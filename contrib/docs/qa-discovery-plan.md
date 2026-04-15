# Solo QA MVP Plan — Backstage Implementation

A minimum-viable QA plan for **one person, on a personal computer**,
whose job is to validate a **Backstage implementation** being rolled
out for their organization.

You are not testing Backstage the open-source framework — upstream CI
already does that. You are testing **your deployment**: your
`app-config.yaml`, your auth integration, your catalog, your plugins,
your scaffolder templates, your TechDocs, your integrations.

This is a contributor-authored reference, not official project
documentation.

---

## 1. What You Are Actually QAing

When an organization "implements Backstage", the QA surface is **not**
the Backstage source code. It is the set of things the organization
customizes and operates:

| Layer | What QA validates |
|---|---|
| **Configuration** | `app-config.yaml`, `app-config.production.yaml`, env vars, secrets |
| **Authentication** | Your chosen SSO provider (Google, GitHub, Microsoft, OIDC, SAML) works, sign-in flows, session handling |
| **Catalog** | Entity ingestion from your sources (GitHub/GitLab orgs, manual YAML), entities are valid, owners resolve, relations are correct |
| **Integrations** | GitHub/GitLab/Bitbucket tokens, Jenkins, SonarQube, PagerDuty, Jira, cloud providers |
| **Plugins** | Each installed plugin loads, renders, and successfully talks to its external system |
| **Scaffolder templates** | Your templates actually produce working repos/projects |
| **TechDocs** | A sample repo's docs build and render; search works |
| **Permissions (if enabled)** | Roles grant and deny the right things |
| **Deployment** | Container builds, database migrations, health checks, logs, backups |
| **Upgrades** | You can bump Backstage versions without breaking your config |

Everything on this list is **your** responsibility. None of it is covered
by upstream CI.

---

## 2. Before You Write Any Plan — Answer These

Your MVP depends on these knobs. Write the answers down before doing
anything else:

1. **Auth provider** — Google? GitHub OAuth? Microsoft / Entra? Okta?
   OIDC? Guest (dev only)?
2. **Catalog sources** — GitHub discovery? GitLab? Manual YAML in a
   single repo? Static URLs?
3. **Plugins installed** — list them. Focus QA only on these, not on
   the 150+ available ones.
4. **Integrations** — which external systems will Backstage call at
   runtime (GitHub API, Jenkins, Jira, cloud APIs)?
5. **Database** — SQLite for dev, Postgres for prod? Same or different
   from your personal machine?
6. **Deployment target** — Docker on a VM? Kubernetes? Managed cloud?
7. **Users** — how many, what SSO groups, any permissioning?
8. **Which Backstage version / release line** — main-line monthly or
   `next`? Pinned to which version?

The rest of this plan is generic. Replace every "your chosen X" with
your actual answers.

---

## 3. Hardware and Environment

- One laptop / desktop. 16 GB RAM comfortable, 8 GB tight.
- Docker Desktop (or Podman / Colima).
- Node 22.x or 24.x and Yarn (via Corepack).
- Disk for `node_modules` (several GB).
- For a realistic dev setup: Postgres in a container.

You can run a meaningful Backstage instance locally. You cannot
reproduce your production cluster, and you should not try.

---

## 4. The MVP Scope

Keep the first pass deliberately narrow. The initial goal is: **prove
the core paths your users will actually touch on day one**.

Minimum:

1. **Log in** with your real auth provider.
2. **Open the catalog** and see at least one real service from your
   real source.
3. **Open a service detail page** and see its metadata, owners, and
   links resolved correctly.
4. **Run one scaffolder template** end-to-end and verify the output.
5. **Open TechDocs** for one real repo and confirm rendering.
6. **Open one installed plugin tab** that talks to an external system
   (e.g., CI builds, Jira issues) and confirm live data appears.

If those six work, you have a portal that is usable. If any of those
six breaks, the portal is not usable — this is the right smoke list.

---

## 5. Two-Week MVP

### Week 1 — Stand it up, answer the knobs, write the smoke test

