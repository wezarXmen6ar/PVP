# Business Requirements Document

## Portfolio & Resource Management System (PRMS)

### General Department of Artificial Intelligence — Dubai Police

---

## 1. Document Control

| Field | Value |
|---|---|
| **Document title** | Business Requirements Document — Portfolio & Resource Management System |
| **Working system name** | PRMS (working title — see §19, OQ-01) |
| **Version** | 0.2 — Draft for internal review |
| **Date** | 13 September 2026 |
| **Author** | Technical Project Manager, General Department of Artificial Intelligence |
| **Owning department** | General Department of Artificial Intelligence (GD of AI) |
| **Intended audience** | Director, GD of AI (primary); GD of AI delivery leadership; the team who will later build the system |
| **Status** | Draft — pending internal review |

### 1.1 Revision History

| Version | Date | Author | Summary |
|---|---|---|---|
| 0.1 | 9 Sep 2026 | Technical PM, GD of AI | Initial draft. Scope, problem statement, business and functional requirements, business rules, conceptual data model. |
| 0.2 | 13 Sep 2026 | Technical PM, GD of AI | Added the internal structure of GD of AI and the requesting-side hierarchy captured so far (§3.1). Added §4.4, delivery dependency problems, as a fourth problem driver, and renumbered the cost of inaction to §4.5. Added OBJ-08, BR-15 to BR-17, the `FR-EXT` requirement group (§10.8), RULE-15 and RULE-16, and the `DeliveryPartnerUnit`, `ExternalCommitment` and `CommitmentDateHistory` entities. **Amended RULE-11 to add external commitment overrun as a fourth term** — without it the reconciliation was not satisfiable for the most common cause of slippage. Added §7.3 delivery partners and the GD of AI visibility boundary; added RISK-09 to RISK-11. |

### 1.2 How to Read This Document

Sections 1–8 are written for business readers and can be read standalone. Sections 9–13 contain the detailed requirements, business rules and data model intended for whoever builds the system. Sections 14–20 cover assumptions, risks, metrics and open items.

---

## 2. Executive Summary

The General Department of Artificial Intelligence is the central delivery department for digitalisation and automation across Dubai Police. Other general departments own the business problems; GD of AI supplies the Business Analysts, Developers, Tech Leads and Technical Project Managers who deliver the solutions.

That model puts GD of AI at the intersection of parties who do not see each other's constraints. Business owners in the other general departments request new requirements without understanding that each one consumes finite capacity and moves a delivery date. Police HQ halts running projects to insert higher-priority work, then expects the halted projects to land on their original dates. GD of AI absorbs the gap between these expectations and its actual capacity, and currently has no way to make that gap visible.

A third source of delay sits closer to home. Mandatory segments of every project — deployment, infrastructure and security testing — are performed by units outside the Department of Internal Applications, which holds the delivery staff. That work is agreed and dated in advance, and the agreed date is routinely missed. The Technical Project Manager carries the delivery date but has no authority over these units, and no means of recording the delay they cause, so it is absorbed silently and ultimately read as GD of AI's own slippage (§4.4).

Today the entire portfolio is tracked in Excel and reported through PowerPoint decks rebuilt by hand for every meeting. The result is status information that is inconsistent between meetings, dependent on whoever prepared it, and unable to answer the one question that matters: *what did this decision actually cost?*

**PRMS is proposed as a single source of truth for the GD of AI project portfolio.** It models the department's people as a finite capacity pool, holds each project's effort estimate in man-days per role, and automatically recalculates delivery dates when scope is added or work is halted. Every change to a date is recorded with its cause and its originator.

The system serves two audiences. GD of AI staff use the full application to manage projects, assignments and change requests. Business owners and HQ stakeholders access a read-only visualization page that answers three questions without a deck being built:

1. **What does this new requirement cost?** — the requested change, the man-days it adds, and the revised delivery date.
2. **What does halting this project cost?** — which project was displaced, by what, for how long, and its revised date.
3. **What is the true state of the portfolio right now?** — live, consistent, and identical for everyone who opens it.

Separately, and visible only within GD of AI, the system records every segment handed to a delivery partner unit with the date that unit committed to and the date it actually delivered. This gives the Director something no current process produces: **a portfolio-wide total of the delivery time lost inside the general department itself**, attributed to the specific unit responsible. A Technical PM cannot make Information Security faster; the Director can, and this is the evidence that makes that a conversation rather than a complaint (§4.4, §7.3).

The central value is not project tracking; established tools already do that. The value is **attribution**: converting "GD of AI is behind schedule" into a specific, auditable, and neutral statement of cause — whether that cause sits with a business owner, with HQ, or inside the department's own walls. This document defines the requirements for that system.

The first release is a locally-running prototype focused on the capacity and assignment engine, intended for demonstration to the Director of GD of AI. Production deployment concerns are documented separately in §13.2 and are explicitly out of scope for this release.

---

## 3. Business Context

### 3.1 Organisational Structure

Dubai Police is organised into general departments, each large and independently managed. Examples include the General Department of Criminal Investigation (CID), General Department of Police Stations, General Department of Anti-Narcotics, General Department of Forensic Science and Criminology, and the General Department of Punitive and Correctional Establishments.

Each general department contains sub-departments, which contain sections, which may in turn contain sectors. Requesting bodies for projects can sit at any level of this hierarchy — the Department of e-Crime, for example, is a sub-department within CID.

**Relevance to this system:** a project's business owner must be identifiable at the correct organisational level, and the portfolio must be reportable by parent general department. The system therefore models the departmental hierarchy, but only to the depth needed to identify and roll up ownership (§12).

#### Requesting side — known structure

Recorded as confirmed; this list is incomplete and will be extended.

| General Department | Sub-departments captured so far |
|---|---|
| **CID** (Criminal Investigation) | e-Crime; Anti-Fraud; Anti-Money Laundering; Criminal Analysis *(further sub-departments to be added)* |
| **Police Stations** | *not yet captured* |
| **Anti-Narcotics** | *not yet captured* |
| **Forensic Science and Criminology** | *not yet captured* |
| **Punitive and Correctional Establishments** (Prisons) | *not yet captured* |

#### Delivery side — the internal structure of GD of AI

This structure is material to the system, not merely descriptive: it defines the boundary between the staff whose capacity GD of AI manages and the units it depends on but does not control (§4.4).

| Unit | Position | Role in delivery |
|---|---|---|
| **Department of Internal Applications** | Sub-department of GD of AI | Holds the Technical PMs, Business Analysts, Tech Leads, Developers and designers. **This is the resource pool modelled in §12, and the department that operates PRMS.** |
| **Department of Smart Operations** | Sub-department of GD of AI | Parent of the sections below. Participates in **every** project, performing work agreed and cleared in advance, but its staff are **not** part of the resource pool and its capacity is not managed by Internal Applications. |
| — Database section | Section of Smart Operations | **Deployment** of every project |
| — Infrastructure section | Section of Smart Operations | Hardware provisioning, rack alignment and related infrastructure work |
| — Networking section | Section of Smart Operations | Network provisioning and configuration |
| — Data Centre section | Section of Smart Operations | Data centre facilities |
| **Department of Information Security** | **Independent** — sits under no general department | **Security testing** of every project |

**The critical distinction.** Internal Applications staff are *resources*: finite, allocatable, and accountable to the Technical PM. Smart Operations sections and Information Security are *delivery partners*: they hold mandatory segments of every project's critical path, they are represented on the project team, but they are **outside the Technical PM's authority and outside the capacity model entirely.** PRMS must never model their capacity or purport to schedule them. What it must do is record what was asked of them, what they committed to, and what actually happened — see §4.4, §10.8 and RULE-15.

### 3.2 The Role of GD of AI

The General Department of Artificial Intelligence is a central delivery function. It does not own the business processes it automates; it owns the capability to automate them. Its role is to develop and manage the projects that digitalise and automate core police business operations on behalf of the other general departments.

This creates a structural asymmetry that underlies every problem in §4:

- **Demand is distributed.** Any general department, and Police HQ, can generate demand on GD of AI.
- **Supply is centralised and finite.** Developers, BAs, Tech Leads and Technical PMs sit in one pool, held by the **Department of Internal Applications** within GD of AI (§3.1).
- **No requesting party sees the whole demand picture.** Each business owner sees only their own project. HQ sees priorities, not capacity.
- **Delivery is not wholly within the delivering department's control.** Mandatory segments of every project — deployment, infrastructure, security testing — are executed by units outside Internal Applications (§3.1). The Technical PM carries the delivery date but does not command every step on the path to it (§4.4).

### 3.3 How Projects Originate

Projects reach GD of AI through two routes:

1. **Direction from Police HQ.** Senior leadership orders a general department to initiate a project. These arrive with implicit high priority and, in some cases, with little notice.
2. **Request from a general department.** A department identifies a need and requests support. These are accepted only after a study establishing the genuine need and expected value.

### 3.4 Current Project Initiation Flow

The established process, using the e-Crime digital complaint channels project as a representative example:

| Step | Actor | Activity |
|---|---|---|
| 1 | Police HQ / General Department | Project is ordered or requested; need and value are established |
| 2 | Owning department (e.g. Dept of e-Crime) | Assigns a business-side Project Manager |
| 3 | Owning department | Presents the project to GD of AI as the central development department |
| 4 | GD of AI | Assigns a Technical Project Manager |
| 5 | GD of AI | Assembles the delivery team (BA, Developers, Tech Lead as required) |
| 6 | GD of AI | Conducts business analysis and produces the delivery plan and timeline |
| 7 | GD of AI | Presents the timeline to the business owner, then to HQ |
| 8 | Business owner + HQ | Approve |
| 9 | GD of AI | Execution begins against the approved plan |

