# Solo QA MVP — Backstage + AI-Assisted API Test Generation

A minimum-viable plan for **one person, on a personal laptop**, with
a specific goal:

> Use Backstage as an **API catalog**, add **AI assistance**, and have
> the AI **generate API automation tests** from the APIs registered
> in the catalog.

This is not about testing Backstage itself. It is about turning
Backstage into your personal QA workbench for API testing.

This is a contributor-authored reference, not official project
documentation.

---

## 1. What This Plan Assumes

Decisions already made, or recommended here:

| Knob | Decision |
|---|---|
| Auth | **Guest auth** (local only, no SSO needed) |
| Catalog sources | **Local YAML file** — recommended below |
| Plugins installed | **Core set + API docs + AI** — recommended below |
| Integrations | **None initially**; add later as needed |
| Database | **SQLite** (the default dev database, no setup) |
| Deployment target | **Local only** — `yarn dev` on your laptop |
| Users / permissions | **Skip permissions entirely for MVP** |
| Backstage version | **Latest stable main-line** |

Ignore anything in the broader Backstage ecosystem that doesn't serve
the goal. You are not running a company portal; you are building a
personal tool.

---

## 2. The End-to-End Flow You Are Building

```
┌────────────────┐   ┌───────────────┐   ┌───────────────────┐
│ OpenAPI spec   │──▶│ Backstage API │──▶│ AI assistant      │
│ (yours or 3rd  │   │ catalog entry │   │ (reads spec +     │
│ party)         │   │               │   │ context)          │
└────────────────┘   └───────────────┘   └─────────┬─────────┘
                                                   │
                                                   ▼
                                         ┌──────────────────────┐
                                         │ Generated test suite │
                                         │ (Postman / Playwright│
                                         │ / Jest+supertest /   │
                                         │ pytest)              │
                                         └──────────────────────┘
```

You don't need every box to exist on day one. Build left to right.

---

## 3. Recommended Plugin and Catalog Choices

### Catalog source (recommended)

- **One local YAML file** at `examples/entities.yaml` inside your
  Backstage app.
- Register each API you want to test as an `API` kind entity with a
  `spec.definition` pointing to an OpenAPI document (inline or a URL).
- Later, when you have a GitHub repo, switch to **GitHub discovery**
  so `catalog-info.yaml` files in repos get auto-imported.

Minimum catalog content for MVP:

- 1 `Component` (a service you own, or a fake one).
- 1 `API` with a real OpenAPI v3 spec (use a public one for starters
  — e.g., Petstore, or a Midtrans sandbox spec if you have it).
- 1 `User` and 1 `Group` (just you, and a "qa" group).

### Plugins to install

Default plugins you already get from `npx @backstage/create-app`:

- `@backstage/plugin-catalog`
- `@backstage/plugin-scaffolder`
- `@backstage/plugin-techdocs`
- `@backstage/plugin-api-docs` ← critical for your goal
- `@backstage/plugin-search`

Add later as needed:

- `@backstage/plugin-catalog-import` — paste a catalog-info URL to
  import quickly.
- An **AI plugin** — options in §4.

Skip everything else until you have a reason.

### Permissions

Skip. The permission framework is powerful but adds friction. A
single-user local setup has no one to permission against.

---

## 4. The AI Piece — Your Three Realistic Options

There is no single "official" Backstage AI plugin. Pick one of these
three paths; all three are viable for a solo local setup.

### Option A — External AI, Backstage as catalog only (simplest)

- Use Backstage purely to browse and manage your API specs.
- When you want tests, **export the OpenAPI spec** from the API entity
  page, paste it into Claude / ChatGPT / Copilot with a prompt like
  *"generate a Playwright test suite for this OpenAPI spec covering
  happy path, 4xx errors, and auth failures"*.
- Commit the generated tests to a repo.

Pros: zero plugin development. Works immediately. You get to pick the
best AI at any moment.
Cons: manual copy-paste; no in-UI magic.

**Recommended for MVP.** Get the workflow right first; automate later.