**Day 1 — Environment**
- Clone `backstage` (or your company's fork).
- `yarn install`, `yarn start`. Confirm `:3000` and `:7007` load.
- At this point you are on default config with guest auth and no real
  catalog. That is fine.

**Day 2 — Configuration**
- Read `app-config.yaml`.
- Swap guest auth for a dev instance of your real provider (most
  providers let you register a dev app against `localhost:7007`).
- Confirm you can log in with your own account.
- Record every config value you changed.

**Day 3 — Catalog**
- Add **one** real source — e.g., one GitHub repo with a
  `catalog-info.yaml`.
- Confirm the entity imports, owners resolve, and the URL link works.
- Deliberately break something (invalid YAML, missing owner) and
  confirm Backstage reports the error cleanly.

**Day 4 — Integrations and plugins**
- Wire **one** integration you actually need (e.g., GitHub personal
  access token in `integrations.github`).
- Install **one** plugin your team will use. Confirm it loads and
  shows real data.
- Document token scopes and rate-limit concerns.

**Day 5 — Smoke checklist**
- Turn the 6 MVP paths from §4 into a written checklist in a markdown
  file.
- Run through it start to finish. Aim for under 15 minutes.
- This becomes your **release-go/no-go checklist** for every upgrade.

### Week 2 — Exercise the things you depend on

**Day 6 — Scaffolder**
- Run one template. Inspect the generated repo: files, owners,
  default branch, CI workflow, first commit.
- Try the template with bad inputs (empty name, invalid owner,
  restricted chars). Confirm it fails cleanly.

**Day 7 — TechDocs**
- Pick a real repo with a `docs/` folder and `mkdocs.yml`.
- Build its TechDocs locally, confirm rendering, try search.
- Verify navigation, images, and internal links.

**Day 8 — Auth and sessions**
- Log out and log back in.
- Sign in on a second browser / incognito; confirm clean isolation.
- Simulate an expired session (clear cookies mid-use) and confirm the
  app recovers gracefully.

**Day 9 — Failure modes**
- Kill the database container while the app is running. Watch the
  logs. Recover.
- Revoke your GitHub token; see how the plugin degrades.
- Point an integration to a bad URL; see how errors surface.
- Record what is user-visible vs. silent.

**Day 10 — Consolidate**
- Write two short documents:
  1. **Smoke checklist** (the 6 paths) — use before every upgrade.
  2. **Known-issues log** — bugs you found, config gotchas, workarounds.
- File any real bugs against the **internal** repo (your
  implementation), not upstream Backstage. Only escalate upstream if
  you can reproduce on an unmodified example app.

---

## 6. Sustainable Weekly Rhythm After MVP

Pick one per week:

- Run the smoke checklist against the latest config branch.
- Add one new path to the smoke checklist as adoption grows.
- Validate one scaffolder template end-to-end.
- Re-test one integration (especially ones with rotating tokens).
- Read upstream release notes for the version you plan to upgrade to.
- Test an upgrade in a disposable local environment before rolling to
  staging.

---

## 7. Where Upstream Backstage QA Tooling Helps You (and Where It Doesn't)

**Useful for an adopter:**
- `yarn start` — local dev loop.
- The example app (`/packages/app`) — sanity check "is Backstage
  itself broken, or is it my config".
- Docs under `/docs/getting-started`, `/docs/integrations`, and
  `/docs/auth` — this is your config reference.
- Plugin docs under each `plugins/*/README.md`.

**Not useful for an adopter:**
- Upstream Jest suites — they test framework internals.
- Upstream Playwright E2E — they test the example app, not yours.
- `yarn build:api-reports` — only matters if you're changing framework
  APIs.
- The 7+ GitHub workflows — those are upstream CI, not yours.

Your QA value comes from testing **your** config, **your** integrations,
**your** templates. Upstream tests can't do that for you.

---

## 8. What Your CI Eventually Needs (Out of Scope for MVP, but Plan For It)

Once the manual MVP smoke is stable, the next step — when time allows —
is to automate it. Minimal shape:

- A CI job that builds your Backstage image.
- A smoke test (Playwright or scripted HTTP) that runs the 6 MVP paths
  against a deployed staging instance.
- A config-lint job that validates `app-config.*.yaml` against the
  Backstage schema.
- Database migration dry-run on upgrade PRs.

Do **not** attempt this in the first two weeks. Get the manual smoke
checklist working first.

---

## 9. Commands Cheat Sheet

```bash
# Install and run locally
yarn install
yarn start                    # frontend :3000, backend :7007

# Build the Docker image (for deployment QA)
yarn tsc
yarn build:backend            # note: only run this when validating a build

# TechDocs local preview (from a repo with mkdocs)
npx @techdocs/cli serve

# Format and lint any config/template changes you made
yarn prettier --write <paths>
yarn lint
```

Note on `yarn build:backend`: the Backstage repo's contributor rules
say not to run `yarn build` in the framework repo. In **your adopter
repo** (a forked or `npx @backstage/create-app`–generated project), you
*do* need to build for deployment — that's different.

---

## 10. How to File Useful Bugs as a Solo QA

Your two bug destinations are different:

- **Your own issue tracker** — for anything caused by your config,
  templates, plugins, or integrations. This is 90%+ of what you'll
  find.
- **github.com/backstage/backstage** — only for bugs you can reproduce
  on an unmodified `npx @backstage/create-app` project, or on this
  repo's example app.

Before filing upstream, always test:
1. Can I reproduce on the default example app? If no → it's your
   config.
2. Is it already reported? Search open and closed issues.
3. Can I produce minimal reproduction steps?

Upstream maintainers are swamped. Clean repro beats volume.

---

## 11. Things You Can Safely Ignore

- Contributing changesets, API reports, or upstream tests.
- The Windows E2E workflow and full database matrix — those are
  upstream CI's problem.
- Legacy frontend system (`/packages/app-legacy`) unless your
  implementation specifically uses it.
- Most of the 150+ upstream plugins — only QA the ones you install.
- Performance benchmarking until you have a production baseline.

---

## 12. Success Criteria for the MVP

You are done with MVP when:

- You can go from cold laptop to your Backstage running with **real
  auth + one real catalog entity + one real integration** in under an
  hour.
- You have a written 6-path smoke checklist and have run it at least
  twice.
- You have a known-issues log with at least the config gotchas you
  tripped over.
- You know how to upgrade Backstage safely (at least conceptually —
  read the upgrade helper docs in `/docs/getting-started/keeping-backstage-updated.md`).
- You have filed any bugs in the right place (internal vs. upstream).

Anything beyond this — permissions, automated smoke CI, load testing —
is bonus and should wait until the basics are rock-solid.

---

## 13. Key References

- `/docs/getting-started/` — adopter setup path.
- `/docs/integrations/` — how to wire each source control / CI system.
- `/docs/auth/` — auth provider configuration.
- `/docs/features/software-catalog/` — catalog modeling.
- `/docs/features/software-templates/` — scaffolder.
- `/docs/features/techdocs/` — TechDocs setup.
- `/docs/permissions/` — permission framework (if used).
- `/docs/deployment/` — production deployment.
- `/docs/getting-started/keeping-backstage-updated.md` — upgrade path.
- `app-config.yaml` in this repo — reference for every config key.