**Step 8 establishes the baseline.** The approved timeline and scope at this point are the reference against which all later variance must be measured and explained. The absence of a durable, system-held baseline is the root cause of the reporting problems described in §4.

---

## 4. Problem Statement

### 4.1 Business-Side Problems (Owning General Departments)

Business owners are experienced police professionals, but they are not project managers. This produces a consistent and predictable set of problems:

| Problem | Consequence for GD of AI |
|---|---|
| **Incomplete requirements at the outset** | Estimates and timelines are built on partial information and are invalidated as the real scope emerges |
| **Scope creep** | New requirements arrive continuously during delivery and are absorbed without formal recognition |
| **No concept of a change request** | A new requirement is experienced by the requester as a clarification, not as a change with a cost |
| **No understanding of requirement weight** | All requests are perceived as similar in size regardless of actual effort |
| **Expectation of unlimited capacity** | GD of AI is perceived as able to deliver anything, at any time, on the original date |
| **Friction when timelines move** | Being told that new scope moves the date is received as an excuse rather than as arithmetic |

The common root is that **the cost of a request is invisible at the moment the request is made.** Nothing in the current process converts "please also add X" into "that is 12 developer-days and moves your go-live from 4 March to 22 March." Because the cost is invisible, it is not weighed, and the requester cannot make an informed decision about whether the change is worth its price.

### 4.2 Police HQ Stakeholder Problems

| Problem | Consequence |
|---|---|
| **Status reporting is rebuilt manually for every meeting** | Presentations consume significant preparation time on each request |
| **Reported information is author-dependent** | The same project can be represented differently by different preparers, undermining trust in all reporting |
| **Reporting is a point-in-time snapshot** | Information is already ageing when presented and cannot be interrogated |
| **Running projects are halted for surprise high-priority work** | The displaced project stops, but the expectation of its original delivery date does not |
| **The consequence of a halt is not visible to the decision-maker** | Prioritisation decisions are made without sight of what they cost elsewhere in the portfolio |

The critical point is the last one. Reprioritisation is a legitimate command decision and the system must not obstruct it. But at present the decision is made **without the trade-off being visible**, and the resulting delay to the displaced project is later attributed to GD of AI rather than to the reprioritisation that caused it.

### 4.3 Internal Capacity Problems (GD of AI)

| Problem | Consequence |
|---|---|
| **No consolidated view of who is working on what** | Assignment decisions rely on individual recollection |
| **No visibility of workload distribution** | Load is unevenly spread; some staff are over-committed while others are underused |
| **No forward view of when resources become free** | Cannot answer "when could we realistically start this?" |
| **No portfolio-level effect analysis** | Cannot show what adding a project does to everything already committed |
| **No record of why a date changed** | Variance cannot be explained after the fact, so it defaults to being GD of AI's fault |

### 4.4 Delivery Dependency Problems (Units Outside Internal Applications)

Every project contains mandatory segments executed by units that the Technical PM does not manage: deployment by the Database section, infrastructure and rack work by the Infrastructure section, and security testing by the independent Department of Information Security (§3.1). This work is agreed and cleared in advance, and a completion date is committed at handover — **the committed date is routinely missed.**

| Problem | Consequence |
|---|---|
| **Work handed to a delivery partner is frequently returned late** | Slippage enters the project from a source the Technical PM cannot influence |
| **Response to a handover is itself often delayed** | Work sits unstarted after submission; the delay accrues before any work begins, and is invisible while it does |
| **The delay is not attributable to a named unit** | The existing hold taxonomy (RULE-12) records only the generic reason *"Blocked by external or technical dependency"* — it cannot say which unit, so nothing can be aggregated |
| **Recording it requires a formal Hold** | A hold is too heavyweight for a routine six-day overrun, so in practice the slip is never recorded at all |
| **No record exists of what was committed** | Because no promised date is held, a late return cannot be distinguished from work that simply took the time it takes |
| **The resulting variance is unexplained** | Under RULE-11 the date moves with no recorded cause, so it is flagged as unexplained residual — the system detects that something moved the date but cannot name it |
| **The delay is ultimately attributed to GD of AI** | Externally, the project is simply late; the segment that caused it is invisible to everyone outside the delivery team |

**This is the internal counterpart of §4.1 and §4.2, and structurally identical to both.** In each case a cost is incurred by one party and paid by another while remaining invisible: business owners impose it through scope, HQ imposes it through reprioritisation, and delivery partners impose it through late returns. The system's answer in the first two cases — record the cause, quantify it in working days, attribute it to its origin — is the same answer required here.

It differs in one respect that shapes the requirement. Business owners and HQ are outside GD of AI and consume a read-only view. Delivery partners sit **inside the same general department**, under the same Director. The Technical PM has no authority to make Information Security faster; the Director of GD of AI does. Aggregated per-unit delay data is therefore directed at an audience that can act on it, and its visibility is deliberately bounded to GD of AI (§7.3, FR-EXT-11).

A second consequence follows from the delay being routine rather than exceptional. Where a unit's returns are *consistently* late by a recognisable margin, that margin is a **planning input, not merely a grievance**: once sufficient history exists, a project can be baselined against what a segment historically takes rather than against what was promised, and a predictable slip stops being absorbed as though it were a surprise (FR-EXT-12).

### 4.5 Cost of Taking No Action

- Preparation effort for status reporting continues to be consumed on every request, indefinitely.
- Delivery dates continue to slip for reasons that are real but undocumented, progressively eroding the credibility of GD of AI's estimates.
- Prioritisation and scope decisions continue to be made without visibility of their cost, producing outcomes that no party would have chosen with full information.
- Staff over-allocation remains invisible until it appears as missed dates or attrition.
- Delay originating in delivery partner units continues to accumulate unrecorded and unattributed, and continues to be absorbed by GD of AI as though it were its own — with no evidence available to the one person, the Director, who could address it.
- Every new project increases the coordination load on Technical PMs superlinearly, because there is no system holding the state.

---

## 5. Current State (As-Is)

| Aspect | Current state |
|---|---|
| **Project tracking** | Microsoft Excel workbooks, maintained manually |
| **Resource allocation** | Held informally; no consolidated record of assignments |
| **Effort estimation** | Produced per project during analysis; not retained in a comparable structure |
| **Timeline management** | Dates maintained manually; revisions overwrite prior values |
| **Baseline retention** | None — the originally approved timeline is not durably retained for comparison |
| **Change requests** | No formal capture, costing, or approval record |
| **Hold / halt records** | Not formally recorded; cause of delay is retained only in individuals' memory |
| **Delivery partner handovers** | Committed dates are agreed verbally or by email and are not retained; no record exists of what was promised, when work was handed over, or how late it was returned |
| **Status reporting** | Microsoft PowerPoint, rebuilt by hand for each meeting |
| **Stakeholder access** | None — stakeholders receive prepared presentations only |
| **Audit history** | None |

**Assessment.** The current tooling can record what is planned but cannot explain what changed or why. Excel holds a single mutable view of the present, so each revision destroys the evidence needed to account for it. This is the specific deficiency PRMS must address; simply moving the same data into a better tracker would not resolve it.

---

## 6. Objectives and Success Criteria

| # | Objective | Success criterion | Addresses |
|---|---|---|---|
| **OBJ-01** | Establish a single source of truth for the GD of AI portfolio | All active projects are recorded in PRMS; status presentations are generated from it rather than assembled by hand | §4.2 |
| **OBJ-02** | Make the cost of a new requirement visible at the moment it is requested | Every change request records added effort in man-days and the resulting revised delivery date before acceptance | §4.1 |
| **OBJ-03** | Make the cost of reprioritisation visible to the decision-maker | Every hold records its reason, its duration, the displacing project where applicable, and the revised delivery date of the displaced project | §4.2 |
| **OBJ-04** | Provide an accurate view of departmental capacity and workload | Utilisation is visible per person and per role; over-allocation is flagged as it occurs | §4.3 |
| **OBJ-05** | Make every timeline change attributable | 100% of variance between baseline and current dates is accounted for by a recorded change request, hold record, override or external commitment slip | §4.1, §4.2, §4.3, §4.4 |
| **OBJ-06** | Eliminate manual preparation of routine status reporting | Stakeholders self-serve current status from the visualization page; no deck is built for routine status meetings | §4.2 |
| **OBJ-07** | Support informed decisions on new demand before commitment | The earliest realistic start date for a proposed project can be determined from current capacity | §4.3 |
| **OBJ-08** | Make delay originating outside Internal Applications visible, attributable to the responsible unit, and quantified | Every segment of work held by a delivery partner records a committed date and an actual date; the working days lost are attributed to the named unit and aggregated across the portfolio for the Director | §4.4 |

---

## 7. Stakeholders and User Roles

### 7.1 System Users (GD of AI)

| Role | System access | Primary use |
|---|---|---|
| **Director / Department management, GD of AI** | Full read; portfolio-level | Portfolio oversight, capacity planning, resourcing decisions, escalation |
| **Technical Project Manager (TPM)** | Full read/write on own projects | Maintains project data, estimates, assignments, change requests and holds. Primary data owner. |
| **Business Analyst (BA)** | Read; write on assigned projects | Requirements and analysis effort; raises change requests arising from analysis |
| **Tech Lead** | Read; write on assigned projects | Technical estimation input; assigned to projects where required |
| **Developer** | Read own assignments | Visibility of own allocation and workload |
| **Resource Manager / Dept coordinator** | Full read/write on resource pool | Maintains the staff roster, roles, availability and leave |