### Option B — Custom Scaffolder action that calls an AI API

- Write a Scaffolder template that takes an `API` entity ref as input.
- Add a custom Scaffolder **action** (small TypeScript function inside
  your Backstage backend) that:
  1. Resolves the API entity from the catalog.
  2. Reads `spec.definition` (the OpenAPI document).
  3. Calls the Anthropic / OpenAI API with a prompt and the spec.
  4. Writes the AI response to files in the workspace.
- The template then commits those files to a new local folder or a
  new GitHub repo.

Pros: one-click test generation from the Backstage UI; reproducible.
Cons: you write and maintain the custom action. Needs an API key in
`app-config.local.yaml`.

**Recommended as the v2 upgrade after Option A works manually.**

### Option C — Community AI chat plugin

- Install something like `backstage-plugin-ai-assistant`,
  `@roadiehq/backstage-plugin-openai-proxy`, or similar community
  plugins (search npm for `backstage-plugin` + `ai` / `llm` / `gpt`).
- Configure it with your API key.
- Ask it to generate tests from catalog context.

Pros: in-UI chat, looks like the end state.
Cons: ecosystem is moving fast, plugin quality varies, some are
abandoned. Vet before installing.

**Optional. Only pursue if Option A feels limiting.**

---

## 5. Two-Week MVP — Concrete Steps

### Week 1 — Stand up Backstage and make the AI-test loop work manually (Option A)

**Day 1 — Scaffold a Backstage app**

This is different from the upstream Backstage repo you currently have
cloned. For a solo adopter, you want your **own app**, not the
framework source.

```bash
# Somewhere outside this repo
npx @backstage/create-app@latest
cd my-backstage
yarn dev
```

Confirm `http://localhost:3000` loads. Guest sign-in is on by default.

**Day 2 — Register one API in the catalog**

Edit `examples/entities.yaml` (or create it). Add:

```yaml
---
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: petstore
  description: Swagger Petstore (sample)
spec:
  type: openapi
  lifecycle: experimental
  owner: qa
  definition:
    $text: https://petstore3.swagger.io/api/v3/openapi.json
```

Register it in `app-config.yaml` under `catalog.locations`. Restart.
Open the API in the Backstage UI and confirm the spec renders.

**Day 3 — Register a real API you'll actually QA**

Replace Petstore with an OpenAPI spec you care about. If you have
none, pick one public sandbox API (Midtrans, Stripe, GitHub).

Confirm the API docs tab renders the endpoints.

**Day 4 — Manual AI test generation (Option A)**

- Copy the OpenAPI spec content from the API entity page.
- Open Claude / ChatGPT with a prompt like:

> "Given this OpenAPI spec, generate a Playwright API test suite in
> TypeScript. Cover: happy path for each endpoint, 400/401/404 cases,
> and response schema assertions. Use `@playwright/test` and
> `request.newContext()`. Output a single file."

- Save the output to a local folder: `qa-tests/<api-name>/`.
- `npm init -y && npm i -D @playwright/test`.
- Run the generated tests: `npx playwright test`.

**Day 5 — Smoke checklist for this workflow**

Write a tiny markdown checklist — your repeatable QA loop:

1. Open API entity in Backstage.
2. Export OpenAPI spec.
3. Paste into AI with the standard prompt.
4. Save output to `qa-tests/<api-name>/`.
5. `npx playwright test`.
6. Review failures, adjust prompt, regenerate.

This is your working MVP. Everything after this is polish.

### Week 2 — Harden and (optionally) automate

**Day 6 — Prompt engineering**

Keep your prompts in a file (`prompts/api-tests.md`). Version them.
Try variants: test depth, assertion style, language (Playwright vs.
Jest+supertest vs. pytest). Record which prompts produced usable
output.

**Day 7 — Run generated tests against a real sandbox**

- Configure base URL and API keys via env vars.
- Point tests at the real sandbox (Midtrans, etc.).
- File bugs against the AI output (prompt refinement needed) vs. the
  API under test (real defect).

