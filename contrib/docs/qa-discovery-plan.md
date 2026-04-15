# Solo QA MVP Plan for Backstage

A minimum-viable QA plan scoped for **one person, on a personal computer,
with no dev team to collaborate with**. The goal is not to cover the whole
project — that is not feasible alone — but to produce useful quality
signal on a small, well-chosen slice of Backstage with effort you can
actually sustain.

This is a contributor-authored reference, not official project
documentation.

---

## 1. Reality Check

Backstage is huge: ~69 framework packages and ~150+ plugins. The CI
infrastructure runs Jest, Playwright, Lighthouse, CodeQL, and database
matrices across Linux and Windows. You cannot reproduce all of that
alone, and you should not try.

What a solo QA on a personal laptop **can** do:

- Run unit tests locally for a handful of packages.
- Run the example app end to end and exercise it by hand.
- Run a small number of Playwright E2E tests locally.
- Read CI output on GitHub for everything else and rely on it as the
  source of truth.
- File clean, reproducible bug reports.
- Keep a lightweight checklist per release.

What you should **not** try to do solo:

- Replicate the full Postgres / MySQL / Redis matrix.
- Run E2E on Windows.
- Build a coverage matrix across all 200+ plugins.
- Design a team-wide flake policy.
- Define metrics for other people to hit.

---

## 2. Hardware and Environment Assumptions

- One laptop / desktop. 16 GB RAM is comfortable; 8 GB will work but
  Playwright + example app + backend + database is tight.
- Docker Desktop (or Podman / Colima) for Postgres when needed.
- Node 22.x or 24.x and Yarn (enabled via Corepack).
- Enough disk for `node_modules` (several GB) and Playwright browsers.

If your machine is small, skip Postgres entirely and use the default
in-memory SQLite-based setup that the example app ships with.

---

## 3. The MVP Scope — Pick a Small Slice

Resist the urge to cover everything. Pick **three targets** and stick to
them for the first couple of weeks:

1. **The example app** (`/packages/app` + `/packages/backend`) — the
   smoke-test surface. If this is broken, everything is broken.
2. **One tier-1 plugin** you actually care about. Reasonable choices:
   - `plugins/catalog` — central to every Backstage install.
   - `plugins/scaffolder` — user-visible, template-driven.
   - `plugins/techdocs` — docs rendering.
   - `plugins/search` — cross-cutting.
   Pick one, not all.
3. **The docs you are most likely to follow** — the "getting started"
   path in `/docs/getting-started`.

Everything outside these three is out of scope for MVP. You can widen
later.

---

## 4. Two-Week MVP

### Week 1 — Get it running and take a baseline

**Day 1 — Install**
- `git clone` and `yarn install`.
- `yarn start` — confirm frontend at `:3000` and backend at `:7007`.
- Click through the default catalog, scaffolder, and techdocs pages.
- Note anything broken; file one issue with a clean reproduction.

**Day 2 — Run the tests you care about**
- `CI=1 yarn test packages/app` — example app unit tests.
- `CI=1 yarn test plugins/<your-chosen-plugin>` — your one plugin.
- `yarn tsc` — type check the repo.
- `yarn lint` — lint the repo.

Record: how long each took, whether anything failed locally that passes
in CI.

**Day 3 — Run one E2E**
- `yarn playwright install` once.
- Run the example app's E2E: look under `/packages/app/e2e-tests`.
- Goal is only to confirm Playwright works on your machine, not to run
  the full suite.

**Day 4 — Read CI on a recent PR**
- On GitHub, open a recently merged PR.
- Walk through each failed and passed check under Actions.
- Skim `.github/workflows/ci.yml` and one `verify_e2e-*.yml`.
- Write yourself a one-paragraph note: "these are the checks I trust CI
  to run so I don't have to."