> **Note on the Developer role.** Developers are modelled as a managed resource, not as active system users. Read access to their own assignments is included for transparency but is not a core requirement of the first release.

### 7.2 Read-Only Consumers (External to GD of AI)

| Stakeholder | Access | Interest |
|---|---|---|
| **Police HQ senior leadership** | Read-only visualization page | Portfolio status; consequence of prioritisation decisions; delivery confidence |
| **Business owner — General Department management** | Read-only visualization page | Status of their own projects; effect of their own change requests on their dates |
| **Business-side Project Manager (owning department)** | Read-only visualization page | Day-to-day status of their project; pending change requests and impacts |

These parties **do not hold accounts and do not enter data.** They consume a presentation-oriented view. This is a deliberate scoping decision: it removes access administration, permission modelling and data-integrity risk from the first release, while still delivering the transparency that resolves §4.1 and §4.2.

### 7.3 Delivery Partners (Recorded, Not Users)

A third category, distinct from both of the above: units that perform mandatory project work but neither operate the system nor consume its output.

| Unit | System access | Represented in PRMS as |
|---|---|---|
| **Database section** (Smart Operations) | **None** | Owner of externally-held deployment work; subject of external commitment records |
| **Infrastructure section** (Smart Operations) | **None** | Owner of externally-held infrastructure work |
| **Networking section** (Smart Operations) | **None** | Owner of externally-held network work |
| **Data Centre section** (Smart Operations) | **None** | Owner of externally-held data centre work |
| **Department of Information Security** (independent) | **None** | Owner of externally-held security testing |

**These units hold no accounts, enter no data, and are not modelled as resources.** Their capacity, staffing and utilisation are outside the scope of this system entirely (§8.2). PRMS records only the interface with them: what was requested, what date was committed, and what date was actually delivered (§10.8).

**Visibility boundary.** Per-unit aggregate delay data (FR-EXT-10, FR-EXT-11) is available to Technical PMs, Junior TPMs and the Director of GD of AI only. It is **not** exposed on the stakeholder visualization page, not shown to Police HQ, and not shown to business owners. The rationale is in §4.4: the data exists to enable a conversation within GD of AI, by the one person with authority over these units — not to relocate blame onto a sibling department in front of an external audience. See RISK-09.

### 7.4 Stakeholder Interest Summary

| Group | What they currently lack | What PRMS gives them |
|---|---|---|
| HQ leadership | Consistent, current, interrogable portfolio status | A live view identical for every viewer, with causes of variance attached |
| Business owners | Understanding of what their requests cost | The man-day cost and revised date of each request, before they commit to it |
| GD of AI management | Visibility of capacity and its limits | Utilisation, bottleneck roles, and realistic start dates for new demand |
| Technical PMs | A system that holds project state and history | Automated recalculation and a defensible, auditable record of every change |
| GD of AI Director | Evidence of where delivery time is actually lost inside the general department | Per-unit totals for delay originating in Smart Operations sections and Information Security, aggregated across the portfolio |

---

## 8. Scope

### 8.1 In Scope — First Release (MVP)

The first release centres on the **capacity and assignment engine**, because every other capability depends on the data it holds.

1. **Project registry and lifecycle management** — record projects, their business owner, priority, dates and status through the defined lifecycle
2. **Resource pool** — the GD of AI staff roster with roles and availability
3. **Effort estimation** — man-days per role, per project, with a retained approved baseline
4. **Assignment engine** — allocate resources to projects; calculate workload; raise over-allocation warnings
5. **Schedule calculation** — derive duration and delivery dates from effort, assigned resources and a working calendar
6. **Change request management** — capture new requirements, cost them in man-days, calculate and record the resulting date impact
7. **Hold management** — record holds with reason codes, link displaced projects to displacing projects, and recalculate downstream dates automatically
8. **External commitment tracking** — record each segment held by a delivery partner unit with its committed and actual dates, attribute the resulting delay to the named unit, and aggregate it per unit across the portfolio
9. **Stakeholder visualization page** — read-only view covering portfolio status, change request impact, and hold impact
10. **Audit history** — a durable record of every status, date, scope and assignment change with cause and timestamp

### 8.2 Out of Scope — First Release

| Excluded | Rationale |
|---|---|
| Task-level or sprint-level work tracking | PRMS operates at portfolio and project level. It is not a replacement for a development tracker. |
| Time sheets / actual hours capture | Effort is planned and estimated, not actually recorded, in this release. Estimate-versus-actual is a later phase (§20). |
| Per-phase resource allocation | Resources are assigned for the whole project duration (RULE-09). Per-phase allocation is a later refinement. |
| Financial and budget management | Cost tracking is not part of the current problem statement. |
| Document management | Requirements documents, designs and deliverables remain in existing repositories. |
| Integration with any external system | Local prototype; no integration surface in this release. |
| User accounts for business owners and HQ | Read-only visualization page only, per §7.2. |
| Authentication, SSO and role-based access control | Local single-user prototype (§13.1). Required for production (§13.2). |
| Arabic language and RTL interface | Required for production (§13.2); not required for the internal prototype. |
| Vendor and contractor resource management | The pool is Internal Applications staff only in this release. |
| Capacity, staffing or scheduling of delivery partner units | Smart Operations sections and Information Security are outside the Technical PM's authority and are modelled as dependencies, not resources (§7.3, FR-EXT-13). PRMS records the interface with them, never their internal workings. |
| Accounts for delivery partner units | These units do not operate the system and do not enter data (§7.3). Allowing a unit to record its own commitment dates is a candidate for a later phase, not this release. |
| Automated notifications and alerting | Warnings are surfaced in the interface, not pushed. |

### 8.3 Deferred to Later Phases

Recorded here so they are not re-litigated as scope gaps during the first release: estimate-versus-actual tracking to improve estimation accuracy over time; per-phase resource modelling; scenario planning with multiple saved what-if scenarios; skill-based resource matching; vendor capacity; dependency modelling between projects; and integration with existing Dubai Police systems. See §20.

---

## 9. Business Requirements

Business requirements state *what the business needs*, in business language. Each is traceable to an objective and is decomposed into functional requirements in §10.

| ID | Requirement | Objective |
|---|---|---|
| **BR-01** | The department must maintain a single, current, authoritative record of every project in its portfolio. | OBJ-01 |
| **BR-02** | The department must maintain a record of its delivery staff, their roles, and their availability, as a finite capacity pool. | OBJ-04 |
| **BR-03** | Each project must carry an estimate of the effort it requires, expressed in man-days for each role it consumes. | OBJ-02, OBJ-04 |
| **BR-04** | Each project must retain the scope and delivery dates approved at the point of business and HQ sign-off, unchanged, as its baseline. | OBJ-05 |
| **BR-05** | The system must calculate a project's delivery date from its effort estimate, its assigned resources and the working calendar, and must allow a Technical PM to override that calculation with justification. | OBJ-02, OBJ-07 |
| **BR-06** | The department must be able to see the current workload of every member of staff and identify who is over-allocated. | OBJ-04 |
| **BR-07** | The department must be alerted when an assignment breaches an allocation rule, but must retain the ability to proceed. | OBJ-04 |
| **BR-08** | Every new requirement raised after baseline approval must be recorded as a change request, costed in man-days, and its effect on the delivery date calculated and presented before it is accepted. | OBJ-02 |
| **BR-09** | Every suspension of work on a project must be recorded with a reason, a duration, and — where the cause is reprioritisation — a link to the project that displaced it. | OBJ-03 |
| **BR-10** | When a project is held, its delivery date must be recalculated automatically to reflect the suspension. | OBJ-03, OBJ-05 |
| **BR-11** | The difference between a project's baseline dates and its current dates must at all times be fully explained by its recorded change requests, holds, overrides and external commitment slips. | OBJ-05 |
| **BR-12** | Business owners and HQ stakeholders must be able to view current portfolio status, and the cause and effect of every timeline change, without a report being prepared for them. | OBJ-01, OBJ-06 |
| **BR-13** | The department must be able to determine the earliest date at which a proposed new project could realistically start, given current commitments. | OBJ-07 |
| **BR-14** | Every material change to a project must be recorded in an audit history showing what changed, when, and why. | OBJ-05 |
| **BR-15** | Each segment of project work performed by a unit outside the Department of Internal Applications must be recorded with the responsible unit named, the date it was handed over, the date that unit committed to, and the date it was actually delivered. | OBJ-08 |
| **BR-16** | Working days lost to a delivery partner returning work later than committed must be attributed to that named unit, counted towards the project's variance, and reportable as a total per unit across the portfolio. | OBJ-05, OBJ-08 |
| **BR-17** | The department must be able to see, per delivery partner unit, how its actual delivery times have historically compared with the dates it committed to, so that future plans can be based on observed performance rather than on the committed date alone. | OBJ-08 |

---

## 10. Functional Requirements

Priority is stated as **M** (must have — first release), **S** (should have — first release if capacity allows), or **C** (could have — first release candidate, otherwise deferred).

### 10.1 Project Registry and Lifecycle — `FR-PRJ`

