# Backstage + AI Test Generation — A Beginner's Walkthrough

**Who this is for**: you have never set up Backstage before, you are
the only person doing this, and you want someone to walk you through
every step with no assumed knowledge.

**What you will have at the end**: Backstage running on your laptop,
one real API registered in its catalog, and an AI-generated test
suite that actually runs against that API.

**Time needed**: 3–4 hours the first time, mostly waiting for
downloads. You can split it across two evenings.

This is the hand-holdy companion to `qa-discovery-plan.md`. That doc
is the strategy. This doc is the button-pressing.

---

## Part 0 — Words You'll See (Quick Glossary)

Don't memorize. Refer back when you hit a term.

- **Backstage** — a piece of software that gives you a website
  (called a "portal") listing all your APIs, services, docs, etc.
  Originally built at Spotify. You run it on your own computer.
- **Catalog** — the database inside Backstage that lists your stuff.
  Each item in it is called an "entity".
- **Entity** — one thing in the catalog. Could be a service, an API,
  a person, a team. Defined in YAML.
- **API entity** — a catalog entry that represents an API, usually
  with a link to its OpenAPI spec.
- **OpenAPI spec** — a JSON or YAML file that describes an HTTP API:
  its endpoints, methods, inputs, outputs. Sometimes called
  "Swagger".
- **Plugin** — a piece of Backstage that adds a feature. Example:
  the "Scaffolder" plugin, the "TechDocs" plugin.
- **Scaffolder** — Backstage's built-in form-based code generator.
  You fill in a form, it runs steps, it writes files.
- **Scaffolder template** — a YAML file that defines one form + the
  steps to run.
- **Ollama** — free software that runs AI models on your laptop. Or
  a cloud service with the same interface.
- **Node.js / Yarn** — the tools that run JavaScript on your
  computer. Backstage is written in JavaScript, so you need these.
- **Terminal** — the black window where you type commands. On Mac
  it's "Terminal.app". On Windows it's "PowerShell" or "Windows
  Terminal". On Linux it's whatever you already have.
- **yarn dev / yarn start** — the command that starts Backstage
  running on your computer.
- **localhost:3000** — an address that means "the website running on
  my own computer, on port 3000". Paste it into your browser.

---

## Part 1 — Prep Your Laptop

You do this once. It takes 20–40 minutes.

### 1.1 Check what you have

Open a terminal and type each of these, pressing Enter after each:

```
node --version
yarn --version
git --version
```

- If `node --version` shows `v22.x.x` or `v24.x.x`, you're good.
- If it says "command not found" or shows `v18` or lower, install
  Node (next step).
- If `yarn` says "command not found", you'll fix it in step 1.3.
- If `git` is missing, install git — on Mac `brew install git`, on
  Windows download from git-scm.com, on Linux `sudo apt install git`.

### 1.2 Install Node.js (if needed)

Easiest way: go to https://nodejs.org and download the LTS version
(the bigger green button). Run the installer. Click Next a few times.

When it's done, **close your terminal and open a new one**, then
retry `node --version`. You should see something like `v22.11.0`.

### 1.3 Enable Yarn

In your terminal, run:

```
corepack enable
```

That's it. `yarn --version` should now work. If it prints a number
like `3.x.x` or `4.x.x`, you're done.

If `corepack` is not found, run `npm install -g corepack` first,
then `corepack enable`.

### 1.4 Install Ollama (optional but useful)

If you want to run AI models **on your own laptop** for free:

- Go to https://ollama.com and download the installer for your OS.
- Run it. It installs a little menu-bar app.
- Open a terminal and run:

```
ollama pull llama3
```

This downloads a 4–5 GB AI model. Go make coffee.

When it finishes, test it:

```
ollama run llama3 "hello"
```

You should see it reply. Press `Ctrl+D` to exit.

If you'd rather skip Ollama and use Claude/ChatGPT in your browser,
that's fine — Part 4 below walks through both paths.

---

## Part 2 — Create Your Own Backstage App

