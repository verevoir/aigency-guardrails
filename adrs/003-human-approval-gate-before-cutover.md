# ADR 003: Human-approval gate before cutover

## Status

Accepted

## Context

PR-driven blue/green deploys with smoke tests gating cutover are tempting to wire as fully automatic — smoke passes, traffic shifts, done. That conflates two different checks:

- **Smoke** answers *"is the service alive?"* — does the container start, do the basic endpoints respond, is the health check green. Service-level.
- **Acceptance** answers *"does the feature behave correctly?"* — does the chat reply sensibly, is the UI usable, does the workflow complete end-to-end. Feature-level.

Smoke is automatable; acceptance often isn't (at least not at first). Conflating them means the only check before traffic shift is the cheap one. A revision that boots cleanly but serves a broken feature would cut over and reach users.

The fix is to insert a human approval step between smoke and cutover. The reviewer opens the preview URL (the no-traffic tagged revision), exercises the feature, and approves only when satisfied. New regressions discovered during review become candidates for automated tests added to the smoke / acceptance suite — so the next time the same class of issue arises, it's caught automatically.

## Decision

Cutover is a **separate workflow** triggered by an explicit human approval signal — a `approved` PR label, a deploy-environment approval, or an equivalent gate the version-control platform offers. The preview deploy workflow handles build → push → no-traffic deploy → smoke, and posts a preview URL to the PR. Cutover is not part of that pipeline — it sits behind the human signal.

The reviewer's loop:

1. PR opens or new commit pushed → preview deploy workflow runs.
2. Smoke passes; preview URL posted as a PR comment.
3. Reviewer opens preview, exercises the feature, iterates if needed (each commit re-runs the preview deploy at the same tagged URL).
4. When satisfied, reviewer applies the approval signal (label / approval-required environment / equivalent).
5. Cutover workflow fires on the approval event; traffic shifts to the tagged revision; cutover-complete confirmation posts to the PR.
6. PR is merged.

Acceptance criteria exercised manually during review *should*, over time, be translated into automated checks added to the smoke suite (or a separate acceptance script once smoke grows beyond simple HTTP probes). The bar: if a regression is caught during a review, write a test for it before merging the fix.

## Consequences

- The delivery loop is now: PR → preview deploy → smoke → **preview review** → apply approval → cutover → merge. The preview is a real revision serving 0% traffic at a tagged URL, identical to production in every dimension except who can reach it.
- Iterative review is supported: each commit pushed to the PR redeploys the preview at the same tagged URL. The reviewer can iterate without merging; only the approval triggers cutover.
- The gate is convention-only on plans that don't enforce branch protection. Upgrade to enforced approval-required when team size or risk profile warrants it.
- The approval signal must be configured up-front (label created in the repo, environment defined with required reviewers, etc.) — without it, applying the signal requires permissions the reviewer might not have.
- Smoke remains automated and cheap. Acceptance review remains human-driven by default. The trajectory is towards more automation over time, but never to the point where smoke alone is enough to ship a new feature.