| ID | Requirement | Pri |
|---|---|---|
| FR-PRJ-01 | The system shall allow a project to be created with: name, reference code, description, owning department, business-side PM name, origin (HQ directive or departmental request), priority, and target start date. | M |
| FR-PRJ-02 | The system shall assign every project exactly one status from: **Analysis, In Progress, On Hold, Delivered, Trial, Launched** (see RULE-01). | M |
| FR-PRJ-03 | The system shall record a project's priority as one of: Critical, High, Medium, Low. | M |
| FR-PRJ-04 | The system shall identify the owning department at its correct organisational level and roll it up to its parent general department for reporting. | M |
| FR-PRJ-05 | The system shall record, for every project, its baseline start date, baseline delivery date, current planned start date and current planned delivery date. | M |
| FR-PRJ-06 | The system shall record every status transition with the date, the user, and — where applicable — the reason. | M |
| FR-PRJ-07 | The system shall calculate and display, for every project, the variance in working days between baseline and current delivery date. | M |
| FR-PRJ-08 | The system shall permit a project to be cancelled, retaining its record and history, and release its assigned resources. | S |
| FR-PRJ-09 | The system shall flag a project whose data has not been updated within a configurable period (default 14 days) as stale. | S |
| FR-PRJ-10 | The system shall record a free-text current-status narrative per project, for stakeholder reporting. | S |

### 10.2 Resource Pool — `FR-RES`

| ID | Requirement | Pri |
|---|---|---|
| FR-RES-01 | The system shall maintain a roster of GD of AI delivery staff with: name, employee reference, primary role, and active/inactive state. | M |
| FR-RES-02 | The system shall support the roles: **Technical Project Manager, Business Analyst, Tech Lead, Developer**. | M |
| FR-RES-03 | The system shall record an availability factor per person (default 80%), representing the proportion of working time available for project delivery after meetings, support and administration. | M |
| FR-RES-04 | The system shall support recording planned absence (leave, training, secondment) as date ranges that reduce available capacity in that period. | S |
| FR-RES-05 | The system shall support a person holding a secondary role, allowing them to be assigned in either capacity. | C |
| FR-RES-06 | The system shall prevent deletion of a person who holds historical assignments, permitting deactivation instead. | M |

### 10.3 Effort Estimation — `FR-EST`

| ID | Requirement | Pri |
|---|---|---|
| FR-EST-01 | The system shall record, per project, an estimated effort in man-days for each role the project consumes. | M |
| FR-EST-02 | The system shall calculate a project's total weight as the sum of estimated man-days across all roles. | M |
| FR-EST-03 | The system shall retain the effort estimate approved at baseline, separately from and unaffected by later revisions. | M |
| FR-EST-04 | The system shall maintain a current effort estimate equal to the baseline estimate plus the effort of all approved change requests. | M |
| FR-EST-05 | The system shall record an optional confidence level per estimate (High, Medium, Low) to indicate estimation reliability. | S |
| FR-EST-06 | The system shall retain a revision history of effort estimates showing what changed, when, and why. | M |

### 10.4 Assignment Engine and Allocation Rules — `FR-ASG`

| ID | Requirement | Pri |
|---|---|---|
| FR-ASG-01 | The system shall allow one or more people to be assigned to a project, each with a role and an allocation percentage (default 100%). | M |
| FR-ASG-02 | The system shall assign a resource for the whole duration of the project, from its planned start date to its planned delivery date (RULE-09). | M |
| FR-ASG-03 | The system shall allow multiple resources of the same role to be assigned to one project. | M |
| FR-ASG-04 | The system shall raise a **warning, not a block**, when a Developer is assigned to more than one project in In Progress status over overlapping dates (RULE-06). | M |
| FR-ASG-05 | The system shall permit a Technical PM to proceed past any allocation warning, recording an acknowledgement with the user and timestamp. | M |
| FR-ASG-06 | The system shall permit Technical Project Managers, Business Analysts and Tech Leads to be assigned to multiple concurrent projects without warning, subject to FR-ASG-07. | M |
| FR-ASG-07 | The system shall raise a warning when any person's total allocation across concurrent projects exceeds 100% of their available capacity (RULE-07). | M |
| FR-ASG-08 | The system shall calculate and display each person's current utilisation as committed allocation against available capacity. | M |
| FR-ASG-09 | The system shall display a capacity view showing, per person and per role, current utilisation and the date on which each becomes available. | M |
| FR-ASG-10 | The system shall release a project's assignments when the project reaches Launched or is cancelled. | M |
| FR-ASG-11 | The system shall release or retain assignments when a project enters On Hold, according to the resource disposition of the hold reason (RULE-12). | M |
| FR-ASG-12 | The system shall identify, for a proposed project with a stated effort estimate, the earliest date at which sufficient capacity exists to start it. | S |
| FR-ASG-13 | The system shall retain a history of all assignments, including ended ones. | M |

### 10.5 Schedule Calculation — `FR-SCH`

| ID | Requirement | Pri |
|---|---|---|
| FR-SCH-01 | The system shall calculate a project's duration in working days from its current effort estimate and its assigned resources (RULE-02). | M |
| FR-SCH-02 | The system shall calculate a project's planned delivery date as its planned start date advanced by its calculated duration across working days only (RULE-03). | M |
| FR-SCH-03 | The system shall maintain a configurable working calendar defining the working week and non-working public holidays (RULE-04). | M |
| FR-SCH-04 | The system shall allow a Technical PM to override a calculated duration or delivery date, requiring a justification, and shall record the override and its reason (RULE-05). | M |
| FR-SCH-05 | The system shall visually distinguish a calculated date from a manually overridden one. | S |
| FR-SCH-06 | The system shall recalculate a project's delivery date whenever its effort estimate, assigned resources, or hold state changes. | M |

### 10.6 Change Request Management — `FR-CHG`

| ID | Requirement | Pri |
|---|---|---|
| FR-CHG-01 | The system shall allow a change request to be raised against a project with: title, description, requester name, requesting department, and date raised. | M |
| FR-CHG-02 | The system shall require an effort estimate in man-days per role for every change request before it can be assessed. | M |
| FR-CHG-03 | The system shall calculate, for a change request under assessment, the resulting revised delivery date and the delta in working days from the current delivery date. | M |
| FR-CHG-04 | The system shall present the calculated impact — added man-days, delta in working days, current date and revised date — as the primary output of the change request record. | M |
| FR-CHG-05 | The system shall support change request states: **Raised, Under Assessment, Presented to Business, Approved, Rejected, Withdrawn**. | M |
| FR-CHG-06 | The system shall apply a change request's effort to the project's current estimate and commit the revised delivery date only when the change request is Approved. | M |
| FR-CHG-07 | The system shall retain rejected and withdrawn change requests, with their assessed impact, as a record of scope that was considered and declined. | M |
| FR-CHG-08 | The system shall record the date on which a change request's impact was presented to the business owner, and the date of their decision. | M |
| FR-CHG-09 | The system shall display, per project, the cumulative effect of all approved change requests on the delivery date. | M |
| FR-CHG-10 | The system shall allow a change request's impact to be viewed without committing it, so it can be presented before a decision is taken. | M |

### 10.7 Hold Management — `FR-HLD`

| ID | Requirement | Pri |
|---|---|---|
| FR-HLD-01 | The system shall allow a project to be placed On Hold with a mandatory reason selected from: **Resources pulled to higher-priority project; Awaiting business owner input; Awaiting approval / budget; Blocked by external or technical dependency**. | M |
| FR-HLD-02 | The system shall require, where the reason is *Resources pulled to higher-priority project*, a link to the project that caused the displacement. | M |
| FR-HLD-03 | The system shall record, for every hold: start date, expected resume date where known, actual resume date, reason, linked displacing project where applicable, the person who recorded it, and the authority who directed it. | M |
| FR-HLD-04 | The system shall record a free-text note against every hold. | M |
| FR-HLD-05 | The system shall recalculate a held project's delivery date on resume, advancing it by the number of working days the project was held (RULE-10). | M |
| FR-HLD-06 | The system shall project a provisional revised delivery date during an open hold, based on the expected resume date where one is recorded. | M |
| FR-HLD-07 | The system shall release the project's assignments to the available pool where the hold reason has a resource disposition of *released*, and retain them where it is *retained* (RULE-12). | M |
| FR-HLD-08 | The system shall support multiple sequential holds on one project, each retained individually in history. | M |
| FR-HLD-09 | The system shall display, for a project that displaced others, the list of projects it displaced and the total delay it caused. | M |
| FR-HLD-10 | The system shall return a project to its prior status on resume. | M |

### 10.8 External Commitments — `FR-EXT`

Covers work forming part of a project but performed by a unit outside the Department of Internal Applications (§3.1, §7.3). An **external commitment** is the record of one such handover.

The design constraint throughout this group: PRMS records the *interface* with these units — request, commitment, delivery — and never their capacity, staffing or internal scheduling.

