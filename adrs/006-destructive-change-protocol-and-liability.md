# ADR 006: Destructive-change protocol and liability acceptance

## Status

Accepted

## Context

Deploys carry asymmetric risk. Additive changes — new tables, new columns, new resources, new endpoints — are cheap to roll back: stop using them, eventually clean them up. Destructive changes — dropping a column, deleting an IaC resource, removing a queue, deleting customer data — are not cleanly reversible once they happen. The data is gone; the resource is gone; the rollback target has nothing to roll back *to*.

A platform that generates and deploys code autonomously must be unusually careful here. The customer authorises changes via the deploy approval gate on the strength of having reviewed the preview, but the preview can't show what *destruction* feels like — a deleted column looks fine in the preview until something needs the data that's now gone in production. So the deploy gate needs two things the preview can't provide: a **clear surface** of what is destructive about this change, and **deliberate acceptance** of that risk in a form that's hard to do accidentally.

The complementary discipline is to *minimise* destructiveness at the technical level by default. The expand/contract pattern (also called parallel-change or two-phase migration) reshapes destructive deploys into a sequence of additive deploys: the new thing is added alongside the old, traffic moves over, the old is removed only after confidence is established. This makes the destructive moment small, late, and isolated — and often skippable entirely if "old" turns out to be cheap to leave in place.

## Decision

Every deploy follows the **expand/contract two-phase pattern**:

- **Schema migrations.** New columns/tables are added first. The application is updated to read either source and write to the new. Once the new path is proven, a separate later deploy removes the old column/table. Three deployments to land a column rename is the default, not the exception.
- **Infrastructure-as-code changes.** New resources stand up alongside old. Traffic / references move to the new. Old resources are removed in a separate later deploy, not in the same plan as the addition.
- **Code paths.** Old call sites are switched to new APIs in one deploy; the old API is removed in a later deploy after the call-site change has soaked in production.
- **Data.** Data deletions are explicit destructive operations and never piggyback on a schema change. They are their own deploy with their own acceptance.

**Plan-before-apply is non-negotiable.** For every IaC deploy:

1. The system runs the IaC tool's plan command (e.g. `tofu plan`, `terraform plan`, `pulumi preview`) and captures the structured plan output.
2. A destructive-profile analyser reads the *plan* — not the source diff — to determine what will actually happen. `drop-and-recreate` of a resource is exactly the canonical surprise: the source diff may look like a property change, but the plan reveals destroy-then-create. The analyser must catch this.
3. If the plan contains operations the analyser flags as unexpectedly destructive given the proposed change's stated intent, the system first attempts **autonomous mitigation**: rewriting the resource to use in-place update, adding `lifecycle.prevent_destroy` to force the failure to surface earlier in future, splitting into a two-phase deploy. Each mitigation is itself a code change that goes through the conversation's approval flow before being applied.
4. When autonomous mitigation isn't possible, the system **escalates to a human** with the plan in hand. The human sees what was proposed, what the plan revealed, what mitigations were attempted, and decides whether to proceed, defer, or rewrite.

The pattern makes most rollback cheap: in the additive-only window, rolling back means undoing one additive deploy. After contract has run, rollback is harder or impossible, and the customer needs to know that before the contract deploy goes out.

For each proposed deploy the system computes the **destructive profile** and surfaces it in the liability dialog:

- **Additive only**: new tables/columns/resources, no removals. Low-friction acceptance — confirmation that the user has reviewed the preview and wants to proceed.
- **Reversible-window contract**: removing resources we added in an earlier deploy that haven't been depended on yet. Medium-friction acceptance — a typed confirmation, language describing what becomes hard to undo.
- **Hard destructive**: dropping columns/tables with data, removing resources holding state, deleting data. High-friction acceptance — typed challenge (e.g. type the project name or a specific phrase), explicit listing of what will be destroyed and what data will be lost, and (where policy requires) developer approval in addition to project-owner approval.

The liability dialog is universal — every deploy gets one — but the language and the challenge are scaled to the destructive profile. The acceptance is recorded on the conversation as a turn: who accepted, when, what they accepted, and the exact destructive profile they accepted. This is the durable record for any future dispute.

Rollback is itself subject to the destructive-change protocol:

- Rolling back an additive-only deploy is itself additive (removing the new thing) and presents the additive-only liability dialog.
- Rolling back something where contract has already run requires its own destructive-change acceptance — because the rollback is removing what's currently *in use* in production.
- Where rollback cannot be performed cleanly, the system says so honestly and offers "roll back what we can, accept the residue as recorded" with a high-friction acceptance.

## Consequences

- The default deployment template in the universal guardrails includes expand/contract pipelines for both IaC and migrations. Projects inherit this without having to author it.
- A deploy plan that contains both add and remove operations on the same resource gets rejected by the planner — it must be split into two deploys per the protocol. This is mechanical; the planner refuses to ship a self-contained "rename in one go" plan even when the model proposes it.
- IaC plan output is the input to destructive-profile analysis, not the source diff. The model can write a one-line property change that the IaC engine implements as drop-and-recreate; only the plan reveals it. Every IaC deploy generates a plan, the plan is inspected, and **no apply runs against production without an inspected plan**. This also applies to preview / test-infrastructure changes — destructive plans against the test database/resources get the same analyser even though the liability gate is downstream.
- The liability dialog is a real UX surface, not a checkbox at the bottom of a page. Typed challenges (project name, "yes, deploy", or similar) are the friction we *want* — they make accidental destructive deploys structurally hard, not just policy-discouraged.
- The destructive profile is computed from the proposed change, not from the natural-language description of it. We don't trust the model to self-describe accurately; we run a deterministic analyser against the diff.
- The acceptance turn in the conversation is the audit trail. In any dispute, that turn is what the customer signed off on — verbatim.
