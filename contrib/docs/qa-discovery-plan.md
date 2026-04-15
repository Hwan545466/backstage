# QA Discovery Plan for Backstage

This document is a structured discovery plan intended to help a QA team ramp up
on the Backstage project. It captures the scope, tooling, quality gates, and
processes relevant to quality assurance, and outlines a phased onboarding path
with deliverables the team can use to produce a durable QA strategy.

It is a contributor-authored reference and is not part of the official project
documentation. Commands, paths, and counts reflect the state of the repository
at the time of writing and should be re-verified during discovery.

---

## 1. Purpose and Audience

- **Audience**: QA engineers, SDETs, and test architects joining or auditing the
  Backstage project.
- **Purpose**: Provide a repeatable plan for discovering what is tested, how it
  is tested, what is not covered, and where quality risks live.
- **Outcomes**: By the end of discovery the team should be able to (a) run all
  supported test suites locally, (b) explain the CI quality gates, (c) map test
  coverage to product surface area, and (d) propose a QA strategy with gaps,
  metrics, and owned deliverables.

---

## 2. Project Snapshot

Backstage is a TypeScript monorepo using Yarn workspaces. It is an open
platform for building developer portals and is composed of a core framework,
a backend system, and a large number of plugins.

- **Packages** (framework core): ~69 under `/packages/*`
- **Plugins**: ~150+ under `/plugins/*`
- **Example apps**: `/packages/app` (new frontend system), `/packages/app-legacy`
  (legacy frontend system), `/packages/backend`
- **Docs**: `/docs` (user- and contributor-facing), `/microsite` (website)
- **Monorepo tooling**: `backstage-cli` drives install, lint, test, build,
  versioning, and publishing
- **Release cadence**: Monthly main-line releases, weekly pre-release `next`
  line (see `docs/overview/versioning-policy.md`)

QA scope is therefore wide: the framework itself, each first-party plugin, the
backend and frontend example apps, the CLI, the release pipeline, and the
published documentation.

---

## 3. Surface Areas That Require QA

| Surface | Location | Notes |
|---|---|---|
| Core frontend framework | `/packages/core-*`, `/packages/frontend-*` | New and legacy frontend systems must both be exercised |
| Backend framework | `/packages/backend-*` | Uses `startTestBackend()` harness |
| Example apps | `/packages/app`, `/packages/app-legacy`, `/packages/backend` | Primary E2E targets |
| First-party plugins | `/plugins/*` | Catalog, Scaffolder, TechDocs, Search, Auth, Permission, Kubernetes, etc. |
| CLI | `/packages/cli` | Affects every adopter's workflow |
| Create templates | `yarn new` scaffolds | Tested indirectly via generation |
| Docs site | `/docs`, `/microsite` | Broken-link and build checks in CI |
| OpenAPI contracts | Various plugins | Breaking-change detection in CI |

---

## 4. Testing Stack

- **Unit / component tests**: Jest, driven by `backstage-cli repo test`, with
  `NODE_OPTIONS='--no-node-snapshot --experimental-vm-modules'`.
- **React component tests**: `@testing-library/react`, with lint rules
  enforcing async queries and discouraging anti-patterns.
- **Frontend test utilities**: `@backstage/test-utils` and
  `@backstage/frontend-test-utils` (`renderInTestApp`, `TestApiProvider`,
  `mockApis`).
- **Backend test utilities**: `@backstage/backend-test-utils`
  (`startTestBackend()`, `mockServices` for logger, config, auth, cache,
  permission, database).
- **E2E tests**: Playwright `@playwright/test`, driven by
  `playwright.config.ts` at the repo root. Projects are auto-generated from
  `@backstage/e2e-test-utils` by scanning for `e2e-tests` folders such as
  `/packages/app/e2e-tests`.
- **Database-backed tests**: Postgres, MySQL, and Redis are started as
  services in CI. Connection strings are provided via
  `BACKSTAGE_TEST_DATABASE_*` environment variables.
- **Accessibility**: Lighthouse CI runs against Catalog, TechDocs, Scaffolder,
  and Search in `.github/workflows/verify_accessibility.yml`, with config and
  helper scripts under `.lighthouseci/`. Storybook uses
  `@storybook/addon-a11y`.
- **Security**: CodeQL scanning (`.github/workflows/verify_codeql.yml`) and
  Snyk policies (`.snyk` files) per package. Process is described in
  `SECURITY.md`.

Jest is configured at the root `package.json` with
`rejectFrontendNetworkRequests: true`, which blocks accidental network traffic
from frontend tests — a useful invariant for QA to know about when triaging
flakes.