| ID | Requirement | Pri |
|---|---|---|
| FR-EXT-01 | The system shall allow a project phase or block to be designated as **externally held**, with a responsible unit selected from the registered delivery partner units (Database, Infrastructure, Networking, Data Centre, Information Security). | M |
| FR-EXT-02 | The system shall record, for every external commitment: the responsible unit, the date the work was handed over, the date committed to by that unit, the actual delivery date, and the current state. | M |
| FR-EXT-03 | The system shall support the following states for an external commitment: **Not yet handed over; Handed over — not started; In progress; Delivered**. | M |
| FR-EXT-04 | The system shall calculate **response lag** as the working days between handover and the unit starting work, and shall display it as a running count while the commitment remains in *Handed over — not started*. | M |
| FR-EXT-05 | The system shall calculate **overrun** as the working days between the committed date and the actual delivery date, where delivery is later than committed. | M |
| FR-EXT-06 | The system shall raise a visible alert on a project when an external commitment passes its committed date without being delivered. | M |
| FR-EXT-07 | The system shall record the overrun of an external commitment as an impact against the project's delivery date, attributed to the named responsible unit, and shall include it in the variance reconciliation (RULE-11, RULE-15). | M |
| FR-EXT-08 | The system shall allow evidence of the commitment — an email, a document or a reference to a meeting decision — to be attached to an external commitment record, as proof of the date committed to. | M |
| FR-EXT-09 | The system shall permit an external commitment's committed date to be revised, retaining every prior committed date in history, so that repeated re-commitment is itself visible. | M |
| FR-EXT-10 | The system shall report, per delivery partner unit and over a selected period: number of commitments, number delivered late, mean response lag, mean overrun, and total working days of project delay attributed to that unit. | M |
| FR-EXT-11 | The system shall restrict the per-unit aggregate reporting of FR-EXT-10 to Technical PMs, Junior TPMs and the Director of GD of AI, and shall exclude it from the stakeholder visualization page (§7.3). | M |
| FR-EXT-12 | The system shall display, when an external segment is being planned, the historical mean overrun of the responsible unit for comparable segments, as an advisory input to the planned duration. | C |
| FR-EXT-13 | The system shall not model the capacity, staffing, utilisation or internal schedule of any delivery partner unit. | M |

### 10.9 Stakeholder Visualization Page — `FR-VIZ`

The visualization page is read-only, requires no account, and is designed to be shown on screen in a meeting without preparation.

| ID | Requirement | Pri |
|---|---|---|
| FR-VIZ-01 | The system shall provide a read-only view of the portfolio requiring no data entry and no login. | M |
| FR-VIZ-02 | The system shall display a portfolio overview: all projects with owning department, status, priority, current delivery date, and variance from baseline. | M |
| FR-VIZ-03 | The system shall display a timeline (Gantt-style) view of the portfolio showing each project's baseline bar and current bar, so slippage is visible as a comparison. | M |
| FR-VIZ-04 | The system shall display, per project, a variance breakdown attributing the total slippage to its component causes — each approved change request, each hold, and each external commitment overrun, with its contribution in working days. Where the cause is an external commitment, the segment is identified but the responsible unit is **not** named on this page (§7.3, FR-EXT-11). | M |
| FR-VIZ-05 | The system shall display a change request impact view showing the requested change, its man-day cost, the current delivery date and the revised delivery date. | M |
| FR-VIZ-06 | The system shall display a hold impact view showing which project was displaced, by which project, for how long, and the resulting revised delivery date. | M |
| FR-VIZ-07 | The system shall display a capacity summary showing departmental utilisation by role and identifying bottleneck roles. | M |
| FR-VIZ-08 | The system shall allow the portfolio view to be filtered by owning general department, status and priority. | M |
| FR-VIZ-09 | The system shall present a single-project view suitable for a business owner reviewing only their own project. | M |
| FR-VIZ-10 | The system shall display the date and time at which the data shown was last updated. | M |
| FR-VIZ-11 | The system shall present all variance information in neutral, factual language, stating cause without attributing fault (see RISK-03). | M |
| FR-VIZ-12 | The system shall support export of the current view to a static format suitable for distribution where a live view cannot be shown. | S |

### 10.10 Reporting and Audit — `FR-RPT`

| ID | Requirement | Pri |
|---|---|---|
| FR-RPT-01 | The system shall maintain an audit history recording every change to project status, dates, effort estimate and assignments, with timestamp, user and reason. | M |
| FR-RPT-02 | The system shall provide a per-project history showing the full sequence of events from baseline to current state. | M |
| FR-RPT-03 | The system shall report portfolio-level totals: active projects, total committed man-days, utilisation by role, projects on hold, and aggregate slippage. | M |
| FR-RPT-04 | The system shall report the number of change requests and total man-days added per owning department, over a selected period. | S |
| FR-RPT-05 | The system shall report total working days lost to holds, broken down by hold reason. | S |
| FR-RPT-06 | The system shall reconcile, per project, baseline delivery date plus all recorded impacts against current delivery date, and flag any unexplained residual variance (RULE-11). | M |
| FR-RPT-07 | The system shall report total working days of delay attributed to delivery partner units, broken down by unit, subject to the visibility restriction of FR-EXT-11. | M |
| FR-RPT-08 | The system shall report, for the portfolio, the three principal sources of accumulated delay side by side — approved change requests, reprioritisation holds, and external commitment overruns — so their relative magnitude is directly comparable. | M |

---

## 11. Business Rules

These rules govern system behaviour and must be implemented exactly as stated.

### RULE-01 — Project Lifecycle

A project occupies exactly one status at a time, from this set:

| Status | Meaning | Resources committed |
|---|---|---|
| **Analysis** | Requirements and analysis work in progress; estimate being produced | Yes — BA, TPM, and Tech Lead where assigned |
| **In Progress** | Approved and under active delivery | Yes — full assigned team |
| **On Hold** | Work suspended; requires a reason (RULE-12) | Per hold reason disposition |
| **Delivered** | Development complete; handed to the business owner | Partially — support-level only |
| **Trial** | Live pilot with a limited user group, prior to full rollout | Partially — support and fixes |
| **Launched** | Full production rollout across the owning department; project closed | No — released |

Permitted transitions:

- Analysis → In Progress, On Hold
- In Progress → On Hold, Delivered
- On Hold → the status held from (RULE-10)
- Delivered → Trial, In Progress *(where trial or acceptance issues require further development)*
- Trial → Launched, In Progress *(where the pilot identifies required changes)*
- Launched → *(terminal)*

Any project may be cancelled from any non-terminal status (FR-PRJ-08).

### RULE-02 — Duration Calculation

For each role consumed by a project:

```
role_capacity_per_day = Σ (assignment.allocation_percentage × resource.availability_factor)
                        for all resources assigned in that role

role_duration_days    = role_effort_man_days ÷ role_capacity_per_day
```

The project's calculated duration is the **maximum** of its role durations, since roles run concurrently across the whole project (RULE-09).

**Limitation, explicitly accepted:** this model does not represent diminishing returns from adding people to a project. Doubling the developers on a project halves its calculated duration, which is not true in practice. This is a deliberate simplification for the first release and is mitigated by RULE-05, which lets the Technical PM override the calculated figure with their professional judgement. See ASM-06.

### RULE-03 — Date Calculation

Delivery dates are calculated across **working days only**, per the working calendar (RULE-04):

```
planned_delivery_date = advance(planned_start_date, ceiling(calculated_duration_days))
```

Where `advance` counts only working days and skips weekends and public holidays.

### RULE-04 — Working Calendar

The working calendar is configurable and defines the working week and the list of non-working public holidays. The default working week is Monday to Friday, with Saturday and Sunday non-working. UAE public holidays are maintained as a configurable list. See ASM-03 — this default must be confirmed against GD of AI's actual working pattern.

### RULE-05 — Manual Override of Calculated Dates

A Technical PM may override a calculated duration or delivery date. An override:

- requires a written justification;
- is recorded with the user and timestamp;
- is displayed distinctly from a calculated value (FR-SCH-05);
- persists until explicitly cleared, and is not silently replaced by recalculation.

Where an override is in force, subsequent impacts from change requests and holds are applied **relative to the overridden date**, not to the calculated one.

### RULE-06 — Developer Exclusivity (Soft)

A Developer is expected to work on one project at a time. Where a Developer is assigned to more than one project in **In Progress** status with overlapping dates, the system raises an over-allocation warning.

**This is a warning, never a block.** The assignment proceeds if the Technical PM acknowledges it (FR-ASG-05). The acknowledgement is recorded.

### RULE-07 — Aggregate Over-Allocation (All Roles)

For any person, the sum of allocation percentages across all concurrently assigned projects, adjusted by their availability factor, must not exceed 100%. Where it does, the system raises an over-allocation warning. This applies to every role, including those exempt from RULE-06.

As with RULE-06, this is a warning and is overridable with a recorded acknowledgement.

### RULE-08 — Multi-Project Roles

Technical Project Managers, Business Analysts and Tech Leads may be assigned to multiple concurrent projects without triggering RULE-06. They remain subject to RULE-07.

### RULE-09 — Whole-Project Assignment

A resource assigned to a project is committed for the project's entire duration, from planned start to planned delivery date. The system does not model per-phase resource demand in this release.

**Known consequence:** this overstates commitment. A Developer assigned to a project in Analysis status is shown as committed although no development work has begun. This is accepted for the first release as the simpler and more conservative model; per-phase allocation is deferred (§8.3). See ASM-05.

### RULE-10 — Hold Recalculation

On resume:

```
held_working_days     = working_days_between(hold.start_date, hold.actual_resume_date)
new_delivery_date     = advance(current_delivery_date, held_working_days)
```

The project returns to the status it held immediately before the hold. A hold record contributes its `held_working_days` to the project's variance breakdown (FR-VIZ-04).

While a hold is open, the system projects a provisional revised delivery date using the expected resume date where one has been recorded (FR-HLD-06). This figure is presented as provisional.

### RULE-11 — Variance Reconciliation

For every project, at all times:

