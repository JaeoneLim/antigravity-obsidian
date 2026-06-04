# antigravity-obsidian: Antigravity CLI Instructions

This repo is a private, Antigravity-CLI–oriented fork of
[claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian) (MIT). It builds
persistent, compounding Obsidian wiki vaults using Andrej Karpathy's LLM Wiki
pattern. The skills are written in the cross-platform Agent Skills format, so they
run under **Antigravity CLI** (`agy`) — Google's Go-based terminal agent that
replaced Gemini CLI — alongside Claude Code, Gemini CLI, Codex, Cursor, and Windsurf.

> Antigravity CLI reads `AGENTS.md` at the project root automatically and prepends
> it to every prompt. This file (`ANTIGRAVITY.md`) is the Antigravity-specific
> companion. For full context, read `AGENTS.md` and the project `CLAUDE.md` too.

## Skills Discovery

Skills live in `skills/<name>/SKILL.md` (kepano Agent-Skills convention: `name` +
`description`; the optional `allowed-tools` field is Claude-Code-only and is ignored
by Antigravity).

Make them available to Antigravity CLI as slash commands:

```bash
bash bin/setup-multi-agent.sh   # wires skills/ for every supported agent, incl. Antigravity
# (the Antigravity step it performs:)
#   project-level:  .agents/skills                          -> ./skills
#   gemini-family:  ~/.gemini/skills/antigravity-obsidian   -> ./skills
agy inspect                     # confirm loaded AGENTS.md + Agent Skills + MCP servers
```

If a skill is not registered as a slash command on your Antigravity build, you can
still run it: tell the agent to "read `skills/<name>/SKILL.md` and follow it."
`AGENTS.md` / `ANTIGRAVITY.md` are always in context, so the agent can always locate
and execute any skill by file.

## Skills

| Skill | What it does |
|---|---|
| `wiki` | Scaffold a vault, manage hot cache, route to sub-skills |
| `wiki-ingest` | Read sources (files, URLs, images) → 8–15 cross-linked wiki pages each |
| `wiki-query` | Answer from the wiki (quick / deep depth modes) |
| `wiki-lint` | Health check: orphans, dead links, stale claims, gaps |
| `wiki-cli` | Obsidian CLI transport wrapper (default desktop mutation path) |
| `wiki-retrieve` | Hybrid BM25 + contextual-prefix + cosine rerank (opt-in) |
| `wiki-mode` | Methodology modes: LYT / PARA / Zettelkasten / Generic |
| `wiki-fold` | Log rollups (DragonScale Mechanism 1, opt-in) |
| `save` | File the current conversation as a wiki note |
| `autoresearch` | Autonomous research loop: search → fetch → synthesize → file |
| `canvas` | Create / edit Obsidian canvas (`.canvas`) files |
| `think` | 10-principle thinking loop for decisions / audits |
| `defuddle` | Clean web pages before ingest (saves 40–60% tokens) |
| `obsidian-markdown` | Obsidian Flavored Markdown reference |
| `obsidian-bases` | Obsidian Bases (`.base`) native database views |

## Trigger Phrases (examples)

- "set up wiki" → `wiki`
- "ingest https://…" / "ingest this pdf" → `wiki-ingest`
- "what do you know about X" → `wiki-query`
- "lint the wiki" → `wiki-lint`
- "research [topic]" → `autoresearch`
- "save this conversation" → `save`

## Subagents

`agents/` holds three subagent definitions (`wiki-ingest`, `wiki-lint`, `verifier`).
Antigravity CLI's **asynchronous subagents** map naturally onto these: dispatch a
long ingest / lint / verify to a background agent and keep prompting in the
foreground. The `verifier` agent is read-only (advisory pre-commit audit).

## Vault Conventions

- `.raw/` — source documents, immutable (never modify)
- `wiki/` — agent-generated knowledge (you own this)
- `wiki/hot.md` — recent-context cache (~500 tokens), read first at session start
- `wiki/index.md` — master catalog
- `.raw/.manifest.json` — delta tracking for ingest

## Bootstrap (first session)

1. Read this file + `AGENTS.md` + the project `CLAUDE.md`.
2. If `wiki/hot.md` exists, read it silently to restore recent context.
3. Wait for the user to type `/wiki`, `ingest`, or `query`.

## MCP (optional)

Antigravity CLI configures MCP via its own `mcp_config.json` (not the
`claude mcp add-json …` form shown in the README). Point it at the same Obsidian MCP
servers — `mcp-obsidian` (Local REST API) or `@bitbonsai/mcpvault` (filesystem) — and
adapt the config shape for `agy`. Server details: `skills/wiki/references/mcp-setup.md`.

## Differences vs Claude Code

- `hooks/hooks.json` and `.claude-plugin/` are Claude-Code-specific and are ignored
  by Antigravity. The PostToolUse auto-commit hook does **not** run under `agy` —
  see **Auto-commit (Obsidian Git)** below to restore that behaviour.
- `allowed-tools` in SKILL.md frontmatter is ignored by Antigravity (harmless).
- This fork keeps full Claude Code compatibility — nothing Claude-specific was removed.

## Auto-commit (Obsidian Git)

Claude Code auto-commits vault writes through `hooks/hooks.json`
(PostToolUse → `git add wiki/ .raw/`). That hook does **not** run under `agy`, so
use the [Obsidian Git](https://github.com/Vinzent03/obsidian-git) plugin to keep the
same "every change gets committed" behaviour:

1. In Obsidian: **Settings → Community plugins → Browse →** search **"Obsidian Git"**
   (by Vinzent03) → **Install → Enable**. (It is *not* bundled with this vault.)
2. **Settings → Obsidian Git** → set **"Vault backup interval (minutes)"** to `15`
   (or your preference). Optionally enable **"Pull updates on startup"** and
   **"Push on backup"** to stay in sync with `origin`.
3. This folder is already a git repo, so commits start on the next interval. Make
   sure `git status` is clean before the first run.

Notes:

- Obsidian Git commits the **whole vault** on a timer, vs. the Claude hook's
  per-write `wiki/ .raw/` staging — coarser but simpler, and fine for single-writer
  `agy` sessions.
- It does **not** consult `scripts/wiki-lock.sh` advisory locks; those only matter
  for parallel multi-writer ingest, which is uncommon in interactive `agy` use.
- Prefer manual control? Skip the plugin and run `git commit` yourself between
  `agy` tasks — nothing in the skills depends on auto-commit.

## Upstream

Private fork of `AgriciDaniel/claude-obsidian` (MIT). Pull updates with:

```bash
git fetch upstream && git merge upstream/main   # or: git rebase upstream/main
```
