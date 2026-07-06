# Marwat Welfare Society — System Architecture Document

| Field | Value |
|---|---|
| **Document Title** | System Architecture Document |
| **Project Name** | Marwat Welfare Society — Blood Donation & Distribution Platform |
| **Document Version** | 1.1.0 |
| **Status** | Approved Baseline |
| **Classification** | Internal — Project Constitution |
| **Last Updated** | 2026-07-06 |
| **Document Owner** | Founder / Principal Architect |
| **Approval Authority** | Founder (solo-founder stage); Board Tech Committee (post-Phase 4) |
| **Depends On** | `01-PRD.md` (v1.1.0 or later) |
| **Consumed By** | `03-TECHNICAL_DESIGN_SPECIFICATION.md`, `04-AI_DEVELOPMENT_RULEBOOK.md` |

> **CONSTITUTION CLAUSE.** This document defines the *how* of the system structure. Every implementation decision MUST conform to the architecture defined here. Where an implementation pattern is not explicitly addressed here, the Technical Design Specification governs; where the TDS is silent, the AI Development Rulebook governs; where all three are silent, the founder (or designated tech lead, post-Phase 4) decides and the decision is **optionally** recorded as an Architecture Decision Record (ADR). ADRs are strongly encouraged for significant decisions but are not merge-blocking during the solo-founder stage.
>
> **v1.1.0 REFACTOR NOTE.** This version adds **Local-First Architecture** (§3.4) as a first-class architectural principle: the platform runs on a single laptop, an office LAN, or the cloud without architectural change. It redefines **Automation Architecture** (§10) to use the **Service Interface pattern** — Phase 1–2 ship with in-process TypeScript services, and Phase 3 swaps in n8n-backed adapters behind the same interfaces. n8n is **never a hard dependency**. It also simplifies governance: ADRs are encouraged but optional during the solo-founder stage. All security, data, and structural decisions from v1.0.0 are preserved unchanged.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Architecture Principles](#2-architecture-principles)
3. [High-Level System Architecture](#3-high-level-system-architecture)
4. [Logical Architecture](#4-logical-architecture)
5. [Component Architecture](#5-component-architecture)
6. [Data Architecture](#6-data-architecture)
7. [Security Architecture](#7-security-architecture)
8. [Integration Architecture](#8-integration-architecture)
9. [Authentication & Authorization Architecture](#9-authentication--authorization-architecture)
10. [Automation Architecture](#10-automation-architecture)
11. [Deployment Architecture](#11-deployment-architecture)
12. [Scalability Architecture](#12-scalability-architecture)
13. [Observability Architecture](#13-observability-architecture)
14. [Disaster Recovery & Business Continuity](#14-disaster-recovery--business-continuity)
15. [Folder & Repository Architecture](#15-folder--repository-architecture)
16. [Technology Stack Decisions](#16-technology-stack-decisions)
17. [Architecture Decision Records (ADRs)](#17-architecture-decision-records-adrs)
18. [Glossary](#18-glossary)
19. [Revision History](#19-revision-history)

---

## 1. Architecture Overview

### 1.1 Purpose of This Document

This System Architecture Document translates the requirements defined in the PRD into a concrete system structure. It defines the major components of the platform, their responsibilities, their interfaces, their interactions, and the principles that govern their evolution. It is the binding architectural contract for every contributor — human or AI — working on the codebase.

This document deliberately stops short of implementation detail. It does not contain SQL DDL, TypeScript interfaces, or API route signatures — those belong to the Technical Design Specification. It does contain architectural diagrams, component responsibilities, data flow descriptions, and the rationale for every significant architectural choice.

### 1.2 Architectural Style (Startup-First, Phased)

The platform is built by a **solo founder using AI coding assistants** with a **zero budget** and a need to ship useful software within weeks, not years. The architecture reflects this reality while preserving the ability to grow into an enterprise platform over 10+ years without major redesign.

The core architectural commitment is **phased, replaceable implementation behind stable interfaces**:

- **Phase 1** ships a working MVP using the simplest possible implementation of each interface (in-process TypeScript services, local Postgres, no automation, no AI).
- **Phase 2** adds operational improvements (portals, email, camps) — still no automation, no AI.
- **Phase 3** swaps in-process service implementations for n8n-backed adapters behind the **same interfaces**. The core application does not change.
- **Phase 4** adds mobile, multi-city, advanced monitoring, predictive AI — without touching Phase 1–3 code paths except where new features are explicitly added.

This is the **Strategy Pattern applied at the architecture level**. The founder codes against `NotificationService.send()`, `ReportService.generate()`, `BackupService.run()`. The implementation behind those interfaces evolves across phases. Call sites do not change.

### 1.3 Document Scope

This document covers:

- The decomposition of the platform into logical components and their responsibilities.
- The deployment topology across the constituent services (Supabase, Vercel, Expo, n8n).
- The security architecture: trust boundaries, authentication, authorization, encryption, audit.
- The integration architecture: how the web app, mobile app, n8n, and external services interact.
- The data architecture: data ownership, lifecycle, partitioning, and consistency model.
- The observability architecture: logging, metrics, tracing, alerting.
- The disaster recovery architecture: backups, failover, RTO/RPO.
- The scalability architecture: how the system grows from 100 users to 100,000+.
- The folder and repository structure for the monorepo.
- The technology stack decisions, with rationale and replaceability analysis.

This document does **not** cover:

- Implementation-level schemas (SQL DDL, TypeScript types) — see TDS.
- Coding standards and conventions — see AI Development Rulebook.
- Per-feature behavior — see PRD §6.
- Operational runbooks — separate document (post-launch).

### 1.4 Architectural Style

The platform follows **Clean Architecture** with **feature-based modular decomposition**. The core ideas:

- **Dependency direction is inward.** Outer layers (UI, infrastructure) depend on inner layers (domain, use cases), never the reverse.
- **Features are self-contained modules.** Each feature (donor management, inventory, requests, etc.) is a vertical slice containing its own UI, business logic, services, types, and validation. Cross-feature dependencies go through published interfaces, never through internal imports.
- **Infrastructure is abstracted.** The application does not call Supabase, Vercel, or n8n directly from feature code. It calls interfaces (Repository, Service, Notifier) that are implemented by infrastructure-specific adapters.
- **State is owned by the server.** The client reflects server state via real-time subscriptions; it does not own authoritative state. This eliminates an entire class of consistency bugs.

### 1.5 Architectural Quality Goals

Every architectural decision is evaluated against five quality attributes, in priority order:

1. **Security** — Does the decision protect PHI and prevent unauthorized access? (Highest priority — never sacrificed.)
2. **Maintainability** — Can a new contributor understand and modify this in 5 years?
3. **Replaceability** — Can we swap the underlying vendor without rewriting the application?
4. **Scalability** — Can this scale 10× without architectural change?
5. **Cost** — Does this fit within the zero-cost constraint for Phase 1–3?

When a decision cannot satisfy all five, the higher-priority attribute wins, and the tradeoff is documented in an ADR (encouraged but not merge-blocking during the solo-founder stage).

---

## 2. Architecture Principles

These principles are the architectural non-negotiables. They supersede convenience, speed of implementation, and personal preference.

### 2.1 P1 — Clean Layering

The system is organized into four layers. Dependencies flow only inward.

```
┌─────────────────────────────────────────────────────────┐
│  Presentation Layer (UI)                                │
│  Next.js pages, React components, Expo screens          │
└─────────────────────────────────────────────────────────┘
                          ↓ depends on
┌─────────────────────────────────────────────────────────┐
│  Application Layer (Use Cases)                          │
│  Services, orchestration, validation, authorization     │
└─────────────────────────────────────────────────────────┘
                          ↓ depends on
┌─────────────────────────────────────────────────────────┐
│  Domain Layer (Business Rules)                          │
│  Entities, value objects, domain events, invariants     │
└─────────────────────────────────────────────────────────┘
                          ↑ implemented by
┌─────────────────────────────────────────────────────────┐
│  Infrastructure Layer (Adapters)                        │
│  Supabase client, n8n client, email/SMS adapters, S3    │
└─────────────────────────────────────────────────────────┘
```

**Rule:** The Domain Layer has zero external dependencies. It is pure TypeScript with no imports from `@supabase/supabase-js`, `next`, `react`, or any infrastructure package. This is enforced by an ESLint rule.

### 2.2 P2 — Feature Modularization

The codebase is decomposed into features, not into file types. A feature is a coherent capability (donor management, inventory, requests, etc.) that has its own UI, business logic, services, types, and validation. Features live under `src/features/<name>/`.

**Rules:**
- A feature MAY export a public API (types, hooks, components) via an `index.ts` barrel.
- A feature MUST NOT import from another feature's internal files — only from the other feature's `index.ts`.
- A feature MUST NOT import UI components from another feature — only shared UI from `src/components/`.
- Shared utilities live under `src/lib/`, not under features.
- Circular dependencies between features are forbidden and enforced by `madge` in CI.

### 2.3 P3 — Server-Authoritative State

The server (Supabase Postgres + RLS) is the single source of truth. Clients reflect server state; they do not own it.

**Implications:**
- All mutations go through Supabase RLS-protected tables or authenticated Edge Functions.
- Optimistic UI updates are allowed but MUST be reconciled with server state within 5 seconds, and rolled back on conflict.
- Real-time updates use Supabase Realtime subscriptions, not client-side polling.
- The client cache (React Query / SWR) is a *cache*, not a store. Cache invalidation is server-driven.

### 2.4 P4 — Zero-Trust Internal APIs

Every API endpoint, every database query, every Edge Function assumes the caller is hostile until proven otherwise. Authentication and authorization are checked at every trust boundary, not just at the perimeter.

**Implications:**
- RLS is enforced at the database level, not at the API level. Even if an API bug exposes a query, RLS prevents unauthorized data access.
- Edge Functions re-validate the JWT and re-check authorization; they do not trust the gateway.
- Server-side props in Next.js never assume the user's role; they fetch it from the session.
- Internal service-to-service calls (e.g., n8n → Edge Function) use signed webhooks with HMAC verification.

### 2.5 P5 — Infrastructure Abstraction

Feature code does not call Supabase, n8n, or any vendor SDK directly. It calls interfaces defined in the domain layer, which are implemented by adapters in the infrastructure layer.

**Example:**
```typescript
// Domain layer — interface
interface DonorRepository {
  findById(id: DonorId): Promise<Donor | null>
  save(donor: Donor): Promise<void>
}

// Infrastructure layer — Supabase implementation
class SupabaseDonorRepository implements DonorRepository {
  constructor(private client: SupabaseClient) {}
  async findById(id: DonorId) { /* Supabase call */ }
  async save(donor: Donor) { /* Supabase call */ }
}
```

This pattern is used for: repositories, notifiers, file storage, AI content generation, and external integrations. Replacing Supabase with another backend means writing new adapters, not rewriting features.

### 2.6 P6 — Explicit over Implicit

Architectural decisions are explicit, visible, and documented. Magic is forbidden.

- Configuration is via environment variables with descriptive names and a `.env.example` template.
- Dependencies between modules are explicit (imports), not implicit (globals, singletons).
- Side effects (file writes, network calls, notifications) are in named functions, not in constructors or hidden lifecycle methods.
- Error handling is explicit; swallowed errors are forbidden.

### 2.7 P7 — Observable by Default

Every component emits structured logs, metrics, and traces. Observability is not added later; it is designed in.

- Every API request has a correlation ID propagated via headers.
- Every background job logs its start, success/failure, and duration.
- Every state transition in a state machine logs the transition with actor and timestamp.
- Admins can see system health from the dashboard without developer access.

### 2.8 P8 — Reversible Decisions

Prefer decisions that can be reversed cheaply. When an irreversible decision is unavoidable, document it as an ADR with explicit rationale and reconsideration triggers.

- Vendor choices are reversible (via P5 abstraction).
- Database schema changes are partially reversible (migrations include down-migrations where possible).
- Public API contracts are partially reversible (versioned, deprecation policy).
- Data migrations are irreversible — these require explicit ADRs and Board sign-off.

---

## 3. High-Level System Architecture

### 3.1 System Context Diagram

```
                          ┌─────────────────────────────────────┐
                          │        End Users (Personas)         │
                          │  Donor  Patient  Volunteer  Hospital│
                          │  Admin   Public  Visitor            │
                          └──────────┬──────────────────────────┘
                                     │
                                     │ HTTPS / WSS
                                     │
              ┌──────────────────────┼──────────────────────┐
              │                      │                      │
              ▼                      ▼                      ▼
   ┌──────────────────┐  ┌────────────────────┐  ┌────────────────────┐
   │  Web App         │  │  Mobile App        │  │  Public Website    │
   │  (Admin/Hospital │  │  (Donor/Volunteer) │  │  (SSR + Static)    │
   │   /Patient)      │  │  Expo React Native │  │  Next.js           │
   │  Next.js         │  │                    │  │                    │
   └────────┬─────────┘  └─────────┬──────────┘  └─────────┬──────────┘
            │                      │                       │
            │  Supabase JS SDK     │  Supabase JS SDK      │  Supabase JS SDK
            │  (auth + data + RT)  │  (auth + data + RT)   │  (auth + data + RT)
            │                      │                       │
            └──────────────────────┼───────────────────────┘
                                   │
                                   ▼
                ┌───────────────────────────────────────┐
                │         Supabase Platform             │
                │                                       │
                │  ┌─────────┐  ┌──────────┐  ┌──────┐ │
                │  │  Auth   │  │ Postgres │  │Storage│ │
                │  │ (GoTrue)│  │  + RLS   │  │ (S3) │ │
                │  └─────────┘  └────┬─────┘  └──────┘ │
                │                      │                │
                │  ┌───────────────────┴──────────┐    │
                │  │     Edge Functions (Deno)    │    │
                │  │  - Webhooks (to n8n)         │    │
                │  │  - Sensitive operations      │    │
                │  │  - File virus scanning       │    │
                │  └──────────────────────────────┘    │
                │                      │                │
                │  ┌───────────────────┴──────────┐    │
                │  │      Realtime (WebSockets)    │    │
                │  └──────────────────────────────┘    │
                └──────────────────────┬───────────────┘
                                       │
                                       │ HTTPS webhooks (HMAC-signed)
                                       │
                                       ▼
                ┌───────────────────────────────────────┐
                │       n8n (Self-Hosted)               │
                │                                       │
                │  - Scheduled workflows                │
                │  - Webhook-triggered workflows        │
                │  - Email/WhatsApp/SMS adapters        │
                │  - AI content generation              │
                │  - Report generation                  │
                └──────────┬────────────────────────────┘
                           │
                           │ HTTPS (signed JWT)
                           │
              ┌────────────┴───────────────┐
              │                            │
              ▼                            ▼
   ┌─────────────────────┐    ┌─────────────────────┐
   │  External Services  │    │   AI Service        │
   │  - Resend (email)   │    │  (self-hosted or    │
   │  - WhatsApp Cloud   │    │   free-tier LLM API)│
   │    API              │    │                     │
   └─────────────────────┘    └─────────────────────┘
```

### 3.2 Component Summary

> **PHASE NOTE.** Components marked `P1` are required for Phase 1. Components marked `P2+` arrive in later phases. The platform MUST run with only `P1` components installed and configured.

| Component | Technology | Hosting | Phase | Purpose |
|---|---|---|---|---|
| Admin Web App | Next.js 15 (App Router) | Local / Vercel | P1 | Authenticated dashboard |
| Database | PostgreSQL 15 | Local Docker / Supabase | P1 | Authoritative state |
| Authentication | GoTrue (Supabase Auth) | Local Docker / Supabase | P1 | JWT-based auth, email + password |
| File Storage | S3-compatible (MinIO / Supabase Storage) | Local Docker / Supabase | P1 | Documents, images |
| Public Website | Next.js 15 (App Router) | Local / Vercel | P2 | SSR + static pages, public-facing |
| Email | Resend (free tier) | Resend | P2 | Transactional email |
| Edge Functions | Deno | Supabase | P2 | Sensitive ops, webhooks, virus scan |
| Realtime | WebSocket | Supabase Realtime | P2 | Live updates (post-v1 portals) |
| Sentry / UptimeRobot | Sentry / UptimeRobot | Sentry / UptimeRobot | P2 | Error tracking, uptime |
| Donor/Volunteer Mobile App | Expo React Native | EAS + app stores | P4 | Mobile experience |
| Automation | n8n | Self-hosted (Railway/Fly.io free tier) | P3 | Workflows (replaces in-process services) |
| WhatsApp | WhatsApp Cloud API (free tier) | Meta | P3 | Notifications |
| SMS | (Optional, paid) | Jazz/Telenor PK gateway | P3+ | Fallback channel |
| AI | Free-tier LLM (e.g., Groq free, or self-hosted) | Mixed | P3 | Content generation (assistive only) |
| DNS / CDN | Cloudflare (free) | Cloudflare | P2 (optional) | DNS, CDN, DDoS protection |
| Source Control | GitHub | GitHub | P1 | Repo, basic Actions CI |

### 3.3 Trust Boundaries

The system has four trust boundaries. Each boundary has explicit authentication and authorization controls.

1. **User → Client.** The user authenticates to the web or mobile app via Supabase Auth. The client holds a JWT (short-lived access token + long-lived refresh token).
2. **Client → Supabase.** Every Supabase call carries the JWT. RLS policies enforce row-level access based on the JWT claims.
3. **Supabase → n8n (Phase 3+).** Database webhooks trigger n8n workflows. The webhook is HMAC-signed with a shared secret. n8n verifies the signature before processing. **In Phase 1–2 this boundary does not exist** — there is no n8n. Notifications and reports are produced by in-process TypeScript services called directly from application code.
4. **n8n → Supabase (Phase 3+).** n8n calls Supabase Edge Functions (or directly the PostgREST API) using a service-role JWT. The service role bypasses RLS, so n8n actions are scoped to specific Edge Functions that perform their own authorization checks.

Public access (no authentication) is allowed only to: the public website's static content (Phase 2+), the aggregate impact dashboard (Phase 2+), the knowledge base (Phase 2+), and the registration/request submission entry points (Phase 2+). **In Phase 1 there is no public access** — the platform is admin-internal only.

### 3.4 Local-First Architecture

The platform is designed to run perfectly on **four deployment targets** without any architectural change:

1. **A single laptop** (founder's development machine, also the production machine during early Phase 1).
2. **A single desktop PC** (the office machine that staff use during the day).
3. **A small office network** (LAN deployment — the desktop PC hosts the database and app, other office machines connect over the LAN).
4. **The cloud** (Supabase + Vercel free tiers, optionally with Cloudflare in front).

The deployment target is selected purely by **environment variables**. The same Docker Compose file used in development can be used for a small-office LAN deployment. The same Next.js code runs on `localhost:3000`, on `192.168.1.10:3000` (LAN), and on `https://app.marwatwelfare.org` (cloud).

**Implementation rules:**

- The application reads `DATABASE_URL`, `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, and `NEXT_PUBLIC_APP_URL` from environment variables. It does **not** hardcode URLs or assume a particular host.
- For local/LAN deployment, `supabase` CLI's local stack (Postgres + GoTrue + Storage + Realtime, all in Docker) is used. The same stack works on a laptop or an office PC.
- For cloud deployment, the same env vars point to the cloud Supabase project. No code change.
- File storage uses MinIO locally (S3-compatible, Docker) and Supabase Storage in the cloud. The Supabase JS SDK treats both identically because MinIO speaks the S3 protocol.
- **No cloud-only features in Phase 1.** Every Phase 1 capability MUST work with zero internet connectivity. Cloud sync (Phase 2+) is an enhancement, not a dependency.
- **The office PC is a production environment.** A power outage or reboot is treated as a production incident. The founder MUST document the recovery procedure (start Docker, start the Next.js process, verify health check).

**Why this matters:**

The organization's office may have intermittent internet. Staff cannot depend on a cloud service that may be unreachable when they need to register a donor or check inventory. Local-first means the platform is **always usable from the office**, with cloud as a backup and remote-access layer.

This also means the founder can develop and demo the platform on a laptop without any cloud account — reducing the barrier to getting started to literally zero cost.

**Migration path (local → cloud):**

1. Run locally on the founder's laptop during Phase 1 development.
2. Deploy to the office desktop PC for staff use (Phase 1 launch).
3. Optionally sync to Supabase cloud for remote access (Phase 2).
4. Optionally deploy Next.js to Vercel for public portals (Phase 2).
5. Add n8n (Phase 3) and mobile app (Phase 4) as separate concerns.

At no point does the application code change to accommodate a different deployment target. Only environment variables change.

---

## 4. Logical Architecture

### 4.1 Bounded Contexts

The platform is decomposed into seven bounded contexts. Each context owns a coherent slice of the domain and exposes a clear interface to other contexts.

| Bounded Context | Responsibility | Owning Module |
|---|---|---|
| **Identity & Access** | Users, authentication, roles, permissions | `features/auth` |
| **Donor Management** | Donor profiles, medical screening, eligibility, donation history | `features/donors` |
| **Inventory** | Blood bags, state machine, storage locations, expiry | `features/inventory` |
| **Request Fulfillment** | Blood requests, matching, allocation, delivery | `features/requests` |
| **Volunteer Management** | Volunteers, tasks, camps | `features/volunteers` + `features/camps` |
| **Content & Communication** | Knowledge base, notifications, reports, AI content | `features/content` + `features/notifications` + `features/reports` |
| **Platform Operations** | Settings, audit logs, automation, monitoring | `features/admin` + `features/automation` |

### 4.2 Cross-Context Communication

Bounded contexts communicate only through published interfaces:

- **Synchronous:** A context calls another context's service via its exported interface (e.g., `RequestsService` calls `InventoryService.checkAvailability()`).
- **Asynchronous:** A context emits a domain event that another context subscribes to (e.g., `DonationRecorded` event triggers `NotificationsService.sendDonorThankYou()`).

Domain events are implemented via Postgres LISTEN/NOTIFY (for real-time) and a `domain_events` table (for durability and replay). n8n subscribes to relevant events via webhooks.

### 4.3 Aggregate Design

Each bounded context has one or more aggregates. An aggregate is a cluster of domain objects treated as a single unit for data changes. Each aggregate has a root entity and a boundary.

| Aggregate | Root Entity | Boundary Includes | Consistency |
|---|---|---|---|
| Donor | `Donor` | Medical screening, communication preferences, emergency availability | Strong (transactional) |
| Donation | `Donation` | Donation record, outcome, linked bag | Strong |
| BloodBag | `BloodBag` | State transitions, location, expiry | Strong (state machine enforced) |
| BloodRequest | `BloodRequest` | Request lines, allocations, status history | Strong |
| Delivery | `Delivery` | Dispatch record, status updates, confirmation | Strong |
| Volunteer | `Volunteer` | Profile, availability, task assignments | Strong |
| Task | `Task` | Assignment, check-in/out, outcome | Strong |
| Camp | `Camp` | Schedule, assigned volunteers, registered donors, post-camp report | Strong |
| Patient | `Patient` | Profile, documents, case history | Strong |
| Hospital | `Hospital` | Profile, staff accounts, partnership status | Strong |
| User | `User` | Account, roles, sessions | Strong |
| Notification | `Notification` | Template instance, delivery attempts, status | Strong |
| Report | `Report` | Generation run, content, approval, publication | Strong |
| AuditEntry | `AuditEntry` | Actor, action, before/after, timestamp | Append-only |

Cross-aggregate consistency is eventual, achieved via domain events. For example, recording a donation (Donation aggregate) updates donor eligibility (Donor aggregate) via an event, not in the same transaction.

---

## 5. Component Architecture

### 5.1 Web Application (Next.js)

The web application is a single Next.js 15 codebase using the App Router, deployed to Vercel. It serves three distinct surfaces:

- **Public website** — unauthenticated, SSR + static, cacheable at the CDN.
- **Admin/Patient/Hospital dashboard** — authenticated, client-rendered with SSR for initial load, real-time via Supabase Realtime.
- **API routes** — server-side endpoints for operations that cannot be done client-side (webhook receivers, complex workflows, file processing).

#### 5.1.1 Routing Architecture

```
src/app/
├── (public)/                  # Public website (no auth)
│   ├── page.tsx               # Home
│   ├── about/page.tsx
│   ├── request-blood/page.tsx
│   ├── donate-blood/page.tsx
│   ├── volunteer/page.tsx
│   ├── knowledge-base/
│   ├── impact/page.tsx
│   └── layout.tsx
├── (auth)/                    # Auth flows
│   ├── login/page.tsx
│   ├── register/page.tsx
│   ├── forgot-password/page.tsx
│   └── layout.tsx
├── (dashboard)/               # Authenticated dashboard
│   ├── admin/                 # SUPER_ADMIN, ADMIN
│   ├── hospital/              # HOSPITAL
│   ├── patient/               # PATIENT
│   ├── donor/                 # DONOR (web view; primary is mobile app)
│   ├── volunteer/             # VOLUNTEER (web view; primary is mobile app)
│   └── layout.tsx             # Auth guard, sidebar, header
├── api/                       # API routes (server-side)
│   ├── webhooks/              # Webhook receivers (HMAC-verified)
│   ├── reports/               # Report generation
│   └── internal/              # Internal admin operations
└── layout.tsx                 # Root layout
```

Route groups (`(public)`, `(auth)`, `(dashboard)`) organize routes without affecting URLs. Each group has its own layout for shared UI and middleware for auth enforcement.

#### 5.1.2 Middleware

Next.js middleware enforces authentication and authorization before routes are rendered.

- Public routes: no auth check, but CSP and security headers are applied.
- Auth routes: redirect to dashboard if already authenticated.
- Dashboard routes: validate JWT, redirect to login if invalid, check role against route, redirect to `/403` if unauthorized.

Middleware is thin — it does not call the database. Authorization decisions that require data (e.g., "is this user allowed to see this specific request?") are made by RLS at the database level.

#### 5.1.3 Server Components vs Client Components

- **Server Components (default):** Data fetching, static rendering, SEO-critical content. Do not use `use client`.
- **Client Components:** Interactivity, real-time subscriptions, form state. Marked with `"use client"` at the top of the file.
- **Rule:** Client Components are leaf nodes. They receive data via props from Server Components and emit events via callbacks. They do not fetch data directly except via the typed Supabase client or React Query hooks that wrap the typed client.

### 5.2 Mobile Application (Expo)

The mobile app is built with Expo React Native, sharing code with the web app where possible.

#### 5.2.1 Shared Code

The following are shared between web and mobile via a `src/shared/` directory:

- Type definitions (domain entities, DTOs).
- Validation schemas (Zod).
- Pure business logic (eligibility calculation, blood type compatibility, state machine transitions).
- API client wrappers (typed Supabase client).
- Domain event types.

The following are NOT shared:
- UI components (web uses Shadcn/Tailwind; mobile uses React Native Paper or Tamagui).
- Routing (web uses Next.js App Router; mobile uses Expo Router).
- Platform-specific integrations (push notifications, camera, geolocation).

#### 5.2.2 Mobile App Surfaces

The mobile app serves two roles:

- **Donor app:** registration, profile, donation history, eligibility, impact, emergency availability toggle, push notifications.
- **Volunteer app:** task list, accept/decline, check-in/out, donor registration at camps, donation recording at camps.

Patients and guardians use the responsive web app (no native app in v1) because their interaction is episodic and document-heavy, better suited to web.

### 5.3 Database (PostgreSQL via Supabase)

The database is the authoritative state of the platform. It is hosted on Supabase (managed PostgreSQL 15) in the Singapore region.

#### 5.3.1 Schema Organization

The database schema is organized by bounded context, with each context's tables prefixed for clarity:

- `auth.*` — managed by Supabase Auth (do not modify directly).
- `public.users_*` — Identity & Access context.
- `public.donors_*`, `public.donations_*` — Donor context.
- `public.blood_bags_*`, `public.storage_locations_*` — Inventory context.
- `public.requests_*`, `public.allocations_*`, `public.deliveries_*` — Request Fulfillment context.
- `public.volunteers_*`, `public.tasks_*`, `public.camps_*` — Volunteer context.
- `public.patients_*`, `public.hospitals_*` — Patient/Hospital context.
- `public.notifications_*`, `public.reports_*`, `public.kb_*` — Content & Communication context.
- `public.audit_*`, `public.settings_*` — Platform Operations context.

A complete schema with DDL is in the TDS.

#### 5.3.2 Row-Level Security

RLS is enabled on every table containing user data. Every policy is documented with its purpose, the role it applies to, and the test that verifies it.

Example RLS policy pattern:
```sql
-- Donors can read their own donor record
CREATE POLICY donors_self_read ON public.donors
  FOR SELECT TO authenticated
  USING (user_id = auth.uid());

-- Admins can read all donor records
CREATE POLICY donors_admin_read ON public.donors
  FOR SELECT TO authenticated
  USING (
    EXISTS (SELECT 1 FROM public.user_roles
            WHERE user_id = auth.uid()
            AND role IN ('SUPER_ADMIN', 'ADMIN'))
  );
```

The full RLS policy set is defined in the TDS.

#### 5.3.3 Triggers

Triggers are used sparingly, only where the database is the only place to enforce a rule reliably:

- `updated_at` auto-update on every table.
- Audit log insertion on sensitive tables (state transitions, role changes).
- Soft-delete timestamp setting.
- Domain event emission (insert into `domain_events` table).

Triggers are NOT used for:
- Business logic that could be in the application layer.
- Cross-aggregate consistency (use application-level events instead).
- Computed values that could be in a materialized view.

### 5.4 Edge Functions

Edge Functions (Deno, hosted on Supabase) are used for operations that cannot be done client-side or that require service-role privileges.

#### 5.4.1 When to Use Edge Functions

- **Webhook receivers.** n8n calls back to the platform via an Edge Function with HMAC verification.
- **Sensitive operations.** Anything that requires the service-role key (which bypasses RLS) — e.g., bulk data exports, cross-tenant operations (none in v1 but designed for).
- **File processing.** Virus scanning, image resizing, PDF generation.
- **Complex queries.** Multi-step operations that cannot be expressed as a single PostgREST call.
- **AI content generation.** Calls to the AI service are made from Edge Functions to keep API keys server-side.

#### 5.4.2 Edge Function Security

- Every Edge Function verifies the JWT from the request header.
- Every Edge Function re-checks authorization; it does not trust the JWT's role claim alone for sensitive operations.
- Service-role key is in environment variables, never in client code.
- Edge Functions return structured errors with correlation IDs.
- Edge Functions are rate-limited via Supabase's built-in rate limiting.

### 5.5 File Storage

File storage uses Supabase Storage (S3-compatible). Files are organized into buckets:

| Bucket | Contents | Access |
|---|---|---|
| `patient-documents` | Prescriptions, hospital letters, lab reports | Private; RLS-based access |
| `donor-medical` | Medical screening forms | Private; donor + admin only |
| `camp-photos` | Photos from blood camps (for marketing) | Private; admin + volunteer who uploaded |
| `public-assets` | Public website images, logos | Public read |
| `reports` | Generated PDF reports | Private; admin only, public bucket for published |
| `avatars` | User profile pictures | Private; owner + admin |

Every file upload goes through:
1. Client requests a signed upload URL from an Edge Function.
2. Edge Function verifies auth, generates signed URL with 5-minute expiry.
3. Client uploads directly to S3 via signed URL.
4. Client notifies Edge Function of completion.
5. Edge Function triggers virus scan (ClamAV via n8n workflow).
6. On clean scan, file is marked available; on infected, file is quarantined and admin alerted.

### 5.6 Realtime

Supabase Realtime (Postgres CDC over WebSocket) is used for live updates:

- **Inventory changes:** Admin dashboard inventory view updates in real time.
- **Request status:** Requester sees status changes without refresh.
- **Notifications:** In-app notification badge updates.
- **Volunteer task assignments:** Volunteer app shows new tasks immediately.

Realtime subscriptions are scoped by RLS — a user only receives events for rows they can read. The client subscribes via the typed Supabase client, with auto-reconnect and backoff.

---

## 6. Data Architecture

### 6.1 Data Ownership

Every piece of data has exactly one owning context. Other contexts access it via the owning context's published interface, never by direct table access.

| Data | Owner | Accessors |
|---|---|---|
| User accounts, roles | Identity & Access | All contexts (via auth context) |
| Donor profiles | Donor Management | Inventory (for collection), Notifications (for reminders) |
| Donation records | Donor Management | Inventory (for bag creation), Reports |
| Blood bags | Inventory | Requests (for allocation), Reports |
| Blood requests | Request Fulfillment | Inventory (for allocation), Notifications |
| Deliveries | Request Fulfillment | Notifications |
| Volunteer profiles | Volunteer Management | Camps (for assignment), Reports |
| Tasks | Volunteer Management | Notifications |
| Camps | Volunteer Management | Donor Management (for donation records), Reports |
| Patients | Request Fulfillment | (None — encapsulated) |
| Hospitals | Request Fulfillment | (None — encapsulated) |
| Notifications | Content & Communication | (None — encapsulated) |
| Reports | Content & Communication | (None — encapsulated) |
| Audit logs | Platform Operations | (Read-only by SUPER_ADMIN) |
| Settings | Platform Operations | All contexts (read-only) |

### 6.2 Data Lifecycle

Every record has a defined lifecycle from creation to disposal:

1. **Creation** — Record is created with `created_at`, `created_by`, and initial state.
2. **Active use** — Record is read, updated, and transitioned through its state machine.
3. **Soft delete** — When no longer active, record is marked `deleted_at` (not physically removed). Soft-deleted records are excluded from default queries.
4. **Archival** — After a configurable period, records are moved to archive tables (same schema, separate physical storage) for query performance.
5. **Disposal** — After the retention period, records are securely deleted. Disposal is logged.

The lifecycle is enforced by scheduled n8n workflows (`AUT-017` audit log archival, plus per-context archival workflows).

### 6.3 Consistency Model

- **Strong consistency** within an aggregate (single Postgres transaction).
- **Eventual consistency** across aggregates (via domain events, typically < 5s lag).
- **Read-after-write consistency** for the same user (Supabase uses primary for writes, replicas for reads with < 1s lag; for v1, primary-only reads to guarantee consistency).
- **Cache invalidation** is event-driven — when a record changes, a domain event triggers cache invalidation for affected queries.

### 6.4 Data Migration

Schema changes are via versioned migrations (Supabase migration files or `dbmate`, decided in TDS). Every migration has:

- Up-migration (apply change).
- Down-migration (revert change, where possible).
- Data migration script (if data transformation is needed).
- Test (verify migration works on a copy of production data).

Migrations are run in CI on a fresh database for every PR. Production migrations are run by a deploy script with rollback on failure.

### 6.5 Backup & Recovery

- **Automated daily backups** by Supabase (managed service, 7-day retention on free tier).
- **Weekly logical backup** (pg_dump) to off-site storage (Cloudflare R2 free tier) for longer retention.
- **Point-in-time recovery** is not available on Supabase free tier; RPO is 24 hours.
- **Restore procedure** is documented and tested quarterly via DR drill.

---

## 7. Security Architecture

### 7.1 Threat Model

The platform's threat model considers the following adversaries and attack vectors:

| Adversary | Capability | Motivation |
|---|---|---|
| Unauthenticated attacker | Internet access | Defacement, data theft, service disruption |
| Authenticated user (low privilege) | Valid account, can submit requests | Privilege escalation, data exfiltration |
| Malicious insider (admin) | Admin access | Data theft, tampering, cover-up |
| Compromised credentials | Stolen JWT or password | Account takeover |
| Compromised vendor | Supabase/Vercel account breach | Mass data breach |
| Misconfigured client | Vulnerable mobile app | Data leakage |
| Social engineering | Manipulation of staff | Credential theft, unauthorized actions |

### 7.2 Security Controls by Layer

#### 7.2.1 Network Layer

- **TLS 1.2+ everywhere** (TLS 1.3 preferred). HTTP requests redirect to HTTPS.
- **HSTS** header with preload.
- **Cloudflare** in front of Vercel for DDoS protection and WAF (free tier).
- **Supabase** restricts API access to specific IPs via Cloudflare Tunnel (post-v1; v1 uses standard HTTPS).
- **n8n** is not internet-exposed; it sits behind Cloudflare Tunnel with HMAC-signed webhooks.

#### 7.2.2 Application Layer

- **Content Security Policy** strict mode: `default-src 'self'; script-src 'self' 'nonce-<random>'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' https://*.supabase.co;`.
- **X-Frame-Options: DENY** (no clickjacking).
- **X-Content-Type-Options: nosniff**.
- **Referrer-Policy: strict-origin-when-cross-origin**.
- **Permissions-Policy** restricting camera, microphone, geolocation to specific routes.
- **Output sanitization**: All user-generated content rendered as text by default; HTML rendering uses `dompurify` with strict allowlist.
- **Input validation**: All inputs validated by Zod schemas at the API boundary; rejected inputs return 400 with descriptive error.
- **SQL injection**: Supabase client uses parameterized queries; no raw SQL in feature code.

#### 7.2.3 Authentication Layer

- **Password hashing**: Argon2id via Supabase Auth (default config).
- **Password policy**: min 12 chars, must include upper, lower, digit, symbol; cannot be in common-passwords list.
- **MFA**: TOTP-based, required for ADMIN and SUPER_ADMIN roles, optional for others.
- **Session**: JWT access token (15-minute expiry) + refresh token (30-day expiry, rotating). Refresh tokens are single-use; reuse indicates theft and invalidates the session.
- **Account lockout**: 5 failed attempts lock the account for 15 minutes and notify the user; 3 locks in 24 hours require admin unlock.
- **Password reset**: expiring token (10-minute expiry), single-use, notified to user via email.

#### 7.2.4 Authorization Layer

- **RBAC**: Seven roles (per PRD §5). Role assignment is in `user_roles` table.
- **RLS**: Every table with user data has RLS policies enforcing row-level access based on JWT claims.
- **Permission checks**: Feature code does not rely on RLS alone; it also checks permissions via `can(user, action, resource)` helper from the auth context.
- **Defense in depth**: Even if RLS is misconfigured, the application layer checks permissions; even if the application layer has a bug, RLS prevents unauthorized data access.

#### 7.2.5 Data Layer

- **Encryption at rest**: Supabase manages database-level encryption (free tier: managed transparent encryption).
- **Column-level encryption**: PHI fields (patient name, disease, medical reports) are encrypted with application-managed keys via pgcrypto (or Supabase Vault for v1).
- **Encryption in transit**: TLS 1.2+ for all connections (client → Supabase, n8n → Supabase).
- **Backups**: Backup files are encrypted at rest in off-site storage.

#### 7.2.6 Audit Layer

- **Audit table**: Append-only `audit_log` table with trigger-based insertion on sensitive tables.
- **Audit fields**: Every record has `created_at`, `created_by`, `updated_at`, `updated_by`, `deleted_at`, `deleted_by`.
- **Audit log entry**: actor, action, entity type, entity ID, before-state (JSON), after-state (JSON), timestamp, IP address, user agent.
- **Tamper evidence**: Audit log inserts are via SECURITY DEFINER function; SELECT is granted only to SUPER_ADMIN; DELETE and UPDATE are revoked from all roles.
- **Retention**: 7 years per regulatory requirement.

#### 7.2.7 File Upload Layer

- **Signed URLs**: All file access via signed URLs with short expiry (5 minutes for upload, 1 hour for download).
- **Virus scanning**: Every uploaded file is scanned by ClamAV (via n8n workflow) before being marked available. Infected files are quarantined.
- **File type validation**: Whitelist of allowed MIME types and extensions per bucket.
- **Size limits**: Per-bucket size limits (e.g., patient documents max 10MB, photos max 5MB).
- **Filename sanitization**: Stored filenames are UUID-generated; original filename kept as metadata only.

#### 7.2.8 Rate Limiting

- **Public endpoints**: 100 requests/minute per IP (Cloudflare WAF rate limit rule).
- **Authenticated endpoints**: 1000 requests/minute per user (application-level rate limit via Redis or Supabase Edge Function with KV store).
- **Auth endpoints**: 5 attempts per minute per IP for login, register, password reset (Supabase Auth built-in).
- **Webhook receivers**: 100 requests/second per webhook (with burst capacity).

#### 7.2.9 Secret Management

- **Environment variables**: All secrets in environment variables, never in code.
- **`.env.example`**: Template with descriptive names, no real values, committed to repo.
- **`.env.local`**: Real values, never committed, in `.gitignore`.
- **Production secrets**: Stored in Vercel environment variables (encrypted at rest), Supabase Edge Function secrets, and n8n credentials.
- **Key rotation**: Documented procedure for rotating each secret; JWT signing key rotation requires session invalidation.

### 7.3 Security Testing

- **SAST**: ESLint security plugin, CodeQL via GitHub Actions.
- **Dependency scanning**: Dependabot, npm audit in CI.
- **Secret scanning**: Git-secrets, GitHub secret scanning.
- **DAST**: OWASP ZAP scan in CI (weekly).
- **RLS test suite**: Dedicated tests that attempt unauthorized access and verify denial.
- **Penetration test**: Before launch and annually thereafter.

---

## 8. Integration Architecture

### 8.1 Integration Inventory

| Integration | Direction | Purpose | Mechanism |
|---|---|---|---|
| Web App ↔ Supabase | Bidirectional | Data, auth, realtime | Supabase JS SDK over HTTPS/WSS |
| Mobile App ↔ Supabase | Bidirectional | Data, auth, realtime | Supabase JS SDK over HTTPS/WSS |
| Supabase → n8n | Outbound | Trigger workflows on data events | Database webhooks (HMAC-signed) |
| n8n → Supabase | Inbound | Workflow actions (create/update records) | Edge Functions (JWT service role) |
| n8n → Resend | Outbound | Send email | Resend REST API |
| n8n → WhatsApp Cloud API | Outbound | Send WhatsApp messages | Meta Graph API |
| n8n → SMS Gateway | Outbound | Send SMS (paid, opt-in) | REST API (provider-specific) |
| n8n → AI Service | Outbound | Content generation | REST API (provider-specific) |
| n8n → GitHub | Outbound | Commit reports to repo | GitHub REST API |
| Cloudflare → Vercel | Inbound | DNS, CDN, WAF | DNS + reverse proxy |
| Sentry ← All | Inbound | Error tracking | Sentry SDK |

### 8.2 Webhook Architecture

n8n workflows are triggered by webhooks from Supabase. The webhook flow:

1. **Database event** — A row is inserted/updated/deleted in a watched table.
2. **Supabase webhook** — Configured in Supabase dashboard, sends HTTP POST to n8n webhook URL.
3. **HMAC signature** — Webhook payload is signed with a shared secret; signature in `X-Supabase-Signature` header.
4. **n8n verification** — n8n verifies the HMAC signature before processing; rejects on mismatch.
5. **Idempotency** — Webhook payload includes an event ID; n8n checks an idempotency table to skip already-processed events.
6. **Response** — n8n responds 200 OK if accepted, 409 Conflict if duplicate, 500 on processing error (triggers Supabase retry).

### 8.3 AI Service Integration

AI service is called from n8n workflows (for content generation automations) and from Edge Functions (for on-demand AI assistance like search and alt text).

**Architecture:**
- AI service is abstracted behind an `AIClient` interface in the domain layer.
- Concrete implementations: `GroqClient` (free tier LLM), `LocalLLMClient` (self-hosted via Ollama, post-v1).
- API keys are stored in n8n credentials (for n8n workflows) and Supabase Edge Function secrets (for Edge Functions).
- All AI calls include a system prompt enforcing the AI boundaries (per PRD §10).
- All AI output is marked `ai_generated: true` and `reviewed: false` until human review.

### 8.4 External Service Failure Handling

Every external service integration has a failure handling strategy:

| Service | Failure Mode | Handling |
|---|---|---|
| Supabase | Transient (network) | Retry with exponential backoff (3 attempts) |
| Supabase | Outage | Read from cache (read-only mode), alert admins |
| n8n | Workflow failure | Retry 3×, then alert admin; workflow state is recoverable |
| Resend (email) | Bounce / complaint | Mark recipient as suppressed, do not retry |
| WhatsApp | Rate limit / failure | Queue and retry with backoff |
| AI service | Timeout / error | Fall back to template-based content (no AI), alert admin |
| SMS gateway | Failure | Fall back to WhatsApp or email, alert admin |

---

## 9. Authentication & Authorization Architecture

### 9.1 Authentication Flow

#### 9.1.1 Registration Flow (Email + Password)

```
User → Web/Mobile App
  ↓
Submit registration form (email, password, profile data)
  ↓
Client calls supabase.auth.signUp()
  ↓
Supabase Auth creates user (unconfirmed), sends confirmation email
  ↓
User clicks confirmation link
  ↓
Supabase Auth confirms user, redirects to app
  ↓
App creates profile record in public.users (via trigger on auth.users)
  ↓
App assigns default role (DONOR / VOLUNTEER / PATIENT / HOSPITAL based on registration flow)
  ↓
User logs in with email + password
  ↓
Supabase Auth returns JWT access token (15min) + refresh token (30d)
  ↓
Client stores tokens (httpOnly cookie for web, secure storage for mobile)
  ↓
Client uses JWT for all subsequent requests
```

#### 9.1.2 Registration Flow (Phone OTP)

```
User submits phone number
  ↓
Client calls supabase.auth.signInWithOtp({ phone })
  ↓
Supabase Auth sends OTP via SMS
  ↓
User enters OTP
  ↓
Client calls supabase.auth.verifyOtp({ phone, token })
  ↓
Supabase Auth returns JWT
  ↓
(If new user) App prompts for profile data, creates profile
```

#### 9.1.3 Login Flow

```
User submits email + password
  ↓
Client calls supabase.auth.signInWithPassword()
  ↓
Supabase Auth validates credentials
  ↓
(If MFA enabled) Client calls supabase.auth.mfa.challenge()
  ↓
User enters TOTP code
  ↓
Client calls supabase.auth.mfa.verify()
  ↓
Supabase Auth returns JWT
  ↓
Client stores tokens, redirects to dashboard
```

#### 9.1.4 Session Refresh

```
Client makes API call with expired access token
  ↓
Supabase returns 401
  ↓
Client calls supabase.auth.refreshSession() with refresh token
  ↓
Supabase Auth validates refresh token, issues new access token + new refresh token
  ↓
(Old refresh token invalidated — rotation)
  ↓
Client retries original request with new token
```

If refresh fails (invalidated refresh token), client redirects to login.

### 9.2 Authorization Architecture

#### 9.2.1 Role Storage

Roles are stored in `public.user_roles` table:
```sql
CREATE TABLE public.user_roles (
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  role TEXT NOT NULL CHECK (role IN ('SUPER_ADMIN','ADMIN','VOLUNTEER','HOSPITAL','DONOR','PATIENT')),
  granted_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  granted_by UUID REFERENCES auth.users(id),
  revoked_at TIMESTAMPTZ,
  PRIMARY KEY (user_id, role)
);
```

A user's *effective roles* are all rows where `revoked_at IS NULL`. The JWT custom claim `user_roles` contains this list, refreshed on each login.

#### 9.2.2 Permission Model

Permissions are defined as `(role, action, resource)` triples. The authorization service exposes:

```typescript
can(user: AuthenticatedUser, action: Action, resource: Resource): boolean
```

Permissions are defined in a central table (`src/features/auth/permissions.ts`) and tested by unit tests. RLS policies mirror these permissions at the database level.

#### 9.2.3 Context-Based Authorization

Some authorization decisions require context — e.g., "can this user see *this specific* request?" The platform uses:

- **RLS for row-level access** — the database filters rows based on the user's identity.
- **Application-level checks for action authorization** — e.g., "can this user *cancel* this request?" is checked in the application layer based on the request's current state and the user's relationship to it.

#### 9.2.4 Multi-Role Users

A user with multiple roles has all roles in their JWT. The application defaults to the most permissive role for the current route, but RLS policies are designed so that the *least permissive applicable policy* applies when roles overlap (e.g., an admin viewing their own donor record is treated as a donor for that record).

### 9.3 MFA Architecture

- **Enrollment:** User scans TOTP QR code (Google Authenticator, Authy, etc.) with `supabase-auth-js` MFA API. Backup codes generated.
- **Challenge:** On login, after password validation, MFA challenge issued. User enters TOTP code. Verified before JWT issued.
- **Enforcement:** ADMIN and SUPER_ADMIN roles MUST have MFA enrolled within 7 days of role assignment; failure to enroll suspends the role.
- **Recovery:** If user loses TOTP device, SUPER_ADMIN can reset MFA after identity verification (documented procedure).

---

## 10. Automation Architecture

> **PHASE NOTE.** Automation is **Phase 3**. Phase 1 and Phase 2 ship with **zero automation infrastructure**. The platform uses the **Service Interface pattern** (§10.1) so that Phase 1–2 implement notifications, reports, and backups as plain TypeScript services called directly from application code, and Phase 3 swaps those implementations for n8n-backed adapters **without changing call sites**. n8n is **never a hard dependency** — disabling n8n reverts to the in-process service implementations.

### 10.1 Service Interface Pattern (Mandatory)

The core application calls **interfaces**, never concrete automation implementations. Each interface has at least one in-process implementation (Phase 1+) and may have an n8n-backed implementation (Phase 3+).

**Required interfaces** (defined in `packages/domain/src/services/`):

```typescript
// The application calls this interface everywhere it needs to notify a user.
interface NotificationService {
  send(notification: NotificationInput): Promise<NotificationResult>;
}

// The application calls this interface everywhere it needs to generate a report.
interface ReportService {
  generate(spec: ReportSpec): Promise<ReportResult>;
}

// The application calls this interface to run a backup.
interface BackupService {
  run(): Promise<BackupResult>;
  verify(backupId: string): Promise<VerificationResult>;
}

// The application calls this interface to generate AI-assisted content.
// In Phase 1-2, this interface throws "not implemented" — AI is Phase 3+.
interface AIContentService {
  draftContent(spec: ContentSpec): Promise<ContentDraft>;
}
```

**Phase 1–2 implementations** (in `infrastructure/services/in-process/`):

- `InProcessNotificationService` — calls `EmailService` (Resend) directly, or no-op if email not configured.
- `InProcessReportService` — generates reports synchronously using database queries + a templating engine.
- `InProcessBackupService` — runs `pg_dump` via `child_process.exec`, stores locally.
- `AIContentService` — throws `NotImplementedError` (Phase 1–2 have no AI).

**Phase 3 implementations** (in `infrastructure/services/n8n/`, added in Phase 3):

- `N8nNotificationService` — POSTs to an n8n webhook that handles multi-channel dispatch (email, WhatsApp, SMS).
- `N8nReportService` — POSTs to an n8n webhook that runs the report workflow (with AI-assisted narrative).
- `N8nBackupService` — POSTs to an n8n webhook that runs the backup workflow (with off-site copy).
- `GroqAIContentService` / `LocalOllamaAIContentService` — calls the configured AI provider.

**Selection mechanism:**

The active implementation is selected by environment variable at startup:

```typescript
// infrastructure/services/index.ts
function createNotificationService(): NotificationService {
  const provider = process.env.NOTIFICATION_PROVIDER ?? 'in-process';
  switch (provider) {
    case 'in-process': return new InProcessNotificationService();
    case 'n8n':        return new N8nNotificationService();
    default: throw new Error(`Unknown NOTIFICATION_PROVIDER: ${provider}`);
  }
}
```

**Rules:**

1. Feature code (`apps/web/src/features/**`) imports only the **interface** from `@marwat/domain/services`. It never imports the concrete implementation.
2. The concrete implementation is wired in `apps/web/src/lib/services/container.ts` (a thin DI container) and injected into features via React context or function parameters.
3. Switching from `in-process` to `n8n` is a single environment variable change. No code change. No redeploy of feature code (only the service container).
4. If the n8n implementation fails (n8n unreachable, workflow error), the system MUST fall back to the in-process implementation and log the failure. **The platform never breaks because n8n is down.**

### 10.2 n8n Deployment (Phase 3 Only)

n8n is self-hosted on Railway (free tier) or Fly.io (free tier). It is not exposed to the public internet; it sits behind Cloudflare Tunnel. Only Supabase (via webhook) and the platform (via authenticated Edge Function call) can reach it.

**Architecture:**
```
Supabase webhook → Cloudflare Tunnel → n8n → External services
                        ↑
                  HMAC verification
```

**Persistence:** n8n uses its own internal SQLite database (free tier) for workflow state. For production durability, n8n's database is backed up daily to off-site storage.

**Phase 1–2 alternative:** No n8n. The `InProcessNotificationService` runs inside the Next.js process. Scheduled tasks use a lightweight in-process scheduler (e.g., `node-cron` or a `setInterval`-based loop) — acceptable for Phase 1–2 scale (a single office, dozens of users).

### 10.3 Workflow Design (Phase 3)

Every n8n workflow follows a standard structure:

1. **Trigger node** — Webhook (from Supabase) or Schedule (cron).
2. **Verification node** — For webhooks, HMAC signature verification. For schedules, none.
3. **Idempotency check** — Query `workflow_executions` table to skip duplicates.
4. **Fetch data** — Query Supabase for any additional context needed.
5. **Business logic** — Execute workflow-specific logic.
6. **Action** — Send notification, generate report, update record, etc.
7. **Log** — Insert into `workflow_executions` table with status, duration, output.
8. **Error handling** — On any node failure, log error, retry 3× with backoff, alert admin on persistent failure.

### 10.4 Workflow Inventory (Phase 3)

Workflows are version-controlled as JSON files in `automations/n8n/`. Each workflow has a README documenting purpose, trigger, inputs, outputs, and failure handling. The full inventory is in PRD §9.2.

### 10.5 Webhook Security (Phase 3)

- Every webhook from Supabase to n8n includes an HMAC-SHA256 signature in `X-Supabase-Signature` header.
- The shared secret is stored in n8n credentials (encrypted at rest by n8n).
- n8n verifies the signature before processing; rejects with 401 on mismatch.
- The webhook URL includes a random token (e.g., `https://n8n.internal/webhook/donation-recorded/<token>`) as defense in depth.
- Webhook retries by Supabase are idempotent (event ID checked).

### 10.6 Workflow Observability (Phase 3)

- Every workflow execution is logged in `workflow_executions` table: workflow ID, trigger, start time, end time, status (success/failure), error message, input summary, output summary.
- The admin dashboard shows: recent executions, failure rate, average duration, currently-running workflows.
- Failed workflows trigger an admin alert (in-app + email) with a retry button.
- A `dry-run` mode executes the workflow without side effects, returning the would-be output for review.

### 10.7 Graceful Degradation

The platform MUST degrade gracefully when automation is unavailable:

| Scenario | Behavior |
|---|---|
| n8n not deployed (Phase 1–2) | `InProcess*` services handle all calls. No degradation. |
| n8n deployed but unreachable | `N8n*` services catch errors, log, fall back to `InProcess*`. Admin alerted. |
| n8n workflow fails | Workflow logs error, sets `workflow_executions.status = 'failed'`. Admin alerted. Underlying data is unchanged (idempotency). |
| AI service unavailable | `AIContentService` throws `ServiceUnavailableError`. Caller catches and either falls back to a template-based content generator or skips the AI feature. **The platform never blocks on AI.** |
| Email provider (Resend) unavailable | `EmailService` queues the email in `notifications` table with `status = 'PENDING'`. A retry worker attempts resend with exponential backoff. |

---

## 11. Deployment Architecture

> **PHASE NOTE.** Deployment evolves across phases. **Phase 1** deploys to a single machine (founder's laptop or office PC) using Docker Compose — zero cloud dependency. **Phase 2** optionally adds Vercel (web hosting) + Supabase cloud (database). **Phase 3** adds n8n. **Phase 4** adds multi-region and high availability. The same application code runs in all deployment topologies; only environment variables change.

### 11.1 Deployment Topologies

The platform supports **four deployment topologies**, all from the same codebase:

| Topology | Phase | Database | Auth | Storage | App Hosting | Use Case |
|---|---|---|---|---|---|---|
| **Solo laptop** | P1+ | Local Docker (Postgres) | Local Docker (GoTrue) | Local Docker (MinIO) | `next dev` or `next start` on `localhost` | Founder development, demo, very early Phase 1 production |
| **Office LAN** | P1+ | Office PC Docker | Office PC Docker | Office PC Docker | Office PC `next start` on LAN IP | Staff use during the day; no internet dependency |
| **Cloud (Supabase + Vercel)** | P2+ | Supabase cloud (free tier) | Supabase cloud | Supabase Storage | Vercel (free tier) | Public portals, remote access, mobile app backend |
| **Hybrid** | P2+ | Office PC + Supabase cloud replica | Both | Both | Office PC + Vercel | Office runs locally; cloud provides public access + remote |

**The Hybrid topology** is the recommended production setup from Phase 2 onward: the office PC runs the local stack for speed and offline resilience, while Supabase cloud + Vercel provide public access and remote availability. Data sync between local and cloud is via Supabase's replication feature (Phase 4) or via a scheduled sync script (Phase 2–3).

### 11.2 Preferred Deployment Order

When bringing the platform online, the founder follows this order. Each step is **optional** — the platform is usable at every step.

1. **Local development** — `docker compose up` starts Postgres + GoTrue + MinIO + Realtime. `pnpm dev` starts Next.js on `localhost:3000`. Zero cloud accounts needed.
2. **GitHub repository** — Push the code to GitHub (free tier). Enables version control and basic GitHub Actions CI.
3. **Office LAN deployment** — Run the same Docker Compose stack on the office desktop PC. Staff access via `http://192.168.1.10:3000` from their machines. Still zero cloud dependency.
4. **Supabase cloud (optional)** — Create a Supabase free-tier project. Point `SUPABASE_URL` and related env vars at the cloud project. Enables remote access to the database.
5. **Vercel (optional)** — Deploy Next.js to Vercel free tier. Enables public access to the web app and public website.
6. **Mobile app (Phase 4)** — EAS Build (Expo Application Services) free tier. Distribute via Play Store.
7. **Automation (Phase 3)** — Self-host n8n on Railway/Fly.io free tier. Configure `NOTIFICATION_PROVIDER=n8n` and related env vars.

### 11.3 Environments

| Environment | Purpose | Phase | Hosting | Data |
|---|---|---|---|---|
| **Local** | Founder development & solo-laptop production | P1+ | Docker Compose on laptop | Seeded synthetic data (dev) or real data (production) |
| **Office LAN** | Staff production | P1+ | Docker Compose on office PC | Real data |
| **Preview** | PR preview | P2+ | Vercel preview + Supabase staging project | Anonymized production subset |
| **Staging** | Pre-production testing | P2+ | Vercel staging + Supabase staging project | Anonymized production subset |
| **Production (cloud)** | Public production | P2+ | Vercel prod + Supabase prod project | Real data |

During Phase 1, only Local and Office LAN environments exist. The founder commits directly to `main` (with self-review) and deploys by `git pull && docker compose up -d` on the office PC. This is acceptable for a solo-founder internal-use Phase 1.

### 11.4 CI/CD Pipeline (Phase 1 — Minimal)

Phase 1 ships with a **minimal** GitHub Actions workflow. The goal is to catch obvious errors, not to enforce enterprise-grade gates.

```yaml
# .github/workflows/ci.yml (Phase 1)
name: CI
on: [push, pull_request]
jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env: { POSTGRES_PASSWORD: postgres }
        ports: ['5432:5432']
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: pnpm }
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm typecheck
      - run: pnpm test:unit
      - run: pnpm test:rls
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/postgres
```

**Phase 2+ adds:**
- Build step (`pnpm build`).
- Integration tests.
- E2E tests (Playwright).
- Security scan (Dependabot, npm audit).
- Preview deployment to Vercel (on PR).
- Production deployment to Vercel (on push to `main`, with manual approval).

**Phase 4 adds:**
- Blue-green deployment.
- Canary releases.
- Multi-environment promotion.

The CI/CD pipeline NEVER blocks Phase 1 deployment to the office PC. If GitHub Actions is down, the founder can deploy by `git pull && docker compose up -d` directly.

### 11.5 Database Migration

```
Migration file added to packages/database/migrations/
  ↓
CI runs migration on fresh database, verifies schema matches expected
  ↓
Founder reviews the migration SQL
  ↓
Migration applied to local database (pnpm db:migrate)
  ↓
Migration applied to office PC database (ssh + pnpm db:migrate, or run on office PC directly)
  ↓
(Phase 2+) Migration applied to Supabase cloud staging, then production
  ↓
If migration fails on production → automatic rollback + alert
```

### 11.6 Mobile App Release (Phase 4 Only)

```
Code merged to main
  ↓
EAS Build (Expo Application Services) free tier
  ↓
Internal testing track
  ↓
Manual QA
  ↓
Production release to Play Store
  ↓
Phased rollout (10% → 50% → 100% over 3 days)
```

### 11.7 n8n Workflow Deployment (Phase 3 Only)

```
Workflow JSON updated in automations/n8n/
  ↓
CI validates JSON structure
  ↓
Founder reviews the workflow
  ↓
Workflow imported to n8n via n8n REST API
  ↓
Manual test (dry-run mode)
  ↓
Workflow activated
  ↓
If new workflow fails → deactivate and revert to previous version
```

### 11.8 Environment Variables

Every environment has its own set of environment variables, managed in:

- **Local / Office LAN:** `.env.local` (in `.gitignore`), loaded by Docker Compose and Next.js.
- **Cloud (Vercel):** Vercel environment variables (UI-managed, encrypted at rest).
- **Supabase Edge Functions:** Supabase dashboard secrets.
- **n8n:** n8n credentials store (encrypted at rest).

A `.env.example` file in the repo documents every required variable with a description, example value, and **phase tag** (which phase first requires it). See TDS §18 for the complete reference.

### 11.9 Domain & TLS

- **Phase 1:** No domain. App accessed via `http://localhost:3000` or `http://192.168.1.10:3000`. No TLS (acceptable for LAN-only internal use; the LAN is trusted).
- **Phase 2+:** Primary domain `marwatwelfare.org` (example; actual domain to be registered). Subdomains: `app.marwatwelfare.org` (dashboard), `api.marwatwelfare.org` (Edge Functions, optional), `n8n.marwatwelfare.org` (n8n, internal). TLS certificates managed by Vercel (for Vercel-hosted) and Cloudflare (for n8n). HTTP strictly redirects to HTTPS.
- **Office LAN with TLS (optional, Phase 2+):** If the founder wants TLS on the LAN, use Caddy (automatic TLS via Let's Encrypt for LAN hostnames) or mkcert (locally-trusted certificates). Not required for Phase 1.

---

## 12. Scalability Architecture

### 12.1 Current Scale (v1 Free Tier)

- **Users:** Up to 50,000 monthly active users (Supabase Auth free tier).
- **Database:** 500MB (Supabase free tier).
- **Storage:** 1GB (Supabase free tier).
- **Bandwidth:** 100GB (Vercel free tier).
- **Serverless execution:** 100 hours/month (Vercel free tier).
- **Edge Function invocations:** 500,000/month (Supabase free tier).
- **Email:** 100 emails/day (Resend free tier).
- **WhatsApp:** 1,000 conversations/month (Meta free tier).

### 12.2 Scale Triggers

Migration to paid tiers triggers at:

| Metric | Free Tier Limit | Trigger Threshold | Action |
|---|---|---|---|
| Database size | 500MB | 400MB | Upgrade to Supabase Pro ($25/mo) |
| Storage | 1GB | 800MB | Upgrade to Supabase Pro |
| MAU | 50,000 | 40,000 | Upgrade to Supabase Pro |
| Bandwidth | 100GB | 80GB | Upgrade to Vercel Pro ($20/mo) |
| Email | 100/day | 80/day | Upgrade to Resend Pro |
| Edge Function invocations | 500K/mo | 400K/mo | Upgrade to Supabase Pro |

### 12.3 Scaling Strategies

#### 12.3.1 Database Scaling

- **Index optimization:** Every query is analyzed with `EXPLAIN ANALYZE` in CI; missing indexes are flagged.
- **Query optimization:** No N+1 queries; Supabase client uses select with relations.
- **Read replicas (post-v1):** When Supabase Pro is enabled, read replicas absorb read traffic.
- **Partitioning (post-v1):** Audit log and notification log tables partitioned by month when they exceed 1M rows.
- **Connection pooling:** Supabase manages PgBouncer; application uses connection pooler endpoint.

#### 12.3.2 Application Scaling

- **Stateless web app:** Next.js on Vercel is stateless; scales horizontally automatically.
- **Caching:** Public website pages cached at CDN (Cloudflare/Vercel). Cache invalidated on content change via webhook.
- **Static generation:** Public pages use `generateStaticParams` where possible.
- **Image optimization:** Vercel Image Optimization (free tier) for responsive images.
- **Code splitting:** Next.js automatic; feature modules lazy-loaded.

#### 12.3.3 Realtime Scaling

- **Selective subscriptions:** Client subscribes only to events it needs (e.g., inventory view subscribes to `blood_bags` table only when the inventory page is open).
- **Debouncing:** Rapid-fire events (e.g., bulk inventory updates) are debounced server-side before realtime broadcast.
- **Fallback to polling:** If Realtime disconnects, client falls back to 30-second polling.

### 12.4 Future Scale (10× Growth)

At 10× growth (500,000 MAU, 5GB database), the architectural changes needed:

- Upgrade to Supabase Pro / Team.
- Add read replicas.
- Partition large tables (audit_log, notifications).
- Move AI content generation to a dedicated worker (separate from n8n).
- Move report generation to a dedicated worker.
- Add a CDN for static assets beyond Vercel's capacity.

None of these require application code changes — they are infrastructure configuration changes. This is the test of P5 (Infrastructure Abstraction).

---

## 13. Observability Architecture

### 13.1 Logging

- **Structured logs (JSON):** Every log entry has `timestamp`, `level`, `message`, `correlation_id`, `user_id` (if applicable), `request_id`, and contextual fields.
- **Log levels:** `error` (failures requiring attention), `warn` (recoverable issues), `info` (significant events), `debug` (verbose, disabled in production).
- **Log sinks:** Vercel logs (web app), Supabase logs (database, Edge Functions), n8n logs, Sentry (errors).
- **Log retention:** 30 days in hot storage (Vercel/Supabase), 1 year in cold storage (Cloudflare R2).
- **Correlation ID:** Generated per request, propagated to all downstream calls (Edge Functions, n8n workflows).

### 13.2 Metrics

- **Application metrics:** API request count, response time (p50, p95, p99), error rate, active users, concurrent requests.
- **Database metrics:** Query count, slow queries, connection count, cache hit ratio.
- **Business metrics:** Donor count, donation count, inventory levels, request count, fulfillment SLA.
- **Automation metrics:** Workflow execution count, failure rate, average duration.

Metrics are visualized in the admin dashboard (for business metrics) and Sentry / Vercel dashboard (for technical metrics). Post-v1, a dedicated observability tool (Grafana Cloud free tier) may be added.

### 13.3 Tracing

- **Distributed tracing:** Sentry traces (free tier) for web app → Edge Function → n8n workflows.
- **Trace propagation:** Correlation ID in HTTP headers across all calls.
- **Sampling:** 100% of errors traced, 10% of successful requests traced (to stay within free tier).

### 13.4 Alerting

- **Error rate > 1%** → Alert admin via email + in-app notification.
- **API p95 response time > 1s** → Alert admin.
- **Database CPU > 80%** → Alert admin (Supabase built-in).
- **Failed workflow > 3 consecutive** → Alert admin.
- **Backup failure** → Alert admin.
- **External uptime check fails** → Alert admin (UptimeRobot).

Alerts include: severity, affected component, error message, correlation ID, link to investigate, suggested action.

### 13.5 Health Checks

- `/api/health` — Returns 200 OK with status of: database connection, auth service, storage service, n8n reachability.
- `/api/health/ready` — Returns 200 if the app is ready to serve traffic (after deploy).
- `/api/health/live` — Returns 200 if the app is alive (for liveness probes).
- External uptime monitoring (UptimeRobot) pings `/api/health` every 5 minutes.

---

## 14. Disaster Recovery & Business Continuity

### 14.1 Recovery Objectives

- **Recovery Time Objective (RTO):** 4 hours (time to restore service after a disaster).
- **Recovery Point Objective (RPO):** 24 hours (max acceptable data loss, based on daily backup).
- **Recovery testing:** Quarterly DR drill, documented results.

### 14.2 Disaster Scenarios

| Scenario | Impact | Recovery Procedure |
|---|---|---|
| Vercel outage | Web app unavailable | Wait for Vercel recovery; Supabase data unaffected; mobile app continues to work |
| Supabase database outage | Platform unavailable | Wait for Supabase recovery; restore from backup if data corruption |
| Supabase region outage | Platform unavailable | Wait for Supabase region recovery; for v1, no multi-region failover |
| n8n outage | Automations pause | Manual operations continue; workflows resume on n8n recovery |
| Cloudflare outage | Public website slow/unavailable | Vercel serves directly (DNS failover); admin dashboard may be affected |
| GitHub outage | CI/CD pauses | No deploys; existing production continues; resume on GitHub recovery |
| Domain registrar issue | Domain unavailable | Switch to backup domain (post-v1); for v1, wait for resolution |
| Data breach | PHI exposed | Incident response plan: contain, assess, notify (per Pakistan DP law), remediate |
| Ransomware | n8n or Vercel compromised | Restore from backups; rotate all credentials; forensic analysis |
| Insider attack | Data theft or corruption | Audit log review; revoke access; restore from backups; legal action |

### 14.3 Backup Strategy

- **Supabase automated backups:** Daily, 7-day retention (free tier managed).
- **Weekly logical backup:** `pg_dump` to Cloudflare R2, 30-day retention (n8n workflow AUT-018).
- **Off-site backup verification:** Daily n8n workflow (AUT-010) verifies backup integrity by restoring to a temporary database and running smoke tests.
- **Storage backup:** Supabase Storage is backed up with the database (managed by Supabase).
- **n8n backup:** n8n's internal database (workflow definitions, credentials) backed up weekly to Cloudflare R2.

### 14.4 Restore Procedure

Documented runbook (in `docs/runbooks/disaster-recovery.md`) covers:

1. Declaration of disaster (who declares, criteria).
2. Communication plan (internal stakeholders, external users, regulators if required).
3. Restoration steps (specific commands, expected output, verification).
4. Post-restore verification (smoke tests, data integrity checks).
5. Post-incident review (root cause, preventive actions, documentation updates).

### 14.5 Business Continuity

If the platform is unavailable for an extended period:

- **Manual fallback procedures:** Documented manual workflows for emergency blood requests (phone-based).
- **Communication channels:** WhatsApp groups (retained for this purpose) for emergency communication.
- **Staff training:** All admins trained on manual fallback procedures.
- **Maximum tolerable downtime:** 8 hours — beyond this, manual operations are activated.

---

## 15. Folder & Repository Architecture

### 15.1 Repository Strategy

The platform uses a **monorepo** containing all code: web app, mobile app, database migrations, n8n workflows, documentation, and shared packages.

**Rationale:** A monorepo ensures atomic changes across components (e.g., a database schema change with the corresponding TypeScript type update in the same PR). It simplifies dependency management, CI/CD, and versioning.

**Tooling:** pnpm workspaces for package management, Turborepo for build orchestration (free, open-source).

### 15.2 Top-Level Repository Structure

```
marwat-welfare-society/
├── apps/                           # Deployable applications
│   ├── web/                        # Next.js web app (public + dashboard)
│   ├── mobile/                     # Expo React Native app
│   └── docs/                       # Documentation site (optional, post-v1)
├── packages/                       # Shared packages
│   ├── database/                   # Supabase migrations, types, RLS
│   ├── domain/                     # Pure domain logic (no infra deps)
│   ├── shared/                     # Shared utilities, types, validation
│   ├── ui/                         # Shared UI components (web)
│   └── config/                     # Shared config (eslint, prettier, tsconfig)
├── automations/                    # n8n workflows + supporting scripts
│   └── n8n/
├── infrastructure/                 # Infrastructure-as-code
│   ├── supabase/                   # Supabase config, migrations, functions
│   ├── n8n/                        # n8n deployment config
│   └── cloudflare/                 # Cloudflare config
├── docs/                           # Project documentation
│   ├── 01-PRD.md
│   ├── 02-SYSTEM_ARCHITECTURE.md
│   ├── 03-TECHNICAL_DESIGN_SPECIFICATION.md
│   ├── 04-AI_DEVELOPMENT_RULEBOOK.md
│   ├── adr/                        # Architecture Decision Records
│   └── runbooks/                   # Operational runbooks
├── .github/                        # GitHub Actions workflows
│   └── workflows/
├── scripts/                        # Dev scripts (setup, seed, etc.)
├── .env.example                    # Environment variable template
├── package.json                    # Root package.json (pnpm workspace)
├── pnpm-workspace.yaml
├── turbo.json
├── tsconfig.base.json
├── .eslintrc.js
├── .prettierrc
└── README.md
```

### 15.3 Application Structure (Web App)

```
apps/web/
├── src/
│   ├── app/                        # Next.js App Router
│   │   ├── (public)/
│   │   ├── (auth)/
│   │   ├── (dashboard)/
│   │   ├── api/
│   │   └── layout.tsx
│   ├── features/                   # Feature modules (vertical slices)
│   │   ├── auth/
│   │   ├── donors/
│   │   ├── inventory/
│   │   ├── requests/
│   │   ├── volunteers/
│   │   ├── camps/
│   │   ├── patients/
│   │   ├── hospitals/
│   │   ├── notifications/
│   │   ├── reports/
│   │   ├── analytics/
│   │   ├── content/
│   │   ├── admin/
│   │   └── automation/
│   ├── components/                 # Shared UI components (Shadcn)
│   ├── lib/                        # Shared utilities
│   ├── hooks/                      # Shared React hooks
│   ├── types/                      # App-wide types
│   ├── validation/                 # App-wide validation schemas
│   ├── config/                     # App configuration
│   ├── middleware.ts               # Next.js middleware
│   └── instrumentation.ts          # Sentry / observability init
├── public/
├── tests/
├── package.json
├── tsconfig.json
├── next.config.js
├── tailwind.config.ts
└── README.md
```

### 15.4 Feature Module Structure

Every feature module follows the same structure:

```
src/features/<feature>/
├── components/                     # Feature-specific UI components
│   ├── <Component>.tsx
│   └── index.ts
├── hooks/                          # Feature-specific React hooks
│   ├── use<Feature>.ts
│   └── index.ts
├── services/                       # Feature-specific services (call infrastructure)
│   ├── <service>.ts
│   └── index.ts
├── repositories/                   # Feature-specific repositories (data access)
│   ├── <repository>.ts
│   └── index.ts
├── types/                          # Feature-specific types
│   ├── <type>.ts
│   └── index.ts
├── validation/                     # Feature-specific validation schemas
│   ├── <schema>.ts
│   └── index.ts
├── utils/                          # Feature-specific utilities
├── README.md                       # Feature documentation
└── index.ts                        # Public API barrel
```

**Rules:**
- Only `index.ts` exports are public; other features import only from `index.ts`.
- Feature folders never import from another feature's internal folders.
- Cross-feature imports go through `index.ts` barrels only.
- README.md in each feature documents: purpose, public API, dependencies, owner.

### 15.5 Shared Packages Structure

```
packages/
├── database/
│   ├── migrations/                 # Versioned SQL migrations
│   ├── seed/                       # Seed data (dev/staging only)
│   ├── rls/                        # RLS policy SQL (separate for testing)
│   ├── functions/                  # Postgres functions
│   ├── triggers/                   # Postgres triggers
│   ├── types/                      # Generated TypeScript types (via supabase gen)
│   └── README.md
├── domain/
│   ├── donors/                     # Donor domain logic (eligibility, etc.)
│   ├── inventory/                  # Inventory domain logic (state machine, expiry)
│   ├── requests/                   # Request domain logic (matching, SLA)
│   ├── blood-types/                # Blood type compatibility
│   ├── events/                     # Domain event types
│   └── README.md
├── shared/
│   ├── types/                      # Shared types (DTOs, enums)
│   ├── validation/                 # Shared Zod schemas
│   ├── utils/                      # Pure utility functions
│   ├── constants/                  # Shared constants
│   └── README.md
├── ui/
│   ├── components/                 # Shared Shadcn components
│   ├── theme/                      # Tailwind theme, design tokens
│   ├── icons/                      # Icon library
│   └── README.md
└── config/
│   ├── eslint/                     # Shared ESLint config
│   ├── prettier/                   # Shared Prettier config
│   ├── tsconfig/                   # Shared tsconfig
│   └── tailwind/                   # Shared Tailwind config
```

### 15.6 Automation Folder Structure

```
automations/
└── n8n/
    ├── workflows/                  # Workflow JSON files (one per workflow)
    │   ├── daily-operations-digest.json
    │   ├── donor-eligibility-reminder.json
    │   └── ...
    ├── credentials/                # Credential reference (actual values in n8n)
    │   └── README.md
    ├── scripts/                    # Deployment scripts
    │   ├── import-workflows.ts
    │   └── export-workflows.ts
    └── README.md
```

---

## 16. Technology Stack Decisions

This section documents every major technology choice with rationale and replaceability analysis.

### 16.1 Next.js (Web App)

**Choice:** Next.js 15 with App Router.
**Rationale:** Industry standard for React; SSR + SSG + API routes in one framework; Vercel-native deployment; strong TypeScript support; large ecosystem.
**Replaceability:** If Vercel becomes unsuitable, Next.js can be self-hosted on any Node.js host (Railway, Fly.io, AWS). If Next.js itself becomes unsuitable, the feature-based architecture allows migration to Remix or plain Vite + React Router over ~2 sprints.
**Risks:** App Router is relatively new (stable since Next.js 13); some patterns still evolving.
**Mitigation:** Stick to documented patterns; avoid experimental features; pin minor versions.

### 16.2 Expo React Native (Mobile App)

**Choice:** Expo SDK 51+ with Expo Router.
**Rationale:** Cross-platform (Android + iOS) from one codebase; OTA updates; EAS Build free tier; large ecosystem; excellent DX.
**Replaceability:** Expo can be ejected to React Native CLI if needed. Code is structured to share domain logic with web (via `packages/domain`).
**Risks:** Expo's free tier has limits (30 builds/month); App Store publishing requires Apple Developer account ($99/year, post-v1).
**Mitigation:** Android-first launch (larger market in Pakistan); iOS post-v1.

### 16.3 Supabase (Database, Auth, Storage, Functions)

**Choice:** Supabase free tier.
**Rationale:** Managed PostgreSQL with auth, storage, edge functions, realtime in one platform; generous free tier; SQL access (no lock-in); row-level security; open-source (can self-host).
**Replaceability:** Database is standard PostgreSQL — can be migrated to any Postgres host (RDS, CrunchyBridge, self-hosted). Auth can be replaced with Keycloak or Clerk. Storage can be replaced with S3 directly. Edge Functions can be replaced with Vercel Functions or self-hosted Deno.
**Risks:** Free tier limits (500MB DB, 1GB storage); Singapore region only (latency from PK); vendor lock-in on dashboard configuration.
**Mitigation:** All configuration as code (migrations, RLS policies, storage bucket definitions); scale triggers documented (§12.2).

### 16.4 Vercel (Web Hosting)

**Choice:** Vercel free tier (Hobby).
**Rationale:** Best-in-class Next.js hosting; automatic preview deployments; edge network; generous free tier; SSL and CDN built-in.
**Replaceability:** Next.js is self-hostable; can move to Railway, Fly.io, Render, or AWS. Build pipeline can move to GitHub Pages + Cloudflare Pages.
**Risks:** Hobby tier is for non-commercial use; commercial use may require Pro ($20/mo) — acceptable cost if/when triggered.
**Mitigation:** Document the trigger for upgrade (commercial activity) and budget for it.

### 16.5 n8n (Automation)

**Choice:** n8n self-hosted on Railway/Fly.io free tier.
**Rationale:** Open-source (no vendor lock-in); powerful visual workflow editor; supports all needed integrations (Supabase, Resend, WhatsApp, AI); fair-code license allows self-hosting.
**Replaceability:** Workflows can be migrated to Inngest, Trigger.dev, or custom Next.js API routes. Workflow JSON is portable.
**Risks:** Self-hosting requires maintenance; free tier of Railway/Fly.io may have cold starts.
**Mitigation:** Backup n8n database weekly; document recovery procedure; monitor uptime.

### 16.6 Shadcn UI + Tailwind CSS (Styling)

**Choice:** Shadcn UI (Radix primitives + Tailwind) + Tailwind CSS 3.4.
**Rationale:** Copy-paste components (no library lock-in); accessible (Radix); consistent design tokens; dark mode built-in; large community.
**Replaceability:** Components are owned code; can be modified or replaced individually. Tailwind can be swapped for CSS Modules if needed (not recommended).
**Risks:** Shadcn requires manual updates (no npm dependency to bump).
**Mitigation:** Document update procedure; CI check for component versions.

### 16.7 TypeScript (Language)

**Choice:** TypeScript 5+ with strict mode.
**Rationale:** Type safety catches bugs at compile time; IDE support is excellent; ecosystem is universal; strict mode enforces discipline.
**Replaceability:** Not applicable — TypeScript is the language.
**Risks:** Overuse of `any` or non-strict mode undermines benefits.
**Mitigation:** ESLint rule forbids `any` unless explicitly justified; strict mode enforced in tsconfig.

### 16.8 Zod (Validation)

**Choice:** Zod 3+.
**Rationale:** TypeScript-native; composable; runtime + compile-time types from one source; widely adopted.
**Replaceability:** Can be replaced with Valibot (smaller bundle) or Yup. The validation interface is abstracted; replacement is localized.
**Risks:** Bundle size for mobile app.
**Mitigation:** Valibot as a future option if bundle size becomes an issue.

### 16.9 Resend (Email)

**Choice:** Resend free tier (100 emails/day).
**Rationale:** Developer-friendly API; generous free tier; reliable deliverability.
**Replaceability:** Can be replaced with AWS SES (free tier), Postmark, or self-hosted Postfix. The email interface is abstracted.
**Risks:** 100 emails/day may be insufficient at scale.
**Mitigation:** Document upgrade trigger; bulk emails batched via n8n.

### 16.10 WhatsApp Cloud API (Notifications)

**Choice:** WhatsApp Cloud API free tier (1,000 conversations/month).
**Rationale:** Dominant communication channel in Pakistan; free tier sufficient for v1; official API (no unofficial clients).
**Replaceability:** Can be replaced with Twilio WhatsApp API (paid) or self-hosted WhatsApp Business.
**Risks:** 1,000 conversations/month limit; Meta policy changes.
**Mitigation:** Use for high-value notifications only (emergency requests, donation thank-yous); fall back to SMS/email for routine.

### 16.11 Sentry (Error Tracking)

**Choice:** Sentry free tier (5,000 errors/month).
**Rationale:** Industry standard; generous free tier; performance monitoring included.
**Replaceability:** Can be replaced with GlitchTip (open-source, self-hosted) or Rollbar.
**Risks:** 5,000 errors/month may be insufficient.
**Mitigation:** Filter expected errors (e.g., 404s) before reporting.

### 16.12 Cloudflare (DNS, CDN, WAF)

**Choice:** Cloudflare free tier.
**Rationale:** Best-in-class DNS; free CDN; free WAF with rate limiting; free DDoS protection.
**Replaceability:** DNS can move to any provider; CDN can move to Vercel's built-in.
**Risks:** Cloudflare's free tier has limitations on WAF rules (5 custom rules).
**Mitigation:** Carefully prioritize WAF rules.

---

## 17. Architecture Decision Records (ADRs)

ADRs document significant architectural decisions, their context, consequences, and status. They are immutable once marked `Accepted`; supersession requires a new ADR.

### 17.1 ADR Discipline (Solo-Founder Stage)

During the solo-founder stage (Phases 1–3), ADRs are **strongly encouraged but not merge-blocking**. The founder uses judgement to decide which decisions warrant an ADR. As a rule of thumb:

- **Write an ADR** for decisions that affect multiple files, introduce a new dependency, change a public interface, or establish a pattern that future contributors must follow.
- **Skip the ADR** for trivial decisions (naming, refactoring within a single file, bug fixes) — write a clear commit message instead.
- **Always write an ADR** for decisions that reverse a previous ADR, or that constrain future options (e.g., choosing a database vendor, choosing an auth provider).

From Phase 4 onward (post-solo-founder), the Tech Lead or Board Tech Committee may mandate ADRs for a broader class of decisions. This governance change is itself documented in an ADR.

### 17.2 ADR Template

```markdown
# ADR-NNNN: Title

## Status
Proposed | Accepted | Deprecated | Superseded by ADR-XXXX

## Context
(Why is this decision needed? What is the problem? What are the constraints?)

## Decision
(What is the decision? Be precise.)

## Consequences
(Positive: ...)
(Negative: ...)
(Neutral: ...)

## Alternatives Considered
(What other options were considered? Why were they rejected?)

## References
(Links to related ADRs, PRs, issues, external resources.)
```

### 17.3 Suggested Initial ADRs

These ADRs are **suggested** for the founder to write during Phase 1 setup. They are not required to start coding, but writing them early saves rework later. The founder may skip any of these with a brief justification in the commit message.

| ADR | Title | Phase |
|---|---|---|
| ADR-0001 | Adopt monorepo with pnpm workspaces + Turborepo | P1 |
| ADR-0002 | Use Next.js App Router (not Pages Router) | P1 |
| ADR-0003 | Use feature-based modular architecture (not file-type) | P1 |
| ADR-0004 | Use Supabase (local Docker for P1; cloud optional from P2) | P1 |
| ADR-0005 | Enforce RLS on every user-data table | P1 |
| ADR-0006 | Use the Service Interface pattern for notifications/reports/backups (n8n is a Phase 3 adapter, not a dependency) | P1 |
| ADR-0007 | Use Shadcn UI + Tailwind (not Material UI or Chakra) | P1 |
| ADR-0008 | Use Expo (not React Native CLI) for mobile app (Phase 4) | P4 |
| ADR-0009 | Use Zod (not Yup or Joi) for validation | P1 |
| ADR-0010 | Domain layer has zero external dependencies (enforced by ESLint) | P1 |
| ADR-0011 | Server-authoritative state (no client-owned authoritative state) | P1 |
| ADR-0012 | All secrets in environment variables (never in code or DB) | P1 |
| ADR-0013 | Soft delete (not hard delete) for all user data | P1 |
| ADR-0014 | Audit log is append-only, retained 7 years | P1 |
| ADR-0015 | AI never makes medical or operational decisions (binding from Phase 3 onward) | P3 |
| ADR-0016 | Local-first deployment: same code runs on laptop, office LAN, and cloud | P1 |

ADRs are stored in `docs/adr/` with filenames `NNNN-title.md`. The README in `docs/adr/` maintains an index of all ADRs by number and status.

---

## 18. Glossary

| Term | Definition |
|---|---|
| **ADR** | Architecture Decision Record — a documented, immutable architectural decision. |
| **Aggregate** | A cluster of domain objects treated as a single unit for data changes. |
| **Bounded Context** | A coherent slice of the domain with its own model and ubiquitous language. |
| **Clean Architecture** | An architecture style where dependencies flow inward, from outer (infrastructure) to inner (domain) layers. |
| **Domain Event** | A significant event in the domain that other parts of the system may react to. |
| **Edge Function** | A serverless function (Deno) hosted on Supabase, running close to the database. |
| **Idempotent** | An operation that produces the same result whether called once or multiple times. |
| **Local-First** | Architecture principle: the platform runs on a single laptop, office LAN, or cloud without code change. |
| **Phase (P1–P4)** | Implementation phase: P1 Core MVP, P2 Operational Improvements, P3 Automation, P4 Scale. |
| **RLS** | Row-Level Security — a PostgreSQL feature for per-row access control. |
| **RTO** | Recovery Time Objective — target time to restore service after a disaster. |
| **RPO** | Recovery Point Objective — maximum acceptable data loss in a disaster. |
| **Service Interface Pattern** | Application calls interfaces (NotificationService, ReportService, etc.); concrete implementations are swappable via environment variables. |
| **State Machine** | A model where an entity transitions between defined states via legal transitions. |
| **Trust Boundary** | A point in the system where authentication/authorization is checked. |

---

## 19. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0.0 | 2026-07-06 | Principal Architect | Initial baseline approved by Board. |
| 1.1.0 | 2026-07-06 | Founder / Principal Architect | **Startup-first refactor.** Added §3.4 Local-First Architecture (laptop → office LAN → cloud, no code change). Rewrote §10 Automation Architecture around the **Service Interface pattern** — n8n is now a Phase 3 adapter behind stable interfaces (`NotificationService`, `ReportService`, `BackupService`, `AIContentService`), never a hard dependency. Added §10.7 Graceful Degradation specifying platform behavior when n8n or AI is unavailable. Rewrote §11 Deployment Architecture with 4 deployment topologies (solo laptop, office LAN, cloud, hybrid) and phased CI/CD (minimal in P1, expanded in P2+). Simplified §17 ADR governance: ADRs are encouraged but not merge-blocking during the solo-founder stage. Added phase tags to the component summary table (§3.2). Preserved all security, data, integration, and scalability decisions from v1.0.0 unchanged. |

---

> **End of System Architecture Document.** The next document is the **Technical Design Specification** (`03-TECHNICAL_DESIGN_SPECIFICATION.md`), which translates this architecture into concrete schemas, types, and API contracts.
