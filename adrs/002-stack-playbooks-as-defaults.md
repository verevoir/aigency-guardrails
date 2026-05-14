# ADR 002: Stack playbooks as defaults with rationale

## Status

Accepted

## Context

The tech-stack stage of a project produces a stack that the walking-skeleton and feature stages then build on. Two anti-patterns to avoid:

- **Open-ended choice every time** — reasoning from first principles for every project means picking differently across projects, losing the leverage of having seen this problem before. Quality is uneven; the assistant is consulted constantly.
- **Rigid template** — one stack fits all customers. Customer constraints ("we already have Datadog", "our team knows Rails") are forced into a shape they don't fit, or rejected outright.

The principle that an uninitiated human developer should be able to maintain the project pushes toward conventional, well-staffed stacks with strong industry support. XP-derived practices (TDD, pair programming, continuous deployment, simple design) are non-negotiable regardless of stack — they are not a stack choice.

## Decision

- **Stack playbooks are *defaults with rationale*.** Each playbook covers a well-defined stack (e.g. Next.js + Postgres + GCP) and includes file structure, naming conventions, deploy pipeline shape, observability defaults, ADR templates, and the *reasoning* behind each.
- **The tech-stack stage selects a playbook based on customer constraints**, not from scratch. The assistant matches constraints to playbooks rather than designing a stack from first principles.
- **Deviations from a playbook produce ADRs** in the project's `docs/adr/`, explaining why. The deviation ADR is the audit trail; the human inheriting the project sees it and understands the local divergence.
- **XP-derived practices apply regardless of playbook.** Tech stack changes do not override XP. A playbook may instantiate the practices differently (test framework, deploy mechanism), but the practices themselves are constant.

## Consequences

- The agency's leverage scales with the playbook library. Adding a playbook (after sufficient real-project use to validate it) makes future engagements cheaper and more predictable.
- Tech-stack conversations are short. The project owner describes constraints; the assistant picks a playbook and explains the rationale; the owner approves or pushes back. If they push back hard enough, escalation to a human consultant produces a deviation ADR.
- The first playbook is whichever stack the agency has the most production experience with. Subsequent playbooks are added only after a real engagement validates them — playbooks are not a speculative backlog.
- Playbook content is versioned. Improvements to a playbook do not retroactively change live customer projects; they flow forward at next-engagement time. A project pinned to playbook version 1.0 stays there until explicitly upgraded.
