# Where This Picks Up

**Date:** 13 September 2026
**Last commit at time of writing:** `5b1b892` — BRD v0.2, delivery dependency delay

This file exists so a new session (or you, coming back later) can resume without re-deriving state from the chat history. Update it as things get resolved — delete resolved items, don't just leave them checked.

---

## 1. Immediately in progress — org structure is half-given

You're in the middle of dictating Dubai Police's organisational structure across two axes. Captured so far, in [BRD §3.1](BRD-portfolio-resource-management.md):

**Requesting side** (who a project is built *for* — feeds the New Project page's Owning Department field):
- CID → e-Crime, Anti-Fraud, Anti-Money Laundering, Criminal Analysis *(explicitly incomplete — more CID sub-departments coming)*
- Police Stations, Anti-Narcotics, Forensics, Prisons — **general departments named, no sub-departments captured yet**

**Delivery side** (who inside GD of AI actually does the work — feeds the Resource Pool and block-owner fields):
- GD of AI → Internal Applications (the resource pool: PMs/BAs/Tech Leads/Devs — this is *you*)
- GD of AI → Smart Operations → Database (owns Deployment), Infrastructure, Networking, Data Centre
- Information Security — independent, under no general department, owns Security testing

**Next action, once you finish giving the list:** rebuild the New Project page's "Owning department" field in [prms-visual-design.html](prms-visual-design.html) as a two-level picker (General Department → sub-department), which also resolves BRD open item B7 (department capture level) for real instead of leaving it flattened.

---

## 2. Proposed but not yet built into the mockup: External Commitment tracking

This is the "always late" delivery-partner problem (deployment/security testing run by units outside your control). Fully designed and written into [BRD v0.2 §4.4, §10.8, RULE-15/16](BRD-portfolio-resource-management.md) — this part is **done**. What's still pending is reflecting it in the actual `prms-visual-design.html` mockup:

- Project Workspace: new "External dependencies" card (alongside Team & Resources / Change Requests / Holds)
- Inbox: overdue external commitments as attention badges (reuse the pattern from the Inbox redesign)
- Portfolio: a third figure in "What's driving that" row — "**Nd lost to external dependencies**" next to the existing 41d (change requests) and 62d (reprioritisation)
- Per-unit rollup visible to TPMs/Director only, excluded from the Stakeholder view (per RISK-09)

**Also open:** OQ-13 in the BRD — *does the GD of AI Director actually sponsor this before the data starts getting produced?* This is a real question for you to answer with your Director, not just a design detail — RISK-09 (measuring a sibling department) is the governing risk on the whole feature.

---

## 3. Open decisions from the solution proposal (§12), not yet confirmed by you

From [solution-proposal-2026-09-13.md §12](solution-proposal-2026-09-13.md):

- **Item 4 — reported-but-unconfirmed sub-steps:** count as 0% (shown separately from PM-confirmed %), or 50%, or show three raw numbers? I explained the tradeoff; you haven't picked.
- **Item 6 — cross-project scenario commits are Director-only:** I flagged a naming collision — does "Director" here mean the *internal* GD of AI Director (who now, per BRD v0.2, has a much bigger role — the External Commitment data is being built specifically for them), or does it refer to an HQ-level stakeholder? You said HQ directors should be view-only. Needs an explicit answer since it affects who can actually touch the sandbox.

Everything else in that list (items 1, 2, 3, 5, 7–12, and B1–B8's proposed resolutions in §10) is the proposal's default and hasn't been contradicted — treat as accepted unless you say otherwise, but it's still technically "proposed," not "written back into the BRD," except where BRD v0.2 already did that (B-items touching org structure and delivery dependencies).

---

## 4. Standing item: "go through B together"

You said early on you wanted to walk through B1–B8 together — this never happened as its own session; it got superseded by the audit → proposal → mockup-build work. BRD v0.2 has now actually resolved the org-structure-adjacent parts of B7 for real (not just proposed). The rest (B1–B6, B8) are still sitting as proposals in the solution doc, not amendments in the BRD itself. Worth explicitly deciding: is the solution proposal's §10 good enough as the record of these decisions, or do you want them formally written into the BRD the way B7 (partially) and the delivery-dependency problem just were?

---

## 5. Mechanical note

`.superpowers/` is gitignored — it holds ~30 early mockup iteration files (`timeline-concept-v1`–`v16`, etc.) superseded by `prms-visual-design.html`. Not in git. Flagged once already; you didn't ask for them to be added, so they're staying out unless you say otherwise.
