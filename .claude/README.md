# `.claude/` Directory Guide

This directory configures Claude Code for the Assurance-CI project. It is read automatically by both `claude-code-action@v1` in CI and the `claude` CLI locally.

---

## Directory Map

```
.claude/
  README.md              ← this file
  settings.json          ← tool permission allowlist + PostToolUse hooks
  mcp.json               ← MCP server declarations
  rules/
    design-system.md     ← UI design tokens (Pfizer blue / clinical navy)
  agents/
    test-writer.md       ← writes .feature + test_*.py into generated/$STORY_ID/
    (research-assistant is a user-level agent now: ~/.claude/agents/research-assistant.md)
    docs-writer.md       ← keeps PROGRESS.md, ADRs, REGISTER.md in sync
  skills/
    test-generation/
      SKILL.md           ← main CI skill; invoked by assurance.yml step 16
    ship-it/
      SKILL.md           ← stage + commit + push helper
  hooks/
    format-file.sh       ← auto-formats Python after every Write/Edit
    pre-commit.sh        ← secrets detection + domain purity check
```

---

## Agents

Agents are sub-sessions spawned inside a Claude Code skill or session. They have a narrower tool allowlist and a focused system prompt.

| Agent | When to invoke | How |
|-------|---------------|-----|
| `test-writer` | Automatically — called by the `/test-generation` skill at step 4 | Not invoked manually |
| `research-assistant` | Before integrating any new library or calling any external API | Type `/research-assistant` in a Claude session with the library name in context |
| `docs-writer` | After a code change that affects public behaviour | Type `/docs-writer` — it syncs PROGRESS.md, ADRs, and REGISTER.md |

---

## Skills

Skills are multi-step protocols that Claude follows when you type `/<name>`.

| Skill | Trigger | What it does |
|-------|---------|--------------|
| `/test-generation` | Typed manually or injected by assurance.yml | Reads story → builds context → invokes `test-writer` → runs pytest → classifies failures |
| `/ship-it` | Type in any Claude session after staging changes | Validates → commits → pushes |

---

## Hooks

Hooks run shell commands in response to Claude tool calls. They are configured in `settings.json` under `hooks`.

| Hook | Trigger | Script | Effect |
|------|---------|--------|--------|
| `PostToolUse` (Write/Edit) | After every `Write` or `Edit` tool call | `format-file.sh` | Runs `ruff format` silently on `.py` files — keeps code formatted with zero agent tokens wasted |
| `PreToolUse` (Write/Edit) | Before every `Write` or `Edit` tool call | inline `echo` | Reminds Claude that `src/domain/` must stay pure — I/O belongs in `scripts/` |

**Debugging hooks:** If a hook fails, check `~/.claude/logs/` for output. `format-file.sh` exits 0 on any non-Python file and on ruff errors — it is intentionally non-blocking.

---

## MCP Servers

Declared in `mcp.json`. Auto-loaded by `claude-code-action@v1` and available locally via `claude` CLI.

| Server | Package | What it enables |
|--------|---------|----------------|
| `context7` | `@upstash/context7-mcp` | Version-specific docs for pytest-bdd, Playwright, Next.js — prevents hallucinated API calls |
| `fetch` | `@modelcontextprotocol/server-fetch` | Convert any web page to Markdown for Claude to read |
| `playwright` | `@playwright/mcp` | Headless Chromium control — used during test generation for live app introspection |

> **Note:** The `playwright` MCP is injected at runtime by assurance.yml step 12 rather than committed here. This keeps the local `mcp.json` lean — the Playwright server is only needed in CI where a running app is available.

---

## Settings (`settings.json`)

Contains the tool permission allowlist. The committed version is **restrictive** (local dev safe). The CI workflow **overrides** it at runtime (step 13) with a broader allowlist so test-generation can read/write freely.

If you need to add a new tool permission for local use, edit `settings.json` under `permissions.allow`. Use the narrowest pattern possible (e.g. `Bash(python scripts/*)` not `Bash(*)`).
