# ADR 005: PR-driven blue/green deploys with auto-revert

## Status

Accepted

## Context

The walking-skeleton stage produces a hello-world application running through every major component the project will use, with autonomous smoke tests and deployment plumbing. It is the moment the project owner sees something real.

The deployment model has to handle that:

- The project owner is non-technical and reviewing previews, not diffs.
- If a deploy makes prod worse, that should be visible in the project's conversation as an event, not a 3am page.
- The system must deploy autonomously after the explicit approval gate without requiring human merge approval at the platform level — but must also not ship broken changes.

## Decision

- **Deploys come from PRs, not from main.** The walking-skeleton stage and every subsequent feature land as a PR.
- **The PR is deployed blue/green to production.** New deployment runs alongside the previous one with traffic gradually cut over.
- **Smoke tests and instrumentation gate the cutover.** Autonomous smoke tests run against the new deployment in prod; instrumentation watches for regression in real traffic. If either fails, traffic reverts to the previous deployment and the PR is *not* merged.
- **Merge happens only after a successful prod cutover.** Main is always a record of what worked in prod, not what was intended.
- **Reverts surface as system-authored turns in the conversation.** The user opens the conversation and sees: "deploy reverted. The smoke test that failed was X. Suggested next step: Y." Same UX shape as a successful deploy; only the content differs.

## Consequences

- The first walking-skeleton playbook must include working smoke tests, real instrumentation, and a blue/green-capable deploy target. This is non-trivial — it's the reason the walking-skeleton stage exists.
- The project's main branch is meaningfully clean: every commit on main has been verified in prod. Bisecting works; rolling back is rolling forward to a known-good state.
- Continuous deployment is automatic, not aspirational — every PR that gets to the approval gate attempts production. This applies maximum pressure on smoke-test and instrumentation quality, which is correct: those are the load-bearing pieces.
- The project owner doesn't see PRs in the UI. They see a deployed preview. The PR exists for record-keeping and for any future technical maintainer; it is not part of the project owner's experience.
- The first stack playbook (see the stack-playbooks ADR) must support this deploy model. Cloud Run with traffic splitting, Kubernetes with blue/green-capable ingress, or an equivalent is a viable starting point.
