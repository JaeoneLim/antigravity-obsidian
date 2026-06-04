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
  by Antigravity. The PostToolUse auto-commit hook does **not** run under `agy`; use
  the Obsidian Git plugin (15-min auto-commit) or an Antigravity hook instead.
- `allowed-tools` in SKILL.md frontmatter is ignored by Antigravity (harmless).
- This fork keeps full Claude Code compatibility — nothing Claude-specific was removed.

## Upstream

Private fork of `AgriciDaniel/claude-obsidian` (MIT). Pull updates with:

```bash
git fetch upstream && git merge upstream/main   # or: git rebase upstream/main
```