**Day 8 — Add a second API**

Repeat the flow with a second API entity in the catalog. Confirm the
process scales.

**Day 9 — (Optional) Start on Option B**

If Option A feels slow, spike on a custom Scaffolder action:

- Read `/docs/features/software-templates/writing-custom-actions.md`
  in this repo.
- Write an action `ai:generate-api-tests` that takes `apiRef` and
  `target` inputs.
- Stub it first — return a hardcoded file. Confirm the Scaffolder
  template works end to end before wiring AI.
- Then replace the stub with a call to the Anthropic or OpenAI API.

Do not attempt this until Option A works manually.

**Day 10 — Consolidate**

- Commit the generated tests to a private repo.
- Write a short README in `qa-tests/` describing the loop.
- Note open questions: prompt stability, spec versioning, flaky AI
  output, secret handling.

---

## 6. Repo Layout Suggestion

```
~/work/
├── backstage/              ← this upstream clone, for reference only
├── my-backstage/           ← your actual Backstage app (Option A+)
│   ├── app-config.yaml
│   ├── app-config.local.yaml   ← your API keys (gitignored)
│   ├── examples/entities.yaml  ← your catalog entries
│   ├── packages/
│   └── plugins/
└── qa-tests/               ← AI-generated tests, version controlled
    ├── petstore/
    ├── midtrans/
    └── prompts/
        └── api-tests.md
```

Keep your Backstage app and your generated tests in separate repos.
Mixing them is tempting and a future pain.

---

## 7. What Good Looks Like for the MVP

You are done when:

- Backstage runs on your laptop with guest auth and SQLite.
- At least one real API is registered in the catalog with an OpenAPI
  spec that renders correctly in the API docs tab.
- You have a written prompt that, given the spec, produces a runnable
  test file.
- You have a checklist describing the 6-step loop (§5 day 5).
- You have at least one generated test suite that runs and produces
  pass/fail results against a real API.

That is the whole MVP. Do not add permissions, SSO, Postgres, a
production deployment, or plugin Options B/C until this loop is
stable.

---

## 8. Things to Intentionally Skip

- SSO / OAuth providers — not needed for local.
- Postgres / MySQL — SQLite is fine until you have a team.
- Production deployment, Docker, Kubernetes — irrelevant at MVP.
- Permissions framework — no one to permission against.
- Automated CI — once the loop works manually, then consider it.
- Windows/Mac/Linux parity testing — you are on one machine.
- The 150+ other Backstage plugins — install only what serves the
  test-generation goal.

---

## 9. Gotchas You Will Hit

Forewarned:

- **OpenAPI spec quality dictates AI output quality.** Missing
  descriptions, examples, or schemas produce weak tests. Spend
  effort on the spec.
- **AI hallucinates endpoints and fields.** Always run the generated
  tests — failing tests often mean the AI invented a route, not that
  the API is broken.
- **Prompts drift.** Version them. Keep a "known good" prompt in the
  repo.
- **Secrets management.** Never commit API keys to the scaffolded
  test repo. Use `.env` files and a `.env.example`.
- **Rate limits.** Both the AI provider and the API under test will
  rate-limit you. Plan batches.
- **Idempotency.** Generated tests that POST must clean up after
  themselves or use fresh IDs.
- **Backstage upgrades.** When you bump Backstage, re-run your smoke
  checklist. The catalog YAML format is stable, but custom
  Scaffolder actions (Option B) break most often.

---

## 10. Key References

- `/docs/getting-started/` — how to scaffold your own app.
- `/docs/features/software-catalog/descriptor-format.md` — entity
  YAML format.
- `/docs/features/api-documentation/` — API entity + OpenAPI docs.
- `/docs/features/software-templates/` — Scaffolder overview.
- `/docs/features/software-templates/writing-custom-actions.md` —
  for Option B.
- Backstage example entities: `examples/entities.yaml` in this repo.
- Anthropic API docs — for wiring Claude into a custom action.
- OpenAI API docs — alternative backend for a custom action.
