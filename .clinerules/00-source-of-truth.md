# Source of truth — do not fork context

This repo is driven by multiple agent harnesses (Claude Code, Codex, Cline, Antigravity).
`AGENTS.md` at the repo root is the single source of truth for every tool.

- **Do not generate a Memory Bank.** The project context already exists as versioned artifacts —
  treat these as your Memory Bank:
  - `AGENTS.md` — stack, commands, Definition of Done, code standards. Authoritative; read first,
    including its Session Start protocol.
  - `PROGRESS.md` — session state. Read at session start, update at end.
  - `DOMAIN.md` / `SPEC.md` — domain model and BDD acceptance contract.
  - `docs/LEARNINGS.md` — append-only compounding log; never reformat past entries.
  - `docs/adr/` — past decisions; check before guessing at architectural questions.
- Path-scoped rules live in `.claude/rules/` (Claude Code loads them automatically on path match;
  read them yourself when touching those paths):
  - `.claude/rules/design-system.md`
- Same mistake made twice → add a one-line rule to `AGENTS.md` (portable), not to a
  tool-specific file.
- Output dense, direct text. Omit pleasantries.
