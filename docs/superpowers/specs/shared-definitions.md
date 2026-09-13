# Shared Definitions (Cross-Spec)

**Status:** Staging document — this does not resolve anything itself. It exists because the three design specs each redefined core concepts locally instead of pointing at one shared source, which is exactly how they ended up contradicting each other and the BRD. Each entry below records what the specs currently say, what the BRD currently says, and which of the pending B-items (from the [design audit](../../design-audit-2026-09-12.md) follow-up conversation) will resolve it.

**Date:** 13 September 2026

**How this file is meant to be used going forward:** once a B-item below is resolved with the user, the decision gets written *here*, the BRD gets edited to match, and the individual specs get simplified to reference this file instead of redefining the concept themselves. Until then, this file is a map of the disagreement, not a fourth opinion added to it.

---

## 1. Phase / Status List

**Specs say:** a project's status is "whatever block it's currently in" — nine phases (Requirements Gathering, Analysis, Development, Testing, Security, UAT, Deployment, Trial, Hold), see [Timeline design §3.1a](2026-09-11-timeline-visual-design.md). No separate status field exists anywhere in the Portfolio List or Project Detail designs.

**BRD says (FR-PRJ-02, RULE-01):** status is one of six fixed values — Analysis, In Progress, On Hold, **Delivered**, Trial, Launched — with an explicit transition table. "In Progress" and "Delivered" don't exist in the specs' vocabulary at all.

**Also conflicting:** Timeline §3.1a places **Deployment before Trial**; RULE-01 places Trial before Launched with no Deployment phase at all — the two documents disagree about the *order* of the project lifecycle, not just its labels.

**Pending as:** design-audit follow-up items B1, B2.

## 2. Roles

**Specs say:** Technical Project Manager, **Junior TPM**, Business Analyst, Tech Lead, Developer — five roles (PM Tool design §4.2/§4.4, resolved via mockup).

**BRD says (FR-RES-02):** four roles. No Junior TPM. Flagged as needing a BRD update in three separate places across the specs; never actioned.

**Pending as:** B5.

## 3. Effort Model

**Specs say (Timeline design §3.1/§4.7):** a block carries one undifferentiated `days` number. `weight` is derived from it. No role dimension exists on a block at all.

**BRD says (FR-EST-01, RULE-02):** effort is man-days **per role** (e.g. 60 dev-days + 15 BA-days + 10 TPM-days); duration is *derived* from effort ÷ assigned capacity — not entered directly.

**Why this one is worse than a simple disagreement:** [PM Tool design §4.6](2026-09-11-pm-tool-design.md) (Change Request Workflow) already assumes the BRD's per-role model when it asks a requester for "effort in man-days per role" — but the timeline sandbox that's supposed to receive that effort has no field to put it in. One spec is already writing checks the other can't cash.

**Pending as:** B3 (also design audit recommendation #3).

## 4. Assignment Allocation %

**Specs say (PM Tool design §4.4):** assigning someone to a project is "pick a person from a role-grouped dropdown." No percentage field.

**BRD says (FR-ASG-01, RULE-07):** every assignment carries an allocation percentage, and the general over-allocation rule (as opposed to the Developer-specific date-overlap rule) is computed from it. Without the field, RULE-07 cannot be evaluated for anyone, in any role.

**Pending as:** B4.

## 5. Department Capture Level

**Specs say (PM Tool design §4.3):** New Project's department field lists **General Departments only** — e.g. "CID." There is nowhere to record that a project is actually owned by e-Crime specifically.

**BRD says (FR-PRJ-04):** capture the *correct organisational level*, and roll it up to the General Department **for reporting** — i.e. the specific level should still be captured; only the *display* rolls up. The specs roll up at the point of capture, which loses information the BRD says must be retained.

**Pending as:** B7.

## 6. New Project Required Fields

**Specs say (PM Tool design §4.3):** Name, Description, standalone/embedded note, Department, Priority, Business PM, Origin, Value Justification. **Explicitly rejects** a project reference code and a target start date as fields, after a ten-candidate review.

**BRD says (FR-PRJ-01):** reference code and target start date are both required at creation.

**Pending as:** B6.

## 7. Developer System Access

**Specs say (PM Tool design §6.1, "Future Ideas," not yet built):** the developer-progress-verification workflow requires Developers to have real accounts and their own screen — acknowledged in that same section as "a genuine expansion beyond every other decision in this document."

**BRD says (§7.1):** "Developers are modelled as a managed resource, not as active system users." Direct contradiction, already flagged once at the point it was written, never followed up.

**Pending as:** B8.

---

## Not yet a contradiction, but will become one if left alone

- **What does "current block" (Portfolio List Status column) actually read from?** Since [Timeline design §3.3](2026-09-11-timeline-visual-design.md) introduced the Event Log, the honest answer is "reconstructed from the Event Log as of today's date" — but no document says this explicitly. Whoever builds the Portfolio List next will otherwise reasonably assume it reads live sandbox state, which is wrong per §3.3's own reasoning (the sandbox is mutable and not the record).
