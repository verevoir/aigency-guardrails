# ADR 004: Project owner at the keyboard, consultant as async escape hatch

## Status

Accepted

## Context

Two failure modes for AI-driven project delivery:

- **Engineer-at-the-keyboard**: the system becomes a sophisticated coding assistant. Useful, but the customer is gated on consultant availability and the conversation runs in technical vocabulary the customer can't sanity-check.
- **Customer-at-the-keyboard with no escape hatch**: the system over-trusts itself on architectural judgement calls. Quality drifts; the customer has no way to know when to push back.

The project owner the system is designed for is non-technical (or technical-but-not-on-this-stack). They speak in business terms — intent, audience, deadlines, constraints, comparable products. They can't evaluate "should we use tRPC or REST."

## Decision

- **The project owner is at the keyboard.** They drive the conversation in business terms.
- **The assistant drives architecture and code.** Stack playbooks and XP-derived practices give it enough overview guidance to make sensible calls. The project owner approves outcomes, not implementation details.
- **A consultant is an async escape hatch invoked behind the scenes.** When the assistant hits a judgement call beyond its confidence, it pings a consultant async. The project owner doesn't necessarily see this happen — to them, the conversation pauses briefly and resumes with a higher-quality answer.
- **Consultant interaction is async by default.** A consultant covering multiple projects responds within a working day. Async-by-default lets one consultant cover many projects; sync interruption is reserved for cases where the project owner is mid-flow and a stall would break trust.

## Consequences

- The project UX is conversational and project-focused. There is no code editor, no terminal, no PR-review interface for the project owner. Reviewable artefacts (rendered ADRs, the intent document, walking-skeleton previews) are presented as documents and previews, not as diffs.
- A separate consultant surface — operator-facing — is where escalations queue, are answered, and feed back into project conversations.
- The stack playbooks (separate ADR) carry the load that would otherwise fall on consultants. A well-shaped playbook keeps the assistant's decisions in-bounds without escalation.
- The system must be willing to escalate and willing to wait. A consultant escalation that takes a day is fine; a consultant escalation that's never made is the failure mode.
- The "uninitiated human can maintain it" principle is measured against the project owner, not the consultant. If the project owner can read the docs repo and product repo and pick up where the assistant left off, the system is succeeding.
