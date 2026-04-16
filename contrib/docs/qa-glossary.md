# QA Backstage Project — Glossary

Terms you'll see in the other docs, in plain English. Don't memorize.
Look things up as needed.

---

**Backstage** — a piece of software that gives you a website (called
a "portal") listing all your APIs, services, docs, etc. Originally
built at Spotify. You run it on your own computer.

**Catalog** — the database inside Backstage that lists your stuff.
Each item in it is called an "entity".

**Entity** — one thing in the catalog. Could be a service, an API, a
person, a team. Defined in YAML.

**API entity** — a catalog entry that represents an API, usually
with a link to its OpenAPI spec.

**Component** — a catalog entry for a piece of software you own
(usually a service or library).

**OpenAPI spec** — a JSON or YAML file that describes an HTTP API:
its endpoints, methods, inputs, outputs. Sometimes called "Swagger".

**Plugin** — a piece of Backstage that adds a feature. Example: the
"Scaffolder" plugin, the "TechDocs" plugin.

**Scaffolder** — Backstage's built-in form-based code generator. You
fill in a form, it runs steps, it writes files.

**Scaffolder template** — a YAML file that defines one form + the
steps to run.

**Scaffolder action** — one step a template can run. Can be built-in
(fetch from GitHub, write a file) or custom (call an AI, call an
external API).

**Ollama** — free software that runs AI models on your laptop. Also
a cloud service with the same interface.

**LLM** — Large Language Model. The generic term for the AI that
generates text. Claude, GPT, Llama, Mistral are all LLMs.

**Node.js / Yarn** — the tools that run JavaScript on your
computer. Backstage is written in JavaScript, so you need these.
Yarn is a "package manager" — it downloads the libraries Backstage
depends on.

**Terminal** — the black window where you type commands. On Mac
it's "Terminal.app". On Windows it's "PowerShell" or "Windows
Terminal". On Linux it's whatever you already have.

**yarn dev / yarn start** — the command that starts Backstage
running on your computer.

**localhost:3000** — an address that means "the website running on
my own computer, on port 3000". Paste it into your browser.

**Port** — a numeric door on your computer. Backstage's frontend
uses port 3000, its backend uses 7007, Ollama uses 11434. Only one
program can use a port at a time.

**app-config.yaml** — the main configuration file for your
Backstage app. Controls which plugins are on, which integrations are
wired up, which catalog sources are loaded.

**Proxy** — a feature of Backstage that forwards HTTP requests from
your frontend to an external server. Used to call Ollama, Jira, etc.
without exposing credentials to the browser.

**Playwright** — a popular test framework for web apps and HTTP
APIs. What we use to run the AI-generated tests.

**MCP** — Model Context Protocol. A standard for AI assistants
(Claude Desktop, Cursor) to connect to external tools like
Backstage. Backstage has an MCP server plugin built in.

**changeset** — a small markdown file in the upstream Backstage
repo that describes a change for release notes. You don't need to
write these unless you're contributing code back to Backstage —
which, as a solo adopter, you are not.

**adopter** — someone who uses Backstage to build a portal for
their organization. Opposite of "contributor", who writes code for
the Backstage framework itself. You are an adopter.
