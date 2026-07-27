# Machine & Instance Setup

This blueprint is cloned to the same path relative to `~` on every machine I use: `~/Documents/GitHub/ai-dev-blueprint/`. `~` resolves to the correct home directory on each platform, so every `@~/Documents/GitHub/ai-dev-blueprint/...` include just works.

| Platform | `~` resolves to |
|---|---|
| macOS | `/Users/<user>` |
| WSL | `/home/<user>` |
| Windows native | `C:\Users\<user>` |

**Windows native caveat:** if `@~/...` imports in `CLAUDE.md` don't expand on Windows-native Claude Code, fall back to a full absolute path in that machine's `~/.claude/CLAUDE.md` only (e.g. `@C:/Users/<user>/Documents/GitHub/ai-dev-blueprint/AGENTS.md` — forward slashes). The blueprint's internal `@setup.md` / `@preferences.md` imports are relative and work unchanged.

## Multi-Instance Setup

Each machine runs multiple Claude Code instances with isolated accounts. The env var `CLAUDE_INSTANCE` identifies which one you are. Check it via `echo $CLAUDE_INSTANCE` if needed.

| Instance | Command | Config Dir | Account | Terminal Color |
|---|---|---|---|---|
| eventus | `claude-eventus` | `~/.claude-eventus/` | Eventus work account | Midnight (blue) |
| tcp | `claude-tcp` | `~/.claude-tcp/` | TCP work account | Aubergine (purple) |
| almir | `claude-almir` | `~/.claude-almir/` | Almir personal account | Lead (gray) |
| (default) | `claude` | `~/.claude/` | Default account | Default |

Shared across instances (symlinked): `plugins/`, `skills/`, `chrome/`, `CLAUDE.md`.
Isolated per instance: auth/credentials, sessions, history, project memory, `.claude.json` (MCPs), `settings.json`.

### How the sharing works

Each non-default instance has these symlinks pointing at the default:

```
~/.claude-<name>/CLAUDE.md → ~/.claude/CLAUDE.md
~/.claude-<name>/skills/   → ~/.claude/skills/
~/.claude-<name>/plugins/  → ~/.claude/plugins/
~/.claude-<name>/chrome/   → ~/.claude/chrome/
```

So adopting the blueprint on a new machine only requires touching `~/.claude/` — the other instances inherit automatically once those symlinks exist.

> **Recreating everything on a fresh machine** — launchers, per-profile `settings.json` deltas, the
> Codex homes, and the cross-tool review policy — is a step-by-step procedure in
> [how-to-profiles.md](how-to-profiles.md). This section is the concept; that doc is the checklist.

## Codex Instances

Codex uses the same isolation idea as Claude, keyed by `CODEX_HOME` instead of `CLAUDE_CONFIG_DIR`.

| Instance | Command | Config Dir | Account | Terminal Color |
|---|---|---|---|---|
| almir | `codex-almir` | `~/.codex-almir/` | Personal | Black |
| vvs | `codex-vvs` | `~/.codex-vvs/` | VVS work | Midnight (blue) |
| (default) | `codex` | `~/.codex/` | Default | Default |

Each named home sets `cli_auth_credentials_store = "file"` in its `config.toml` so logins stay
isolated. The Codex review policy ([codex-review-policy.md](codex-review-policy.md)) is symlinked
into each home as `AGENTS.md`, so `git pull` in the blueprint updates it everywhere — same pattern as
Claude's `CLAUDE.md`. Named Codex launchers run `codex --yolo`.

## MCP Servers

MCPs are configured **per instance** in each instance's `.claude.json` file (not shared across instances).

Currently installed: none.

Perplexity (`perplexity-ask`) was removed 2026-07-02 — Gemini grounded search superseded it for all web research. Do not re-add it.

Gemini is **not** an MCP. It's the `/gemini` skill (CLI-based) — see [tool-usage.md](tool-usage.md). This is the web-research tool, and it's a **required** link in the chain (CLI-only; no MCP). If an old `~/.claude/settings.json` still carries a `mcpServers.gemini` (or `perplexity-ask`) block, remove it — those are stale.

To manage MCPs for a specific instance:

```bash
CLAUDE_CONFIG_DIR=~/.claude-<name> claude mcp add <server> ...
CLAUDE_CONFIG_DIR=~/.claude-<name> claude mcp list
CLAUDE_CONFIG_DIR=~/.claude-<name> claude mcp remove <server>
```
