# Marwat Welfare Society — Master Product Requirements Document (PRD)

| Field | Value |
|---|---|
| **Document Title** | Master Product Requirements Document |
| **Project Name** | Marwat Welfare Society — Blood Donation & Distribution Platform |
| **Document Version** | 1.1.0 |
| **Status** | Approved Baseline |
| **Classification** | Internal — Project Constitution |
| **Last Updated** | 2026-07-06 |
| **Document Owner** | Founder / Principal Architect, Marwat Welfare Society |
| **Approval Authority** | Founder (solo-founder stage); Board Tech Committee (post-Phase 4) |
| **Next Review Date** | 2027-01-06 (semi-annual review) |

> **CONSTITUTION CLAUSE.** This document is the supreme source of truth for the Marwat Welfare Society platform. Every subsequent design document (System Architecture, TDS, AI Rulebook), every implementation prompt, every pull request, and every AI coding session **MUST** conform to this PRD. Where any artifact conflicts with this PRD, this PRD prevails. Changes require founder approval (during solo-founder stage) or Board Tech Committee approval (post-Phase 4) and a version bump recorded in `CHANGELOG.md`.
>
> **v1.1.0 REFACTOR NOTE.** This version transforms the document from enterprise-first to **startup-first, phased, AI-assisted** without removing any long-term capability. Every requirement now carries a **Phase tag** (`P1`, `P2`, `P3`, `P4`) indicating when it is implemented. Future capabilities are preserved verbatim and collected under §15.5 *Future Expansion Roadmap*. Security and architectural rigor are unchanged.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Vision, Mission & Strategic Objectives](#2-vision-mission--strategic-objectives)
3. [Problem Statement & Current-State Analysis](#3-problem-statement--current-state-analysis)
4. [Target Audience & User Personas](#4-target-audience--user-personas)
5. [Roles & Access Control Overview](#5-roles--access-control-overview)
6. [Functional Requirements — Module by Module](#6-functional-requirements--module-by-module)
7. [Non-Functional Requirements](#7-non-functional-requirements)
8. [User Stories & Acceptance Criteria](#8-user-stories--acceptance-criteria)
9. [Automation Requirements](#9-automation-requirements)
10. [AI Assistance Boundaries](#10-ai-assistance-boundaries)
11. [Data & Compliance Requirements](#11-data--compliance-requirements)
12. [Success Metrics & KPIs](#12-success-metrics--kpis)
13. [Constraints, Assumptions & Dependencies](#13-constraints-assumptions--dependencies)
14. [Phase Classification & Out-of-Phase Items](#14-phase-classification--out-of-phase-items)
15. [Phased Roadmap](#15-phased-roadmap)
16. [Glossary](#16-glossary)
17. [Revision History](#17-revision-history)

---

## 1. Executive Summary

### 1.1 Purpose of This Document

This Product Requirements Document (PRD) defines, in unambiguous and testable terms, what the **Marwat Welfare Society platform** must do, for whom, under what constraints, and to what measurable standard. It is the first of four foundational documents that together form the project's "constitution":

1. **PRD (this document)** — *What* we are building and *why*.
2. **System Architecture Document** — *How* the system is structured at a high level.
3. **Technical Design Specification** — *How* the system is implemented in concrete detail (schemas, APIs, types).
4. **AI Development Rulebook** — *How* every future AI-assisted coding session must behave to keep the codebase consistent.

This PRD is deliberately written to remain valid for **10+ years**. Technology choices, library names, and infrastructure vendors will change; the *product* described here must not. Where the PRD references a specific technology, that reference is illustrative of a *category* (e.g., "Supabase" means "managed PostgreSQL with auth, storage, and edge functions"), and the category requirement is binding even if the vendor is later replaced.

### 1.2 Organization Background

Marwat Welfare Society is a registered welfare organization operating in Pakistan. Its core mission is the **collection, storage, management, and distribution of blood** to patients in need, conducted in a manner that minimizes manual administrative work through intelligent automation. The organization operates across multiple axes simultaneously:

- **Donor acquisition and retention** through public outreach, blood camps, and ongoing donor relationship management.
- **Clinical logistics** through hospital partnerships, patient case management, and just-in-time blood delivery.
- **Inventory governance** through strict tracking of every blood bag from collection to disposal, with full chain-of-custody.
- **Volunteer enablement** through a structured volunteer program with role-based task assignment and recognition.
- **Public accountability** through transparent reporting of impact metrics, financial stewardship, and operational efficiency.

The organization currently relies on a mix of paper records, spreadsheets, WhatsApp groups, and individual staff memory. This approach has reached its breaking point as the organization scales. Lost donor information, expiry of blood bags without notification, missed donation eligibility windows, and slow response to emergency requests are recurring operational failures that this platform exists to eliminate.

### 1.3 Product in One Paragraph

The Marwat Welfare Society platform is a **multi-tenant-capable single-tenant SaaS application** that provides a public-facing website, an administrative dashboard, a hospital portal, and (post-Phase 4) a mobile app — all backed by a normalized PostgreSQL database with row-level security, end-to-end audit logging, and an **optional** automation layer (n8n, added in Phase 3) that handles notifications, reminders, reports, and AI-assisted content generation. The platform is engineered for **zero operational cost** at the free tier of its constituent services, runs **local-first** (a single laptop, a small office LAN, or the cloud — all without architectural change), and ships in **four incremental phases** so a solo founder with AI assistance can deliver Phase 1 within weeks, not months.

### 1.4 Why This Platform, Why Now

Pakistan's blood donation ecosystem faces three structural problems that this platform directly addresses:

1. **Information asymmetry.** Patients and guardians do not know where blood is available. Hospitals cannot find willing donors quickly. Donors do not know when they are needed. The platform collapses this asymmetry by making inventory visible (to authorized roles) and by triggering targeted outreach to eligible donors when demand spikes.

2. **Waste through expiry.** Whole blood and components have strict shelf lives (35 days for whole blood, 5 days for platelets, 1 year for plasma). Without active expiry management, bags expire on shelves while patients elsewhere go without. The platform's inventory and automation layers actively surface nearing-expiry bags and route them to high-urgency requests first.

3. **Donor attrition through silence.** A donor who gives blood once and never hears from the organization again is unlikely to donate a second time. The platform's automation layer maintains ongoing donor relationships — eligibility reminders, impact summaries, appreciation messages, and emergency availability pings — converting one-time donors into recurring ones.

### 1.5 Primary Success Criteria

Success criteria are **phase-scoped**, so the founder can claim measurable wins at each release rather than only at the end of an annual cycle.

**Phase 1 (Core MVP) success criteria:**

- The organization replaces its paper-and-spreadsheet workflow with the platform for **all daily operations** within 4 weeks of Phase 1 launch.
- **100% audit trail coverage** — every state change on every blood bag, every patient request, and every role escalation is recoverable.
- **Zero data breaches** — no unencrypted PHI at rest, no unauthenticated API access, no RLS bypass.
- Platform runs on a **single office laptop or desktop PC** with no internet dependency for core operations (sync to cloud when connectivity returns).
- A non-technical admin (Imran persona) can perform every routine task without developer assistance.

**Phase 2 (Operational Improvements) success criteria:**

- Donors, patients, and hospitals each have a self-service web portal.
- Email notifications deliver within 60 seconds of the triggering event.
- Blood bag expiry rate drops below 5% (baseline: 12%).

**Phase 3 (Automation) success criteria:**

- 90% of routine notifications and reports are produced without human intervention.
- **90% reduction** in time-to-locate-blood for emergency requests (baseline: ~4 hours via phone calls; target: under 25 minutes via platform).
- AI-generated content volume is at least 4 use cases, all subject to human review.

**Phase 4 (Scale) success criteria:**

- **75% reduction** in blood bag expiry rate (target: under 3%).
- **2× increase** in recurring donor rate (donors who have given 2+ times in a 12-month window).
- Platform supports multi-city, multi-branch operation with no architectural change.

Full metric definitions are in §12.

### 1.6 Document Conventions

Throughout this PRD, the words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are used in the RFC 2119 sense. Every requirement tagged with MUST is a release blocker for the phase in which it is scheduled; SHOULD requirements are strongly preferred but negotiable with documented justification; MAY requirements are optional enhancements.

**Phase tags.** Every functional requirement (FR-XXX-NNN) carries a phase tag in its table row or in a parenthetical note: `P1` = Phase 1 Core MVP, `P2` = Phase 2 Operational Improvements, `P3` = Phase 3 Automation, `P4` = Phase 4 Scale. When a requirement spans phases, the tag indicates the **first** phase in which it appears; later phases extend it. This lets a solo founder implement Phase 1 without reading Phase 3+ requirements, while keeping the long-term contract visible.

**Solo-founder assumption.** Where the document refers to "the team", "the maintainer", or "staff", this means the founder and any AI coding assistants the founder has delegated to. Where the document refers to "Board approval", during the solo-founder stage (Phases 1–3) this means founder approval; the Board Tech Committee is consulted from Phase 4 onward when the organization has scaled enough to warrant formal governance.

---

## 2. Vision, Mission & Strategic Objectives

### 2.1 Vision Statement

> *A Pakistan in which no patient dies for want of blood, no donor's gift is wasted, and no welfare organization's resources are consumed by paperwork that a machine could handle.*

### 2.2 Mission Statement

> *Marwat Welfare Society exists to ensure that every unit of blood donated reaches the patient who needs it, on time, with full traceability, and with the dignity that both donor and recipient deserve — by building and operating software that automates the bureaucracy of compassion.*

### 2.3 Strategic Objectives (5-Year Horizon)

The platform's design is anchored to five strategic objectives. Every feature in this PRD must map to at least one of them; any feature that does not map is, by definition, out of scope.

| # | Strategic Objective | Owner | Target by Year 5 |
|---|---|---|---|
| SO-1 | **Operational excellence in blood logistics** | Operations Director | 99.5% on-time fulfillment of validated requests |
| SO-2 | **Donor lifetime value maximization** | Donor Relations Lead | 5+ donations per active recurring donor over 5 years |
| SO-3 | **Volunteer force scalability** | Volunteer Coordinator | 1,000+ active volunteers with >80% task acceptance |
| SO-4 | **Public transparency & trust** | Executive Director | Quarterly impact reports published within 15 days of quarter close |
| SO-5 | **Platform self-sufficiency** | CTO / Tech Lead | All routine operations executable by staff without developer intervention |

### 2.4 Product Principles

These principles are the **non-negotiable design philosophy** of the platform. Where two requirements appear to conflict, the principle higher in this list prevails.

1. **Patient first.** When any decision trades donor convenience, volunteer efficiency, or operational cost against a patient receiving blood in time, the patient wins.
2. **Privacy by default.** Personal and health data is collected only when necessary, exposed only to roles that need it, retained only as long as required, and never sold or shared outside the organization without explicit consent.
3. **Automation over delegation.** If a task is repetitive, rule-based, and frequent, a machine must do it. Human time is reserved for judgment, empathy, and exception handling.
4. **Fail safe, fail loud.** When the platform fails (and it will), it must fail in a way that protects patient data and surfaces the failure to operators immediately. Silent failures are forbidden.
5. **Replaceability over loyalty.** Every vendor choice (Supabase, Vercel, n8n, Expo) is made with the assumption that it may need to be replaced. Abstractions must be in place to make replacement a project, not a rewrite.
6. **Documentation is a deliverable.** Code without documentation is incomplete work. Every module, every endpoint, every schema ships with documentation that a new contributor can understand without asking.
7. **Accessibility is not optional.** The platform must be usable by people with visual impairments, motor limitations, low-bandwidth connections, and old devices. WCAG 2.1 AA is the floor, not the ceiling.
8. **Honesty in metrics.** Internal dashboards show real numbers, including the bad ones. Vanity metrics are forbidden. If a metric cannot be defined unambiguously, it is not a metric.

---

## 3. Problem Statement & Current-State Analysis

### 3.1 The Problem Space

Blood donation in Pakistan, like in most developing countries, operates as a fragmented informal network. The Marwat Welfare Society currently functions within this fragmentation, achieving meaningful impact despite — not because of — its tooling. The platform must address the following concrete problems, each described with its current-state manifestation and the desired future state.

#### 3.1.1 Problem: Donor records are lost or duplicated

**Current state.** Donor information is captured on paper forms at blood camps, transcribed (sometimes) into a spreadsheet, and frequently re-collected on the donor's next visit because the previous record cannot be located. Donors find this disrespectful and report it as a primary reason for not returning.

**Future state.** A donor registers once — via the public website, the mobile app, or at a blood camp (with a volunteer capturing via the volunteer app) — and that record is canonical for life. Every subsequent donation, eligibility check, communication, and impact summary references the same donor record. Donors can self-update their contact details, availability status, and medical eligibility without staff intervention.

#### 3.1.2 Problem: Blood inventory is opaque

**Current state.** The organization knows roughly how many bags are in the refrigerator but not, without physically inspecting each bag, which blood types are available, which are nearing expiry, or which are reserved for pending requests. Hospital staff call the office to ask; the office staff open the refrigerator and count. This takes 10–30 minutes per query and produces answers that are stale by the time they are received.

**Future state.** Inventory is updated in real time at every state transition (collection, screening, storage, reservation, issuance, return, discard). Any authorized user (admin, hospital triage nurse) can see current inventory by blood type, component, and storage location in under 2 seconds. The system proactively alerts when inventory drops below threshold for any blood type.

#### 3.1.3 Problem: Emergency requests are slow and noisy

**Current state.** When a hospital has an emergency need, they call the office. Office staff broadcast the need via WhatsApp groups to whoever is in the group. Response rate is low because most donors in the group are not eligible (recent donation, medical deferral, geographic distance). The noise of irrelevant broadcasts causes group members to mute the group, reducing future response rates further.

**Future state.** A hospital submits an emergency request through the hospital portal. The system filters the donor database for eligible donors (correct blood type, not donated within deferral window, marked as available, geographically proximate) and sends targeted notifications only to those donors — via the channel they have opted into (SMS, WhatsApp, push notification, email). Response rate per notification rises because each donor is only contacted when they are genuinely able to help.

#### 3.1.4 Problem: Expiry waste

**Current state.** Without active expiry management, approximately 12% of collected whole blood units expire before they can be issued. Platelets, with a 5-day shelf life, expire at an even higher rate. Expired units are discarded, representing both a waste of a donor's gift and a financial loss to the organization.

**Future state.** The system tracks expiry date on every unit from the moment of collection. Nearing-expiry units (configurable threshold, default 7 days for whole blood, 24 hours for platelets) are surfaced in the admin dashboard, included in the daily operations digest, and prioritized in fulfillment matching for compatible requests. Target expiry rate: under 3%.

#### 3.1.5 Problem: Reporting is manual and infrequent

**Current state.** Monthly and annual reports are compiled by hand from spreadsheets and paper records, taking 3–5 person-days per report and frequently containing arithmetic errors. Reports are delivered late, if at all, undermining donor and institutional confidence.

**Future state.** Reports are generated automatically on schedule (daily operations digest, monthly impact report, annual report) with zero manual compilation. Reports are reviewed and approved by a human before publication, but the underlying data compilation and initial drafting are automated. AI assists in generating narrative content (success stories, appreciation messages, statistics summaries) subject to human review.

#### 3.1.6 Problem: Volunteer coordination is ad hoc

**Current state.** Volunteers are recruited through personal networks, given informal task assignments via WhatsApp, and have no visibility into the organization's broader activities or their own contribution history. Volunteer retention is poor — most volunteers participate in one or two events and then disengage.

**Future state.** Volunteers register through the platform, complete a profile indicating their skills, availability, and geographic area, and receive task assignments through the volunteer app. They can see their contribution history, earned certificates (AI-drafted, admin-approved), and upcoming opportunities. The platform's automation layer sends reminders, appreciation messages, and re-engagement prompts.

#### 3.1.7 Problem: Patient and guardian experience is stressful and opaque

**Current state.** A patient's guardian seeking blood faces the worst day of their life compounded by bureaucratic friction. They do not know what the organization has, what documents they need, what the process is, or what the status of their request is. They call repeatedly, are put on hold, and receive inconsistent information from different staff members.

**Future state.** A guardian (or hospital staff on their behalf) submits a request through the patient portal or hospital portal with guided document upload and clear status tracking. The platform acknowledges receipt, sets expectations on response time, and pushes status updates at each stage of fulfillment. The guardian can see the same status the organization sees, eliminating the need to call for updates.

### 3.2 Current-State Tooling Inventory

The following table documents the tools currently in use and the platform's disposition toward each. This inventory exists to ensure the platform is designed to *replace* these tools cleanly, not coexist with them indefinitely.

| Tool | Current Use | Future State | Migration Path |
|---|---|---|---|
| Paper registration forms | Donor sign-up at camps | Retired | Volunteer app captures directly into platform |
| Excel spreadsheets | Donor master, inventory, finance | Retired | Platform database, finance module (post-v1) |
| WhatsApp groups (multiple) | Donor broadcasts, volunteer coordination, hospital communication | Retired for donor broadcast (replaced by targeted notifications); retained for human-to-human conversation only | Opt-in channel migration; platform becomes source of truth |
| Phone calls (manual) | Hospital-to-office requests | Retired for routine requests; retained for escalations | Hospital portal becomes primary intake |
| Paper blood bag tags | Bag identification | Supplemented by platform-generated QR code labels | Print QR labels from platform; phase out paper tags |
| Manual monthly report compilation | Reporting | Retired | Automated report generation + human review |

### 3.3 Stakeholder Map

| Stakeholder | Relationship | Primary Need from Platform |
|---|---|---|
| **Patients** | Recipients of service | Timely blood, dignified treatment, transparent status |
| **Patient guardians** | Proxy for patients | Submit requests, track status, upload documents |
| **Donors** | Voluntary contributors | Easy registration, donation history, impact visibility, respect for time |
| **Volunteers** | Operational workforce | Clear task assignment, contribution tracking, recognition |
| **Hospitals** | Institutional partners | Submit requests reliably, view inventory, receive fulfillment confirmation |
| **Doctors** | Clinical decision-makers | Request specific blood products with clinical justification |
| **Office staff** | Internal operators | Efficient workflows, reduced manual work, audit trail |
| **Board of Directors** | Governance | Transparent reporting, risk visibility, compliance assurance |
| **Regulators** | Compliance oversight | Traceability, consent management, data protection |
| **General public** | Potential donors and supporters | Information about the organization, ways to contribute, impact stories |

---

## 4. Target Audience & User Personas

The platform serves seven distinct user populations, each with different needs, technical fluency, devices, and emotional contexts. Designing for the average user is forbidden; every feature must be designed for the specific persona it serves. The personas below are composites based on real user archetypes and are referenced by name throughout the rest of this PRD.

### 4.1 Persona: Bilal — The Recurring Donor

**Demographics.** Bilal is 28, male, lives in Peshawar, works as a software engineer, owns a mid-range Android smartphone on a 4G data plan, and is comfortable with mobile apps and online forms.

**Motivation.** Bilal donated blood for the first time at a university blood camp after a friend's family member needed it. He found the experience meaningful and wants to donate regularly, but he forgets when he is eligible again, and the organization never contacts him. He has donated three times in four years but considers this less than he would like.

**Frustrations with current state.** He receives WhatsApp broadcasts about every blood request in the city, most of which are not his blood type (B+). He has muted the group. When he tries to find out when he can donate next, no one can tell him. He has never been told what happened to the blood he donated — he assumes it helped someone but does not know.

**What the platform must do for Bilal.**

- **One-time registration** with his blood type, contact details, and communication preferences (channel + topics).
- **Automatic eligibility tracking** — the platform calculates his next eligible date based on his last donation and reminds him via his chosen channel 7 days before, on the day, and 3 days after.
- **Targeted notifications** — he is only contacted about requests for B+ blood, and only when he is eligible and marked available.
- **Impact visibility** — every donation is linked (anonymized) to the recipient case so he can see "your donation was issued to a patient at X Hospital on Y date for Z condition." He sees a lifetime impact counter.
- **Emergency availability toggle** — he can mark himself as "available this week" or "unavailable" without losing his donor status.
- **Medical self-service** — he can update his medical history (new medications, recent travel to malaria-endemic regions) which automatically adjusts his eligibility calculation.

### 4.2 Persona: Saima — The Patient's Guardian

**Demographics.** Saima is 41, female, lives in a small town 60 km from the nearest tertiary hospital, owns a basic Android smartphone on a 3G/4G plan with limited data, has primary-school literacy in Urdu and limited English, and is the primary caregiver for her mother who is undergoing chemotherapy.

**Motivation.** Saima's mother needs regular blood transfusions as part of her treatment. Each transfusion cycle requires Saima to arrange 2–3 units of blood. The process of arranging blood is the most stressful part of an already stressful medical situation. She wants the blood arranged quickly, with minimum paperwork, and with clear communication so she knows what to expect.

**Frustrations with current state.** She does not know whom to call. When she does reach someone, she is asked different questions each time. She has to physically visit the office with documents, taking a full day off work. She does not know the status of her request after submitting it. She has been told "we will call you back" multiple times and the call never comes.

**What the platform must do for Saima.**

- **Urdu-language interface** with simple, large text, minimal jargon, and clear iconography.
- **Guided request submission** — step-by-step form that asks only what is necessary, with help text in Urdu for every field, and document upload via phone camera.
- **Status tracking** — she can see exactly where her request is in the fulfillment pipeline (Submitted → Verified → Matching → Allocated → Issued → Delivered → Received).
- **Proactive notifications** — she receives a notification at every status change via SMS (her preferred channel, as she has limited data).
- **No office visits required** — the entire process is completable from her phone. Documents are uploaded as photos; verification happens remotely.
- **Emergency contact escalation** — if her request has not progressed within a defined SLA, the platform automatically escalates to a senior admin and surfaces it on the dashboard.

### 4.3 Persona: Dr. Khan — The Hospital Blood Bank Technician

**Demographics.** Dr. Khan is 38, male, works at a mid-sized hospital in Peshawar, has a medical degree and fluent English, uses a desktop computer at the hospital and a smartphone personally, and is responsible for the hospital's blood bank liaison with external donors and welfare organizations.

**Motivation.** Dr. Khan's job is to ensure his hospital has the blood it needs. He interacts with multiple welfare organizations, and the one that responds fastest and most reliably is the one he calls first. He wants Marwat Welfare Society to be the one he calls first.

**Frustrations with current state.** He has to call during office hours. He has to verbally specify what he needs and trust that the person on the other end writes it down correctly. He does not know what is available until he asks. He does not have documentation of what was issued for his hospital's records. He has to follow up by phone to confirm dispatch.

**What the platform must do for Dr. Khan.**

- **Hospital portal** with role-based login for his hospital's blood bank staff.
- **Inventory visibility** — read-only view of available inventory at Marwat Welfare Society filtered by blood type and component, updated in real time.
- **Structured request submission** — he specifies blood type, component, quantity, urgency level, patient details, and clinical justification. The platform validates against inventory and confirms receipt.
- **Request history** — full history of his hospital's requests, issued units, and deliveries, exportable for his hospital's records.
- **Delivery tracking** — when a unit is dispatched, he receives a notification with the delivery method and expected arrival time.
- **SLA transparency** — every request shows its committed response time and current status against that SLA.

### 4.4 Persona: Ayesha — The Volunteer

**Demographics.** Ayesha is 21, female, a university student in Islamabad, owns a recent Android phone on a 4G plan, fluent in English and Urdu, comfortable with mobile apps, has variable availability (heavy during semester breaks, light during exams).

**Motivation.** Ayesha wants to contribute to her community. She has time but not money. She has volunteered at two blood camps and found the experience rewarding but disorganized — she showed up and was given ad hoc tasks, with no clear briefing or follow-up. She would do more if the experience were more structured and if she could see the cumulative impact of her contribution.

**Frustrations with current state.** She hears about opportunities through a WhatsApp group that she mutes during exams. When she does volunteer, no one follows up afterward — no thank you, no impact summary, no invitation to the next event. She has stopped checking the WhatsApp group.

**What the platform must do for Ayesha.**

- **Volunteer profile** with skills, availability calendar, geographic area, and preferred task types.
- **Task assignment** — she is assigned tasks that match her profile and receives push notifications. She can accept, decline, or request reassignment.
- **Briefing materials** — every task includes a brief, location, expected duration, and contact person.
- **Contribution history** — she can see every task she has completed, hours contributed, and people helped (anonymized aggregate).
- **Recognition** — she earns certificates (AI-drafted, admin-approved) for milestones (10 camps, 50 hours, 100 donors registered) that she can share on LinkedIn.
- **Re-engagement** — if she has been inactive for 30 days, the platform sends a personalized re-engagement message with upcoming opportunities that match her profile.

### 4.5 Persona: Imran — The Office Administrator

**Demographics.** Imran is 45, male, has worked at Marwat Welfare Society for 6 years, uses a Windows desktop computer at the office, is moderately tech-comfortable (can use spreadsheets and email, struggles with new interfaces), and is the primary person who currently maintains the Excel-based donor and inventory records.

**Motivation.** Imran wants to do his job well. He is overworked — most of his day is consumed by phone calls, data entry, and report compilation. He is initially skeptical of the new platform because he has seen previous "modernization" attempts fail, but he is willing to engage if it genuinely reduces his workload rather than adding a new system on top of his existing one.

**Frustrations with current state.** He spends 60% of his time on data entry and report compilation that he knows a machine could do. He is blamed when records are lost or reports are late, even though the underlying problem is the system, not him. He is afraid that the new platform will make him redundant.

**What the platform must do for Imran.**

- **Reduce, not increase, his workload.** Every workflow he currently does manually must have a faster equivalent in the platform.
- **Familiar interface patterns** — the admin dashboard uses standard SaaS conventions (sidebar, table views, modal forms) so he can transfer his existing skills.
- **Bulk operations** — for the tasks that remain manual (e.g., entering historical paper records), he can do them in bulk via CSV upload, not one-by-one.
- **Role continuity** — his role (Admin) is preserved and elevated; the platform makes him more effective, not redundant. He becomes the system's super-user and trainer.
- **Comprehensive audit trail** — when something goes wrong, he can prove what he did and when, rather than being blamed by default.

### 4.6 Persona: Fatima — The Super Admin (Executive Director)

**Demographics.** Fatima is 50, female, co-founded the organization, uses a laptop and smartphone, fluent in English and Urdu, splits her time between operations, fundraising, and governance. She is the accountable owner of the platform.

**Motivation.** Fatima needs the platform to give her organization-wide visibility, support governance and compliance, and produce the reports she needs for the board, donors, and regulators. She does not have time to learn a complex system — she needs dashboards that answer her questions in 30 seconds.

**Frustrations with current state.** She cannot answer board questions without asking Imran, who takes a day to compile the answer. She does not have real-time visibility into operations. She worries about compliance but does not have a clear picture of where the organization stands.

**What the platform must do for Fatima.**

- **Executive dashboard** — one screen that answers "how is the organization doing right now?" (inventory levels, pending requests, expiring units, donor count, volunteer hours this month).
- **Drill-down on every metric** — she can click any number to see the underlying records, without learning SQL or asking Imran.
- **Audit trail access** — she can see who did what, when, without developer help.
- **Report generation** — she can generate any standard report on demand, and the platform produces scheduled reports automatically.
- **User management** — she can create, deactivate, and role-assign users without developer help.
- **Configuration management** — she can change operational parameters (deferral windows, SLA thresholds, notification templates) without code changes.

### 4.7 Persona: Anonymous Public Visitor

**Demographics.** Unknown. Could be anyone with an internet connection — a potential donor, a journalist, a regulator, a student doing research, a family member of a patient considering their options.

**Motivation.** They want to understand what Marwat Welfare Society does, whether it is legitimate, how to donate blood, how to request blood, and what impact the organization has had. They may have no technical fluency and may be on a slow connection or old device.

**What the platform must do for them.**

- **Fast, accessible public website** — loads in under 3 seconds on a 3G connection, works on a 5-year-old Android phone, is fully usable without JavaScript (progressive enhancement).
- **Clear calls to action** — "Donate blood", "Request blood", "Volunteer", "Donate money" (post-v1), each leading to a guided flow.
- **Impact transparency** — public dashboard with key metrics (units collected, units issued, donors registered, active volunteers), updated in real time.
- **Trust signals** — registration details, partner hospitals, success stories (anonymized, consented), financial summary (post-v1).
- **No login required for essential information** — anyone can see what blood types are urgently needed (without revealing specific inventory counts), read success stories, and find contact information.

---

## 5. Roles & Access Control Overview

The platform implements **role-based access control (RBAC)** with seven defined roles. Each user has exactly one primary role; certain users (notably internal staff) MAY have additional secondary roles granted explicitly by a Super Admin. Role assignments are auditable and revocable.

### 5.1 Role Definitions

| Role Code | Role Name | Description | Typical Holder |
|---|---|---|---|
| `SUPER_ADMIN` | Super Admin | Unrestricted access to all data and configuration. Accountable owner of the platform. | Executive Director, Board-appointed tech lead |
| `ADMIN` | Admin | Full operational access to all modules except platform-wide configuration and user-role assignment for SUPER_ADMIN accounts. | Office staff, operations managers |
| `VOLUNTEER` | Volunteer | Self-service access to own profile, assigned tasks, and contribution history. Limited create access for donor registration at camps. | Active volunteers |
| `HOSPITAL` | Hospital | Submit and track blood requests on behalf of patients at their hospital. View inventory availability (aggregate, not bag-level). | Hospital blood bank staff |
| `DONOR` | Donor | Self-service access to own donor profile, donation history, eligibility status, and impact summary. | Registered donors |
| `PATIENT` | Patient | Submit and track blood requests (typically used by guardians on behalf of patients). | Patient guardians |
| `PUBLIC` | Public Visitor | Anonymous, unauthenticated. Browse public website, view aggregate impact metrics, initiate registration flows. | Anyone |

### 5.2 Role Capability Matrix (Summary)

The complete capability matrix is defined in the Technical Design Specification. The summary below conveys the principle: each role can access only what it needs, and nothing more.

| Capability | SUPER_ADMIN | ADMIN | VOLUNTEER | HOSPITAL | DONOR | PATIENT | PUBLIC |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| View own profile | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| View all donor records | ✓ | ✓ | — | — | — | — | — |
| View own donor record | ✓ | ✓ | — | — | ✓ | — | — |
| Create blood request | ✓ | ✓ | — | ✓ | — | ✓ | — |
| View all blood requests | ✓ | ✓ | — | (own only) | — | (own only) | — |
| View inventory (bag-level) | ✓ | ✓ | — | — | — | — | — |
| View inventory (aggregate) | ✓ | ✓ | — | ✓ | — | — | ✓ |
| Manage volunteers | ✓ | ✓ | (self only) | — | — | — | — |
| Configure platform | ✓ | — | — | — | — | — | — |
| View audit logs | ✓ | — | — | — | — | — | — |
| Manage user roles | ✓ | — | — | — | — | — | — |

### 5.3 Role Assignment Rules

1. **PUBLIC** is the default for all unauthenticated users. No assignment is required.
2. **DONOR**, **VOLUNTEER**, **PATIENT**, and **HOSPITAL** roles are self-assigned during the registration flow (with email verification).
3. **ADMIN** role is assigned only by a **SUPER_ADMIN**, with documented justification recorded in the audit log.
4. **SUPER_ADMIN** role is assigned only by an existing **SUPER_ADMIN**, requires Board approval (documented in the audit log), and is limited to a maximum of three concurrent SUPER_ADMIN accounts.
5. Role revocation is immediate and auditable. A revoked user's session MUST be invalidated within 60 seconds.
6. Role escalation (e.g., VOLUNTEER → ADMIN) follows the same rules as initial assignment to the target role.

### 5.4 Multi-Role Users

A user MAY hold multiple roles simultaneously (e.g., a staff member who is also a registered donor). The platform MUST:

- Allow the user to switch active role context from their profile.
- Apply the *most restrictive applicable permission* when contexts overlap (e.g., a user who is both ADMIN and DONOR cannot use their ADMIN privileges to modify their own DONOR record).
- Log every role context switch in the audit trail.

---

## 6. Functional Requirements — Module by Module

> **PHASE TAGS.** Each module section below begins with a **Phase** line indicating when the module first becomes available and when later phases extend it. Requirements within a module that are not in the module's first phase are tagged inline with their phase. The phase tags are: `P1` = Phase 1 Core MVP, `P2` = Phase 2 Operational Improvements, `P3` = Phase 3 Automation, `P4` = Phase 4 Scale. Requirements without an explicit tag belong to the module's first listed phase.

This section defines every module of the platform, the capabilities it MUST provide, and the acceptance criteria for each capability. Requirements are tagged with stable identifiers (e.g., `FR-DON-001`) for traceability through the architecture and TDS documents.

### 6.1 Module: Public Website (`WEB`)

The public website is the platform's front door. It serves the Anonymous Public Visitor persona primarily but is also the entry point for registration flows for donors, volunteers, patients, and hospitals.

#### 6.1.1 Functional Requirements

| ID | Requirement | Priority |
|---|---|---|
| FR-WEB-001 | The website MUST display the organization's mission, vision, and registration details on a publicly accessible About page. | MUST |
| FR-WEB-002 | The website MUST display a real-time aggregate impact dashboard (units collected, units issued, active donors, active volunteers) on the home page. | MUST |
| FR-WEB-003 | The website MUST provide a "Request Blood" call-to-action leading to a guided request submission flow. | MUST |
| FR-WEB-004 | The website MUST provide a "Donate Blood" call-to-action leading to donor registration. | MUST |
| FR-WEB-005 | The website MUST provide a "Volunteer" call-to-action leading to volunteer registration. | MUST |
| FR-WEB-006 | The website MUST publish consented, anonymized success stories submitted by recipients and reviewed by an admin. | MUST |
| FR-WEB-007 | The website MUST publish a publicly accessible list of partner hospitals. | MUST |
| FR-WEB-008 | The website MUST display currently urgently needed blood types (without revealing specific inventory counts). | MUST |
| FR-WEB-009 | The website MUST be fully accessible without JavaScript (progressive enhancement). | MUST |
| FR-WEB-010 | The website MUST load in under 3 seconds on a 3G connection measured by Lighthouse. | MUST |
| FR-WEB-011 | The website MUST support Urdu and English, with a clear language toggle. | MUST |
| FR-WEB-012 | The website MUST include a public Knowledge Base with articles on donation eligibility, what to expect, FAQs, and post-donation care. | SHOULD |
| FR-WEB-013 | The website MUST publish a public annual report (auto-generated, admin-approved) within 30 days of fiscal year close. | SHOULD |
| FR-WEB-014 | The website MUST expose an RSS feed for success stories and announcements. | MAY |

#### 6.1.2 Content Management

The website's content (with the exception of real-time data) is managed through a lightweight CMS integrated into the admin dashboard. Content types include:

- **Static pages** — About, Contact, Privacy Policy, Terms of Service, Knowledge Base articles.
- **Success stories** — narrative content with optional anonymized photograph, submitted by recipients or drafted by AI and reviewed by admin.
- **Announcements** — time-bound notices (blood camps, urgent appeals, system maintenance).
- **Knowledge base articles** — structured articles with categories, tags, and search.

### 6.2 Module: Admin Dashboard (`ADM`)

The admin dashboard is the primary interface for Super Admins and Admins. It is a desktop-first responsive web application organized around the operational workflow.

#### 6.2.1 Information Architecture

The dashboard sidebar is organized into the following sections, each corresponding to a module:

1. **Overview** — executive dashboard with KPI tiles and recent activity.
2. **Donors** — donor management.
3. **Volunteers** — volunteer management.
4. **Patients** — patient management.
5. **Requests** — blood request management.
6. **Inventory** — blood bag inventory.
7. **Deliveries** — delivery tracking.
8. **Hospitals** — hospital partner management.
9. **Camps** — blood camp management.
10. **Reports** — report generation and history.
11. **Analytics** — analytical views (cohort, trend, geographic).
12. **Knowledge Base** — content management.
13. **Notifications** — notification template and history.
14. **Automation** — n8n workflow status and history.
15. **Audit Log** — immutable activity log (SUPER_ADMIN only).
16. **Users** — user management and role assignment.
17. **Settings** — platform configuration.

#### 6.2.2 Functional Requirements

| ID | Requirement | Priority |
|---|---|---|
| FR-ADM-001 | The dashboard MUST display role-appropriate content based on the logged-in user's role. | MUST |
| FR-ADM-002 | Every list view MUST support search, filter, sort, and pagination. | MUST |
| FR-ADM-003 | Every detail view MUST show a complete activity timeline for the entity (created, updated, state transitions, related entities). | MUST |
| FR-ADM-004 | Every mutating action MUST require confirmation for irreversible operations (delete, discard, role change). | MUST |
| FR-ADM-005 | Every mutating action MUST produce an audit log entry with actor, timestamp, before-state, and after-state. | MUST |
| FR-ADM-006 | Bulk operations MUST be supported for: status updates, tag assignment, export, and (soft) delete. | MUST |
| FR-ADM-007 | The dashboard MUST support dark mode and light mode, persisted per user. | MUST |
| FR-ADM-008 | The dashboard MUST be fully keyboard-navigable (Tab, Shift+Tab, Enter, Escape, arrow keys for menus). | MUST |
| FR-ADM-009 | The dashboard MUST support saved filters and saved views per user. | SHOULD |
| FR-ADM-010 | The dashboard MUST support export of any list view to CSV and PDF. | MUST |
| FR-ADM-011 | The dashboard MUST display real-time updates via WebSocket (inventory changes, new requests, status transitions) without page refresh. | MUST |
| FR-ADM-012 | The dashboard MUST display a notification center with unread badge, populated by in-app notifications and system alerts. | MUST |

### 6.3 Module: Donor Management (`DON`)

#### 6.3.1 Donor Registration

| ID | Requirement | Priority |
|---|---|---|
| FR-DON-001 | A donor MUST be able to self-register via the public website or mobile app with email or phone verification. | MUST |
| FR-DON-002 | A donor MUST be able to register at a blood camp via a volunteer using the volunteer app. | MUST |
| FR-DON-003 | Registration MUST capture: full name, CNIC (Pakistani national ID) or passport, date of birth, gender, blood type, contact details, communication preferences, emergency contact. | MUST |
| FR-DON-004 | Registration MUST capture medical screening information per WHO blood donor selection guidelines. | MUST |
| FR-DON-005 | The platform MUST verify email and phone via OTP before activating the donor account. | MUST |
| FR-DON-006 | The platform MUST detect duplicate registrations (same CNIC, same phone, same email) and prevent creation of duplicate records. | MUST |
| FR-DON-007 | The platform MUST NOT allow a donor to register without consenting to the privacy policy and data processing terms. | MUST |

#### 6.3.2 Donation History & Eligibility

| ID | Requirement | Priority |
|---|---|---|
| FR-DON-010 | The platform MUST maintain a complete donation history per donor with date, location, bag ID, component donated, and outcome (successful, deferred, adverse reaction). | MUST |
| FR-DON-011 | The platform MUST calculate donor eligibility based on: last donation date (deferral window: 56 days whole blood, 16 weeks platelets per WHO), medical screening results, current medications, recent travel, recent tattoos/procedures. | MUST |
| FR-DON-012 | The platform MUST display the donor's next eligible date prominently in their profile. | MUST |
| FR-DON-013 | The platform MUST send reminders to the donor via their preferred channel: 7 days before eligible, on the eligible date, and 14 days after if no donation recorded. | MUST |
| FR-DON-014 | The platform MUST allow an admin to manually override eligibility (with documented justification in audit log). | MUST |
| FR-DON-015 | The platform MUST allow a donor to self-update their medical information, which MUST trigger re-evaluation of eligibility. | MUST |

#### 6.3.3 Donor Self-Service

| ID | Requirement | Priority |
|---|---|---|
| FR-DON-020 | A donor MUST be able to view their own profile, donation history, eligibility status, and impact summary. | MUST |
| FR-DON-021 | A donor MUST be able to update their contact details and communication preferences. | MUST |
| FR-DON-022 | A donor MUST be able to set an emergency availability status (available / unavailable / limited) for a specified time window. | MUST |
| FR-DON-023 | A donor MUST be able to view their donation impact (units donated, estimated lives impacted, hospitals served). | MUST |
| FR-DON-024 | A donor MUST be able to download their donation history as PDF. | SHOULD |
| FR-DON-025 | A donor MUST be able to delete their own account (subject to data retention requirements for active donation records). | MUST |

#### 6.3.4 Donor-Admin Functions

| ID | Requirement | Priority |
|---|---|---|
| FR-DON-030 | An admin MUST be able to view, search, filter, and export the donor database. | MUST |
| FR-DON-031 | An admin MUST be able to manually add a donation record for a donor (e.g., historical data migration). | MUST |
| FR-DON-032 | An admin MUST be able to mark a donor as deferred (temporary or permanent) with reason, which MUST suspend notifications and emergency availability. | MUST |
| FR-DON-033 | An admin MUST be able to broadcast a targeted message to filtered donor segments (e.g., "all B+ donors in Peshawar eligible this week"). | MUST |
| FR-DON-034 | The platform MUST generate an automatic appreciation message to a donor after each successful donation, via their preferred channel. | MUST |
| FR-DON-035 | The platform MUST generate an annual impact summary per donor (AI-drafted, admin-approved) sent via their preferred channel. | SHOULD |

### 6.4 Module: Volunteer Management (`VOL`)

| ID | Requirement | Priority |
|---|---|---|
| FR-VOL-001 | A volunteer MUST be able to self-register with skills, availability calendar, geographic area, and preferred task types. | MUST |
| FR-VOL-002 | An admin MUST be able to create and assign tasks to volunteers matching the task requirements. | MUST |
| FR-VOL-003 | A volunteer MUST be able to accept, decline, or request reassignment of an assigned task. | MUST |
| FR-VOL-004 | The platform MUST send task reminders via the volunteer's preferred channel: 24 hours before, 2 hours before, and at task start time. | MUST |
| FR-VOL-005 | A volunteer MUST be able to check in and check out of a task to record actual hours. | MUST |
| FR-VOL-006 | A volunteer MUST be able to view their contribution history (tasks completed, hours, camps attended, donors registered). | MUST |
| FR-VOL-007 | The platform MUST issue certificates (AI-drafted, admin-approved) for milestones: 10 tasks, 50 hours, 100 donors registered, 1 year of service. | SHOULD |
| FR-VOL-008 | An admin MUST be able to mark a volunteer as inactive, suspending task assignments and notifications. | MUST |
| FR-VOL-009 | The platform MUST send a re-engagement message to volunteers inactive for 30+ days. | SHOULD |
| FR-VOL-010 | A volunteer MUST be able to register a new donor at a blood camp via the volunteer app, with the donor's OTP verification. | MUST |
| FR-VOL-011 | A volunteer MUST be able to record donation outcomes at a blood camp (successful donation, deferral reason, adverse reaction). | MUST |

### 6.5 Module: Patient Management (`PAT`)

| ID | Requirement | Priority |
|---|---|---|
| FR-PAT-001 | A patient record MUST be created when a blood request is first submitted, with patient name, age, gender, blood type, disease/condition, treating hospital, treating doctor, and guardian contact. | MUST |
| FR-PAT-002 | A guardian MUST be able to submit a blood request on behalf of a patient without creating a full patient profile (minimal intake, expandable later). | MUST |
| FR-PAT-003 | The platform MUST support upload of patient documents (doctor's prescription, hospital letter, lab reports) with virus scanning and access control. | MUST |
| FR-PAT-004 | An admin MUST be able to view, verify, and update patient records. | MUST |
| FR-PAT-005 | The platform MUST link all blood requests, units issued, and deliveries to the patient record for a complete case history. | MUST |
| FR-PAT-006 | The platform MUST anonymize patient identity in all public-facing content (success stories, impact metrics) unless explicit written consent is recorded. | MUST |
| FR-PAT-007 | A patient (or guardian) MUST be able to view their own case history and request status. | MUST |
| FR-PAT-008 | The platform MUST support soft-delete of patient records after the retention period (default 7 years per Pakistan DP requirements) with audit trail preserved. | MUST |

### 6.6 Module: Blood Inventory (`INV`)

This is the operational heart of the platform. Every blood bag has a unique identity, a state machine, and a chain-of-custody log.

#### 6.6.1 Bag Lifecycle

A blood bag moves through the following states. The platform enforces legal state transitions; illegal transitions are rejected.

```
COLLECTED → SCREENED → STORED → RESERVED → ISSUED → DELIVERED → RETURNED (if unused)
                                           ↓
                                        EXPIRED
                                           ↓
                                        DISCARDED
```

#### 6.6.2 Functional Requirements

| ID | Requirement | Priority |
|---|---|---|
| FR-INV-001 | Every blood bag MUST have a unique ID (format: `MWS-YYYY-NNNNNN`) generated by the platform at collection time. | MUST |
| FR-INV-002 | Every blood bag MUST have a QR code label printed at collection time, linking to a verification page accessible by hospital staff. | MUST |
| FR-INV-003 | Every blood bag MUST record: collection date/time, donor, collector (volunteer/staff), blood type, component type, volume, expiry date, storage location. | MUST |
| FR-INV-004 | The platform MUST calculate expiry date automatically based on component type (whole blood: 35 days, platelets: 5 days, plasma: 365 days, cryoprecipitate: 365 days). | MUST |
| FR-INV-005 | Every state transition MUST be logged with actor, timestamp, location, and reason. | MUST |
| FR-INV-006 | The platform MUST enforce legal state transitions and reject illegal ones with a descriptive error. | MUST |
| FR-INV-007 | The platform MUST display inventory by blood type, component, storage location, and status. | MUST |
| FR-INV-008 | The platform MUST send low-inventory alerts (per blood type) when stock drops below configurable threshold. | MUST |
| FR-INV-009 | The platform MUST send expiry alerts for bags nearing expiry (default: 7 days for whole blood, 24 hours for platelets). | MUST |
| FR-INV-010 | An admin MUST be able to manually adjust inventory (with documented justification in audit log). | MUST |
| FR-INV-011 | The platform MUST support inventory transfers between storage locations. | SHOULD |
| FR-INV-012 | The platform MUST support batch operations: bulk check-in at camps, bulk status updates. | MUST |
| FR-INV-013 | The platform MUST generate an inventory reconciliation report comparing system state to physical count. | MUST |
| FR-INV-014 | The platform MUST prevent issuance of expired or untested bags. | MUST |

### 6.7 Module: Blood Requests (`REQ`)

#### 6.7.1 Request Lifecycle

```
SUBMITTED → VERIFIED → MATCHING → ALLOCATED → DISPATCHED → DELIVERED → RECEIVED → CLOSED
                ↓           ↓           ↓
            REJECTED    CANCELLED  CANCELLED
```

#### 6.7.2 Functional Requirements

| ID | Requirement | Priority |
|---|---|---|
| FR-REQ-001 | A blood request MUST capture: requester (hospital or guardian), patient, blood type, component, quantity, urgency (routine/urgent/emergency), clinical justification, required-by date. | MUST |
| FR-REQ-002 | The platform MUST validate blood type compatibility (donor/recipient) when matching inventory to requests. | MUST |
| FR-REQ-003 | The platform MUST auto-allocate available inventory to verified urgent/emergency requests, subject to admin approval. | MUST |
| FR-REQ-004 | The platform MUST trigger targeted donor outreach when demand cannot be met from current inventory. | MUST |
| FR-REQ-005 | The platform MUST enforce SLA commitments per urgency level (emergency: 30 minutes, urgent: 4 hours, routine: 24 hours). | MUST |
| FR-REQ-006 | The platform MUST escalate requests breaching SLA to senior admins automatically. | MUST |
| FR-REQ-007 | A requester MUST be able to track request status in real time. | MUST |
| FR-REQ-008 | A requester MUST be able to cancel a request before dispatch. | MUST |
| FR-REQ-009 | An admin MUST be able to override auto-allocation (with audit). | MUST |
| FR-REQ-010 | The platform MUST record the chain-of-custody from request to delivery to receipt. | MUST |
| FR-REQ-011 | The platform MUST support request splitting (partial fulfillment from multiple bags). | MUST |
| FR-REQ-012 | The platform MUST support request linking (related requests for the same patient). | SHOULD |

### 6.8 Module: Blood Delivery (`DLV`)

| ID | Requirement | Priority |
|---|---|---|
| FR-DLV-001 | The platform MUST support dispatch records linking issued bags to a delivery (courier, vehicle, dispatch time, expected arrival). | MUST |
| FR-DLV-002 | The platform MUST support delivery status updates: dispatched, in-transit, delivered, returned. | MUST |
| FR-DLV-003 | The platform MUST notify the requester and patient/guardian at each delivery status change. | MUST |
| FR-DLV-004 | The platform MUST record delivery confirmation (signature or OTP from recipient). | MUST |
| FR-DLV-005 | The platform MUST support cold-chain compliance logging (temperature readings during transit). | SHOULD |
| FR-DLV-006 | The platform MUST support return of unused units with condition assessment. | MUST |
| FR-DLV-007 | The platform MUST generate delivery manifests for courier handoff. | MUST |

### 6.9 Module: Hospital Portal (`HSP`)

| ID | Requirement | Priority |
|---|---|---|
| FR-HSP-001 | A hospital MUST be able to register with verification (admin approval required). | MUST |
| FR-HSP-002 | A hospital MUST be able to manage multiple staff accounts under one hospital profile. | MUST |
| FR-HSP-003 | A hospital MUST be able to view aggregate inventory availability by blood type (not bag-level). | MUST |
| FR-HSP-004 | A hospital MUST be able to submit, track, and manage their own blood requests. | MUST |
| FR-HSP-005 | A hospital MUST be able to view their request history and delivery records. | MUST |
| FR-HSP-006 | A hospital MUST be able to download compliance records (units received, dates, batch numbers) for their internal audit. | MUST |
| FR-HSP-007 | The platform MUST enforce that a hospital can only see its own requests and deliveries, never another hospital's. | MUST |
| FR-HSP-008 | An admin MUST be able to suspend or terminate a hospital partnership (with audit and notification). | MUST |

### 6.10 Module: Blood Camps (`CMP`)

| ID | Requirement | Priority |
|---|---|---|
| FR-CMP-001 | An admin MUST be able to schedule a blood camp with date, time, location, target collection, assigned volunteers, and partner institution. | MUST |
| FR-CMP-002 | The platform MUST publish upcoming camps on the public website with registration link. | MUST |
| FR-CMP-003 | The platform MUST support pre-registration of donors for a camp (with reminder notifications). | SHOULD |
| FR-CMP-004 | A volunteer MUST be able to check in donors at a camp via QR code scanning of the donor's app. | MUST |
| FR-CMP-005 | A volunteer MUST be able to record donation outcomes at a camp in real time, updating inventory as donations complete. | MUST |
| FR-CMP-006 | The platform MUST generate a camp report (collections, deferrals, adverse reactions, volunteer hours, donor demographics) auto-generated at camp close. | MUST |
| FR-CMP-007 | The platform MUST support post-camp follow-up: thank-you messages to donors, certificates to volunteers, impact summary to partner institution. | MUST |

### 6.11 Module: Reports (`RPT`)

| ID | Requirement | Priority |
|---|---|---|
| FR-RPT-001 | The platform MUST auto-generate a daily operations digest (collections, issues, expirations, pending requests, low inventory) delivered to admins by 08:00 PKT daily. | MUST |
| FR-RPT-002 | The platform MUST auto-generate a monthly impact report (collections, donors, volunteers, hospitals served, expiry rate, financial summary post-v1) by the 5th of the following month. | MUST |
| FR-RPT-003 | The platform MUST auto-generate an annual report by January 31st of the following year. | MUST |
| FR-RPT-004 | An admin MUST be able to generate ad-hoc reports with custom date ranges, filters, and groupings. | MUST |
| FR-RPT-005 | All reports MUST be exportable to PDF, CSV, and Excel. | MUST |
| FR-RPT-006 | All reports MUST be reviewable and editable by an admin before publication. | MUST |
| FR-RPT-007 | Published monthly and annual reports MUST be publicly accessible on the website (with sensitive data redacted). | SHOULD |
| FR-RPT-008 | AI MUST assist in drafting narrative content for reports (executive summary, success stories, key findings) subject to admin review and approval. | MUST |

### 6.12 Module: Analytics (`ANL`)

| ID | Requirement | Priority |
|---|---|---|
| FR-ANL-001 | The platform MUST provide a donor analytics view: acquisition trends, retention cohorts, donation frequency distribution, geographic distribution. | MUST |
| FR-ANL-002 | The platform MUST provide an inventory analytics view: collection vs. issuance trends, expiry rate by component, inventory turnover. | MUST |
| FR-ANL-003 | The platform MUST provide a request analytics view: volume trends, fulfillment SLA performance, urgency distribution, hospital demand. | MUST |
| FR-ANL-004 | The platform MUST provide a volunteer analytics view: active volunteers, hours contributed, task completion rate, retention. | MUST |
| FR-ANL-005 | The platform MUST provide a geographic analytics view: collections, requests, and donors mapped by district. | SHOULD |
| FR-ANL-006 | All analytics views MUST support date range, filter, and comparison (vs. previous period). | MUST |
| FR-ANL-007 | Analytics MUST be computed from the operational data in real time (no separate data warehouse for v1). | MUST |

### 6.13 Module: Notifications (`NOT`)

| ID | Requirement | Priority |
|---|---|---|
| FR-NOT-001 | The platform MUST support notification delivery via: in-app, email, SMS (paid, opt-in), WhatsApp (free tier via n8n), and push notification (mobile app). | MUST |
| FR-NOT-002 | A user MUST be able to configure channel preferences per notification category. | MUST |
| FR-NOT-003 | The platform MUST use templates for all notifications, manageable by admins. | MUST |
| FR-NOT-004 | Templates MUST support personalization variables (donor name, blood type, date, etc.). | MUST |
| FR-NOT-005 | The platform MUST log every notification sent (recipient, channel, template, content, timestamp, delivery status). | MUST |
| FR-NOT-006 | The platform MUST retry failed notifications with exponential backoff (max 3 retries). | MUST |
| FR-NOT-007 | The platform MUST respect quiet hours (configurable per user, default 22:00–07:00 PKT) for non-emergency notifications. | MUST |
| FR-NOT-008 | Emergency notifications MUST bypass quiet hours. | MUST |

### 6.14 Module: Automation (`AUT`)

| ID | Requirement | Priority |
|---|---|---|
| FR-AUT-001 | The platform MUST integrate with n8n for workflow automation, with workflows version-controlled in the repository. | MUST |
| FR-AUT-002 | Every workflow MUST be idempotent (safe to re-run) and observable (status, logs, error tracking). | MUST |
| FR-AUT-003 | Every workflow MUST have a documented trigger, input schema, output schema, and failure handling. | MUST |
| FR-AUT-004 | The platform MUST provide a webhook layer for n8n to trigger platform actions securely. | MUST |
| FR-AUT-005 | An admin MUST be able to enable, disable, and manually trigger workflows from the dashboard. | MUST |
| FR-AUT-006 | Failed workflows MUST be retried with exponential backoff, and persistent failures MUST alert an admin. | MUST |
| FR-AUT-007 | The platform MUST support workflowdry-run / preview before live execution. | SHOULD |

### 6.15 Module: Knowledge Base (`KB`)

| ID | Requirement | Priority |
|---|---|---|
| FR-KB-001 | The platform MUST include a structured knowledge base with categories, tags, and search. | MUST |
| FR-KB-002 | Knowledge base articles MUST be authored in Markdown with rich media support. | MUST |
| FR-KB-003 | An admin MUST be able to publish, unpublish, and version articles. | MUST |
| FR-KB-004 | The knowledge base MUST be publicly accessible without login. | MUST |
| FR-KB-005 | The knowledge base MUST include at minimum: donor eligibility guide, donation process, post-donation care, blood type compatibility chart, FAQs. | MUST |

### 6.16 Module: Settings (`SET`)

| ID | Requirement | Priority |
|---|---|---|
| FR-SET-001 | An admin (SUPER_ADMIN only for security-sensitive settings) MUST be able to configure: deferral windows, SLA thresholds, inventory alert thresholds, notification templates, quiet hours, retention periods. | MUST |
| FR-SET-002 | All settings MUST be versioned with audit trail (who changed what, when, previous value). | MUST |
| FR-SET-003 | Settings changes MUST NOT require code deployment. | MUST |
| FR-SET-004 | Settings MUST be exportable and importable (for backup and migration). | MUST |
| FR-SET-005 | Sensitive settings (encryption keys, third-party API tokens) MUST be stored in environment variables, never in the database. | MUST |

### 6.17 Module: User Management (`USR`)

| ID | Requirement | Priority |
|---|---|---|
| FR-USR-001 | A SUPER_ADMIN MUST be able to create, deactivate, and reactivate user accounts. | MUST |
| FR-USR-002 | A SUPER_ADMIN MUST be able to assign and revoke roles. | MUST |
| FR-USR-003 | The platform MUST enforce password complexity (min 12 chars, mix of upper, lower, digit, symbol) and rotation policies. | MUST |
| FR-USR-004 | The platform MUST support multi-factor authentication (TOTP) for ADMIN and SUPER_ADMIN roles. | MUST |
| FR-USR-005 | The platform MUST enforce session timeout (configurable, default 8 hours for admin roles, 30 days for donor/patient with refresh). | MUST |
| FR-USR-006 | The platform MUST log all authentication events (login, logout, failed attempt, MFA challenge, password reset). | MUST |
| FR-USR-007 | The platform MUST lock accounts after 5 failed login attempts, with unlock by SUPER_ADMIN only. | MUST |
| FR-USR-008 | The platform MUST support password reset via email verification with expiring tokens. | MUST |

---

## 7. Non-Functional Requirements

Non-functional requirements (NFRs) define *how well* the system must behave, independent of what it does. NFRs are testable, measurable, and treated with the same rigor as functional requirements.

### 7.1 Performance

| ID | Requirement | Target | Measurement |
|---|---|---|---|
| NFR-PER-001 | Public website First Contentful Paint | < 1.5s on 4G, < 3s on 3G | Lighthouse, CI-gated |
| NFR-PER-002 | Public website Time to Interactive | < 3s on 4G, < 5s on 3G | Lighthouse, CI-gated |
| NFR-PER-003 | Admin dashboard initial load | < 2s on broadband | Lighthouse |
| NFR-PER-004 | API response time (read, p95) | < 300ms | APM, alerting |
| NFR-PER-005 | API response time (write, p95) | < 500ms | APM, alerting |
| NFR-PER-006 | Database query time (p99) | < 100ms | pg_stat_statements |
| NFR-PER-007 | Real-time notification delivery | < 30s end-to-end | E2E test |
| NFR-PER-008 | Mobile app cold start | < 3s on mid-range Android | Manual + automated |
| NFR-PER-009 | Concurrent user support (v1) | 500 concurrent without degradation | Load test |
| NFR-PER-010 | Search response (across donors/requests) | < 500ms | E2E test |

### 7.2 Scalability

| ID | Requirement |
|---|---|
| NFR-SCL-001 | The platform MUST scale horizontally without code changes; vertical scaling is acceptable for v1. |
| NFR-SCL-002 | The database MUST support 100,000 donor records, 1,000,000 donation records, and 500,000 blood bag records without performance degradation. |
| NFR-SCL-003 | Read replicas SHOULD be supported via Supabase read replica (available on free tier as of design date). |
| NFR-SCL-004 | Background job throughput MUST handle 10,000 notifications per hour without backlog. |
| NFR-SCL-005 | The platform MUST NOT have architectural bottlenecks that prevent 10× growth. |

### 7.3 Availability & Reliability

| ID | Requirement |
|---|---|
| NFR-AVL-001 | The platform MUST target 99.5% uptime in v1 (Supabase free tier SLA). |
| NFR-AVL-002 | The platform MUST degrade gracefully — read operations continue during write outages (cached reads). |
| NFR-AVL-003 | The platform MUST have a documented disaster recovery plan with RTO < 4 hours and RPO < 1 hour. |
| NFR-AVL-004 | Database backups MUST be automated daily with 30-day retention. |
| NFR-AVL-005 | The platform MUST have health check endpoints monitored externally. |
| NFR-AVL-006 | The platform MUST support maintenance mode with public-facing notice. |

### 7.4 Security

Detailed security requirements are in the System Architecture Document. The NFR summary:

| ID | Requirement |
|---|---|
| NFR-SEC-001 | All PHI MUST be encrypted at rest (database-level + column-level for sensitive fields). |
| NFR-SEC-002 | All transport MUST be TLS 1.2+ (TLS 1.3 preferred). |
| NFR-SEC-003 | All passwords MUST be hashed with Argon2id via Supabase Auth. |
| NFR-SEC-004 | All API endpoints MUST require authentication except a documented public allowlist. |
| NFR-SEC-005 | All mutating API endpoints MUST implement CSRF protection. |
| NFR-SEC-006 | All file uploads MUST be virus-scanned (ClamAV via n8n or Supabase Edge Function). |
| NFR-SEC-007 | All inputs MUST be validated server-side (Zod schemas) before database writes. |
| NFR-SEC-008 | All outputs MUST be sanitized to prevent XSS (no raw HTML rendering without sanitization). |
| NFR-SEC-009 | Row Level Security MUST be enabled on every table containing user data. |
| NFR-SEC-010 | Rate limiting MUST be applied: 100 req/min per IP for public, 1000 req/min per user for authenticated. |
| NFR-SEC-011 | Security headers MUST be set: CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy. |
| NFR-SEC-012 | Audit logs MUST be append-only and retained for 7 years. |

### 7.5 Accessibility

| ID | Requirement |
|---|---|
| NFR-ACC-001 | The platform MUST conform to WCAG 2.1 AA. |
| NFR-ACC-002 | All interactive elements MUST be keyboard accessible. |
| NFR-ACC-003 | All images MUST have meaningful alt text or be marked decorative. |
| NFR-ACC-004 | Color contrast MUST meet WCAG AA ratios (4.5:1 for body text, 3:1 for large text). |
| NFR-ACC-005 | Forms MUST have associated labels, error messages, and instructions. |
| NFR-ACC-006 | The mobile app MUST support dynamic font sizing and screen readers (TalkBack, VoiceOver). |

### 7.6 Internationalization & Localization

| ID | Requirement |
|---|---|
| NFR-I18N-001 | All user-facing strings MUST be externalized (i18n framework). |
| NFR-I18N-002 | The platform MUST support Urdu (ur-PK) and English (en-PK) at launch. |
| NFR-I18N-003 | The platform MUST be designed to add more languages without code changes. |
| NFR-I18N-004 | Date, time, and number formatting MUST be locale-aware. |
| NFR-I18N-005 | RTL layout MUST be supported (for future Arabic addition). |

### 7.7 Maintainability

| ID | Requirement |
|---|---|
| NFR-MNT-001 | The codebase MUST have ≥ 80% test coverage for business logic. |
| NFR-MNT-002 | All public APIs MUST have OpenAPI documentation. |
| NFR-MNT-003 | All database changes MUST be via versioned migrations. |
| NFR-MNT-004 | The codebase MUST pass ESLint, Prettier, and TypeScript strict mode without errors. |
| NFR-MNT-005 | The architecture MUST allow replacement of any single dependency (Supabase, Vercel, n8n) without rewriting the application. |
| NFR-MNT-006 | Every module MUST have a README documenting its purpose, interfaces, and dependencies. |

### 7.8 Observability

| ID | Requirement |
|---|---|
| NFR-OBS-001 | The platform MUST emit structured logs (JSON) with correlation IDs. |
| NFR-OBS-002 | The platform MUST expose Prometheus-compatible metrics (or equivalent). |
| NFR-OBS-003 | The platform MUST have error tracking (Sentry free tier or equivalent). |
| NFR-OBS-004 | The platform MUST have uptime monitoring (UptimeRobot or equivalent). |
| NFR-OBS-005 | Admins MUST have visibility into system health from the dashboard (no developer-only access). |

### 7.9 Cost

| ID | Requirement |
|---|---|
| NFR-CST-001 | The platform MUST operate at zero cost at the free tier of its constituent services for the first 12 months or until 10,000 registered users, whichever comes first. |
| NFR-CST-002 | The platform MUST have a documented cost projection for growth beyond free tier. |
| NFR-CST-003 | Paid services (e.g., SMS gateway) MUST be opt-in only, with explicit user consent for any charge. |
| NFR-CST-004 | The platform MUST NOT use any paid third-party API for v1. |

---

## 8. User Stories & Acceptance Criteria

This section translates the functional requirements into user-story format with concrete acceptance criteria, suitable for direct use in implementation prompts and QA testing. Stories are tagged with stable IDs (`US-XXX-NNN`).

### 8.1 Donor Stories

**US-DON-001: First-time donor registration**

> As a potential donor, I want to register on the platform so that I can be contacted when my blood type is needed.

Acceptance criteria:
- Given I am on the public website, when I click "Donate Blood", then I am taken to a guided registration flow.
- The form requires: full name, CNIC/passport, date of birth, gender, blood type, phone, email, password, address, emergency contact.
- The form includes medical screening questions per WHO guidelines.
- I must consent to the privacy policy and data processing terms.
- After submission, I receive an OTP via SMS and email; my account is activated only after OTP verification.
- If my CNIC or phone matches an existing record, I am informed and offered a password reset flow instead of creating a duplicate.
- After activation, I am taken to my donor dashboard with a welcome message.

**US-DON-002: Donation eligibility check**

> As a registered donor, I want to know when I am next eligible to donate so that I can plan my donation.

Acceptance criteria:
- My donor dashboard displays "Next eligible: DD MMM YYYY" prominently.
- If I am currently eligible, it displays "You are eligible to donate now" with a "Find a camp" CTA.
- If I am deferred (medical reason), it displays the deferral reason and projected eligible date.
- The calculation is based on my last donation date + deferral window, adjusted for medical screening responses.
- If I update my medical information, the eligibility is recalculated and the new date is displayed.

### 8.2 Patient/Guardian Stories

**US-PAT-001: Submit a blood request**

> As a patient's guardian, I want to submit a blood request so that the organization can find blood for my family member.

Acceptance criteria:
- I can access the request form via the public website "Request Blood" CTA without logging in (account created as part of submission).
- The form is presented in Urdu by default (toggle to English available) with large text and clear iconography.
- The form captures: patient name, age, gender, blood type, hospital, doctor's name, disease/condition, units needed, urgency, required-by date, contact, and document upload (prescription/hospital letter).
- I can upload a photo of the prescription from my phone camera.
- After submission, I receive a request ID and a tracking link via SMS.
- The request appears in my dashboard (account auto-created) with status "Submitted".
- I receive a notification within 30 minutes confirming the request was received and providing an estimated response time.

### 8.3 Hospital Stories

**US-HSP-001: Check inventory and submit a request**

> As a hospital blood bank technician, I want to check available inventory and submit a request so that I can plan my hospital's blood supply.

Acceptance criteria:
- After logging into the hospital portal, I see an aggregate inventory view (units available by blood type and component) updated in real time.
- I cannot see bag-level details (donor, collection date) — only counts.
- I can submit a request specifying blood type, component, quantity, urgency, patient details, clinical justification.
- The system confirms receipt and displays a request ID with SLA commitment.
- I can view all my hospital's historical requests and deliveries.
- I cannot see another hospital's requests or deliveries.

### 8.4 Volunteer Stories

**US-VOL-001: Accept and complete a task**

> As a volunteer, I want to accept an assigned task and check in/out so that my contribution is recorded.

Acceptance criteria:
- I receive a push notification when a task matching my profile is assigned.
- In the volunteer app, I see task details: type, location, date/time, briefing, contact person.
- I can accept, decline, or request reassignment.
- On the task day, I receive reminders 24h, 2h, and at start time.
- I can check in via the app (location-verified) and check out at completion.
- My actual hours are recorded in my contribution history.
- After completion, I receive a thank-you message and updated impact summary.

### 8.5 Admin Stories

**US-ADM-001: Handle an emergency blood request**

> As an admin, I want to be alerted to an emergency blood request and fulfill it within SLA so that the patient receives blood in time.

Acceptance criteria:
- When a request with urgency "emergency" is submitted, I receive an immediate in-app + push notification.
- The request appears at the top of my dashboard with a countdown timer to SLA breach.
- The system suggests available matching inventory and eligible donors.
- I can allocate inventory with one click, triggering the dispatch workflow.
- If allocation is not possible from inventory, the system triggers targeted donor outreach automatically.
- The requester is notified at each status change.
- If SLA is at risk (50% elapsed), the system escalates to a senior admin.
- After fulfillment, the case is recorded with full chain-of-custody.

### 8.6 Public Visitor Stories

**US-PUB-001: Find urgent blood needs**

> As a public visitor, I want to see which blood types are urgently needed so that I can donate if I am eligible.

Acceptance criteria:
- The public website homepage displays a "Blood Urgently Needed" panel listing blood types below threshold.
- I do not need to log in to see this.
- The panel does not reveal specific unit counts (only "urgently needed" / "needed" / "sufficient").
- Clicking a blood type takes me to the donor registration flow with the blood type pre-filled.

---

## 9. Automation Requirements

> **PHASE NOTE.** All automation is **Phase 3 (Automation)**. The Phase 1 and Phase 2 platform MUST function correctly with **zero automation running**. The application uses the **Service Interface pattern** (see Architecture §10) so that Phase 1 implements notifications and reports with plain TypeScript services, and Phase 3 swaps those services for n8n-backed adapters without changing call sites. **n8n is never a hard dependency.** If n8n is unavailable or disabled, the application continues to work using the in-process service implementations.

Automation is a first-class concern of this platform, but it is **optional and pluggable**, not foundational. The platform uses n8n (self-hosted, added in Phase 3) as its automation engine, with workflows triggered by database webhooks, schedules, or external events. The complete automation architecture is in the System Architecture Document §10; this section defines the *requirements*.

### 9.1 Automation Principles

1. **Automations are version-controlled.** Every n8n workflow is exported as JSON and committed to the repository under `automations/n8n/`. Changes go through normal PR review (founder self-merge is acceptable during solo-founder stage).
2. **Automations are observable.** Every workflow execution is logged with status, duration, and errors visible in the admin dashboard.
3. **Automations are idempotent.** Re-running a workflow with the same input produces the same result without side effects.
4. **Automations fail safe.** A failed automation does not corrupt data; it alerts an admin and leaves the system in a consistent state.
5. **Automations do not make medical decisions.** They may compute eligibility, send reminders, and generate reports, but never classify medical conditions or recommend treatment.
6. **Automations respect user preferences.** Notification automations respect channel preferences, quiet hours, and opt-outs.
7. **Automations are replaceable.** Any n8n workflow can be replaced with an in-process Next.js service (or vice versa) without changing the calling code, because the calling code goes through a `NotificationService` / `ReportService` interface.
8. **The core application never imports n8n.** Only the adapter implementations in `infrastructure/automation/n8n/` reference n8n; feature code calls interfaces.

### 9.2 Required Automations

All automations below are **Phase 3** unless explicitly noted. In Phase 1 and Phase 2, the equivalent behavior is implemented as an in-process service call (e.g., a direct `EmailService.send()` call after a donation is recorded). Phase 3 swaps that call for an n8n workflow behind the same `NotificationService` interface.

| ID | Automation | Trigger | Frequency | Phase |
|---|---|---|---|---|
| AUT-001 | Daily operations digest | Schedule (08:00 PKT) | Daily | P3 |
| AUT-002 | Donor eligibility reminder | Schedule (09:00 PKT) | Daily | P3 (basic in-process version P2) |
| AUT-003 | Donor post-donation thank-you | DB webhook (donation recorded) | Event | P3 (in-process version P1) |
| AUT-004 | Low inventory alert | DB webhook (inventory change) | Event | P3 (dashboard indicator P1) |
| AUT-005 | Blood expiry alert | Schedule (every 6 hours) | 4× daily | P3 (dashboard indicator P1) |
| AUT-006 | Volunteer task reminder | Schedule (every hour) | Hourly | P3 |
| AUT-007 | Volunteer re-engagement | Schedule (weekly) | Weekly | P3 |
| AUT-008 | Monthly impact report | Schedule (1st of month, 00:30 PKT) | Monthly | P3 (manual report P1) |
| AUT-009 | Annual report | Schedule (Jan 15) | Annual | P3 (manual report P1) |
| AUT-010 | Backup verification | Schedule (daily after backup) | Daily | P3 (manual backup+verify P1) |
| AUT-011 | Social media content generation | Schedule (Mon/Wed/Fri 10:00 PKT) | 3× weekly | P3 |
| AUT-012 | Daily motivational post | Schedule (07:00 PKT) | Daily | P3 |
| AUT-013 | Emergency donor outreach | DB webhook (request unmatched) | Event | P3 (manual outreach P1) |
| AUT-014 | Request status notifications | DB webhook (request state change) | Event | P3 (in-app notification P2) |
| AUT-015 | Delivery status notifications | DB webhook (delivery state change) | Event | P3 |
| AUT-016 | Failed login monitoring | DB webhook (auth event) | Event | P3 |
| AUT-017 | Audit log archival | Schedule (monthly) | Monthly | P3 |
| AUT-018 | Backup to off-site storage | Schedule (weekly) | Weekly | P3 (local backup P1) |
| AUT-019 | Camp follow-up (donors + volunteers) | DB webhook (camp closed) | Event | P3 |
| AUT-020 | Certificate generation | DB webhook (milestone reached) | Event | P3 |

### 9.3 AI-Assisted Content Automations

> **PHASE NOTE.** All AI-assisted automations are **Phase 3+**. The platform MUST be fully functional in Phase 1 and Phase 2 with **zero AI involvement**.

The platform uses AI (via n8n-hosted LLM API or self-hosted model, subject to free-tier constraints) for content generation only, never for medical or operational decisions. AI-generated content is **always** reviewed and approved by a human admin before publication or sending.

| ID | Automation | AI Task | Human Review | Phase |
|---|---|---|---|---|
| AUT-AI-001 | Daily motivational post | Draft post from theme + recent activity | Admin approves before publish | P3 |
| AUT-AI-002 | Social media caption | Draft caption from inventory need / event | Admin approves before publish | P3 |
| AUT-AI-003 | Success story narrative | Draft narrative from anonymized case data | Admin + recipient consent before publish | P3 |
| AUT-AI-004 | Appreciation message | Draft personalized message to donor | Admin approves before send | P3 |
| AUT-AI-005 | Volunteer certificate | Draft certificate text for milestone | Admin approves before issue | P3 |
| AUT-AI-006 | Monthly report narrative | Draft executive summary + key findings | Admin approves before publish | P3 |
| AUT-AI-007 | Annual report narrative | Draft full annual report narrative | Board approves before publish | P4 |
| AUT-AI-008 | Statistics summary | Draft plain-language summary of monthly stats | Admin approves before publish | P3 |

---

## 10. AI Assistance Boundaries

> **PHASE NOTE.** All AI features are **Phase 3+**. The platform in Phase 1 and Phase 2 has **no AI integration whatsoever**. The boundaries below govern AI use from Phase 3 onward; they do not retroactively constrain Phase 1–2 development.

The platform uses AI as an assistant, never as an autonomous decision-maker. This section defines, in unambiguous terms, what AI MAY and MUST NOT do. These boundaries are non-negotiable and apply to every AI integration past, present, and future.

### 10.1 What AI MAY Do

1. **Generate draft content** for human review — reports, captions, success stories, appreciation messages, certificates, statistics summaries.
2. **Summarize existing content** — e.g., summarize a month's activity into a paragraph for a report.
3. **Translate content** between Urdu and English (subject to human review for accuracy and tone).
4. **Suggest categorizations** — e.g., suggest categories for a knowledge base article (subject to admin confirmation).
5. **Generate alt text** for uploaded images (subject to admin review for accuracy).
6. **Power search** — semantic search over knowledge base and FAQ content.

### 10.2 What AI MUST NOT Do

1. **Make medical decisions.** AI MUST NOT classify medical conditions, recommend treatments, evaluate donor eligibility on medical grounds (eligibility is rule-based per WHO guidelines), or interpret lab results.
2. **Make operational decisions.** AI MUST NOT allocate blood bags, prioritize requests, assign volunteers, or modify inventory. These are rule-based or human decisions.
3. **Send communications autonomously.** AI-generated content MUST be reviewed and approved by a human before sending or publishing. The platform MUST NOT have an "auto-send AI content" mode.
4. **Access PHI beyond what is necessary.** AI content generation MUST use anonymized data; recipient names, hospital names, and identifying details MUST be redacted before AI processing.
5. **Make financial decisions.** AI MUST NOT authorize payments, manage budgets, or interact with financial systems (post-v1 finance module).
6. **Modify access controls.** AI MUST NOT grant or revoke roles, permissions, or access.
7. **Be the sole source of truth.** AI-generated content is always a draft. The system MUST clearly mark AI-generated content with `ai_generated: true` and `reviewed: false` until human review.

### 10.3 AI Provenance & Audit

- Every AI-generated artifact MUST be stored with: prompt, model identifier, timestamp, generating workflow ID, and reviewer identity.
- AI-generated content MUST be watermarked in the audit trail (not necessarily visible to end users).
- The platform MUST publish, in its knowledge base, a clear explanation of where and how AI is used, in plain language accessible to non-technical users.

---

## 11. Data & Compliance Requirements

### 11.1 Applicable Regulations

The platform operates in Pakistan and handles sensitive personal and health data. The following regulations apply:

| Regulation | Scope | Platform Response |
|---|---|---|
| **Pakistan Personal Data Protection Bill 2023** | Personal data processing, consent, retention, cross-border transfer | Consent management, retention enforcement, data residency disclosure (Supabase region disclosure), data subject rights (access, correction, deletion) |
| **Safe Blood Transfusion Act of Pakistan 2017** | Blood bank licensing, donor screening, traceability | Donor screening per WHO guidelines, full chain-of-custody, batch tracking, adverse event reporting |
| **WHO Blood Safety Guidelines** | Donor selection, screening, processing, storage | Eligibility rules, screening questions, component processing standards, storage temperature monitoring |
| **PECA 2016 (Prevention of Electronic Crimes Act)** | Cybercrime, unauthorized access | Access controls, audit logging, incident response plan |

### 11.2 Data Classification

The platform classifies all data into four tiers, each with different handling rules:

| Tier | Classification | Examples | Handling |
|---|---|---|---|
| T0 | **Public** | Impact metrics, success stories (consented), partner hospitals | No access control; can be cached publicly |
| T1 | **Internal** | Operational statistics, aggregate reports, internal knowledge base | Authenticated users only; RLS-enforced |
| T2 | **Confidential** | Donor contact details, hospital staff details, volunteer profiles | Role-based access; field-level encryption; audit logged |
| T3 | **Protected Health Information (PHI)** | Patient identity, disease/condition, medical reports, donor medical screening | Strict RBAC; column-level encryption; access audited; minimum necessary principle |

### 11.3 Consent Management

- The platform MUST record explicit consent for: data collection, data processing, communication channels, public use of success stories, and AI-assisted content generation involving the data subject.
- Consent records MUST include: who consented, when, what version of the policy, what specific consent was given, and how it was obtained (UI checkbox, signed form, etc.).
- Consent MUST be withdrawable at any time; withdrawal MUST propagate to all downstream uses within 24 hours.
- The platform MUST NOT use data for purposes beyond the original consent without re-consent.

### 11.4 Data Retention

| Data Type | Retention Period | Rationale |
|---|---|---|
| Donor records | Lifetime + 7 years | Regulatory requirement for blood bank traceability |
| Donation records | 7 years after donation | Regulatory requirement |
| Blood bag records | 10 years after disposal | Traceability for adverse event investigation |
| Patient records | 7 years after last activity | Regulatory requirement |
| Audit logs | 7 years | Security and compliance |
| Notification logs | 1 year | Operational reference |
| Documents (prescriptions, etc.) | Tied to patient record | Consistency |
| AI-generated drafts | 90 days (rejected) / permanent (approved) | Provenance |
| Web server logs | 30 days | Operational |
| Backups | 30 days rolling | Disaster recovery |

### 11.5 Data Subject Rights

The platform MUST support the following data subject rights:

- **Right to access** — a data subject can request a copy of all their data.
- **Right to correction** — a data subject can correct inaccurate data.
- **Right to deletion** — subject to regulatory retention requirements; data not under retention MUST be deleted on request.
- **Right to portability** — a data subject can export their data in machine-readable format.
- **Right to object** — a data subject can object to specific processing (e.g., marketing communications).

### 11.6 Data Residency & Cross-Border Transfer

- Supabase (as of design date) does not offer a Pakistan-region deployment. The primary database will be hosted in Supabase's Singapore region (closest to Pakistan).
- The platform MUST disclose this to data subjects in the privacy policy and consent flow.
- Cross-border transfer of PHI MUST be minimized; PHI MUST NOT be sent to third-party AI APIs without explicit consent and anonymization.
- The platform MUST maintain a data flow map documenting where every category of data is stored and processed.

---

## 12. Success Metrics & KPIs

### 12.1 Strategic KPIs

| KPI | Definition | Target (Year 1) | Target (Year 3) | Owner |
|---|---|---|---|---|
| Time-to-locate-blood (emergency) | Median time from emergency request submission to allocation | < 25 min | < 15 min | Operations |
| Blood bag expiry rate | Expired units / collected units, monthly | < 3% | < 1.5% | Operations |
| Recurring donor rate | Donors with 2+ donations in trailing 12 months / active donors | 40% | 60% | Donor Relations |
| Volunteer retention (12-month) | Volunteers active in month 12 / volunteers active in month 1 | 50% | 70% | Volunteer Coordinator |
| Request fulfillment SLA | % of requests fulfilled within SLA by urgency | 90% | 98% | Operations |
| Audit trail coverage | % of state-changing operations with audit log entry | 100% | 100% | CTO |
| Platform uptime | % of measured time platform is operational | 99.5% | 99.9% | CTO |
| Public report timeliness | % of scheduled reports published within target window | 90% | 100% | Executive Director |

### 12.2 Operational KPIs (Tracked Daily)

- Units collected (today, MTD, YTD)
- Units issued (today, MTD, YTD)
- Units expired (today, MTD, YTD)
- Active donors (rolling 12-month)
- New donor registrations (today, MTD)
- Pending requests by urgency
- Volunteers checked in (today)
- Notification delivery success rate
- API error rate (5xx)
- Average API response time (p95)

### 12.3 Anti-Metrics (Vanity Metrics to Avoid)

The platform explicitly does NOT track or optimize for:

- **Total registered users** (without active engagement, registration is meaningless).
- **Page views** (without conversion to action, views are noise).
- **WhatsApp group member count** (group membership ≠ engagement).
- **AI content generation volume** (volume without quality is waste).

---

## 13. Constraints, Assumptions & Dependencies

### 13.1 Constraints

1. **Zero-cost operation** at free tier of all constituent services for Phases 1–3.
2. **No paid third-party APIs** in Phases 1–3 (SMS, paid AI APIs, paid CDN).
3. **Solo-founder development** with AI assistance (Claude Code, ChatGPT, Cursor, GLM, etc.) — the architecture must be implementable by one person working alone, with AI as the second pair of hands.
4. **Local-first deployment** — the platform MUST run on a single laptop, a single desktop PC, a small office LAN, or the cloud, without architectural change. Cloud deployment is **optional**, not mandatory.
5. **Pakistan-based users** primarily on mid-range Android devices with 3G/4G connectivity (matters from Phase 2 onward when public portals launch; Phase 1 is admin-internal only).
6. **Urdu + English** at launch of public-facing surfaces (Phase 2); admin-internal Phase 1 may ship English-only with Urdu added in Phase 2.
7. **Supabase free-tier limits** (500MB database, 1GB storage, 50,000 monthly active users auth) define Phase 1–3 scale ceiling.
8. **Vercel free-tier limits** (100GB bandwidth, 100 hours serverless function execution) define Phase 1–3 traffic ceiling (Phase 1 traffic is near-zero — admin-internal only).
9. **Single-region deployment** (Singapore for cloud; or local server for office LAN) — multi-region is Phase 4+.
10. **No mandatory n8n dependency** — Phase 1 and Phase 2 MUST ship and operate with no n8n installed. n8n becomes available in Phase 3 as a pluggable automation provider.
11. **No mandatory AI dependency** — Phase 1 and Phase 2 MUST ship and operate with no AI integration. AI features arrive in Phase 3 and are always assistive, never blocking.
12. **Production quality from day one** — Phase 1 ships with full RLS, audit logging, validation, and security headers; there is no "prototype" or "throwaway" Phase 1.

### 13.2 Assumptions

1. The founder is the primary maintainer across Phases 1–3, with AI coding assistants as force multipliers.
2. The organization has at least one staff member (Imran persona) who will serve as primary platform operator.
3. The organization will maintain a GitHub account (free) for source control — solo user or organization, both fine.
4. A domain name (annual cost ~$10–15) is the **only** unavoidable cash cost, and only when public portals launch (Phase 2). Phase 1 can run on `localhost` or a LAN IP with zero cash cost.
5. Internet connectivity at the office is intermittent but sufficient for occasional cloud sync (Phase 1) and routine platform use (Phase 2+).
6. Volunteers and donors have access to smartphones — relevant from Phase 2 onward when public portals launch; Phase 1 is admin-internal and needs only the founder's laptop.
7. The founder will use AI coding assistants (Claude Code, ChatGPT, Cursor, GLM) following the **AI Development Rulebook** (`docs/04-AI_DEVELOPMENT_RULEBOOK.md`) to keep codebase consistency across AI sessions.

### 13.3 Dependencies

> **PHASE NOTE.** Only the dependencies marked `P1` are required for Phase 1. Dependencies marked `P2+` arrive in later phases. The platform MUST run with only `P1` dependencies installed.

| Dependency | Type | Phase | Mitigation if Unavailable |
|---|---|---|---|
| PostgreSQL (local or Supabase) | Infrastructure | P1 | Self-hosted Postgres via Docker Compose on the founder's laptop |
| Supabase Auth (local or cloud) | Infrastructure | P1 | Self-hosted GoTrue via Docker Compose; or local-only auth (Phase 1 internal users) |
| Next.js runtime | Infrastructure | P1 | None — open-source, runs anywhere Node runs |
| GitHub (free tier) | Infrastructure | P1 | GitLab free tier |
| Supabase Storage (local or cloud) | Infrastructure | P1 | MinIO via Docker Compose for local |
| Resend (email) | Service | P2 | Self-hosted Postfix; or Supabase Auth email (limited) |
| Vercel (cloud hosting) | Infrastructure | P2 (optional) | Self-hosted Next.js on the office PC; or Railway/Fly.io free tier |
| Cloudflare (DNS, CDN, WAF) | Infrastructure | P2 (optional) | Vercel built-in CDN; or skip CDN for office-LAN deployment |
| n8n self-hosted | Infrastructure | P3 | In-process TypeScript services (the default Phase 1–2 implementation) |
| WhatsApp Cloud API (free tier) | Service | P3 | Email + in-app notifications |
| AI service (free-tier LLM) | Service | P3 | Manual content authoring (the platform's default mode) |
| Expo / EAS (mobile app) | Infrastructure | P4 | Responsive web app on mobile browser |
| Sentry (error tracking) | Service | P2 | Console logging + structured logs (Phase 1 default) |
| UptimeRobot (uptime monitoring) | Service | P2 | Manual checks (Phase 1 default) |

---

## 14. Phase Classification & Out-of-Phase Items

This section classifies every capability of the platform into one of four phases (P1–P4) or marks it as **Out-of-Phase** (deliberately deferred indefinitely). The phases are defined in §15. The classification is the canonical answer to *"when does this feature ship?"* — every implementation prompt and PR must reference this section to justify its scope.

### 14.1 Phase 1 — Core MVP (Internal Use Only)

The goal of Phase 1 is software the organization can immediately start using internally, replacing paper and spreadsheets. **No public-facing surfaces, no automation, no AI, no mobile app.**

**Included in Phase 1:**

| Capability | Notes |
|---|---|
| Authentication | Email + password; roles: SUPER_ADMIN, ADMIN, VOLUNTEER, HOSPITAL, DONOR, PATIENT (roles may have no portal yet — they exist for forward compatibility) |
| Admin Dashboard | Sidebar, list views, detail views, search, filter, sort, pagination, dark/light mode |
| Donor Management | CRUD, basic eligibility calculation (rule-based per WHO), donation history, soft delete |
| Patient Management | CRUD, guardian info, treating hospital, disease, documents upload (virus scan deferred to Phase 2) |
| Hospital Management | CRUD, partnership status, staff accounts |
| Blood Inventory | Bag tracking, unique ID, state machine (COLLECTED → SCREENED → STORED → RESERVED → ISSUED → DELIVERED → RETURNED → EXPIRED → DISCARDED), expiry date auto-calc, storage locations |
| Blood Requests | Submission (admin-only in P1), status tracking, allocation, SLA display (no auto-escalation) |
| Blood Issuance | Allocate bag to request, mark issued, state transition |
| Blood Delivery History | Record dispatch, delivery confirmation (manual, no OTP) |
| Volunteer Management | CRUD, basic profile, no task assignment (P2) |
| Search | Postgres full-text search across donors, patients, requests, bags |
| Manual Reports | CSV export of any list view; basic PDF via print stylesheet; no scheduled reports |
| Dashboard Statistics | Counts, simple charts (Recharts), trend over last 30 days |
| Settings | Deferral windows, SLA thresholds, inventory alert thresholds (no notification dispatch — display only) |
| Database Backup & Restore | `pg_dump` script + restore script; manual run; verified by founder |
| Audit Logging | Append-only audit_log table, trigger-based on sensitive tables |
| RLS | Enabled on every user-data table, policies tested |
| Security Headers | CSP, HSTS, X-Frame-Options, etc. enforced via middleware |
| Local-First Deployment | Runs on `localhost` or office LAN; cloud sync optional |

**Explicitly EXCLUDED from Phase 1** (deferred to later phases, not built):

- n8n, AI, social media automation, WhatsApp automation, hospital APIs, QR codes, barcode systems, predictive AI, advanced analytics, public donor portal, mobile app, complex notification systems, CI/CD requirements beyond a basic GitHub workflow, blood camps, blood test reports, donor/patient/hospital self-service portals, email notifications, donation reminders, blood expiry alerts (active notifications), scheduled reports.

### 14.2 Phase 2 — Operational Improvements

| Capability | Notes |
|---|---|
| Blood Camp Management | Schedule, assign volunteers, record collections on-site |
| Blood Test Reports | Donor screening records, link to blood bag |
| Donor Portal | Self-service web (profile, donation history, eligibility, impact) |
| Patient Portal | Self-service web (submit request, track status, upload documents) |
| Hospital Portal | Self-service web (submit request, view aggregate inventory, history) |
| Email Notifications | Via Resend (in-process, not n8n); post-donation thank-you, request status, delivery status |
| Donation Reminders | In-process scheduled check (basic; n8n version in P3) |
| Blood Expiry Alerts | Dashboard banner + email (in-process; n8n version in P3) |
| Scheduled Reports | In-process cron-like (basic; n8n version in P3) |
| Better Analytics | Cohort, trend, geographic views |
| Public Website | Home, about, request blood, donate blood, volunteer, knowledge base, impact dashboard |
| Virus Scanning | ClamAV via Edge Function or in-process daemon |
| Urdu Language | i18n with Urdu translations for public surfaces |
| Cloud Deployment | Vercel + Supabase cloud (optional; local-first still supported) |
| Sentry / UptimeRobot | Error tracking and uptime monitoring |

### 14.3 Phase 3 — Automation

| Capability | Notes |
|---|---|
| n8n Self-Hosted | Railway or Fly.io free tier; behind Cloudflare Tunnel |
| Scheduled Workflows | All AUT-001 through AUT-020 |
| AI-Generated Reports | Subject to admin review (AUT-AI-006, AUT-AI-007) |
| AI-Generated Social Posts | Subject to admin review (AUT-AI-001, AUT-AI-002) |
| Website News Automation | Auto-publish approved content |
| Facebook Automation | Auto-post approved content |
| Instagram Automation | Auto-post approved content |
| LinkedIn Automation | Auto-post approved content |
| X (Twitter) Automation | Auto-post approved content |
| WhatsApp Automation | WhatsApp Cloud API free tier |
| Telegram Automation | Bot API |
| Scheduled Backups | n8n-driven pg_dump to off-site storage |
| Scheduled Maintenance | n8n-driven (reindex, vacuum, archive) |
| Volunteer Reminders | AUT-006, AUT-007 |
| NotificationService → n8n Adapter | Swap in-process EmailService for n8n-backed implementation behind the same interface |
| AI Content Generation | All AUT-AI-001 through AUT-AI-008 |

### 14.4 Phase 4 — Scale

| Capability | Notes |
|---|---|
| Mobile Application | Expo React Native (donor + volunteer apps) |
| Hospital API Integrations | REST API for hospital information systems |
| Multi-City Deployment | Multiple storage locations across cities |
| Multi-Branch Management | Branch-level roles, inventory, reporting |
| High Availability | Read replicas, failover |
| Advanced Monitoring | Grafana / Prometheus, distributed tracing |
| Disaster Recovery Improvements | Multi-region backup, lower RTO/RPO |
| Predictive AI / ML | Demand forecasting, donor churn prediction |
| National Blood Network | Federation with other blood banks |
| Government Integrations | Safe Blood Authority reporting |
| Enterprise DevOps | Multi-environment CI/CD, blue-green deploys, canary releases |

### 14.5 Out of Phase (Deferred Indefinitely)

These capabilities are explicitly **out of scope indefinitely**. They may be reconsidered if the organization's mission expands, but the platform is not designed to support them in any current phase.

| Capability | Reason |
|---|---|
| **Financial management module** | Donations, expenses, accounting, financial reporting — out of scope; the platform is a blood logistics platform, not a finance platform. Use a separate accounting tool. |
| **Fundraising campaigns** | Online donation collection, campaign pages — out of scope for the same reason. |
| **Multi-organization / multi-tenant SaaS** | The platform serves Marwat Welfare Society. Multi-tenancy would require governance changes the organization has not committed to. The architecture is multi-tenant-*capable* (so a future fork can serve others), but the deployed product is single-tenant. |
| **Telemedicine / doctor consultation** | The platform is a blood logistics platform, not a healthcare platform. |
| **Blockchain-based traceability** | Over-engineered; the conventional audit trail (append-only audit_log + state history) provides equivalent traceability at a fraction of the complexity. |
| **Paid advertising integration** | Google Ads, Meta Ads — out of scope indefinitely; the platform does not handle marketing spend. |
| **Native iOS/Android apps (non-Expo)** | Expo is sufficient; native apps would double maintenance cost for no benefit. |
| **Custom AI model training** | Use pre-trained models only; training custom models is out of scope indefinitely. |

---

## 15. Phased Roadmap

The development roadmap is structured in four phases. Each phase has a clear goal, entry criteria, deliverables, and exit criteria. The phases are **cumulative** — each phase extends the previous, never replaces it. Detailed task breakdowns are in the System Architecture Document §11.

### 15.1 Phase 1 — Core MVP (Weeks 1–4)

**Goal:** Software the organization can immediately start using internally, replacing paper and spreadsheets.

**Entry criteria:**
- Founder has a development laptop with Node 20+ and Docker.
- GitHub repository created.
- Four constitution documents (`docs/01-PRD.md` through `docs/04-AI_DEVELOPMENT_RULEBOOK.md`) committed to the repo.

**Deliverables (see §14.1 for the full list):**
- Monorepo scaffolded (pnpm workspaces + Turborepo).
- Database schema (core tables only) with migrations and RLS policies.
- Authentication (Supabase Auth, email + password; local Supabase via Docker for local-first).
- Admin dashboard with sidebar, list views, detail views, search, filter, sort.
- Donor, patient, hospital, volunteer, inventory, request, delivery CRUD.
- Manual reports (CSV export).
- Dashboard statistics (counts + simple charts).
- Settings (operational parameters).
- Backup & restore scripts (pg_dump).
- Audit logging infrastructure.
- Basic GitHub Actions workflow: lint + typecheck + test on push.
- Local-first deployment: runs on `localhost` or office LAN with zero cloud dependency.

**Exit criteria:**
- The organization uses the platform for all daily operations (register donors, record donations, submit and fulfill requests, manage inventory).
- 100% audit trail coverage.
- Local deployment works on the founder's laptop and the office desktop PC.
- Optional: cloud sync to Supabase free tier works.
- A non-technical admin (Imran persona) can perform every routine task without developer assistance.

### 15.2 Phase 2 — Operational Improvements (Weeks 5–12)

**Goal:** Self-service portals for donors, patients, and hospitals; basic email notifications; blood camp management.

**Entry criteria:**
- Phase 1 launched and in daily use.
- Domain name registered (only unavoidable cash cost).
- Resend account created (free tier).

**Deliverables (see §14.2 for the full list):**
- Blood camp management.
- Blood test reports (donor screening).
- Donor portal, patient portal, hospital portal.
- Email notifications (in-process, not n8n).
- Donation reminders (in-process scheduled check).
- Blood expiry alerts (dashboard + email).
- Scheduled reports (in-process).
- Better analytics (cohort, trend, geographic).
- Public website (home, about, request blood, donate blood, volunteer, knowledge base, impact dashboard).
- Virus scanning for file uploads.
- Urdu language support (i18n).
- Cloud deployment (Vercel + Supabase cloud) — optional, local-first still supported.
- Sentry error tracking + UptimeRobot monitoring.

**Exit criteria:**
- A donor can self-register, view their history, and update their profile without admin assistance.
- A guardian can submit a blood request from the public website and track its status.
- A hospital can submit and track requests through their portal.
- Email notifications deliver within 60 seconds of the triggering event.
- Blood bag expiry rate drops below 5% (baseline: 12%).

### 15.3 Phase 3 — Automation (Weeks 13–20)

**Goal:** Replace manual and in-process automations with n8n workflows; introduce AI-assisted content generation (with human review).

**Entry criteria:**
- Phase 2 in production.
- n8n self-hosted (Railway or Fly.io free tier) or willingness to use in-process services indefinitely.

**Deliverables (see §14.3 for the full list):**
- n8n self-hosted and integrated.
- All AUT-001 through AUT-020 workflows implemented.
- All AUT-AI-001 through AUT-AI-008 AI-assisted content workflows implemented.
- NotificationService and ReportService swapped to n8n-backed adapters (interface unchanged).
- Social media automation (Facebook, Instagram, LinkedIn, X) — auto-post approved content.
- WhatsApp automation (Cloud API free tier).
- Telegram automation (Bot API).
- Scheduled backups (off-site to Cloudflare R2 free tier).
- Scheduled maintenance (reindex, vacuum, archive).

**Exit criteria:**
- 90% of routine notifications and reports are produced without human intervention.
- 90% reduction in time-to-locate-blood for emergency requests (target: under 25 minutes).
- AI-generated content for at least 4 use cases, all subject to human review.
- The platform continues to function correctly if n8n is disabled (in-process services remain as fallback).

### 15.4 Phase 4 — Scale (Months 6+)

**Goal:** Mobile application, multi-city/multi-branch operation, advanced monitoring, predictive AI.

**Entry criteria:**
- Phase 3 in production.
- Organization operating across multiple cities or planning to.
- Justification for mobile app (donor/volunteer demand exceeding web portal).

**Deliverables (see §14.4 for the full list):**
- Mobile application (Expo React Native — donor + volunteer apps).
- Hospital API integrations.
- Multi-city deployment.
- Multi-branch management.
- High availability (read replicas, failover).
- Advanced monitoring (Grafana / Prometheus, distributed tracing).
- Disaster recovery improvements (multi-region backup, lower RTO/RPO).
- Predictive AI / ML (demand forecasting, donor churn prediction).
- National blood network federation (if applicable).
- Government integrations (Safe Blood Authority reporting).
- Enterprise DevOps (multi-environment CI/CD, blue-green deploys, canary releases).

**Exit criteria:**
- 75% reduction in blood bag expiry rate (target: under 3%).
- 2× increase in recurring donor rate.
- Platform supports multi-city, multi-branch operation with no architectural change.

### 15.5 Future Expansion Roadmap

The capabilities below are **not scheduled** for any specific phase. They are documented here so the architecture can accommodate them if the organization's mission expands. Inclusion in this list does **not** imply commitment — only that the architecture does not preclude them.

| Capability | Trigger for Consideration | Architecture Impact |
|---|---|---|
| Financial management module | Organization decides to handle donations/expenses in-platform | New bounded context; no change to existing contexts |
| Fundraising campaigns | Organization launches public fundraising | New module; payment gateway integration (paid; budget required) |
| Multi-organization SaaS | Other welfare organizations request access to the platform | Tenant isolation already designed in (RLS, schema prefixes); operational governance changes needed |
| Telemedicine / doctor consultation | Strategic pivot beyond blood logistics | New platform, not extension of this one |
| Blockchain-based traceability | Regulatory requirement or partner mandate | Append-only audit_log already provides equivalent traceability; blockchain would add complexity without clear benefit |
| Custom AI model training | Use case emerges where pre-trained models are insufficient | Requires ML infrastructure (GPU, training pipeline) — significant scope expansion |
| IoT temperature monitoring | Cold-chain compliance becomes mandatory | New sensor integration; webhook ingestion; minimal architecture change |

**Rule:** Adding any capability from this list to a scheduled phase requires a constitution amendment (founder approval during solo-founder stage; Board Tech Committee post-Phase 4) and a version bump.

---

## 16. Glossary

| Term | Definition |
|---|---|
| **Bag** | A unit of collected blood or blood component; the atomic unit of inventory. |
| **Chain of Custody** | The documented sequence of custody for a blood bag from collection to disposal. |
| **Component** | A blood product derived from whole blood: red cells, platelets, plasma, cryoprecipitate. |
| **Deferral** | A temporary or permanent state in which a donor is not eligible to donate. |
| **Deferal Window** | The minimum time between donations for a donor (per WHO: 56 days whole blood, 16 weeks platelets). |
| **Guardian** | A person submitting a blood request on behalf of a patient. |
| **PHI** | Protected Health Information — patient identity, conditions, medical records. |
| **RLS** | Row-Level Security — PostgreSQL feature enforcing per-row access control. |
| **SLA** | Service Level Agreement — committed response time for a request by urgency. |
| **Soft Delete** | Marking a record as deleted without removing it from the database (for audit and recovery). |
| **Triage** | The process of prioritizing requests by urgency and clinical need. |
| **Whole Blood** | Unseparated blood as collected from a donor. |

---

## 17. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0.0 | 2026-07-06 | Principal Architect | Initial baseline approved by Board |
| 1.1.0 | 2026-07-06 | Founder / Principal Architect | **Startup-first refactor.** Replaced enterprise-first 7-phase roadmap with 4-phase phased roadmap (Core MVP → Operational Improvements → Automation → Scale). Added phase tags (P1–P4) across all functional requirements, automation requirements, and dependencies. Made n8n and AI explicitly optional (not Phase 1 dependencies) via the Service Interface pattern. Added Local-First Architecture constraint (single laptop → office LAN → cloud, no architectural change). Reframed §14 as Phase Classification & Out-of-Phase Items. Added §15.5 Future Expansion Roadmap preserving long-term capabilities. Reframed success criteria as phase-scoped. Simplified governance: founder approval during solo-founder stage, Board Tech Committee from Phase 4. Preserved all security, all architecture, and all long-term capabilities — only implementation order changed. |

---

> **End of Product Requirements Document.** This document is the project's constitution. The next document in the set is the **System Architecture Document** (`02-SYSTEM_ARCHITECTURE.md`), which defines *how* the PRD is realized in system structure.
