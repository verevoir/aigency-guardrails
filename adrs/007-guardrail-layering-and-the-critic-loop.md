# ADR 007: Guardrail layering and the critic loop

## Status

Accepted

## Context

When code generation is autonomous — the assistant proposes changes, applies them, and opens PRs without a human reviewing each diff — the human-approval gate at deploy time catches the worst at cutover. But it's a coarse instrument: by the time a reviewer sees the preview, work has already happened. We need guardrails earlier in the pipeline that constrain *what gets generated in the first place*, not just *what gets shipped*.

Two failure modes the deploy-time gate alone does not address:

- **Drift.** Small unjustified deviations from project intent or established principles, each individually defensible but collectively pulling the project off-course.
- **Overreach.** The proposer takes scope beyond what was asked, often "while we're in here" — refactors that touch areas the conversation never authorised.

Universal principles (404 on protected URIs, tests follow intent, low-dependency discipline, expand-and-contract for destructive changes, etc.) are the kind of constraint that should make those failures harder. But they only work if they're consulted *before* the proposer commits to a change — and if there is an explicit check that the proposal honours them.

## Decision

Project guardrails live in three layers plus one cross-cutting artefact, with strict temporal order around code change:

- **L1 — Principles.** Universal cross-cutting practices that apply to every project regardless of stack. Maintained centrally and pointed at, not snapshotted. Update *before* a change that violates them, or document the exception explicitly.
- **L2 — Decision records.** Architectural decisions, project-specific or universal. Numbered, supersede each other via reference, never silently overwritten. Update *before* a change that would conflict.
- **L3 — Per-module / per-feature notes** (typically a CLAUDE.md or AGENTS.md in the relevant directory). Derived description of current state, updated *after* a change to reflect what is.
- **Intent.** Why we're still doing this project — pivots, rejected directions, constraints from outside the codebase. Separate from L2: L2 decides *how*; intent records *why we continue*.

The temporal order is load-bearing. If L1/L2 update *after* code changes, they become rationalisation rather than ballast. Enforce mechanically: any stage that proposes changes to a structurally-important area refuses to generate code unless the same diff includes the precedent L1/L2 change or a recorded exemption in the intent file.

Every autonomous generation runs a **two-role critic loop**:

- **Proposer** generates the change against the task, with access to L1, the relevant L2, the intent, and L3 notes for the area.
- **Critic** reads the proposed change plus the same context, and is explicitly prompted to argue *against* the proposal: "what in the intent contradicts this? which decision record would need to be superseded? is the proposer overreaching scope?"
- If proposer and critic converge, the change proceeds to the human-approval gate.
- If they cannot converge, **a human is the tie-breaker.** The disagreement itself is signal: the human sees the proposer's case and the critic's objection side-by-side. This is a higher-leverage human moment than reviewing a finished PR.

Two distinct approval roles exist, and a project declares which apply:

- **Project approval** — "this preview behaves as I intended and can be merged." The project owner or another product-accountable party, exercising judgement about product behaviour, not code. The universal default: every project has at least one project approver.
- **Dev approval** — "I agree with the state of the code and this can merge." A developer exercising judgement about craft, security, and code-level fit. Opt-in per project: some customers have no developers; others have periodic access to them; a few have constant access.

The two roles answer different questions and read different artefacts. Project approval looks at the running preview. Dev approval looks at the diff, the critic's findings, and the relevant principles. A project may require either, both, or both-with-different-triggers (e.g. dev approval required for changes touching auth/security, project approval sufficient for everything else). When dev approval is required but the dev reviewer is unavailable, the policy must declare a fallback — defer until they're back, or escalate to the project approver with a translation of what the critic flagged.

The critic-loop tie-breaker is dev-shaped when a dev approver exists. When none does, the tie-breaker escalates to the project approver with a plain-language summary of the disagreement and a clear decision surface ("allow, defer, abandon"). The critic's job in that case includes translating its objection out of code-jargon.

The critic prompt is short relative to generation; the loop is cheap.

## Consequences

- Autonomous generation isn't allowed to run without the critic wired up. From day one of any generation feature, both roles exist; there is no "just generate" path that ships.
- The critic must have teeth. A critic that always green-lights is theatre; one too adversarial blocks normal change. Calibrate early on real diffs from the project's own usage, watch both failure modes, refine the prompt accordingly.
- Intent becomes a first-class artefact, on par with decision records. Every new project starts with an intent file, seeded from the initial scope conversation.
- PR descriptions for autonomous-generated PRs include the critic's findings and how they were resolved. This makes the gate reviewer's job easier and creates a corpus of "what the critic catches" for future tuning.
- Project setup includes naming approval roles: who is the project approver (mandatory), is there a dev approver (optional), and what change types — if any — require both. The default profile for a no-dev customer is project-approval-only, with the critic loop translating any escalations into product-language; this is the model most engagements should start with.
- The system must be honest when a required reviewer is unavailable. A change blocked on a dev approval that hasn't come stays explicitly pending — we don't silently promote it via fallback unless the project's policy authorises that path.
- L1 principles live in a shared, pointer-referenced location (e.g. the universal guardrails repo). L2 decision records start empty per project, accumulate over time. L3 notes are seeded at project creation and drift with the code.
- This layering applies recursively. The discipline holds whether the project being built is a customer's product or the platform that builds customer products.