```
baseline_delivery_date
  + Σ (working-day impact of every approved change request)
  + Σ (held working days of every hold)
  + Σ (working-day effect of every recorded manual override)
  + Σ (working-day overrun of every external commitment)
  = current_delivery_date
```

Any residual difference is **unexplained variance** and must be flagged (FR-RPT-06). This rule is the mechanism that delivers OBJ-05: it makes it structurally impossible for a date to move without a recorded cause.

> **On the fourth term.** External commitment overrun was added in version 0.2. Without it the rule was not satisfiable in practice: a delivery partner returning work later than committed moves the delivery date, but that movement corresponded to no term in the equation unless the Technical PM raised a formal Hold — which, for a routine overrun, does not happen (§4.4). The most frequent single cause of slippage was therefore the one cause the reconciliation could not name, and it surfaced as unexplained residual. The fourth term closes that gap.

### RULE-12 — Hold Reasons and Resource Disposition

| Reason | Displacing project link | Resource disposition | Attribution |
|---|---|---|---|
| **Resources pulled to higher-priority project** | **Required** | Released to pool | Reprioritisation decision |
| **Awaiting business owner input** | Not applicable | Retained | Business owner |
| **Awaiting approval / budget** | Not applicable | Released to pool | Approval authority |
| **Blocked by external or technical dependency** | Not applicable | Retained | External dependency |

*Resource disposition* determines whether the project's assignments are freed for other work during the hold (FR-ASG-11, FR-HLD-07). Where resources are released, they become available capacity immediately; where retained, they remain committed and unavailable.

*Attribution* records where the cause of the delay originated. It is used for factual reporting only and is presented neutrally (FR-VIZ-11).

> **Relationship to external commitments (RULE-15).** The reason *Blocked by external or technical dependency* is **not** the mechanism for recording delivery partner delay, and must not be used for it. A hold records that the project **stopped**; an external commitment overrun records that a segment of work **took longer than committed while remaining in progress**. The two are distinct events with distinct data, and conflating them was the defect identified in §4.4. This hold reason remains available for genuine stoppages caused by dependencies outside the project altogether — an unavailable third-party system, an unreleased platform, an external vendor.

### RULE-13 — Baseline Establishment and Re-Baselining

A project's baseline is established when the timeline is approved by the business owner and HQ (§3.4, step 8). At that moment the current effort estimate and dates are copied to the baseline fields and become immutable.

A project may be **re-baselined** only by explicit action, requiring a justification. Re-baselining archives the previous baseline together with all variance accumulated against it, and starts a new baseline period. This exists so that a project which has changed beyond recognition does not carry meaningless variance forever — but it must be a deliberate, visible act, not a means of erasing history.

### RULE-14 — Change Request Impact Application

A change request's effort is applied to the project's current estimate, and its date impact committed, **only on approval** (FR-CHG-06). Prior to approval the impact is calculated and displayed but not applied, so it can be presented to the business owner as a projection (FR-CHG-10).

### RULE-15 — External Commitment Measurement and Attribution

Applies to every project segment designated as externally held (FR-EXT-01).

**1. Two independent measurements are taken, not one.**

```
response_lag = working days between handover date and start date
overrun      = working days between committed date and actual delivery date   (0 where delivered on or before the committed date)
```

Response lag measures *delay before work began*; overrun measures *delay in the work itself*. They are reported separately because they indicate different problems and, in the case described in §4.4, the former is frequently the larger.

**2. Attribution is to a named unit.** Every external commitment names one responsible unit. Generic or unattributed external delay is not permitted — if the responsible unit cannot be named, the segment is not an external commitment and the delay must be recorded by another mechanism.

**3. Only overrun moves the delivery date.** The planned duration of an externally held segment is already contained in the baseline. Therefore only the *excess* over the committed date propagates to the project's delivery date (RULE-03) and enters the variance reconciliation (RULE-11). Response lag is measured and reported, but is not double-counted: where it delays the eventual delivery, that delay is already expressed in the overrun.

**4. Committed dates are versioned, never overwritten.** Where a unit revises its committed date, the prior date is retained (FR-EXT-09). Overrun is always measured against the **first** date committed after handover. A segment re-committed three times and delivered against the third date has not been delivered on time; the reconciliation must reflect that, and the history must show the re-commitments.

**5. Aggregation is bounded.** Per-unit aggregate figures are visible only within GD of AI (FR-EXT-11, §7.3). Per-project variance remains visible to stakeholders, but identifies the delayed *segment* rather than the responsible unit.

### RULE-16 — Observed Performance as a Planning Input

Where a delivery partner unit has a sufficient history of completed commitments, the system derives that unit's **mean overrun** and **mean response lag** for comparable segments, and offers them as advisory figures when a new segment of the same type is planned (FR-EXT-12).

These figures are **advisory only.** They are never applied automatically to a plan, never alter a baseline, and never appear as a committed date. The planned duration remains the Technical PM's decision; the system's role is to ensure that decision is made in sight of what has actually happened before, rather than solely on the date most recently committed.

A minimum history is required before any such figure is displayed; below that threshold the system shows the sample size and no derived mean, rather than presenting an average of two observations as though it were a pattern.

---

## 12. Conceptual Data Model

### 12.1 Entities

| Entity | Purpose | Key attributes |
|---|---|---|
| **Department** | Organisational unit that owns projects | Name, level (General Department / Sub-department / Section / Sector), parent department |
| **Project** | A unit of delivery work | Name, reference code, description, owning department, business PM name, origin, priority, status, baseline start/delivery dates, current planned start/delivery dates, status narrative, last updated |
| **Resource** | A member of GD of AI delivery staff | Name, employee reference, primary role, availability factor, active state |
| **Role** | A capacity type | Name (TPM, BA, Tech Lead, Developer), multi-project permitted flag |
| **EffortEstimate** | Estimated effort for a project in one role | Project, role, baseline man-days, current man-days, confidence |
| **Assignment** | Commitment of a resource to a project | Project, resource, role, allocation percentage, start date, end date, active state, override acknowledgement |
| **ChangeRequest** | A requirement raised after baseline | Project, title, description, requester, requesting department, date raised, state, effort per role, calculated day impact, date presented, date decided |
| **HoldRecord** | A suspension of work | Project, reason, displacing project, start date, expected resume date, actual resume date, held working days, resource disposition, directing authority, note |
| **DeliveryPartnerUnit** | An organisational unit that performs project work but is not part of the resource pool | Name, parent department, segment type normally owned (deployment / infrastructure / networking / data centre / security testing), active state. **No capacity, staffing or availability attributes** (FR-EXT-13) |
| **ExternalCommitment** | One handover of a project segment to a delivery partner unit | Project, segment or block reference, responsible unit, handover date, first committed date, current committed date, start date, actual delivery date, state, response lag, overrun, evidence reference, note |
| **CommitmentDateHistory** | A superseded committed date | External commitment, previous committed date, revised committed date, date of revision, reason |
| **StatusHistory** | Record of a status transition | Project, from status, to status, date, user, reason |
| **AuditEntry** | Record of any material change | Entity type, entity reference, field, previous value, new value, timestamp, user, reason |
| **WorkingCalendar** | Definition of working time | Working days of week, public holiday dates |
| **Absence** | Planned unavailability of a resource | Resource, type, start date, end date |

### 12.2 Relationships

```
Department ──(parent)──> Department                    self-referencing hierarchy
Department ──(owns)────> Project                       one department owns many projects

Project ───(estimated by)──> EffortEstimate ──(for)──> Role
Project ───(staffed by)────> Assignment     ──(of)───> Resource ──(has)──> Role
Project ───(changed by)────> ChangeRequest  ──(adds effort per)──> Role
Project ───(suspended by)──> HoldRecord
HoldRecord ──(displaced by)──> Project                 the displacing project
Project ───(tracked by)────> StatusHistory

Project ───(depends on)────> ExternalCommitment ──(held by)──> DeliveryPartnerUnit
ExternalCommitment ──(re-committed via)──> CommitmentDateHistory
DeliveryPartnerUnit ──(belongs to)──> Department       Smart Operations, or none for Information Security

Resource ──(unavailable during)──> Absence
```

**Note the deliberate asymmetry.** `Resource` carries availability and is consumed by `Assignment`; `DeliveryPartnerUnit` carries neither and is never assigned. A project's relationship to its own staff is *allocation*; its relationship to a delivery partner is *dependency*. This mirrors the organisational reality in §3.1 and is the reason the two are separate entities rather than one entity with a flag.

### 12.3 Derived Values

These are calculated, never stored as independent editable values:

| Value | Derivation |
|---|---|
| Project total weight | Σ current man-days across all roles (FR-EST-02) |
| Project calculated duration | RULE-02 |
| Project planned delivery date | RULE-03, unless overridden per RULE-05 |
| Delivery variance | Current delivery date − baseline delivery date, in working days |
| Variance breakdown | Σ change request impacts + Σ hold impacts + Σ override effects + Σ external commitment overruns (RULE-11) |
| External commitment response lag | Start date − handover date, in working days (RULE-15) |
| External commitment overrun | Actual delivery date − **first** committed date, in working days, floored at zero (RULE-15) |
| Per-unit delay total | Σ overruns of all commitments held by that unit, over a selected period (FR-EXT-10) |
| Per-unit mean overrun | Mean overrun across that unit's completed commitments, shown only above a minimum sample size (RULE-16) |
| Resource utilisation | Σ allocation percentages of active assignments ÷ availability factor |
| Role capacity | Σ availability across all active resources in that role |
| Earliest available start | First date on which unallocated capacity meets a proposed project's role requirements (FR-ASG-12) |

