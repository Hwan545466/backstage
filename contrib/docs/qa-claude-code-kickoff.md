# Starting Claude Code on Your Laptop — Copy-Paste Prompts

When you switch to Claude Code on your personal laptop, you lose
the chat history from wherever you started. This doc gives you the
exact prompts to paste so you pick up cleanly.

All prompts assume you're in a terminal, in a folder that has this
repo cloned (`git clone <your-fork-url>` and
`git checkout claude/backstage-qa-discovery-plan-zF03Z`).

If you haven't cloned yet, see the bottom of this doc.

---

## Prompt 1 — First Time, Never Run Anything Yet

Paste this into Claude Code:

> I'm a solo QA new to Backstage, working on my personal laptop.
> I want to build a local QA workbench: Backstage as an API
> catalog, plus AI-assisted API test generation.
>
> Context lives in this repo on branch
> `claude/backstage-qa-discovery-plan-zF03Z`:
> - `contrib/docs/qa-discovery-plan.md` — the strategy
> - `contrib/docs/qa-getting-started-for-beginners.md` — my
>   step-by-step guide
> - `contrib/docs/qa-glossary.md`, `qa-troubleshooting.md` — lookup
>
> Read the beginner guide. Then walk me through **Part 1 — Prep
> Your Laptop**, one sub-step at a time. Wait for me to confirm
> each step before moving on.

---

## Prompt 2 — Resuming, You Know Which Part

Replace `<N>` with the part number you stopped at (e.g. Part 3 if
you already have Backstage running with no API yet).

> I'm continuing the QA workbench project. Branch:
> `claude/backstage-qa-discovery-plan-zF03Z`.
>
> I already finished through Part <N-1> of
> `contrib/docs/qa-getting-started-for-beginners.md`. My Backstage
> app is at `~/Desktop/my-backstage` (adjust if different).
>
> Pick up at Part <N>. Walk me through it one sub-step at a time.

---

## Prompt 3 — You Hit an Error

> I'm on **Part <N>, sub-step <M>** of
> `contrib/docs/qa-getting-started-for-beginners.md`.
>
> I ran this command:
>
> ```
> <paste the command>
> ```
>
> And got this error:
>
> ```
> <paste the full error output>
> ```
>
> First check `contrib/docs/qa-troubleshooting.md` for a matching
> entry. If it matches, tell me the fix. If it doesn't, help me
> diagnose.

---

## Prompt 4 — You Want to Level Up (Custom Scaffolder Action)

Only use this after Parts 1–5 are fully working.

> I've finished Parts 1–5 of the beginner guide and my manual AI
> test loop works. Now I want to build the **custom Scaffolder
> action** so I can generate tests from a form in the Backstage UI.
>
> The skeleton is in `contrib/docs/qa-discovery-plan.md` Appendix A
> section A.3. Walk me through creating the scaffolder-backend
> module, writing the action, registering it in
> `packages/backend/src/index.ts`, and testing the template
> end-to-end. One sub-step at a time.

---

## Prompt 5 — You Want to Wire Ollama Through the Proxy

Use after Part 5 works and Ollama is installed.

> Add the Ollama proxy config to my Backstage app. See
> `contrib/docs/qa-discovery-plan.md` §A.2 for the config block.
> My Ollama is running at http://localhost:11434. After adding,
> verify by curling through the proxy and show me the result.

---

## Prompt 6 — You're Lost, Show Me Where I Am

> I've lost track of where I am in the QA workbench project.
> Branch: `claude/backstage-qa-discovery-plan-zF03Z`.
>
> Investigate my current state: do I have a Backstage app created?
> Where? Is it running? What entities are in the catalog? Have I
> generated any tests yet?
>
> Report back with a one-paragraph summary and tell me what the
> logical next step is.

---

## If You Haven't Cloned the Repo Yet

On your laptop:

```
git clone https://github.com/hwan545466/backstage.git
cd backstage
git checkout claude/backstage-qa-discovery-plan-zF03Z
```

Then start Claude Code inside that folder:

```
claude
```

Then paste **Prompt 1** above.

If you don't actually need the Backstage source (you're building
your own app with `npx @backstage/create-app`), you can clone
somewhere smaller or just download the four `contrib/docs/qa-*.md`
files individually and reference them.

---

## Tips for Working Smoothly with Claude Code

- **Use `/fast` mode** for routine stuff (editing YAML, running
  shell commands). It's the same Opus model but with faster output.
- **Don't dump huge error logs** into the chat. Paste the first ~30
  lines and the stack trace; Claude will ask for more if needed.
- **Tell Claude what you don't want** as often as what you want —
  e.g. "don't add error handling I didn't ask for".
- **Commit after each working step.** If a later step breaks
  things, you can recover with `git reset --hard HEAD~1`.
- **Take breaks at the session boundaries** marked in the beginner
  guide (look for "STOP HERE — good place to break" notes).

---

## If You Want to Throw Away the Branch and Start Fresh

Totally valid. The docs were generated mid-process and some parts
may feel over-engineered. On your laptop:

```
git clone https://github.com/backstage/backstage.git
cd backstage
# ignore the branch — just use the official repo as reference
```

Then in Claude Code:

> I'm a solo QA setting up Backstage locally to build an
> AI-assisted API test generator. I've scanned some planning docs
> before but I want to start fresh. Guide me through:
> 1. Creating a Backstage app with `npx @backstage/create-app`
> 2. Adding one OpenAPI API to the catalog
> 3. Generating one Playwright test from that API using Claude
>    (copy/paste, no plugin)
> Keep it minimal. No permissions, no SSO, no Postgres.

That's the whole MVP. The detailed docs are nice-to-have, not
must-have.
