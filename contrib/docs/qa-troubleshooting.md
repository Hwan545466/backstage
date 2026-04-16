# QA Backstage Project — Troubleshooting

Fast lookup for errors you'll hit. Scan for the error text you're
seeing; follow the fix.

---

## Toolchain

### `command not found: yarn`
Run `corepack enable`. Close your terminal and open a new one. Retry
`yarn --version`.

If `corepack` is also missing: `npm install -g corepack`, then
`corepack enable`.

### `command not found: node` or Node version too old
Install or upgrade Node from https://nodejs.org (pick the LTS
version). Close and reopen your terminal after install.

### `command not found: git`
- Mac: `brew install git` (install Homebrew first if missing).
- Windows: download from https://git-scm.com.
- Linux: `sudo apt install git` or equivalent for your distro.

---

## Backstage install / create-app

### `npx @backstage/create-app@latest` hangs or fails

- Check your internet. The installer pulls hundreds of packages.
- Make sure you're not behind a strict corporate proxy without
  config — try on a personal network first.
- If it fails halfway, delete the partial folder and retry.

### `EACCES` or permission errors during npm / yarn install

**Do not use `sudo`**. Instead:

- Mac / Linux: `sudo chown -R $(whoami) ~/.npm`
- Or install Node via the official installer from nodejs.org,
  which sets permissions correctly.

### `yarn install` fails with "cannot find module..."

Delete and retry:

```
rm -rf node_modules .yarn/cache
yarn install
```

---

## Starting Backstage

### Port 3000 or 7007 already in use

Something else is running there. Either:

- Stop the other thing. To find it on Mac/Linux:
  `lsof -i :3000` then `kill <pid>`.
- Or change Backstage's ports in `app-config.yaml`:

```yaml
app:
  baseUrl: http://localhost:3001
backend:
  baseUrl: http://localhost:7008
  listen:
    port: 7008
```

### `yarn dev` starts but the browser shows "can't connect"

- Wait for both `[0]` and `[1]` lines in the terminal. Backend
  takes longer to start than frontend.
- Check the ports: frontend at :3000, backend at :7007.
- If only one started, scroll up and look for errors.

### Guest sign-in button missing

Your `app-config.yaml` may have an `auth.providers` block that
doesn't include `guest`. For local dev, make sure it has:

```yaml
auth:
  providers:
    guest: {}
```

---

## Catalog

### API entity doesn't appear in the UI

- Did you restart `yarn dev` after editing `examples/entities.yaml`?
- Check the backend terminal output for YAML parse errors.
- Did you add the file path to `catalog.locations` in
  `app-config.yaml`? By default `examples/entities.yaml` is already
  listed — if you renamed it, you need to update the path.

### YAML parse error in `examples/entities.yaml`

YAML is strict:
- **No tabs** — only spaces for indentation.
- **Consistent indentation** — 2 spaces per level is standard.
- **Each entity starts with `---`** on its own line.
- **No trailing colons without values** unless you mean empty.

Copy a working snippet exactly, then modify one field at a time.

### OpenAPI spec in the API entity doesn't render

- If using `$text: <url>`, the URL must be reachable from your
  laptop right now. Open it in a browser to check.
- If the content is inline, make sure the YAML indentation is
  correct — OpenAPI specs have deep nesting.
- Check the backend terminal for fetch errors.

---

## AI / Ollama

### Ollama is super slow

Your model is too big for your RAM. Try smaller:

```
ollama pull mistral:7b       # 4 GB
ollama pull phi3:mini        # 2 GB
```

On 8 GB RAM, stick to 7B parameter models or smaller. On 16 GB you
can try 13B.

### `ollama: command not found`

Install from https://ollama.com. After install, you may need to
open a new terminal for the PATH to update.

### AI output is nonsense / invents endpoints

The prompt needs tightening. Good prompts:

- Name the exact test framework (Playwright, Jest+supertest, pytest).
- State the base URL explicitly.
- Tell it to output **only code, no explanations**.
- Feed it **one endpoint at a time**, not the whole spec, for better
  quality.
- Include an example of how you want a single test to look.

If the AI keeps inventing endpoints, the spec you gave it is
probably too big. Summarize or split.

### AI call from Backstage proxy returns 502 / timeout

- Is Ollama actually running? `curl http://localhost:11434/api/tags`
  should return a JSON list.
- Did you restart `yarn dev` after editing the proxy config?
- Check the proxy config spelling: `target: http://localhost:11434`
  (no trailing slash).

---

## Running Generated Tests

### `npx playwright test` says "browser not installed"

```
npx playwright install chromium
```

### All tests time out

- The API is unreachable. Open the base URL in your browser.
- You're being rate-limited. Add `await page.waitForTimeout(500)`
  between requests, or use a sandbox with no rate limits.
- The base URL in the generated test is wrong. Check and fix.

### Tests all pass — suspiciously

AI sometimes generates tests with empty assertions (`expect(true).
toBe(true)`) or assertions that can't fail. Read the code. If in
doubt, deliberately break the API (wrong URL) and re-run — tests
should turn red.

---

## Git / Branch

### `git push` rejected: "updates were rejected because the remote..."

Someone (or you, from another machine) pushed to the branch since
your last pull. Run:

```
git pull --rebase origin <branch-name>
git push -u origin <branch-name>
```

### Wrong branch

Check current branch: `git branch --show-current`. Switch:
`git checkout <branch-name>`. Create if missing: `git checkout -b
<branch-name>`.

---

## When None of the Above Works

1. Copy the exact error message.
2. Note which step of the beginner guide you were on.
3. Paste both into Claude Code with:
   > "I'm on step X of `qa-getting-started-for-beginners.md`. I ran
   > `<command>` and got this error: `<error>`. What now?"

Giving Claude the step number + exact error is 10x better than
"it's broken".
