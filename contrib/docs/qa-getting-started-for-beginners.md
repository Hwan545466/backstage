# Backstage + AI Test Generation — A Beginner's Walkthrough

**Who this is for**: first-time Backstage user, working alone, wants
every step spelled out.

**What you will have at the end**: Backstage running on your laptop,
one real API in its catalog, and an AI-generated test suite that
actually runs.

**Time**: 3–4 hours split across 2–3 sessions.

**Companion docs** (in this folder, open as needed):
- `qa-glossary.md` — plain-English definitions of every term.
- `qa-troubleshooting.md` — fixes for errors you'll hit.
- `qa-discovery-plan.md` — the strategy behind all this.
- `qa-claude-code-kickoff.md` — prompts to paste when you start Claude Code.

---

## Progress Checklist

Mark these off as you go. Each session is a natural stopping point.

**Session 1 — One-time laptop setup** (20–40 min, once)
- [ ] 1.1 Check Node, Yarn, git versions
- [ ] 1.2 Install Node if needed
- [ ] 1.3 Enable Yarn via Corepack
- [ ] 1.4 (Optional) Install Ollama and pull a model

⏸ *Break point — come back next session*

**Session 2 — Get Backstage running with one API** (60–90 min)
- [ ] 2.1 Pick a folder and scaffold a Backstage app
- [ ] 2.2 Start it with `yarn dev`
- [ ] 2.3 Open it in a browser, sign in as guest
- [ ] 3.1 Pick an OpenAPI spec (Petstore recommended)
- [ ] 3.2 Add an API entity to `examples/entities.yaml`
- [ ] 3.3 Restart, verify the API shows in the UI

⏸ *Break point — good place to stop for the day*

**Session 3 — Generate and run your first AI test** (45–90 min)
- [ ] 4.1 Copy the OpenAPI spec
- [ ] 4.2 Prompt an AI to generate tests
- [ ] 4.3 Save the test file
- [ ] 4.4 Install Playwright
- [ ] 4.5 Run the test
- [ ] 5.1 Save your prompt and loop as a README

✅ **MVP complete.** Come back for Part 6 when you want to level up.

---

# Session 1 — Prep Your Laptop

Do this once. ~20–40 min depending on what you have installed.

## 1.1 Check what you have

Open a terminal and run each:

```
node --version
yarn --version
git --version
```