You will **not** be working inside the `/home/user/backstage` folder
(that's the Backstage source code). You'll create your own app
elsewhere.

### 2.1 Pick a folder

Pick somewhere on your laptop — Desktop, Documents, whatever. In
your terminal, navigate there:

```
cd ~/Desktop
```

(On Windows PowerShell: `cd $env:USERPROFILE\Desktop`.)

### 2.2 Run the creator

Copy-paste this **exactly**:

```
npx @backstage/create-app@latest
```

It will:

1. Ask you to confirm installing `create-app`. Press `y` and Enter.
2. Ask you for a name. Type something like `my-backstage` and press
   Enter.
3. Download a lot of stuff. This takes 5–15 minutes. You'll see
   many log lines. That's normal.

When it's done you'll see a message like:

> ✅ Successfully created my-backstage

There is now a folder called `my-backstage` on your Desktop.

### 2.3 Start it up

```
cd my-backstage
yarn dev
```

This takes 2–5 minutes the first time (it's compiling). You'll see
many lines scroll by. Watch for:

> [0] webpack compiled successfully
> [1] Listening on :7007

That means: the website is running at `http://localhost:3000` and
the backend is running at `http://localhost:7007`.

### 2.4 Open it

In your browser, go to:

```
http://localhost:3000
```

You should see the Backstage home page. Click "Guest" to sign in
(it's a yellow button). You're in.

Click around: "Home", "Catalog", "APIs", "Docs", "Create...". Most
pages will show example entries.

**If this step fails**: see Part 7 (troubleshooting).

Leave it running. To stop it later, go to the terminal and press
`Ctrl+C`.

---

## Part 3 — Put One Real API into the Catalog

Right now the catalog shows example services. Let's add a real one.

### 3.1 Find an OpenAPI spec

Easiest: use the public Swagger Petstore spec. URL:
`https://petstore3.swagger.io/api/v3/openapi.json`

If you have your own API spec, even better. Save it as a `.yaml` or
`.json` file you can refer to.

### 3.2 Create a catalog YAML file

**Stop Backstage first** (`Ctrl+C` in the terminal running it).

In your `my-backstage` folder, find the `examples` subfolder. Open
`examples/entities.yaml` in a text editor (VS Code, Notepad++,
anything).

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

Save the file.

### 3.3 Start Backstage again

```
yarn dev
```

Wait for "webpack compiled successfully" again. Refresh your browser.

### 3.4 Find your API

In the Backstage sidebar click "APIs". You should see "petstore" in
the list. Click it. You'll see the API spec rendered with every
endpoint, request body, response schema, etc.

**If petstore doesn't appear**: check the terminal for YAML errors.
Most common: wrong indentation (YAML wants spaces, not tabs).

---

## Part 4 — Generate Your First AI Test (Manual, No Plugin)

This is the **Option A** path from the strategy doc. Simplest,
works today, no plugin development.

### 4.1 Copy the OpenAPI spec

Open the raw URL in a new tab:
`https://petstore3.swagger.io/api/v3/openapi.json`

Select all (`Ctrl+A` / `Cmd+A`), copy.

### 4.2 Ask an AI to generate tests

Pick your AI:

**Option 4.2a — Claude / ChatGPT in your browser (easiest)**

Open Claude (https://claude.ai) or ChatGPT (https://chat.openai.com).
Paste this prompt:

> I'm going to give you an OpenAPI spec. Generate a Playwright API
> test suite in TypeScript that:
> - Tests the happy path for each endpoint (GET, POST, PUT, DELETE)
> - Tests one 400 error case and one 404 error case
> - Uses `@playwright/test` and `request.newContext()`
> - Uses `https://petstore3.swagger.io/api/v3` as the base URL
> - Outputs one single .ts file
>
> Here is the spec:
>
> [paste the OpenAPI spec here]

Press Enter. Wait for the response. Copy the code it gives you.

**Option 4.2b — Ollama on your laptop**

In a new terminal (leave Backstage running in the other one):

```
ollama run codellama "generate a Playwright TypeScript test suite for the Petstore API at https://petstore3.swagger.io/api/v3. Cover GET /pet/{petId}, POST /pet, and one 404 case. Output only code."
```

It will print the code directly. Copy it.

Note: local Ollama on 8 GB RAM is slow (30–90 seconds per
response) and lower quality than cloud AI. For your first test, use
the browser AI. Come back to Ollama once you want privacy or want to
stop paying.

### 4.3 Save the test file

Make a new folder **outside** `my-backstage`:

```
cd ~/Desktop
mkdir qa-tests
cd qa-tests
mkdir petstore
cd petstore
```

Open a text editor in that folder. Create a file called
`petstore.spec.ts` and paste the code the AI gave you.

### 4.4 Install Playwright

Still inside `~/Desktop/qa-tests/petstore`:

```
npm init -y
npm install -D @playwright/test
npx playwright install chromium
```

Wait for it to finish (couple minutes).

### 4.5 Run the test

```
npx playwright test
```

You should see test results — some passing, maybe some failing.
**Both outcomes are fine for now.** You have an end-to-end loop:
Backstage → OpenAPI spec → AI → Test code → Real HTTP calls →
Results.

### 4.6 If tests fail, that's interesting

Failures usually mean one of:

- The AI invented an endpoint or field (most common). Fix the
  prompt and regenerate.
- The API changed since the spec was written (bug!).
- The test has bad assertions. Fix manually or regenerate.
- You're being rate-limited. Slow down or add delays.

This *is* the QA work. Welcome.

---

## Part 5 — Save Your Working Loop

You now have a 6-step loop. Write it down in a file so future-you
doesn't forget:

Create `~/Desktop/qa-tests/README.md`:

```markdown
# My QA Loop

1. Open Backstage → APIs tab.
2. Pick an API entity.
3. Copy its OpenAPI spec URL or content.
4. Paste into Claude/ChatGPT with the prompt from prompts/api-tests.md.
5. Save the response to qa-tests/<api-name>/.
6. Run `npx playwright test`.
7. Fix or regenerate as needed.
```

Then save your prompt in `~/Desktop/qa-tests/prompts/api-tests.md`
so you can reuse it.

This tiny doc + your working code **is your MVP done**.

---

## Part 6 — Level Up (Only After the Above Works)

Only after you've done Parts 1–5 successfully, consider these:

### 6.1 Add a second API

Repeat Part 3 and Part 4 with a different OpenAPI spec. Good ones
for practice:

- Midtrans sandbox: https://api-docs.midtrans.com
- JSONPlaceholder: https://jsonplaceholder.typicode.com
- Your own company's API, if you have a spec

### 6.2 Wire Ollama through Backstage's proxy

So you can call Ollama from the Backstage UI instead of from
terminal. Edit `my-backstage/app-config.yaml` and add:

```yaml
proxy:
  endpoints:
    /ollama:
      target: http://localhost:11434
      credentials: dangerously-allow-unauthenticated
      allowedMethods: ['POST', 'GET']
      allowedHeaders: ['Content-Type']
```

Restart Backstage. Now any fetch to
`/api/proxy/ollama/api/generate` will hit your local Ollama.

### 6.3 Build a custom Scaffolder action

This is the big upgrade: a Backstage form that takes an API and a
model, and outputs generated tests with one click. See
§A.3 in `qa-discovery-plan.md` for the full code skeleton. Don't
attempt this until Parts 1–5 are comfortable — it needs you to edit
TypeScript and understand the Backstage backend wiring.

### 6.4 Try Ollama Cloud

If you have access to a hosted Ollama endpoint, change the `target`
URL in the proxy config from `http://localhost:11434` to your cloud
URL. Everything else stays the same.

---

## Part 7 — Things That Break, and Fixes

### "command not found: yarn"
Run `corepack enable` again. Close and reopen the terminal.

### "EACCES" or permission errors during npm/yarn install
Don't use `sudo`. Instead:
- On Mac/Linux: `sudo chown -R $(whoami) ~/.npm`
- Or install Node via https://nodejs.org properly (it fixes perms).

### Port 3000 or 7007 already in use
Something else is running there. Either stop that thing, or change
Backstage's port in `app-config.yaml` under `app.baseUrl` /
`backend.baseUrl`.

### Backstage won't start: "Cannot find module ..."
Run `yarn install` inside `my-backstage` again. If that fails,
delete `node_modules` and `.yarn/cache` and retry.

### Petstore spec doesn't load in the UI
Make sure you have internet. The `$text` URL is fetched live. If
you're offline, download the spec JSON and reference it with a
local path instead.

### YAML error in `examples/entities.yaml`
YAML is strict. Common problems:
- Tabs instead of spaces (it hates tabs).
- Missing `---` separator before a new entity.
- Wrong indentation under `spec:`.
Copy the snippet in §3.2 exactly.

### Ollama is super slow
Your model is too big for your RAM. Try a smaller one:
`ollama pull mistral:7b` or `ollama pull phi3:mini`.

### Playwright tests all fail with timeouts
The API is unreachable or rate-limiting you. Try the URL in your
browser first. If you're being rate-limited, add waits or use a
sandbox with no rate limits (Petstore is usually fine).

### AI output is garbage
Your prompt needs work. Specifically:
- Tell it the base URL explicitly.
- Tell it the test framework explicitly.
- Tell it to output ONLY code.
- Give it one endpoint at a time, not the whole spec.

---

## Part 8 — Done. What You Have

If you followed all of Parts 1–5, you now have:

- Backstage running on your laptop.
- One real API registered in its catalog.
- A working prompt that produces usable test code.
- A test suite that actually runs against a real API.
- A written 6-step loop you can repeat for any API.
- Any bugs you find during testing.

This is the complete QA MVP described in
`qa-discovery-plan.md`. Everything past this (custom Scaffolder
actions, MCP integration, CI automation, permissioning) is
optional polish.

Nice work.