---

## 13. Non-Functional Requirements

### 13.1 First Release — Local Prototype

| ID | Requirement |
|---|---|
| NFR-01 | The system shall run entirely on a single local machine, with no external service dependency. |
| NFR-02 | The system shall persist data locally and retain it across restarts. |
| NFR-03 | The system shall be startable by a single documented command. |
| NFR-04 | The system shall support the stated scale without perceptible degradation: 60 concurrent active projects, 80 staff, and three years of accumulated history. |
| NFR-05 | Any portfolio view shall render within two seconds at the stated scale. |
| NFR-06 | The visualization page shall be legible when projected in a meeting room — adequate type size, sufficient contrast, and no reliance on fine detail or hover interaction to convey primary information. |
| NFR-07 | The system shall support seeding with representative sample data for demonstration purposes. |
| NFR-08 | The system shall permit the complete data set to be exported and re-imported, so a demonstration state can be preserved and restored. |
| NFR-09 | The interface shall be in English for the first release. |
| NFR-10 | Data entry for a routine project update shall require no more than a small number of fields, to keep the maintenance burden low enough to be sustained (see RISK-01). |

### 13.2 Production Readiness — Later Phase

Recorded here so the gap between the prototype and a deployable system is explicit and not discovered late. **None of these are in scope for the first release.**

| ID | Requirement |
|---|---|
| NFR-P-01 | Authentication integrated with Dubai Police Active Directory / single sign-on. |
| NFR-P-02 | Role-based access control aligned to §7.1, restricting write access to a user's own projects. |
| NFR-P-03 | Controlled access for external stakeholders to the visualization page, replacing open read access. |
| NFR-P-04 | Compliance with Dubai Police information security and data classification policy. |
| NFR-P-05 | Deployment within the approved Dubai Police hosting environment. |
| NFR-P-06 | Backup and recovery to the standard required for a departmental system of record. |
| NFR-P-07 | Full Arabic language support including right-to-left interface layout. |
| NFR-P-08 | Availability and support commitments appropriate to a system relied upon for HQ reporting. |
| NFR-P-09 | Tamper-evident audit log meeting the department's record-keeping requirements. |

---

## 14. Assumptions

| ID | Assumption | If wrong |
|---|---|---|
| ASM-01 | Technical Project Managers will keep project data current; the system's output is only as good as its input. | The system's reporting becomes untrustworthy and reverts to being a parallel Excel. See RISK-01. |
| ASM-02 | Effort estimates in man-days can be produced with sufficient consistency across projects to make capacity arithmetic meaningful. | Capacity figures mislead. Mitigated by estimate confidence levels (FR-EST-05) and, later, estimate-versus-actual tracking (§20). |
| ASM-03 | The default working week is Monday to Friday. This must be confirmed against GD of AI's actual working pattern before build. | Every calculated date is wrong. The calendar is configurable (RULE-04), so this is correctable, but it must be set correctly from the outset. |
| ASM-04 | A default availability factor of 80% reasonably represents productive project time after meetings, support and administration. | Durations are systematically optimistic or pessimistic. The factor is configurable per person (FR-RES-03). |
| ASM-05 | Whole-project resource assignment (RULE-09) is acceptable for the first release, despite overstating commitment during analysis phases. | Capacity appears more constrained than it is, and the department under-commits. Per-phase allocation is the deferred remedy (§8.3). |
| ASM-06 | Linear duration scaling with headcount (RULE-02) is acceptable given the manual override in RULE-05. | Calculated durations for larger teams are optimistic. Technical PM override is the primary control. |
| ASM-07 | Business owners and HQ stakeholders will engage with a self-service visualization view rather than continuing to request prepared presentations. | OBJ-06 is not met even if the system works. Mitigated by making the view directly presentable in meetings (NFR-06). |
| ASM-08 | Making the cause of delay visible is organisationally acceptable and will be received as factual reporting. | The system is resisted for political rather than functional reasons. See RISK-03 — this is the most significant non-technical risk. |
| ASM-09 | The first release is a prototype for internal demonstration and will not hold operational data requiring formal security classification. | Production security requirements (§13.2) become immediate rather than deferred. |
| ASM-10 | Projects are sufficiently independent that inter-project dependencies need not be modelled in the first release. | Delivery dates ignore real sequencing constraints. Dependency modelling is deferred (§20). |

---

## 15. Constraints

| ID | Constraint |
|---|---|
| CON-01 | The first release must run locally on a single machine, without departmental infrastructure provisioning. |
| CON-02 | No integration with existing Dubai Police systems is available for the first release. |
| CON-03 | GD of AI delivery capacity is finite and already committed; effort spent building PRMS competes with the project portfolio it manages. This argues strongly for a tightly scoped first release. |
| CON-04 | Business owners and HQ stakeholders will not enter data into the system under any circumstances; all input is from GD of AI staff. |
| CON-05 | Any production deployment will require Dubai Police information security review and approval. |
| CON-06 | The system must accommodate reprioritisation by HQ as a legitimate and frequent event. It must record and cost such decisions; it must never obstruct them. |

---

## 16. Risks and Mitigations

| ID | Risk | Impact | Likelihood | Mitigation |
|---|---|---|---|---|
| **RISK-01** | **Data currency.** Technical PMs do not keep project data up to date, and the system's output becomes untrustworthy. This is the single most common failure mode for systems of this type. | High | High | Keep required fields to a minimum (NFR-10); assign clear data ownership to the TPM; display staleness indicators (FR-PRJ-09) so out-of-date projects are visibly flagged rather than silently wrong; establish a weekly update cadence; ensure TPMs get direct value from the system so maintaining it is self-interested rather than administrative. |
| **RISK-02** | **Estimation quality.** Poor man-day estimates produce confident-looking but wrong dates, which is worse than no dates. | High | Medium | Record estimate confidence (FR-EST-05); present calculated dates as calculated rather than committed; retain estimate history (FR-EST-06); introduce estimate-versus-actual tracking in a later phase to improve calibration over time. |
| **RISK-03** | **Political sensitivity of attribution.** The system makes visible that a delay was caused by a specific reprioritisation decision or a specific business owner's change requests. This is the system's core purpose, but it can be perceived as blame. | High | Medium | Present all variance in neutral, factual language (FR-VIZ-11); frame the visualization around forward-looking decision support — *what will this cost* — rather than retrospective fault-finding; position attribution as protecting the decision-maker's ability to decide with full information; secure the Director's sponsorship for this framing before wider rollout. |
| **RISK-04** | **Adoption failure.** The system becomes a second place to record what is already in Excel, and is abandoned. | High | Medium | Make PRMS the sole source for status reporting so the alternative disappears; ensure the first release delivers immediate visible value to the TPMs who maintain it; deliberately restrict first-release scope so it is usable quickly. |
| **RISK-05** | **Scope expansion of PRMS itself.** The system acquires task tracking, time sheets, budgets and document management, and is never completed — an ironic instance of the problem it exists to solve. | Medium | Medium | Hold the boundaries in §8.2 firmly; treat additions as change requests against this BRD; retain §8.3 as the register for deferred requests so they are visibly recorded rather than argued. |
| **RISK-06** | **Model oversimplification.** Whole-project allocation (RULE-09) and linear duration scaling (RULE-02) produce figures that experienced PMs recognise as wrong, undermining confidence in the whole system. | Medium | Medium | Manual override with recorded justification (RULE-05); state the limitations openly in the interface rather than presenting calculated figures as authoritative; treat per-phase allocation as the first enhancement after the first release. |
| **RISK-07** | **Prototype-to-production gap.** A local prototype is approved by the Director and immediately expected in production, where §13.2 requirements apply. | Medium | High | Document the production gap explicitly (§13.2); present the prototype as a validation of the model, not as a deployable system; include the production readiness effort in any subsequent proposal. |
| **RISK-08** | **Single point of knowledge.** The system depends on one person who understands both the model and the build. | Medium | Medium | This BRD; documented business rules in §11 sufficient for another team to implement; avoid undocumented behaviour. |
| **RISK-09** | **Internal political exposure of delay attribution.** Per-unit figures name a *sibling department under the same Director* — Smart Operations sections and Information Security — rather than an external customer. This is materially more sensitive than RISK-03: those units are colleagues the delivery team must continue working with daily, and being measured without consultation may harden the very relationships the data is meant to improve. | High | Medium | Bound visibility to GD of AI and exclude it from the stakeholder page (FR-EXT-11, §7.3); report response lag and overrun as observations against *committed dates the units set themselves*, never as performance ratings; secure the Director's sponsorship before the data is used in any forum; consider informing the partner units that commitments are being recorded, rather than presenting the figures for the first time as an accusation. |
| **RISK-10** | **External commitment data is not captured at handover.** The whole mechanism depends on the committed date being recorded at the moment of handover. If the Technical PM records it late or not at all, there is nothing to measure against and the reconciliation reverts to unexplained variance. | High | High | Make handover a single state change on a block the PM is already viewing, never a separate form (NFR-10); default the handover date to today; allow the commitment to be recorded with a date and no evidence, so a missing attachment never blocks capture; surface un-captured external segments as a project data-quality flag alongside staleness (FR-PRJ-09). |
| **RISK-11** | **Observed-performance figures become self-fulfilling.** If planned durations are routinely inflated to a unit's historical mean overrun, the padding is absorbed and the unit is never held to its committed date, entrenching the delay rather than reducing it. | Medium | Medium | Keep RULE-16 figures strictly advisory and never auto-applied; continue to measure overrun against the unit's own committed date rather than against the padded plan, so the gap remains visible even where the plan has absorbed it; present the historical mean as context for negotiating the commitment, not as a substitute for it. |