---

## 5. Quality Gates and CI

Primary workflows in `.github/workflows/`:

- **`ci.yml`** — main PR verification on Node 22.x and 24.x. Contains:
  - `verify` — changesets, peer deps, type deps, API reports, TypeScript
    fullcheck, catalog-info consistency, OpenAPI validation, doc link check,
    plugin directory consistency.
  - `test` — Jest suite with Postgres 18/14, MySQL 8, and Redis 7 service
    containers; cache layers; optional coverage.
- **`verify_e2e-linux.yml`** — Playwright E2E against a built example app
  with Postgres.
- **`verify_e2e-windows.yml`** — Windows-specific E2E parity.
- **`verify_e2e-techdocs.yml`** — TechDocs end-to-end flow.
- **`verify_accessibility.yml`** — Lighthouse CI.
- **`verify_codeql.yml`** — CodeQL static analysis.
- **`api-breaking-changes.yml`** — OpenAPI breaking-change gate.
- **`deploy_packages.yml`** — publishes to npm on merges to master.

Local equivalents for the most important gates:

| Gate | Local command |
|---|---|
| Lint | `yarn lint --fix` |
| Formatting | `yarn prettier --write <paths>` |
| Type check | `yarn tsc` (use `tsc:full` for declarations) |
| Unit tests | `CI=1 yarn test <path>` |
| All tests | `yarn test:all` |
| E2E | `yarn test:e2e` |
| API reports | `yarn build:api-reports` |
| Changesets | files under `.changeset/` |

Note: `yarn build`, `yarn changesets version`, and `yarn release` are reserved
for release workflows and must not be run as part of development.

---

## 6. Release and QA Process

Summarized from `docs/overview/versioning-policy.md` and `docs/publishing.md`:

1. Contributors add changesets under `.changeset/` describing user-facing
   impact and bump level (patch / minor / major).
2. A "Version Packages" PR is generated from accumulated changesets.
3. Merging that PR to master triggers `deploy_packages.yml`, which publishes
   to npm and posts to Discord.
4. Main-line releases ship monthly (Tuesday before the third Wednesday);
   `next` line ships weekly.
5. Security fixes of high severity or above are backported for 6 months.
6. Emergency patches are shipped through the `.patches/` mechanism.

The gate before a release is the green status of the CI workflows above plus
human review under the rules in `REVIEWING.md` (formal Approve / Request
Changes reviews, cross-area owner sign-off, 14-day stale rule).

---

## 7. Risks and Known Quality Concerns

These are starting hypotheses for the QA team to validate during discovery,
not confirmed findings:

- **Surface area vs. coverage**: With ~200+ publishable units, per-plugin test
  depth is uneven. Coverage should be mapped per plugin.
- **Legacy vs. new frontend system**: Both `/packages/app` and
  `/packages/app-legacy` are maintained. E2E coverage across both should be
  verified.
- **E2E flake**: Playwright config uses `retries` in CI; a flake-rate metric
  should be collected per project.
- **Database matrix**: Postgres 14 and 18 plus MySQL 8 are exercised in CI;
  local parity requires Docker and may hide environment-specific issues.
- **Windows parity**: A dedicated Windows E2E workflow exists, implying
  platform-specific risks worth monitoring.
- **API / OpenAPI drift**: API reports and OpenAPI breaking-change detection
  are automated but depend on reports being regenerated by contributors.
- **Accessibility coverage**: Lighthouse runs on four plugins only. Other
  plugins are not covered by automated a11y checks.
- **Docs rot**: Broken link and doc-link checks are in CI, but content
  accuracy of testing docs under `/docs` should be spot-checked.

---

## 8. Discovery Plan — Phased

### Phase 0 — Access and Environment (day 1)
- Fork / clone the repo; install Node (22.x or 24.x) and Yarn.
- Run `yarn install`.
- Start the example app: `yarn start` (frontend at :3000, backend at :7007).
- Install Docker; confirm Postgres / MySQL / Redis containers can be started
  for database-backed tests.
- Install Playwright browsers: `yarn playwright install`.

**Deliverable**: Environment checklist committed to the QA team's workspace.

### Phase 1 — Run the Suites (days 2–3)
- Run targeted unit tests: `CI=1 yarn test packages/core-plugin-api`.
- Run a plugin suite end-to-end: e.g. `CI=1 yarn test plugins/catalog`.
- Run full local E2E: `yarn test:e2e`.
- Run `yarn lint`, `yarn tsc`, `yarn build:api-reports` to understand
  non-test gates.
