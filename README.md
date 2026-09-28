# chatgpt-book-writer

**Write a whole book on your ChatGPT plan.** The Codex CLI that ships with ChatGPT Plus, Pro,
Business and Enterprise connects to the [Proseify](https://proseify.xyz) MCP server, and your book-writing
agent goes from "generate 30 chapters" to "plan, draft, self-edit and score a novel".

[![Listed on mcpservers.org](https://mcpservers.org/badge.svg)](https://mcpservers.org/servers/dionisselami/proseify-mcp) [![MCP Registry](https://img.shields.io/badge/MCP_registry-io.github.Dionisselami%2Fproseify--mcp-4a3f35)](https://registry.modelcontextprotocol.io/v0/servers?search=proseify)

---

## First, the honest part

Two different things are called "ChatGPT", and only one of them can reach Proseify today:

| Surface | Can it connect? | Why |
|---------|-----------------|-----|
| **Codex CLI** (included with your ChatGPT plan) | Yes | Streamable HTTP MCP with a bearer token from an environment variable |
| **ChatGPT desktop app**, where your plan exposes Codex-based MCP config | Yes | Same config file, same env-var token |
| **ChatGPT web custom connectors** (developer mode) | No, not today | Custom connectors accept OAuth or no authentication only — ChatGPT will not present a static API key or custom header to an MCP server |

So this repo wires up the first two, and does not pretend the third works. If OpenAI adds
header-based auth for web connectors, this file changes.

## What Proseify actually is

A hosted MCP server: 80+ public-domain books, 2,500+ searchable chapters, a genre recipe for each
of 11 genres, and a plan → draft → score pipeline. It calls no LLM. Your plan's models do all the
writing, so there is no second token bill hiding in this.

```text
premise  →  genre recipe  →  corpus passages  →  outline  →  chapter drafts  →  evaluate_book  →  revision
```

Never let the model start at "chapter drafts". The ordering is the whole product: a genre
recipe read *before* prose beats any amount of "write like Stephen King" prompting, because it
hands the model concrete numbers — pacing beats, dialogue ratio, sentence-length profile — that
its defaults would otherwise flatten to its own house style.

---

## Quick start

### 1. Get a Proseify key

https://proseify.xyz — sign in, pick a plan (from $9/mo, or the one-time Founding Lifetime tier),
key issued on payment.

### 2. Put the key in your environment

```bash
# macOS / Linux — add to ~/.zshrc or ~/.bashrc, then open a NEW shell
export PROSEIFY_API_KEY="sk-..."

# Windows PowerShell (persists for the user)
[Environment]::SetEnvironmentVariable("PROSEIFY_API_KEY","sk-...","User")

# Windows cmd
setx PROSEIFY_API_KEY "sk-..."
```

Then **restart the app**. A GUI-launched Codex/ChatGPT does not inherit variables set after it
started, and the MCP client can look configured while sending no `Authorization` header at all —
the failure mode is silent, and the tools simply are not there.

### 3. Connect the MCP server

Fastest — let the CLI write the config:

```bash
codex mcp add proseify --url https://mcp.proseify.xyz/mcp --bearer-token-env-var PROSEIFY_API_KEY
```

Or append `config.toml.example` to `~/.codex/config.toml` by hand:

```toml
[mcp_servers.proseify]
url = "https://mcp.proseify.xyz/mcp"
bearer_token_env_var = "PROSEIFY_API_KEY"
enabled = true
```

Confirm with `codex mcp list` — you want `proseify` present, enabled, with your env var named.

### 4. Give the agent its discipline

Copy `AGENTS.md` into your project root. Codex reads it automatically, and it is the difference
between an agent that uses the tools and an agent that uses the tools *properly* — genre recipe
first, corpus grounding, one chapter per turn, per-chapter edit pass, `evaluate_book` as the gate.

### 3. Install the writing skill

```bash
npx skills add https://proseify.xyz --skill anti-prose-slop -y
```

The MCP server gives the agent the *tools*. The `anti-prose-slop` skill gives it the *method* —
genre recipe first, corpus grounding, chapter discipline, and the per-chapter edit pass. Install
both or you get tool access and the same generic prose you had before.

### 5. Premise

> Write a 12-chapter locked-room mystery: a retired detective, a snowed-in country house, a study
> locked from the inside. Budget 32,000 words.

---

## What's in here

| File | Purpose |
|------|---------|
| `config.toml.example` | The `[mcp_servers.proseify]` block for `~/.codex/config.toml` |
| `AGENTS.md` | Agent instructions — drop into any project you want to write a book in |
| `LICENSE` | MIT |

### The five-step loop

1. **Pick the genre before any prose exists.** `list_genres`, then `get_genre_recipe` for the
   closest fit. Blends are allowed — name both and let the recipe argue for one.
2. **Ground the register.** `search_corpus` (or `get_style_references`) for 3–5 model passages
   in the target genre. Read them. This is the calibration step, not decoration.
3. **`plan_book` once.** Hold the outline. Do not re-plan mid-draft; if the outline is wrong,
   fix it deliberately and say so.
4. **Draft chapter by chapter**, pausing after each for the per-chapter edit pass below.
5. **`evaluate_book` before you call it done.** Fix the chapters it flags and re-run the gate.

### The per-chapter edit pass

The failure mode of AI prose is not grammar — it is sameness. Every chapter gets struck against
this list before it counts as drafted:

- `seemed to`, `began to`, `started to`, `could feel` — delete or recast
- filter words: *felt, noticed, watched, saw, heard* — put the reader in the perception instead
- stacked adverbs on dialogue tags; `said` is not a problem, `said loudly, angrily` is
- sentence openings repeated across the chapter (and the weather-opening default)
- three-item lists used as rhythm filler
- dialogue that exists to explain the plot to the reader
- simile endings: the last line becoming a metaphor for the chapter

### If you only read one paragraph

The agent's own model does all the writing — Proseify calls no LLM and stores no manuscript.
There is no token bill from us: it is a corpus, a set of genre recipes, and a pipeline.


---

## Codex notes and gotchas

- **`bearer_token_env_var` is a name, not a value.** It resolves at process launch, so the variable
  must already exist when Codex starts. The single most common failure here is setting the variable
  and not restarting the app.
- **`codex mcp list` can look healthy while auth is broken.** The status line reports configured
  state, not whether the header was sent. If the Proseify tools are missing, check the environment
  of the running process, not the config file.
- **Use `http_headers` only if you must.** It works, but it puts the key in a plain-text file that
  is easy to commit by accident. The env var path exists to prevent exactly that.
- **Long books: keep chapters in files.** Have the agent write `chapters/01.md` after each one. A
  book is longer than any single context window, and the agent's memory of its own manuscript is
  not durable.
- **`evaluate_book` is a gate, not a formality.** It returns structure/pacing/coverage numbers per
  book and flags weak chapters; the loop is fix → re-run until it passes.

| Tool | What it does |
|------|--------------|
| `list_genres` | The 11 genres the corpus covers (theatre, horror, romance, adventure, literary, mystery, gothic, sci-fi, fantasy, comedy, children's) |
| `get_genre_recipe` | Pacing beats, dialogue ratio, sentence-length profile and stylistic anchors derived from that tradition |
| `search_corpus` | FTS5 full-text search across 2,500+ chapters of public-domain classics (quoted phrases, AND/OR/NOT, wildcards) |
| `get_style_references` | Model passages from books in the target genre — the register you are aiming at |
| `plan_book` | A full chapter-by-chapter outline from a one-line premise, each beat carrying a corpus style reference |
| `write_book` | The one-shot flow: plan → draft → evaluate, in a single session |
| `evaluate_book` | Scores a draft against genre benchmarks (structure, pacing, chapter coverage, word budget) so weak chapters get revised |

## The corpus

80+ books and 2,500+ chapters of public-domain literature — Project Gutenberg and similar
sources — every chapter verified against its own file header at download time, so nothing in
the library has a copyright question hanging over it. Works from Austen, Stevenson, Hugo,
Conrad, Verne, the Brontës, Shelley and the gothic masters, grouped by genre and indexed for
full-text search.

Commercial use of what you write on top of it is safe. Full-text searchable chapter by chapter,
not a scrape of titles.

## Pricing

| Plan | Price | Rate limit |
|------|-------|-----------|
| Starter | $9/mo | 120 req/min |
| Pro | $19/mo | 400 req/min |
| Studio | $49/mo | 9999 req/min |
| Founding Lifetime | one-time | 400 req/min, all genres |

Sign in at https://proseify.xyz, pick a plan, and the key is issued the moment the purchase
clears. Cancel from https://proseify.xyz/account; 14-day refund window
(https://proseify.xyz/refunds).

## Other clients

Same Proseify server, same workflow, different config file. The cookbook is the method itself.

- [claude-book-writer — write a book with Claude Code](https://github.com/Dionisselami/claude-book-writer)
- [cursor-book-writer — write a book in Cursor](https://github.com/Dionisselami/cursor-book-writer)
- [gemini-cli-book-writer — write a book with Gemini CLI](https://github.com/Dionisselami/gemini-cli-book-writer)
- [copilot-book-writer — write a book in VS Code with Copilot](https://github.com/Dionisselami/copilot-book-writer)
- [windsurf-book-writer — write a book in Windsurf](https://github.com/Dionisselami/windsurf-book-writer)
- [claude-book-cookbook — recipes for writing a whole book with Claude](https://github.com/Dionisselami/claude-book-cookbook)

## License

MIT for everything in this repository. The corpus texts themselves are public domain.

## Disclaimer

Unofficial. This repository is not affiliated with, endorsed by, or sponsored by OpenAI, ChatGPT or Codex. Client names appear only to describe which configuration file and transport the instructions are for. Proseify is an independent product — https://proseify.xyz.

Keep your Proseify key in local agent config or an environment variable. Never commit it, print it, log it, or paste it into a repository file.

## Links

- **Sign up / key issuance:** https://proseify.xyz
- **Filled-in config for your key:** https://proseify.xyz/agent
- **FAQ (ownership, KDP, what an MCP server is):** https://proseify.xyz/faq
- **Server repo & MCP registry entry:** https://github.com/Dionisselami/proseify-mcp
- **Support:** support@proseify.xyz