---

## 17. Success Metrics

| ID | Metric | Baseline | Target |
|---|---|---|---|
| **KPI-01** | Preparation time for routine HQ status reporting | Manual deck rebuilt per meeting | Zero — status presented live from PRMS |
| **KPI-02** | Proportion of active portfolio recorded in PRMS | 0% | 100% of active projects |
| **KPI-03** | Proportion of delivery-date variance with a recorded, attributable cause | Not measurable | 100% (enforced by RULE-11) |
| **KPI-04** | Proportion of post-baseline requirements captured as formal change requests | Effectively 0% | 100% |
| **KPI-05** | Proportion of change requests whose impact was presented to the business owner before acceptance | Not measured | 100% |
| **KPI-06** | Proportion of holds recorded with a reason and, where applicable, a displacing project | Not recorded | 100% |
| **KPI-07** | Over-allocation incidents visible at the time of assignment rather than discovered later | 0% | 100% flagged at assignment |
| **KPI-08** | Consistency of reported status between consecutive stakeholder meetings | Author-dependent | Single source; no discrepancy |
| **KPI-09** | Projects delivered within their current (re-calculated) delivery date | Not measured | Measured from first release; improvement target set after one reporting period |
| **KPI-10** | Proportion of externally held segments with a committed date recorded at handover | Not recorded | 100% — the precondition for KPI-11 and for RULE-11 (see RISK-10) |
| **KPI-11** | Working days of portfolio delay attributable to named delivery partner units | Unknown and unmeasurable | Measured from first release; reduction target set by the Director after one reporting period |
| **KPI-12** | Proportion of delivery partner commitments delivered on or before the first date committed | Unknown | Measured from first release; establishes the baseline for §4.4 |

> KPI-09 is deliberately measured against the *current* date rather than the baseline. Measuring against a baseline that legitimate change requests and directed holds have moved would penalise GD of AI for decisions taken elsewhere — which is precisely the distortion this system exists to correct.

---

## 18. Glossary

| Term | Definition |
|---|---|
| **Baseline** | The scope, effort estimate and delivery dates approved by the business owner and HQ at the point of sign-off. Immutable thereafter except by explicit re-baselining (RULE-13). |
| **BA** | Business Analyst |
| **Business owner** | The general department, sub-department or section that owns the business process being digitalised and on whose behalf the project is delivered. |
| **Change request** | A requirement raised after the baseline is approved, requiring effort estimation, impact calculation and a decision before it is accepted. |
| **Committed date** | The completion date a delivery partner unit gives when project work is handed over to it. Retained permanently; overrun is measured against the *first* such date (RULE-15). |
| **Delivery partner unit** | An organisational unit that performs a mandatory segment of project work but is not part of the resource pool and is outside the Technical PM's authority — the Smart Operations sections and the Department of Information Security (§3.1, §7.3). |
| **Displacing project** | A project whose initiation or prioritisation caused resources to be withdrawn from another project, placing it on hold. |
| **External commitment** | The record of one handover of a project segment to a delivery partner unit: the unit responsible, the handover date, the date committed to, the date delivered, and the resulting delay (§10.8). |
| **GD of AI** | General Department of Artificial Intelligence — the central delivery department for digitalisation across Dubai Police. |
| **General Department** | A top-level organisational unit of Dubai Police (e.g. CID, Anti-Narcotics, Forensics). |
| **Higher-ups / HQ** | Police Headquarters senior leadership, who direct priorities and receive portfolio reporting. |
| **Hold** | A recorded suspension of work on a project, with a mandatory reason (RULE-12). |
| **Launched** | Full production rollout across the owning department; the project's terminal status. |
| **Man-day** | One person working for one working day. The unit of effort estimation throughout this system. |
| **Over-allocation** | A resource committed beyond their available capacity (RULE-07), or a Developer committed to more than one active project (RULE-06). |
| **PRMS** | Portfolio & Resource Management System — the working name for the system defined by this document. |
| **Re-baselining** | Explicitly replacing a project's baseline, archiving the prior baseline and its accumulated variance (RULE-13). |
| **Scope creep** | The accumulation of additional requirements after baseline approval without formal recognition of their cost. |
| **Trial** | A live pilot of a delivered solution with a limited user group, preceding full rollout. |
| **TPM** | Technical Project Manager — the GD of AI role accountable for delivery and for maintaining the project's record in PRMS. |
| **Variance** | The difference in working days between a project's baseline delivery date and its current delivery date. |
| **Weight** | A project's total estimated effort, being the sum of its man-days across all roles. |

---

## 19. Open Questions

| ID | Question | Why it matters | Needed by |
|---|---|---|---|
| **OQ-01** | What should the system be formally named? "PRMS" is a working title only. | Naming affects how the system is positioned to the Director and to stakeholders. | Before demonstration |
| **OQ-02** | What is GD of AI's actual working week and public holiday schedule? | Directly determines every calculated date (RULE-04, ASM-03). | Before build |
| **OQ-03** | Is 80% a realistic default availability factor for GD of AI delivery staff? | Systematically shifts all calculated durations (ASM-04). | Before build |
| **OQ-04** | Is the baseline established at HQ approval (§3.4 step 8), or at business owner approval, where these differ? | Determines the reference point for all variance reporting (RULE-13). | Before build |
| **OQ-05** | Who is authorised to approve a change request on the business side, and is HQ approval required for changes above a certain size? | Determines whether the change request workflow needs an approval hierarchy beyond FR-CHG-05. | Before build |
| **OQ-06** | Should the read-only visualization page be reachable by stakeholders directly, or presented on screen by GD of AI in meetings? | Affects hosting and access approach even for the prototype (FR-VIZ-01, NFR-01). | Before build |
| **OQ-07** | Are Technical PMs and BAs also subject to a practical limit on concurrent projects, beyond the aggregate rule (RULE-07)? | May require a per-role concurrent project ceiling in addition to percentage-based allocation. | Before build |
| **OQ-08** | Are there existing Dubai Police project governance standards or templates that PRMS status reporting should conform to? | Alignment improves acceptance at HQ and may constrain the visualization design. | Before demonstration |
| **OQ-09** | What are the remaining sub-departments of CID, and the sub-departmental structure of the other general departments? | Required to complete the requesting-side hierarchy in §3.1 and the owning-department selection at project creation (FR-PRJ-04). Partially captured; explicitly incomplete. | Before build |
| **OQ-10** | Is the commitment from a delivery partner unit given as a date, a duration, or informally — and is it given in writing? | Determines whether FR-EXT-08 evidence capture is realistic or whether the committed date will usually be the TPM's own record of a verbal agreement. Recorded position: a date is given per task, and it is routinely missed (§4.4). | Before build |
| **OQ-11** | Do the Smart Operations sections and Information Security work to any published service standard or turnaround target? | If one exists, overrun should be measured against it as well as against the per-task committed date, which materially strengthens the §4.4 case. | Before build |
| **OQ-12** | Should delivery partner units eventually record their own commitment dates and completion directly, rather than the TPM recording on their behalf? | Removes the single largest data-quality risk in this area (RISK-10) but requires accounts for units explicitly excluded from the first release (§8.2). Candidate for Phase 4. | Phase 2 review |
| **OQ-13** | Is the Director of GD of AI willing to sponsor per-unit delay reporting before it is produced? | RISK-09 is the governing risk on §4.4. Producing the data without sponsorship risks the mechanism being shut down and damaging working relationships. | Before demonstration |

---

## 20. Appendix — Phased Roadmap

### Phase 1 — Prototype (this BRD, first release)

Local, single-machine system delivering the capacity and assignment engine, change request and hold management with automatic date recalculation, and the read-only stakeholder visualization page. Seeded with representative data. Purpose: **validate the model and demonstrate it to the Director of GD of AI.**

### Phase 2 — Departmental Pilot

Deploy within GD of AI with real portfolio data. Add authentication and role-based access (NFR-P-01, NFR-P-02). Run a full reporting cycle against real HQ meetings. Purpose: **prove the model against reality and establish the data discipline of RISK-01.**

### Phase 3 — Production

Full production readiness per §13.2: security review, approved hosting, Arabic and RTL support, backup and recovery, controlled stakeholder access. Purpose: **make PRMS the department's system of record.**

### Phase 4 — Model Refinement

Enhancements deliberately deferred from Phase 1, in expected order of value:

1. **Per-phase resource allocation** — resolves the overstatement in RULE-09 and materially improves capacity accuracy
2. **Estimate-versus-actual tracking** — closes the loop on RISK-02 and improves estimation over time
3. **Scenario planning** — multiple saved what-if scenarios, allowing options to be compared before a prioritisation decision is taken
4. **Inter-project dependency modelling** — resolves ASM-10
5. **Delivery partner self-recording** — units record their own commitment dates and completions directly, removing the data-quality risk of RISK-10 and the appearance of being measured by a third party (OQ-12)
6. **Skill-based resource matching** — beyond role-based assignment
7. **Vendor and contractor capacity** — extends the resource pool beyond Internal Applications staff
8. **Integration with existing Dubai Police systems** — removes duplicate data entry

---

*End of document.*