**Expected**:
- `node` → `v22.x` or `v24.x`. Older is a problem.
- `yarn` → any version, or "not found" (we'll fix).
- `git` → any version.

Anything missing → see next steps. Any of these errors → check
`qa-troubleshooting.md`.

## 1.2 Install Node.js (if needed)

- Go to https://nodejs.org.
- Download LTS (the bigger green button).
- Run the installer, click Next a few times.
- **Close your terminal, open a new one.**
- Re-run `node --version`.

## 1.3 Enable Yarn

```
corepack enable
```

That's it. `yarn --version` should now work.

## 1.4 Install Ollama (optional)

Only if you want **free, local** AI generation. If you plan to use
Claude/ChatGPT in your browser instead, skip this.

- Download from https://ollama.com, run the installer.
- Open a new terminal and run:

```
ollama pull llama3
```

(Downloads 4–5 GB. Go make coffee.)

- Test:

```
ollama run llama3 "hello"
```

Press `Ctrl+D` to exit.

**Low on RAM (8 GB)?** Use `ollama pull phi3:mini` instead — much
smaller.

⏸ *Session 1 done. Next time you open Claude Code, say you
finished Session 1 and want Session 2.*

---

# Session 2 — Create Your Backstage App and Add One API

## 2.1 Scaffold the app

**Important**: do this **outside** any existing Backstage source
folder. Pick a fresh location like your Desktop.

```
cd ~/Desktop
npx @backstage/create-app@latest
```

When prompted:
- Install `create-app`? → `y`
- App name? → `my-backstage` (or whatever you want)

Wait 5–15 minutes. Lots of download output is normal.

**Success looks like**: `✅ Successfully created my-backstage`

## 2.2 Start it

```
cd my-backstage
yarn dev
```

First run compiles — takes 2–5 minutes. Watch for these two lines:

```
[0] webpack compiled successfully
[1] Listening on :7007
```

Both = success.

## 2.3 Open it

In your browser: `http://localhost:3000`

Click the yellow **Guest** button to sign in. Click around the
sidebar: Home, Catalog, APIs, Docs, Create…

Leave `yarn dev` running. `Ctrl+C` stops it when needed.

## 3.1 Pick an OpenAPI spec

Easy starter: Swagger Petstore —
`https://petstore3.swagger.io/api/v3/openapi.json`

If you have your own spec, use that instead.

## 3.2 Add the API to the catalog

**Stop Backstage** (`Ctrl+C` in its terminal).

Open `my-backstage/examples/entities.yaml` in a text editor.

At the bottom of the file, add:

```yaml
---
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: petstore
  description: Sample Petstore API
  tags:
    - rest
    - example
spec:
  type: openapi
  lifecycle: experimental
  owner: guests
  definition:
    $text: https://petstore3.swagger.io/api/v3/openapi.json
```

Save. **YAML is picky** — no tabs, only spaces, indent by 2.

## 3.3 Restart and verify

```
yarn dev
```

Wait for "webpack compiled successfully". Refresh your browser.

Click **APIs** in the sidebar → you should see `petstore`. Click it
→ you should see every endpoint rendered (GET /pet, POST /pet, etc).

**If it didn't appear**: check `qa-troubleshooting.md` → "Catalog"
section.

⏸ *Session 2 done. This is a great place to commit/take a break.
Next session we generate your first AI test.*

---

# Session 3 — Generate and Run Your First AI Test

This is **Option A** from the strategy doc: manual copy-paste, no
plugin development. Get this working first.

## 4.1 Copy the OpenAPI spec

Open `https://petstore3.swagger.io/api/v3/openapi.json` in a new
browser tab. `Ctrl+A`, `Ctrl+C`.

## 4.2 Prompt an AI

Pick one:

### 4.2a — Claude or ChatGPT (recommended)

Open https://claude.ai or https://chat.openai.com. Paste:

> I'm going to give you an OpenAPI spec. Generate a Playwright API
> test suite in TypeScript that:
> - Tests the happy path for each endpoint (GET, POST, PUT, DELETE)
> - Tests one 400 error case and one 404 error case
> - Uses `@playwright/test` and `request.newContext()`
> - Uses `https://petstore3.swagger.io/api/v3` as the base URL
> - Outputs ONE single .ts file with no explanations
>
> Here is the spec:
>
> [paste the spec]

Wait for the response. Copy the code.

### 4.2b — Local Ollama (free but slower)

In a new terminal:

```
ollama run codellama "generate a Playwright TypeScript test suite for the Petstore API at https://petstore3.swagger.io/api/v3. Cover GET /pet/{petId}, POST /pet, and one 404 case. Output only code."
```

Quality will be lower than Claude/GPT — good enough for practice.

## 4.3 Save the test file

```
cd ~/Desktop
mkdir -p qa-tests/petstore
cd qa-tests/petstore
```

Create a file `petstore.spec.ts` and paste the AI's code into it.

## 4.4 Install Playwright

Still in `qa-tests/petstore`:

```
npm init -y
npm install -D @playwright/test
npx playwright install chromium
```

## 4.5 Run it

```
npx playwright test
```

You'll see test results. **Some passing, some failing is fine.**
You now have the full loop working end to end.

**If tests fail**: common causes —
- AI invented an endpoint → fix prompt, regenerate.
- API changed → real QA finding.
- Rate-limited → slow down or add delays.
- Assertions are wrong → fix manually or regenerate.

This **is** the QA work.

## 5.1 Save your loop

Create `~/Desktop/qa-tests/README.md`:

```markdown
# My QA Loop

1. Open Backstage → APIs tab.
2. Pick an API entity.
3. Copy the OpenAPI spec.
4. Paste into Claude/ChatGPT with prompts/api-tests.md.
5. Save to qa-tests/<api-name>/.
6. Run `npx playwright test`.
7. Fix or regenerate as needed.
```

Also save your working prompt to
`~/Desktop/qa-tests/prompts/api-tests.md` so you can reuse it.

✅ **MVP complete.** You have:
- Backstage running locally
- One real API registered
- A working prompt
- A runnable test suite
- A written 6-step loop

---

# Part 6 — Level Up (When You're Ready)

Only after the above works smoothly. Each of these is a separate
session, probably 1–2 hours each.

## 6.1 Add a second API

Repeat Session 2 step 3.2 and Session 3 with another API. Good
practice targets:
- Midtrans sandbox: https://api-docs.midtrans.com
- JSONPlaceholder: https://jsonplaceholder.typicode.com
- Your own company's API

## 6.2 Wire Ollama through Backstage's proxy

Edit `my-backstage/app-config.yaml`, add:

```yaml
proxy:
  endpoints:
    /ollama:
      target: http://localhost:11434
      credentials: dangerously-allow-unauthenticated
      allowedMethods: ['POST', 'GET']
      allowedHeaders: ['Content-Type']
```

Restart. Now you can call Ollama from the Backstage frontend.

## 6.3 Build a custom Scaffolder action

The big one: a Backstage form that generates tests with one click.

See `qa-discovery-plan.md` Appendix A.3 for the full code skeleton.
Don't attempt until 6.1 and 6.2 work.

## 6.4 Switch to Ollama Cloud

Change `target` in the proxy config from `localhost:11434` to your
cloud URL. Nothing else changes.

---

# When Things Break

See `qa-troubleshooting.md`. Has dedicated sections for toolchain,
install, Backstage startup, catalog, AI/Ollama, and test runner
errors.

If it's not there, paste your exact error + step number into Claude
Code using **Prompt 3** from `qa-claude-code-kickoff.md`.