**Day 5 — Write your MVP test charter**
- One page of plain text. Sections:
  - What I test locally (example app smoke + one plugin's unit tests).
  - What I trust CI for (everything else).
  - What I skip (Windows, DB matrix, 200+ plugin coverage).
  - How I file bugs (template + reproduction steps).

### Week 2 — Start producing signal

**Day 6–7 — Exploratory testing on the example app**
- Run `yarn start`.
- Go through every top-level nav item. Try: empty states, long strings,
  unicode, very wide/narrow viewports, keyboard-only navigation,
  browser back/forward.
- Log findings in a simple markdown file, one line per issue.

**Day 8 — Pick one page for an accessibility pass**
- Run the example app in Chrome, open DevTools → Lighthouse →
  Accessibility on one page of your chosen plugin.
- Compare with CI's Lighthouse report if available
  (`.github/workflows/verify_accessibility.yml`).
- Record the gap.

**Day 9 — Write one new test**
- Either a unit test for a bug you found, or a Playwright test for a
  flow that wasn't covered.
- Use `renderInTestApp` from `@backstage/test-utils` (frontend) or
  `startTestBackend()` from `@backstage/backend-test-utils` (backend).
- Don't aim for a great test — aim for one test that runs.

**Day 10 — Consolidate**
- Turn your findings into 1–3 GitHub issues using the templates in
  `.github/ISSUE_TEMPLATE/`.
- Write a short retro: what you learned, what was slower than expected,
  what to cut from the plan.

At the end of two weeks you should have: a working local environment, a
written charter, an exploratory log, at least one issue filed, and
ideally one test contributed or drafted.

---

## 5. Sustainable Weekly Rhythm After MVP

Once the two-week MVP is done, keep it small — pick one of these per
week:

- One hour of exploratory testing on the example app against the latest
  `master`.
- Run your chosen plugin's unit tests against `master`; file an issue
  if anything regresses.
- Add one new test to your chosen plugin.
- Read the changesets under `.changeset/` for the upcoming release and
  spot-check one user-facing change.
- Re-run the Lighthouse check on one page and compare with last week.

Resist adding a second plugin until you are genuinely bored with the
first.

---

## 6. What to Cut When Time Is Short

In priority order, cut from the bottom:

1. Always keep: **example app smoke test** (`yarn start`, click around).
2. Keep if you can: **unit tests on your one chosen plugin**.
3. Keep if you can: **one Playwright run per week**.
4. Drop first: accessibility checks.
5. Drop next: adding new tests.
6. Drop last: reading CI output on other people's PRs.

If you only ever have 30 minutes, spend them on item 1.

---

## 7. Commands Cheat Sheet

```bash
# Install
yarn install

# Start example app (frontend :3000, backend :7007)
yarn start

# Unit tests for one path
CI=1 yarn test packages/app
CI=1 yarn test plugins/catalog

# Type check and lint
yarn tsc
yarn lint

# Playwright
yarn playwright install
yarn test:e2e

# Format code you changed
yarn prettier --write <paths>
```

Do **not** run `yarn build`, `yarn changesets version`, or `yarn release`
— those are reserved for the release pipeline.

---

## 8. How to File a Useful Bug (Solo Edition)

Without a dev team to triage, your issues compete for attention with
everyone else's. Raise the signal-to-noise ratio:

- Use the bug template at `.github/ISSUE_TEMPLATE/01_bug.yaml`.
- State Backstage commit SHA and Node version.
- Steps must be runnable from a fresh clone with `yarn install && yarn start`.
- Include exact error text, not a paraphrase.
- Screenshot or short screen recording if the bug is visual.
- One bug per issue. Don't batch.

If you can include a failing test, even a draft one, do it — that is
often the difference between "confirmed" and "closed as can't
reproduce".

---

## 9. Things You Can Safely Ignore as a Solo QA

- The Windows E2E workflow — you are on one machine.
- The Postgres 14 and 18 + MySQL 8 + Redis matrix — CI has it.
- Cross-plugin regression testing — out of scope solo.
- API reports (`yarn build:api-reports`) unless you are changing APIs.
- The legacy frontend system (`/packages/app-legacy`) unless your chosen
  plugin has a legacy variant you actually use.
- Performance benchmarking — no baseline, no signal.

---

## 10. Success Criteria for the MVP

You are done with MVP when all of the following are true:

- You can go from cold laptop to a running example app in under 15
  minutes.
- You have run at least one unit test and one Playwright test locally
  and seen them pass.
- You have a written, one-page charter describing what you test and
  what you skip.
- You have filed at least one well-reproduced issue, or drafted one
  test.
- You know where to find CI results on GitHub and trust them for
  everything you don't test locally.

That's it. Anything beyond this is bonus.
