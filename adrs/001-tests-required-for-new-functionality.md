# ADR 001: Tests are required for new functionality

## Status

Accepted

## Context

Early in a project, no automated tests is sometimes defensible — every PR is reviewed in preview by a human, and the surface is small enough to hold in one head. That window closes quickly. As scope grows — multiple services, auth, document handling, integrations — relying on preview review alone for catching regressions becomes unrealistic. Subtle behavioural drift (auth bypassed for a specific path, ownership checks slipping, persistence guarantees silently broken during a refactor) won't surface in a click-through of a preview URL.

The acceptance-becomes-smoke trajectory described in ADR 003 implies a growing automated suite. The discipline this ADR establishes is that the suite is non-optional from the first PR that changes behaviour.

## Decision

**Every PR that adds or changes behaviour ships tests for that behaviour.** The suite is in-process and fast wherever possible — slow tests get used less, eroding the discipline they're meant to support.

Default layers:

- **Unit tests** for pure helpers and prompt loading.
- **Integration tests** at the API / route boundary, invoked directly with mocks for external services (LLM, third-party APIs). No HTTP server, no real network.
- **Component tests** render UI in a JS DOM environment with `fetch` and identity context stubbed. Assertions on semantic markup, not visual snapshots.
- **End-to-end tests** are deferred until the lighter layers stop catching real regressions — they're slower and more expensive to maintain. Add them when the pain arrives, not before.

Tests run in the deploy workflow as part of the `test` job, before deploy and smoke. A failing test fails the PR; cutover and merge are blocked.

**Acceptance findings during preview review become regression tests before the fix is merged.** If a reviewer spots something on the preview that smoke didn't catch, the fix PR includes a test that would have caught the same class of issue. The automated suite grows asymmetrically — driven by real failures, not anticipation — and human review keeps catching the long tail.

## Consequences

- New PRs are larger by a small fraction (the test code). The trade is a regression-resistant codebase as scope grows.
- The test stack is hermetic: each test runs against an in-memory store; auth and external services are mocked; no network calls. Tests are runnable offline.
- The project's `test` task runs lint + format + the test suite. A watch mode supports the development loop.
- Existing untested code is retrofitted when the test infrastructure first lands — the surface is typically small enough at that point for a one-time effort to cover the helpers, API routes, and key components.
- Tests live in `tests/` mirroring the source tree. The specific runner / assertion library / DOM environment is a project-level choice (per the stack-playbook ADR) — what's universal is the discipline of shipping tests with every behavioural change.
