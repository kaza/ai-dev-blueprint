# AI tool usage

Which AI handles which job, and why. The short answer: use multiple models for independent review, and let **Fable** arbitrate.

## Roles

| Task | Tool | Why |
|---|---|---|
| Primary coding / driver | Claude Opus | Best at judgment, synthesis, multi-file reasoning |
| **Final review arbitration** | **Fable** | The tiebreaker across all reviewers — not Opus |
| Plan review (pre-code) | Gemini + Codex | Independent second opinions catch bad plans early |
| Code review (post-code) | Gemini + Codex + CodeRabbit | Multi-review; every finding must be addressed or rejected |
| Optional third review + brainstorm | Cursor (`/cursor` skill) | Composer 2.5. **Optional / experimental — reintroducing.** Not in the default loop. |
| Web research / search / brainstorm | Gemini (`/gemini` skill) | **Required** link. CLI-only (no MCP). Default model `pro`. |

## Hard rules

- **Gemini MUST run on a PRO model.** **Never Flash** — too weak for brainstorm/research/visual work; using it is a mistake.
  - **CLI (`/gemini` skill):** `-m pro`. This is the required link for web research.
  - **MCP (`mcp__gemini__*`, where a project has it):** the working Pro model is **`gemini-3.1-pro-preview`**.
    🚨 **`gemini-3-pro-preview` 404s** — the call fails with `MCP error -32603 ... Not Found`. It is
    still listed in the tool's enum, so it looks valid and isn't. Pass `gemini-3.1-pro-preview`
    explicitly rather than relying on the default. (Confirmed 2026-07-29.)
- **Perplexity is removed** — Gemini replaced it for all web research. Do not reinstall or call it.
- **"Ask Codex" / "check with Codex"** always goes through the `/codex-review` skill, never a standalone tool.
- **Default code review = Gemini + Codex + CodeRabbit in parallel, in the background**; **Fable** reads all three and arbitrates (not Opus). Cursor is **optional/experimental** — add it when you want a fourth angle; explicit "cursor review" / "check with Cursor" = Cursor only (`/cursor`), skipping the others.
- **Cursor uses Composer 2.5 only** (never `-fast`, never another model family). If `cursor-agent` fails with "Connection lost / exceeded max retries", **check Tailscale first** (MagicDNS breaks its streaming connection) — see the `/cursor` skill's troubleshooting.
- **Never install Python packages into system Python.** Always `python3 -m venv` first.

## Why three reviewers

Different models have different blind spots:

- **Gemini** — strong at architectural / diagnostic critique, **and the strongest of the three at
  VISUAL/SPATIAL work** (reading renders and screenshots, 3D geometry, "does this shape do the job").
  Claude cannot see 3D at all, so on anything geometric Gemini Pro is the eyes — send it images.
- **Codex** — strong at subtle bugs and edge cases
- **CodeRabbit** — strong at conventions, lint-class issues, PR-level scope
- **Fable** — reads all three and arbitrates; rejects bad suggestions with written rationale (Opus drives, Fable arbitrates)

### Proof in practice

A recent triple-review on a real PR caught issues that would have slipped through any single reviewer:

**Codex found:**
- "Base-class fields are authoritative" — an invariant violation
- "Int narrowing + zero-fallback" — a silent numeric bug
- "All trades visible always" — a visibility/scope error
- "Trade timestamps use consumer state" — a data-model mix-up

**Gemini found:**
- The architectural and diagnostic issues Codex glossed over

Neither model alone would have caught both buckets. Running them in parallel and then arbitrating is what makes the loop worth the cost.

Running all three is mandatory for real features. Fast track (bug fixes) drops to one. See [how-to-review-code.md](how-to-review-code.md) for the full flow.

## See also

- [how-to-review-spec.md](how-to-review-spec.md) — the pre-code plan review flow
- [how-to-review-code.md](how-to-review-code.md) — the post-code review loop with Fable arbitration
- [how-to-profiles.md](how-to-profiles.md) — recreate all Claude/Codex profiles on a new machine
- [skills/](skills/) — the actual `/gemini` and `/codex-review` skill definitions
