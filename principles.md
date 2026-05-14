# L1 universal principles

The practices below apply to every project regardless of stack, scale, or domain. They are short by intent — short enough that an assistant or a human can hold all of them in mind at once. They constrain _how_ a project is built; _what_ the project is for is the project's own concern.

## Defer architectural decisions to the last responsible moment

For every slice of work, name the risk you're addressing. Pick the smallest implementation that resolves it. Defer every other architectural decision until the cost of _not_ deciding outweighs the cost of _having_ decided wrong. Cheap, replaceable architecture is a valid placeholder when the real decision can still be made cheaply later.

Acceptable placeholders: in-process queues, polling, JSON files, plain `if` chains, copy-and-paste. Unacceptable: placeholders that leak into the URL, the database schema, the wire protocol, or external contracts — those become expensive to change and the deferral is no longer cheap.

## Low-dependency discipline

Resist "conventional" libraries that solve problems you don't actually have. Reach for the standard library and existing first-party dependencies first. Add a new dependency only when its absence is actively painful, when its surface is small enough to read end-to-end, and when its transitive footprint is sane.

The corollary holds: once a library is in, use it everywhere it helps. A dependency you've paid for (install, learning curve, security review) should be put to its proportional best.

## Tests follow intent, not implementation

Test what the system does for the user or caller, not its internals. No snapshot tests of UI. No "X is visible on the page" tests. Behavioural shape, not structural shape. When a regression is caught in review, write a test for it before the fix ships — the test suite grows asymmetrically from real failures, not anticipation.

## Threat-model every cycle

Every PR-sized change includes a quick adversarial pass: who can interact with this, what happens if they're not who we think, what bypasses the UI, what replays a token. Cheap mitigations close in the same cycle; expensive ones get documented as accepted risks.

Patterns to apply by default:

- Uniform 404 on protected URIs — no existence oracle.
- Minimum response-time floor on 404s — no timing-attack enumeration.
- Asymmetric crypto over shared secrets where the choice exists. JWT with public-key verification, mTLS for service-to-service, signed challenges over passwords.
- Secrets in a dedicated store (never in committed env files), minimum need-to-know access, named rotation cadence.
- Stateful services live in private subnets; public ingress goes through a load balancer with TLS termination; no direct internet access to databases.

## Expand-and-contract for destructive changes

Schema migrations, infrastructure changes, and code paths follow the two-phase pattern: add the new thing alongside the old, prove it, then remove the old in a separate later deploy. Three deploys to land a column rename is the default, not the exception. The plan-before-apply discipline is non-negotiable on infrastructure changes — the destructive-profile analyser reads the actual plan output, not the source diff.

## Plain CSS, not utility frameworks

Per-component `.css` files, classes that describe the component. No utility frameworks, no CSS-in-JS, no styled-components. The argument is for legibility and for the absence of a build-system dependency on a fashionable layer that may not outlive the project.

This principle is more opinionated than the rest — fork the guardrails and remove it if your standard is different (and the rest of L1 still applies).

## Universal disciplines that don't fit a section

- **Async-with-no-observers.** The system must keep working when nobody's watching. Materialisers complete, queues drain, dependents unblock — all without UI presence. When an observer returns, surface what advanced.
- **Documentation is part of the deliverable.** A change isn't done until the docs reflect it. The audience is a future developer (often the same developer six months on) reading the docs cold.
- **One conversation, one focused outcome.** When a discussion drifts to a meaningfully different concern, propose a separate piece of work rather than absorbing scope. The discipline supports decomposition over absorption.
