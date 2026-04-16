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

---

## Appendix A — Ollama Integration

Ollama gives you local or cloud LLM inference with an
OpenAI-compatible API. This appendix shows three ways to wire it
into your Backstage app, from simplest to most integrated.

### A.1 Ollama Basics

- **Local**: install Ollama, `ollama pull llama3`, runs at
  `http://localhost:11434`.
- **Ollama Cloud**: hosted endpoint (e.g.
  `https://ollama.example.com`), same API shape.
- **API shape**: POST `/api/generate` (streaming) or POST
  `/api/chat` (chat completions). Also exposes an
  OpenAI-compatible endpoint at `/v1/chat/completions`.

### A.2 Option 1 — Backstage Proxy (5-minute setup)

The proxy backend plugin (`@backstage/plugin-proxy-backend`,
source at `plugins/proxy-backend/`) forwards requests from
the Backstage frontend to an external target. Add this to your
`app-config.yaml`:

```yaml
proxy:
  endpoints:
    /ollama:
      target: http://localhost:11434   # or your cloud URL
      credentials: dangerously-allow-unauthenticated
      allowedMethods: ['POST', 'GET']
      allowedHeaders: ['Content-Type']
```

Then from any frontend plugin or a simple fetch in the browser
console:

```ts
const response = await fetch('/api/proxy/ollama/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    model: 'llama3',
    prompt: 'Generate a Playwright test for the GET /pets endpoint',
    stream: false,
  }),
});
const data = await response.json();
console.log(data.response);
```

This is the fastest way to get AI responses flowing through
Backstage. No custom plugins needed.

For full proxy configuration options see
`docs/plugins/proxying.md` and `plugins/proxy-backend/config.d.ts`.

### A.3 Option 2 — Custom Scaffolder Action (recommended for test generation)

This is the best fit for the "generate API tests" goal. You
create a Scaffolder template with a form (pick a model, pick an
API entity, pick a test framework), and a backend action that
calls Ollama and writes the generated files.

**Step 1 — Scaffold the module**

```bash
cd my-backstage
yarn backstage-cli new    # select "scaffolder-backend-module"
```

**Step 2 — Write the action**

Inside the generated module, create the action:

```ts
// plugins/scaffolder-backend-module-ollama/src/actions/generate-tests.ts
import { createTemplateAction } from '@backstage/plugin-scaffolder-node';
import fs from 'fs';
import path from 'path';

export const createOllamaTestGenAction = () => {
  return createTemplateAction({
    id: 'ollama:generate-api-tests',
    description: 'Generate API automation tests using Ollama',
    schema: {
      input: {
        type: 'object' as const,
        required: ['model', 'openApiSpec', 'testFramework'],
        properties: {
          model: {
            type: 'string' as const,
            title: 'Ollama model',
            description: 'e.g. llama3, codellama, mistral',
          },
          openApiSpec: {
            type: 'string' as const,
            title: 'OpenAPI spec (JSON or YAML)',
          },
          testFramework: {
            type: 'string' as const,
            title: 'Test framework',
            enum: ['playwright', 'jest-supertest', 'pytest'],
          },
        },
      },
    },
    async handler(ctx) {
      const { model, openApiSpec, testFramework } = ctx.input;

      const prompt = [
        `Given this OpenAPI spec, generate a ${testFramework} test suite.`,
        `Cover: happy path for every endpoint, 400/401/404 error cases,`,
        `and response schema assertions.`,
        `Output only the test code, no explanations.`,
        `\n\n${openApiSpec}`,
      ].join(' ');

      const res = await fetch('http://localhost:11434/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ model, prompt, stream: false }),
      });

      const data = await res.json();
      const outputFile = path.join(ctx.workspacePath, 'generated-tests.ts');
      fs.writeFileSync(outputFile, data.response);

      ctx.logger.info(`Tests written to ${outputFile}`);
    },
  });
};
```

**Step 3 — Register the action** in your backend
(`packages/backend/src/index.ts`) by adding the module.

**Step 4 — Write the Scaffolder template**

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: generate-api-tests
  title: Generate API Tests (Ollama)
  description: Use a local LLM to generate API automation tests
spec:
  owner: qa
  type: qa-tool

  parameters:
    - title: Configuration
      required:
        - model
        - testFramework
        - apiSpec
      properties:
        model:
          title: Ollama Model
          type: string
          enum:
            - llama3
            - codellama
            - mistral
            - deepseek-coder
          default: llama3
        testFramework:
          title: Test Framework
          type: string
          enum:
            - playwright
            - jest-supertest
            - pytest
          default: playwright
        apiSpec:
          title: OpenAPI Spec
          type: string
          ui:widget: textarea
          ui:options:
            rows: 15

  steps:
    - id: generate
      name: Generate tests via Ollama
      action: ollama:generate-api-tests
      input:
        model: ${{ parameters.model }}
        openApiSpec: ${{ parameters.apiSpec }}
        testFramework: ${{ parameters.testFramework }}

  output:
    text:
      - title: Done
        content: |
          Tests generated. Check the workspace output.
```

This gives you a form in the Backstage UI: pick a model from a
dropdown, paste (or pipe) the OpenAPI spec, pick a test
framework, and click Create. The backend calls Ollama and writes
the test file.

### A.4 Option 3 — OpenAI-Compatible SDK via Ollama

Ollama exposes `/v1/chat/completions` which is wire-compatible
with the OpenAI SDK. If you prefer to use the `openai` npm
package (e.g., for a community plugin that expects it):

```ts
import OpenAI from 'openai';

const client = new OpenAI({
  baseURL: 'http://localhost:11434/v1', // Ollama's OpenAI compat
  apiKey: 'ollama',                     // required but ignored
});

const completion = await client.chat.completions.create({
  model: 'llama3',
  messages: [
    { role: 'user', content: 'Generate a test for GET /pets' },
  ],
});

console.log(completion.choices[0].message.content);
```

This means any Backstage community plugin built for OpenAI will
work with Ollama by changing `baseURL` and `apiKey` in its
config. No code changes needed.

### A.5 Ollama + MCP Actions Backend (bonus)

Backstage has an MCP Actions Backend plugin
(`plugins/mcp-actions-backend/`) that exposes Backstage actions
as MCP tools. If you use Claude Desktop, Cursor, or another MCP
client, you can connect it to your Backstage instance and have
the AI assistant call catalog and scaffolder actions directly.

This is the most advanced integration and is not required for the
MVP. Explore it once Options A/B are working. See
`docs/ai/mcp-actions.md` for setup.

### A.6 Which Ollama Models to Try

For code generation, start with these (in rough quality order):

| Model | Size | Good for |
|---|---|---|
| `deepseek-coder` | 6.7B | Best code quality at small size |
| `codellama` | 7B–34B | Code-focused Llama variant |
| `llama3` | 8B–70B | General purpose, good reasoning |
| `mistral` | 7B | Fast, decent code output |

On 8 GB RAM, stick to 7B models. On 16 GB, you can try 13B.
The 70B models need 40+ GB RAM or a GPU — skip them on a
laptop.

### A.7 When to Use Ollama vs. a Cloud AI

| Use Ollama (local) when | Use cloud AI (Claude/GPT) when |
|---|---|
| You want free, unlimited calls | You need the best output quality |
| You have no internet or API keys | Spec is large (>4K tokens) |
| Latency doesn't matter (7B is slow) | You want streaming chat UX |
| Privacy matters (spec stays local) | You need function calling / tool use |

For MVP: use cloud AI manually (Option A in §4) to get good
baseline tests, then try Ollama locally to see if the quality is
sufficient for your APIs. If it is, switch to Ollama to save
costs.
