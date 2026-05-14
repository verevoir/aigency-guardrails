# aigency-guardrails

Universal principles and decision records that every aigency-built project follows by default. Public so projects can read them at any time, fork them where their context diverges, and contribute back where the principles improve.

## What's here

- **`principles.md`** — L1 universal principles. Apply to every project regardless of stack, scale, or domain. These are short, opinionated, and meant to constrain. They are reviewed periodically, not casually.
- **`adrs/`** — L2 decision records. Reusable architectural decisions projects adopt as defaults. Numbered, each in `STATUS / CONTEXT / DECISION / CONSEQUENCES` form. Project-specific overrides go in the project's own `docs/adr/`, not here.

## How projects use these

Each aigency-built project keeps a pointer to this repo (default) or a fork of it. The project's own `CLAUDE.md` says "this project follows the guardrails at <URL>" — the URL is editable. To diverge from a guardrail for a single project, the project records the deviation in its own ADR; to diverge persistently across all your projects, fork this repo and edit.

## L1 / L2 / L3

- **L1 — principles** (here, in `principles.md`): universal practices. Stack-agnostic.
- **L2 — decision records** (here, in `adrs/`): reusable architectural choices. Slightly more specific than L1; still cross-stack.
- **L3 — project-specific guardrails** (in each project's docs repo, in `CLAUDE.md` and `docs/adr/`): the project's own adaptations, additions, and divergences. Always overrides L1/L2 for that project.

## How to fork

Standard GitHub fork. Then update your project's `CLAUDE.md` (or the equivalent pointer file) to point at your fork's URL. The aigency runtime uses whichever URL is recorded, falling back to this canonical repo if the pointer is unset or unreachable.

Fork when you want to:
- Adapt principles to your team's context (e.g. you've codified that all your projects use a specific test framework).
- Add organisation-wide ADRs that should apply across every project you build.
- Remove guardrails you don't want (e.g. the plain-CSS principle if your standard is a CSS framework).

Don't fork to make a one-off change for a single project — that belongs in that project's own ADRs.

## Contributing back

PRs welcome. Substantive guardrail changes should reference real experience (an incident, a recurring miss, a clarification born of confusion). Don't change a guardrail without a story.

## Versioning

The `main` branch is canonical and live — pointers default to it. Projects that want stable, pinned guardrails can replace their pointer with a tag URL (e.g. `https://github.com/verevoir/aigency-guardrails/tree/v1.0`). Tags are cut periodically; consumers choose between always-current and pinned-stable.