- Capture timings, failures, and flakes.

**Deliverable**: Baseline report of local run times and any local-only
failures vs. CI.

### Phase 2 — Read the Pipelines (days 3–5)
- Walk through `.github/workflows/ci.yml`, `verify_e2e-*.yml`,
  `verify_accessibility.yml`, and `api-breaking-changes.yml`.
- For each, note: triggers, matrix, required services, failure modes,
  retry behavior, and artifacts.
- Map which workflow is the gate for which surface area.

**Deliverable**: A one-page "CI gate map" diagram linking workflows to
product areas.

### Phase 3 — Map Coverage (week 2)
- For each top-level plugin, record: presence of unit tests, presence of E2E
  tests, presence of Storybook a11y, presence of OpenAPI contract.
- Identify plugins with no or minimal tests.
- Cross-reference with issue template traffic in `.github/ISSUE_TEMPLATE/`
  and the public issue tracker to spot high-defect areas.

**Deliverable**: Coverage matrix (plugin × test type) with risk scores.

### Phase 4 — Study Test Utilities and Patterns (week 2)
- Read `/docs/backend-system/building-plugins-and-modules/02-testing.md`,
  `/docs/frontend-system/building-plugins/02-testing.md`, and
  `/docs/plugins/testing.md`.
- Inspect `packages/test-utils`, `packages/backend-test-utils`,
  `packages/frontend-test-utils`, `packages/e2e-test-utils`.
- Write one sample test using each harness to validate understanding.

**Deliverable**: Internal "How to write a Backstage test" cheat sheet
targeted at your team's conventions.

### Phase 5 — Release and Process (week 3)
- Read `CONTRIBUTING.md`, `REVIEWING.md`, `SECURITY.md`,
  `docs/overview/versioning-policy.md`, `docs/publishing.md`.
- Shadow one "Version Packages" PR from creation to publish.
- Understand the changeset writing rules in `CONTRIBUTING.md` and practice by
  drafting a changeset for a sample change.

**Deliverable**: Release-cycle runbook with QA checkpoints annotated.

### Phase 6 — Propose a QA Strategy (week 4)
- Combine outputs into a strategy document covering:
  - Test pyramid per surface (unit, integration, E2E, a11y, security).
  - Gaps by plugin and by platform (Linux vs. Windows).
  - Flake budget and quarantine policy.
  - Metrics to publish (pass rate, flake rate, coverage, defect escape rate).
  - QA tasks owned by the team vs. responsibilities shared with plugin
    owners.

**Deliverable**: QA strategy doc reviewed with maintainers.

---

## 9. Metrics to Establish Early

- **CI pass rate** per workflow over a rolling 7- and 30-day window.
- **E2E flake rate** per Playwright project.
- **Test duration** per workflow and per package.
- **Coverage** (Jest `--coverage`) reported per package; trend over time.
- **Open defect count** by plugin, tagged from GitHub issues.
- **Time to fix** for regressions introduced between releases.
- **A11y scores** from Lighthouse CI on gated plugins.

---

## 10. Key References

- `CONTRIBUTING.md` — setup, test commands, changeset rules.
- `REVIEWING.md` — review requirements and gates.
- `STYLE.md` — code style.
- `SECURITY.md` — security reporting process.
- `docs/overview/versioning-policy.md` — release lines and SemVer rules.
- `docs/publishing.md` — release mechanics.
- `docs/plugins/testing.md` — plugin test guidance.
- `docs/backend-system/building-plugins-and-modules/02-testing.md` — backend
  testing.
- `docs/frontend-system/building-plugins/02-testing.md` — frontend testing.
- `playwright.config.ts` — E2E configuration.
- `.github/workflows/` — all CI gates.
- `.github/ISSUE_TEMPLATE/` — bug, docs, feature, maintenance templates.

---

## 11. Open Questions for Maintainers

The QA team should bring these to a maintainer sync early in discovery:

1. Is there an official coverage target per package, or is coverage only
   informational?
2. Which plugins are considered "tier 1" and therefore must have E2E and a11y
   coverage?
3. What is the accepted flake rate for Playwright E2E before a test is
   quarantined?
4. Who owns cross-plugin regression testing during the monthly release
   freeze?
5. Are there plans to extend accessibility automation beyond the four
   plugins currently covered by Lighthouse CI?
6. How should external QA findings (performance, security, accessibility) be
   filed — issue templates, security advisories, or elsewhere?
