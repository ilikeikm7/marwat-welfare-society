# Marwat Welfare Society — Technical Design Specification (TDS)

| Field | Value |
|---|---|
| **Document Title** | Technical Design Specification |
| **Project Name** | Marwat Welfare Society — Blood Donation & Distribution Platform |
| **Document Version** | 1.1.0 |
| **Status** | Approved Baseline |
| **Classification** | Internal — Project Constitution |
| **Last Updated** | 2026-07-06 |
| **Document Owner** | Founder / Principal Architect |
| **Approval Authority** | Founder (solo-founder stage); Board Tech Committee (post-Phase 4) |
| **Depends On** | `01-PRD.md` (v1.1.0+), `02-SYSTEM_ARCHITECTURE.md` (v1.1.0+) |
| **Consumed By** | All implementation prompts, all AI coding sessions, all PR reviews |

> **CONSTITUTION CLAUSE.** This document defines the *how* of the implementation. Every line of code written for this platform MUST conform to the schemas, types, contracts, and patterns defined here. Where this document is silent, the AI Development Rulebook governs. Where a pattern in this document conflicts with a real-world constraint discovered during implementation, the founder may deviate with a documented justification in the commit message (solo-founder stage) or an ADR (post-Phase 4).
>
> **v1.1.0 REFACTOR NOTE.** This version adds **phase tags** (`P1`–`P4`) to all schemas, API endpoints, Edge Functions, and environment variables, so the founder knows exactly what to build in Phase 1 vs later. It adds the **Service Interface implementations** reference (§8.7) showing the in-process Phase 1–2 services and the n8n-backed Phase 3 adapters. It simplifies the **CI/CD pipeline** (§17) to a minimal Phase 1 workflow with explicit Phase 2+ additions. All schemas, types, RLS policies, and validation schemas from v1.0.0 are preserved unchanged — Phase 1 simply implements a subset of them.

---

## Table of Contents

1. [Document Purpose & Conventions](#1-document-purpose--conventions)
2. [Project Folder Structure (Detailed)](#2-project-folder-structure-detailed)
3. [Database Schema — Complete DDL](#3-database-schema--complete-ddl)
4. [Entity-Relationship Diagram](#4-entity-relationship-diagram)
5. [Row-Level Security Policies](#5-row-level-security-policies)
6. [TypeScript Type System](#6-typescript-type-system)
7. [Validation Schemas (Zod)](#7-validation-schemas-zod)
8. [API Design](#8-api-design)
9. [Edge Function Specifications](#9-edge-function-specifications)
10. [n8n Workflow Specifications](#10-n8n-workflow-specifications)
11. [Authentication & Session Management](#11-authentication--session-management)
12. [Authorization — Permission Matrix](#12-authorization--permission-matrix)
13. [State Machines](#13-state-machines)
14. [Frontend Routing & Component Architecture](#14-frontend-routing--component-architecture)
15. [Mobile App Architecture](#15-mobile-app-architecture)
16. [Testing Strategy](#16-testing-strategy)
17. [CI/CD Pipeline Specification](#17-cicd-pipeline-specification)
18. [Environment Variables Reference](#18-environment-variables-reference)
19. [Migration & Seeding Strategy](#19-migration--seeding-strategy)
20. [Glossary](#20-glossary)
21. [Revision History](#21-revision-history)

---

## 1. Document Purpose & Conventions

### 1.1 Purpose

This Technical Design Specification (TDS) is the implementation-level companion to the PRD and System Architecture Document. It provides the concrete schemas, types, contracts, and patterns that every contributor — human or AI — must follow when writing code for the platform. Where the architecture document says *"we use a Repository abstraction"*, this document says *"here is the `DonorRepository` interface and its Supabase implementation"*.

### 1.2 Phase Tags

Every schema, endpoint, function, and environment variable in this document carries a **phase tag** (`P1`, `P2`, `P3`, `P4`) indicating when it is first implemented. The founder implementing Phase 1 can ignore everything tagged `P2+`. Later phases implement their tagged items without changing Phase 1 code.

- `P1` = Phase 1 Core MVP — admin-internal, no automation, no AI, no public surfaces.
- `P2` = Phase 2 Operational Improvements — portals, email, camps, public website.
- `P3` = Phase 3 Automation — n8n, AI, scheduled workflows, social media.
- `P4` = Phase 4 Scale — mobile app, multi-city, advanced monitoring, predictive AI.

Where a phase tag is absent, the item belongs to Phase 1 (the default).

### 1.3 Conventions

- **Code blocks** in this document are normative reference implementations. They may be adapted for the actual codebase, but the *interface* and *contract* they define must be preserved.
- **Naming** follows the conventions in the AI Development Rulebook. When this document and the Rulebook appear to conflict, this document governs for the specific case; the Rulebook is updated to reflect the resolution.
- **Identifiers** (requirement IDs, schema names, API paths) are stable and referenced from code comments, tests, and PR descriptions.
- **SQL** is PostgreSQL 15 dialect, compatible with Supabase.
- **TypeScript** is 5.4+ with strict mode enabled.

---

## 2. Project Folder Structure (Detailed)

Building on §15 of the System Architecture Document, this section specifies the contents of each folder at file level.

### 2.1 Root Files

```
marwat-welfare-society/
├── .env.example                    # All required env vars, no real values
├── .env.local                      # Local dev values (gitignored)
├── .gitignore
├── .editorconfig
├── .nvmrc                          # Node version pin
├── .prettierrc                     # Prettier config (extends @marwat/config)
├── .prettierignore
├── .eslintrc.cjs                   # ESLint config (extends @marwat/config)
├── .eslintignore
├── package.json                    # Root workspace package.json
├── pnpm-workspace.yaml             # pnpm workspace definition
├── pnpm-lock.yaml
├── turbo.json                      # Turborepo pipeline config
├── tsconfig.base.json              # Base TypeScript config
├── README.md                       # Project overview, setup, links to docs/
├── CONTRIBUTING.md                 # Contribution guide
├── LICENSE                         # Open-source license (MIT or AGPL)
└── CHANGELOG.md                    # Versioned changelog
```

### 2.2 `apps/web/` (Next.js Web App)

```
apps/web/
├── src/
│   ├── app/                                  # Next.js App Router
│   │   ├── (public)/                         # Public routes (no auth)
│   │   │   ├── layout.tsx                    # Public layout (header, footer)
│   │   │   ├── page.tsx                      # Home
│   │   │   ├── about/page.tsx
│   │   │   ├── request-blood/
│   │   │   │   ├── page.tsx                  # Form
│   │   │   │   └── success/page.tsx          # Confirmation
│   │   │   ├── donate-blood/
│   │   │   │   ├── page.tsx                  # Form
│   │   │   │   └── success/page.tsx
│   │   │   ├── volunteer/
│   │   │   │   ├── page.tsx                  # Form
│   │   │   │   └── success/page.tsx
│   │   │   ├── knowledge-base/
│   │   │   │   ├── page.tsx                  # Article list
│   │   │   │   └── [slug]/page.tsx           # Article detail
│   │   │   ├── impact/page.tsx               # Public dashboard
│   │   │   ├── partner-hospitals/page.tsx
│   │   │   ├── contact/page.tsx
│   │   │   ├── privacy/page.tsx
│   │   │   └── terms/page.tsx
│   │   ├── (auth)/
│   │   │   ├── layout.tsx                    # Auth layout (minimal chrome)
│   │   │   ├── login/page.tsx
│   │   │   ├── register/page.tsx
│   │   │   ├── register/donor/page.tsx
│   │   │   ├── register/volunteer/page.tsx
│   │   │   ├── register/hospital/page.tsx
│   │   │   ├── forgot-password/page.tsx
│   │   │   ├── reset-password/page.tsx
│   │   │   ├── verify-otp/page.tsx
│   │   │   └── mfa-setup/page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx                    # Dashboard layout (sidebar, header)
│   │   │   ├── admin/                        # SUPER_ADMIN, ADMIN
│   │   │   │   ├── page.tsx                  # Overview
│   │   │   │   ├── donors/
│   │   │   │   │   ├── page.tsx              # List
│   │   │   │   │   ├── [id]/page.tsx         # Detail
│   │   │   │   │   └── new/page.tsx
│   │   │   │   ├── inventory/
│   │   │   │   │   ├── page.tsx
│   │   │   │   │   └── [bagId]/page.tsx
│   │   │   │   ├── requests/
│   │   │   │   │   ├── page.tsx
│   │   │   │   │   └── [id]/page.tsx
│   │   │   │   ├── volunteers/
│   │   │   │   ├── camps/
│   │   │   │   ├── patients/
│   │   │   │   ├── hospitals/
│   │   │   │   ├── deliveries/
│   │   │   │   ├── reports/
│   │   │   │   ├── analytics/
│   │   │   │   ├── notifications/
│   │   │   │   ├── automation/
│   │   │   │   ├── content/
│   │   │   │   ├── audit-log/
│   │   │   │   ├── users/
│   │   │   │   └── settings/
│   │   │   ├── hospital/                     # HOSPITAL
│   │   │   │   ├── page.tsx
│   │   │   │   ├── requests/
│   │   │   │   ├── inventory/page.tsx        # Aggregate view only
│   │   │   │   └── deliveries/
│   │   │   ├── patient/                      # PATIENT
│   │   │   │   ├── page.tsx
│   │   │   │   ├── requests/
│   │   │   │   └── documents/
│   │   │   ├── donor/                        # DONOR (web view of mobile app)
│   │   │   │   ├── page.tsx
│   │   │   │   ├── donations/
│   │   │   │   └── profile/
│   │   │   └── volunteer/                    # VOLUNTEER (web view)
│   │   ├── api/
│   │   │   ├── health/route.ts               # Health check
│   │   │   ├── webhooks/                     # Webhook receivers (HMAC)
│   │   │   │   ├── n8n/route.ts              # From n8n
│   │   │   │   └── supabase/route.ts         # From Supabase (if needed)
│   │   │   ├── reports/
│   │   │   │   └── [id]/route.ts             # Generate / download report
│   │   │   └── internal/                     # Internal admin ops
│   │   ├── error.tsx                         # Error boundary
│   │   ├── not-found.tsx                     # 404
│   │   ├── loading.tsx                       # Loading UI
│   │   ├── globals.css                       # Global styles
│   │   └── layout.tsx                        # Root layout
│   ├── features/                             # Feature modules (see §2.4)
│   ├── components/                           # Shared UI components
│   │   ├── ui/                               # Shadcn components
│   │   ├── layout/                           # Layout components (Header, Footer, Sidebar)
│   │   ├── feedback/                         # Toast, Alert, Skeleton
│   │   ├── forms/                            # Form primitives
│   │   ├── data/                             # Table, List, Pagination
│   │   └── index.ts
│   ├── lib/                                  # Shared utilities
│   │   ├── supabase/                         # Supabase client factory
│   │   │   ├── client.ts                     # Browser client
│   │   │   ├── server.ts                     # Server client
│   │   │   ├── middleware.ts                 # Middleware client
│   │   │   └── index.ts
│   │   ├── auth/                             # Auth helpers
│   │   ├── utils.ts                          # cn(), formatters
│   │   ├── logger.ts                         # Structured logger
│   │   ├── constants.ts                      # App constants
│   │   └── env.ts                            # Env var validation (Zod)
│   ├── hooks/                                # Shared React hooks
│   │   ├── use-auth.ts
│   │   ├── use-realtime.ts
│   │   ├── use-debounce.ts
│   │   ├── use-media-query.ts
│   │   └── index.ts
│   ├── types/                                # App-wide types
│   │   ├── api.ts                            # API request/response types
│   │   ├── env.ts                            # Env type augmentation
│   │   └── index.ts
│   ├── validation/                           # App-wide Zod schemas
│   ├── config/                               # App configuration
│   │   ├── site.ts                           # Site metadata
│   │   ├── nav.ts                            # Navigation config
│   │   └── i18n.ts                           # i18n config
│   ├── middleware.ts                         # Auth + security middleware
│   └── instrumentation.ts                    # Sentry init
├── public/
│   ├── images/
│   ├── icons/
│   └── fonts/
├── tests/
│   ├── e2e/                                  # Playwright E2E tests
│   ├── integration/                          # Integration tests
│   └── rls/                                  # RLS policy tests
├── package.json
├── tsconfig.json
├── next.config.mjs
├── tailwind.config.ts
├── postcss.config.mjs
├── playwright.config.ts
└── README.md
```

### 2.3 `apps/mobile/` (Expo App)

```
apps/mobile/
├── src/
│   ├── app/                                  # Expo Router
│   │   ├── (auth)/
│   │   ├── (donor)/                          # Donor tabs
│   │   │   ├── home.tsx
│   │   │   ├── donations.tsx
│   │   │   ├── profile.tsx
│   │   │   └── impact.tsx
│   │   ├── (volunteer)/                      # Volunteer tabs
│   │   │   ├── home.tsx                      # Task list
│   │   │   ├── tasks/[id].tsx
│   │   │   ├── check-in.tsx
│   │   │   └── profile.tsx
│   │   └── _layout.tsx
│   ├── components/                           # Mobile-specific UI
│   ├── hooks/
│   ├── lib/
│   │   ├── supabase.ts                       # Supabase client (React Native)
│   │   ├── notifications.ts                  # Push notifications
│   │   └── haptics.ts
│   ├── features/                             # Re-uses shared packages
│   ├── theme/                                # Mobile theme
│   └── assets/
├── app.config.ts                             # Expo config
├── package.json
├── tsconfig.json
└── README.md
```

### 2.4 Feature Module Internal Structure

Every feature in `apps/web/src/features/<feature>/` follows this structure:

```
src/features/donors/
├── components/
│   ├── DonorList.tsx
│   ├── DonorDetail.tsx
│   ├── DonorForm.tsx
│   ├── DonorEligibilityBadge.tsx
│   └── index.ts
├── hooks/
│   ├── useDonors.ts                          # List query
│   ├── useDonor.ts                           # Single query
│   ├── useCreateDonor.ts                     # Mutation
│   ├── useUpdateDonor.ts
│   ├── useDonorRealtime.ts
│   └── index.ts
├── services/
│   ├── donor-service.ts                      # Business logic
│   └── index.ts
├── repositories/
│   ├── donor-repository.ts                   # Supabase data access
│   └── index.ts
├── types/
│   ├── donor.ts                              # Donor entity type
│   ├── donor-dto.ts                          # API DTOs
│   └── index.ts
├── validation/
│   ├── create-donor.ts                       # Zod schema
│   ├── update-donor.ts
│   └── index.ts
├── utils/
│   ├── eligibility.ts                        # Eligibility calculation
│   └── index.ts
├── README.md                                 # Feature docs
└── index.ts                                  # Public API barrel
```

**Barrel `index.ts` exports only the public API:**

```typescript
// src/features/donors/index.ts
export * from './types';
export * from './validation';
export * from './services';
export * from './hooks';
export * from './components';
```

Other features import only from the barrel:
```typescript
// ✅ Correct
import { useDonor, type Donor } from '@/features/donors';

// ❌ Forbidden — accessing internal modules
import { useDonor } from '@/features/donors/hooks/use-donor';
```

### 2.5 `packages/database/` (Schema Source of Truth)

```
packages/database/
├── migrations/                               # Versioned SQL migrations
│   ├── 0001_create_extensions.sql
│   ├── 0002_create_user_roles.sql
│   ├── 0003_create_donors.sql
│   ├── 0004_create_donations.sql
│   ├── 0005_create_blood_bags.sql
│   ├── 0006_create_patients.sql
│   ├── 0007_create_hospitals.sql
│   ├── 0008_create_requests.sql
│   ├── 0009_create_allocations.sql
│   ├── 0010_create_deliveries.sql
│   ├── 0011_create_volunteers.sql
│   ├── 0012_create_tasks.sql
│   ├── 0013_create_camps.sql
│   ├── 0014_create_notifications.sql
│   ├── 0015_create_reports.sql
│   ├── 0016_create_audit_log.sql
│   ├── 0017_create_settings.sql
│   ├── 0018_create_kb_articles.sql
│   ├── 0019_create_storage_buckets.sql
│   ├── 0020_enable_rls_policies.sql
│   ├── 0021_create_triggers.sql
│   ├── 0022_create_indexes.sql
│   └── README.md
├── rls/                                       # RLS policy SQL (for testing)
│   ├── donors.sql
│   ├── blood_bags.sql
│   └── ...
├── functions/                                # Postgres functions
│   ├── soft_delete.sql
│   ├── audit_log.sql
│   └── ...
├── triggers/                                 # Postgres triggers
│   ├── updated_at.sql
│   └── audit.sql
├── seed/                                     # Seed data (dev/staging)
│   ├── 0001_roles.sql
│   ├── 0002_super_admin.sql
│   ├── 0003_sample_donors.sql
│   └── ...
├── types/                                    # Generated TS types
│   └── database.types.ts                     # via `supabase gen types`
├── queries/                                  # Reusable SQL queries (as TS)
│   └── ...
└── README.md
```

### 2.6 `packages/domain/` (Pure Business Logic)

```
packages/domain/
├── src/
│   ├── donors/
│   │   ├── eligibility.ts                    # Eligibility calculation (pure)
│   │   ├── deferral.ts                       # Deferral rules
│   │   └── index.ts
│   ├── inventory/
│   │   ├── state-machine.ts                  # Bag state machine (pure)
│   │   ├── expiry.ts                         # Expiry calculation
│   │   └── index.ts
│   ├── requests/
│   │   ├── matching.ts                       # Blood type matching
│   │   ├── sla.ts                            # SLA calculation
│   │   ├── state-machine.ts
│   │   └── index.ts
│   ├── blood-types/
│   │   ├── compatibility.ts                  # Donor/recipient compatibility
│   │   ├── types.ts
│   │   └── index.ts
│   ├── events/
│   │   ├── types.ts                          # Domain event types
│   │   └── index.ts
│   └── index.ts
├── tests/
│   ├── donors.test.ts
│   ├── inventory.test.ts
│   └── ...
├── package.json
├── tsconfig.json
└── README.md
```

**Rule:** `packages/domain/` MUST have zero imports from `@supabase/supabase-js`, `next`, `react`, `expo`, or any infrastructure package. Enforced by ESLint `no-restricted-imports` rule.

### 2.7 `packages/shared/` (Cross-Cutting Code)

```
packages/shared/
├── src/
│   ├── types/
│   │   ├── enums.ts                          # Role, BloodType, urgency, etc.
│   │   ├── api.ts                            # API envelope types
│   │   ├── pagination.ts
│   │   └── index.ts
│   ├── validation/
│   │   ├── common.ts                         # Common Zod schemas (phone, CNIC)
│   │   └── index.ts
│   ├── utils/
│   │   ├── date.ts                           # Date formatting (locale-aware)
│   │   ├── id.ts                             # ID generation
│   │   ├── crypto.ts                         # Hashing, etc.
│   │   ├── string.ts
│   │   └── index.ts
│   ├── constants/
│   │   ├── blood-types.ts
│   │   ├── roles.ts
│   │   ├── deferral-windows.ts               # WHO deferral windows
│   │   └── index.ts
│   └── index.ts
├── package.json
└── README.md
```

### 2.8 `packages/ui/` (Shared Web UI)

```
packages/ui/
├── src/
│   ├── components/                           # Shadcn components
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   ├── dialog.tsx
│   │   └── ... (all Shadcn components)
│   ├── theme/
│   │   ├── tokens.ts                         # Design tokens (colors, spacing)
│   │   ├── dark.ts                           # Dark theme
│   │   ├── light.ts                          # Light theme
│   │   └── index.ts
│   ├── icons/
│   │   └── index.ts
│   └── index.ts
├── package.json
└── README.md
```

### 2.9 `automations/n8n/` (Workflow Source of Truth)

```
automations/n8n/
├── workflows/
│   ├── AUT-001-daily-operations-digest.json
│   ├── AUT-002-donor-eligibility-reminder.json
│   ├── AUT-003-post-donation-thank-you.json
│   ├── ... (one JSON per workflow)
│   └── README.md
├── credentials/                              # Credential docs (no secrets)
│   └── README.md
├── scripts/
│   ├── import-to-n8n.ts                      # Deploy workflows via API
│   ├── export-from-n8n.ts                    # Pull workflows for review
│   └── validate-json.ts                      # CI validation
├── shared/                                   # Shared n8n nodes / helpers
└── README.md
```

### 2.10 `infrastructure/` (IaC & Deployment Config)

```
infrastructure/
├── supabase/
│   ├── config.toml                           # Supabase CLI config
│   ├── functions/                            # Edge Functions (Deno)
│   │   ├── _shared/
│   │   │   ├── cors.ts
│   │   │   ├── auth.ts                       # JWT verification
│   │   │   ├── supabase.ts                   # Service client
│   │   │   ├── hmac.ts                       # HMAC verification
│   │   │   └── error.ts
│   │   ├── webhook-receiver/
│   │   │   └── index.ts
│   │   ├── report-generator/
│   │   │   └── index.ts
│   │   ├── virus-scan/
│   │   │   └── index.ts
│   │   ├── ai-content/
│   │   │   └── index.ts
│   │   ├── bulk-export/
│   │   │   └── index.ts
│   │   └── ...
│   └── seed.sql                              # Initial seed
├── n8n/
│   ├── docker-compose.yml                    # Self-hosted n8n
│   ├── Caddyfile                             # Reverse proxy + TLS
│   └── README.md
├── cloudflare/
│   ├── wrangler.toml
│   ├── rules/                                # WAF rules
│   └── README.md
└── README.md
```

### 2.11 `docs/` (Project Documentation)

```
docs/
├── 01-PRD.md                                 # Product Requirements
├── 02-SYSTEM_ARCHITECTURE.md                 # System Architecture
├── 03-TECHNICAL_DESIGN_SPECIFICATION.md      # This document
├── 04-AI_DEVELOPMENT_RULEBOOK.md             # AI Rulebook
├── adr/                                      # Architecture Decision Records
│   ├── 0001-monorepo-pnpm-turbo.md
│   ├── 0002-nextjs-app-router.md
│   └── ...
├── runbooks/                                 # Operational runbooks
│   ├── disaster-recovery.md
│   ├── deployment.md
│   ├── on-call.md
│   └── incident-response.md
├── guides/                                   # Developer guides
│   ├── getting-started.md
│   ├── database-migrations.md
│   ├── adding-a-feature.md
│   ├── adding-an-automation.md
│   └── testing.md
├── api/                                      # API docs (OpenAPI generated)
│   └── openapi.yaml
└── README.md
```

---

## 3. Database Schema — Complete DDL

This section defines the complete database schema. All DDL is versioned in `packages/database/migrations/` and applied via Supabase CLI.

### 3.1 Extensions

```sql
-- File: 0001_create_extensions.sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";          -- trigram search
CREATE EXTENSION IF NOT EXISTS "citext";           -- case-insensitive text
```

### 3.2 Identity & Access

```sql
-- File: 0002_create_user_roles.sql

-- user_profiles extends auth.users with platform-specific data
CREATE TABLE public.user_profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email CITEXT UNIQUE NOT NULL,
  phone TEXT UNIQUE,
  phone_verified BOOLEAN NOT NULL DEFAULT FALSE,
  email_verified BOOLEAN NOT NULL DEFAULT FALSE,
  full_name TEXT NOT NULL,
  preferred_language TEXT NOT NULL DEFAULT 'en-PK' CHECK (preferred_language IN ('en-PK', 'ur-PK')),
  avatar_url TEXT,
  status TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'suspended', 'deactivated')),
  last_login_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  deleted_at TIMESTAMPTZ
);

CREATE INDEX idx_user_profiles_email ON public.user_profiles(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_user_profiles_phone ON public.user_profiles(phone) WHERE deleted_at IS NULL AND phone IS NOT NULL;

-- user_roles: many-to-many user → role
CREATE TABLE public.user_roles (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES public.user_profiles(id) ON DELETE CASCADE,
  role TEXT NOT NULL CHECK (role IN ('SUPER_ADMIN', 'ADMIN', 'VOLUNTEER', 'HOSPITAL', 'DONOR', 'PATIENT')),
  granted_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  granted_by UUID REFERENCES public.user_profiles(id),
  revoked_at TIMESTAMPTZ,
  revoked_by UUID REFERENCES public.user_profiles(id),
  reason TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (user_id, role)
);

CREATE INDEX idx_user_roles_user_active ON public.user_roles(user_id) WHERE revoked_at IS NULL;
CREATE INDEX idx_user_roles_role ON public.user_roles(role) WHERE revoked_at IS NULL;

-- user_sessions: tracks active sessions (for audit and forced logout)
CREATE TABLE public.user_sessions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES public.user_profiles(id) ON DELETE CASCADE,
  session_token_hash TEXT NOT NULL,
  ip_address INET,
  user_agent TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at TIMESTAMPTZ NOT NULL,
  revoked_at TIMESTAMPTZ,
  revoked_reason TEXT
);

CREATE INDEX idx_user_sessions_user ON public.user_sessions(user_id) WHERE revoked_at IS NULL;
CREATE INDEX idx_user_sessions_expires ON public.user_sessions(expires_at) WHERE revoked_at IS NULL;

-- audit_log: append-only audit trail
CREATE TABLE public.audit_log (
  id BIGSERIAL PRIMARY KEY,
  actor_id UUID REFERENCES public.user_profiles(id),
  actor_role TEXT,
  action TEXT NOT NULL,
  entity_type TEXT NOT NULL,
  entity_id TEXT,
  before_state JSONB,
  after_state JSONB,
  ip_address INET,
  user_agent TEXT,
  correlation_id UUID,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_audit_log_actor ON public.audit_log(actor_id);
CREATE INDEX idx_audit_log_entity ON public.audit_log(entity_type, entity_id);
CREATE INDEX idx_audit_log_created ON public.audit_log(created_at DESC);

-- Revoke update/delete on audit_log from all roles
REVOKE UPDATE, DELETE ON public.audit_log FROM PUBLIC, authenticated, anon, service_role;
```

### 3.3 Donor Management

```sql
-- File: 0003_create_donors.sql

CREATE TABLE public.donors (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES public.user_profiles(id) ON DELETE CASCADE,
  donor_number TEXT UNIQUE NOT NULL,               -- human-readable: MWS-D-NNNNNN
  blood_type TEXT NOT NULL CHECK (blood_type IN ('A+','A-','B+','B-','AB+','AB-','O+','O-')),
  date_of_birth DATE NOT NULL,
  gender TEXT NOT NULL CHECK (gender IN ('male', 'female', 'other')),
  cnic TEXT UNIQUE,                                -- Pakistani national ID
  passport TEXT UNIQUE,
  address TEXT NOT NULL,
  city TEXT NOT NULL,
  district TEXT NOT NULL,
  province TEXT NOT NULL CHECK (province IN ('Punjab','Sindh','KPK','Balochistan','Islamabad','GB','AJK')),
  postal_code TEXT,
  emergency_contact_name TEXT NOT NULL,
  emergency_contact_phone TEXT NOT NULL,
  emergency_contact_relation TEXT,
  height_cm INTEGER,
  weight_kg NUMERIC(5,2),
  medical_screening JSONB NOT NULL DEFAULT '{}',   -- per WHO donor selection
  current_medications JSONB DEFAULT '[]',
  recent_travel JSONB DEFAULT '[]',                -- malaria-endemic regions
  recent_tattoos_procedures JSONB DEFAULT '[]',
  deferral_until DATE,                             -- temporary deferral end
  deferral_reason TEXT,
  permanent_deferral BOOLEAN NOT NULL DEFAULT FALSE,
  permanent_deferral_reason TEXT,
  emergency_availability_status TEXT NOT NULL DEFAULT 'unavailable'
    CHECK (emergency_availability_status IN ('available', 'unavailable', 'limited')),
  emergency_availability_until TIMESTAMPTZ,
  communication_preferences JSONB NOT NULL DEFAULT '{
    "channels": { "email": true, "sms": false, "whatsapp": true, "push": true },
    "topics": { "requests": true, "reminders": true, "reports": true, "marketing": false }
  }',
  consent_data_processing BOOLEAN NOT NULL DEFAULT FALSE,
  consent_marketing BOOLEAN NOT NULL DEFAULT FALSE,
  consent_ai_use BOOLEAN NOT NULL DEFAULT FALSE,
  consent_public_story BOOLEAN NOT NULL DEFAULT FALSE,
  consent_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES public.user_profiles(id),
  updated_by UUID REFERENCES public.user_profiles(id),
  deleted_at TIMESTAMPTZ,
  deleted_by UUID REFERENCES public.user_profiles(id)
);

CREATE UNIQUE INDEX idx_donors_user_active ON public.donors(user_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_donors_blood_type ON public.donors(blood_type) WHERE deleted_at IS NULL;
CREATE INDEX idx_donors_city ON public.donors(city) WHERE deleted_at IS NULL;
CREATE INDEX idx_donors_district ON public.donors(district) WHERE deleted_at IS NULL;
CREATE INDEX idx_donors_availability ON public.donors(emergency_availability_status, emergency_availability_until)
  WHERE deleted_at IS NULL;
CREATE INDEX idx_donors_deferral ON public.donors(deferral_until, permanent_deferral)
  WHERE deleted_at IS NULL;
CREATE INDEX idx_donors_search ON public.donors USING gin (to_tsvector('english', full_name || ' ' || COALESCE(cnic,'')))
  WHERE deleted_at IS NULL;

-- File: 0004_create_donations.sql

CREATE TABLE public.donations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  donor_id UUID NOT NULL REFERENCES public.donors(id) ON DELETE RESTRICT,
  donation_date TIMESTAMPTZ NOT NULL,
  location_type TEXT NOT NULL CHECK (location_type IN ('camp', 'office', 'hospital')),
  location_id UUID,                                -- references camps.id or hospitals.id
  location_name TEXT NOT NULL,
  collected_by UUID REFERENCES public.user_profiles(id),  -- volunteer or staff
  component_type TEXT NOT NULL CHECK (component_type IN ('whole_blood', 'plasma', 'platelets', 'cryoprecipitate', 'double_red_cells')),
  volume_ml INTEGER NOT NULL CHECK (volume_ml > 0),
  bag_id UUID,                                     -- FK to blood_bags.id (set after bag creation)
  hemoglobin_pre NUMERIC(4,1),
  blood_pressure_pre TEXT,
  pulse_pre INTEGER,
  outcome TEXT NOT NULL CHECK (outcome IN ('successful', 'deferred', 'incomplete', 'adverse_reaction')),
  deferral_reason TEXT,
  adverse_reaction TEXT,
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES public.user_profiles(id),
  updated_by UUID REFERENCES public.user_profiles(id),
  deleted_at TIMESTAMPTZ
);

CREATE INDEX idx_donations_donor_date ON public.donations(donor_id, donation_date DESC) WHERE deleted_at IS NULL;
CREATE INDEX idx_donations_date ON public.donations(donation_date DESC) WHERE deleted_at IS NULL;
CREATE INDEX idx_donations_location ON public.donations(location_type, location_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_donations_outcome ON public.donations(outcome) WHERE deleted_at IS NULL;
```

### 3.4 Inventory

```sql
-- File: 0005_create_blood_bags.sql

CREATE TABLE public.storage_locations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  code TEXT UNIQUE NOT NULL,                       -- e.g., FRIDGE-01
  location_type TEXT NOT NULL CHECK (location_type IN ('refrigerator', 'freezer', 'transport')),
  capacity INTEGER,
  temperature_min NUMERIC(4,1),                    -- Celsius
  temperature_max NUMERIC(4,1),
  address TEXT,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE public.blood_bags (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  bag_number TEXT UNIQUE NOT NULL,                 -- MWS-YYYY-NNNNNN
  qr_code_token UUID NOT NULL DEFAULT uuid_generate_v4(),  -- for QR verification
  donor_id UUID REFERENCES public.donors(id) ON DELETE RESTRICT,
  donation_id UUID REFERENCES public.donations(id) ON DELETE RESTRICT,
  blood_type TEXT NOT NULL CHECK (blood_type IN ('A+','A-','B+','B-','AB+','AB-','O+','O-')),
  component_type TEXT NOT NULL CHECK (component_type IN ('whole_blood', 'plasma', 'platelets', 'cryoprecipitate', 'double_red_cells')),
  volume_ml INTEGER NOT NULL CHECK (volume_ml > 0),
  collection_date TIMESTAMPTZ NOT NULL,
  expiry_date TIMESTAMPTZ NOT NULL,
  storage_location_id UUID REFERENCES public.storage_locations(id),
  status TEXT NOT NULL DEFAULT 'COLLECTED'
    CHECK (status IN ('COLLECTED', 'SCREENED', 'STORED', 'RESERVED', 'ISSUED', 'DELIVERED', 'RETURNED', 'EXPIRED', 'DISCARDED')),
  screening_result JSONB,                          -- test results: HIV, HepB, HepC, etc.
  screening_status TEXT NOT NULL DEFAULT 'PENDING'
    CHECK (screening_status IN ('PENDING', 'CLEARED', 'REJECTED')),
  current_state_changed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  current_state_changed_by UUID REFERENCES public.user_profiles(id),
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES public.user_profiles(id),
  updated_by UUID REFERENCES public.user_profiles(id),
  deleted_at TIMESTAMPTZ
);

CREATE INDEX idx_blood_bags_blood_type_status ON public.blood_bags(blood_type, status)
  WHERE deleted_at IS NULL;
CREATE INDEX idx_blood_bags_expiry ON public.blood_bags(expiry_date) WHERE deleted_at IS NULL AND status IN ('STORED', 'RESERVED');
CREATE INDEX idx_blood_bags_storage ON public.blood_bags(storage_location_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_blood_bags_donor ON public.blood_bags(donor_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_blood_bags_status ON public.blood_bags(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_blood_bags_qr ON public.blood_bags(qr_code_token);

-- blood_bag_state_history: append-only state transitions
CREATE TABLE public.blood_bag_state_history (
  id BIGSERIAL PRIMARY KEY,
  bag_id UUID NOT NULL REFERENCES public.blood_bags(id) ON DELETE CASCADE,
  from_status TEXT,
  to_status TEXT NOT NULL,
  changed_by UUID REFERENCES public.user_profiles(id),
  changed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  reason TEXT,
  location_id UUID REFERENCES public.storage_locations(id)
);

CREATE INDEX idx_bag_history_bag ON public.blood_bag_state_history(bag_id, changed_at DESC);
```

### 3.5 Patients & Hospitals

```sql
-- File: 0006_create_patients.sql

CREATE TABLE public.patients (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  patient_number TEXT UNIQUE NOT NULL,             -- MWS-P-NNNNNN
  full_name TEXT NOT NULL,
  date_of_birth DATE,
  age INTEGER,
  gender TEXT CHECK (gender IN ('male', 'female', 'other')),
  blood_type TEXT CHECK (blood_type IN ('A+','A-','B+','B-','AB+','AB-','O+','O-')),
  cnic TEXT,
  disease_condition TEXT NOT NULL,
  disease_details TEXT,
  treating_hospital_id UUID REFERENCES public.hospitals(id),
  treating_doctor_name TEXT,
  treating_doctor_contact TEXT,
  guardian_name TEXT NOT NULL,
  guardian_relation TEXT,
  guardian_contact TEXT NOT NULL,
  guardian_user_id UUID REFERENCES public.user_profiles(id),  -- if registered
  status TEXT NOT NULL DEFAULT 'ACTIVE' CHECK (status IN ('ACTIVE', 'INACTIVE', 'DECEASED')),
  consent_data_processing BOOLEAN NOT NULL DEFAULT FALSE,
  consent_public_story BOOLEAN NOT NULL DEFAULT FALSE,
  consent_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES public.user_profiles(id),
  updated_by UUID REFERENCES public.user_profiles(id),
  deleted_at TIMESTAMPTZ
);

CREATE INDEX idx_patients_hospital ON public.patients(treating_hospital_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_patients_guardian ON public.patients(guardian_user_id) WHERE deleted_at IS NULL AND guardian_user_id IS NOT NULL;
CREATE INDEX idx_patients_search ON public.patients USING gin (to_tsvector('english', full_name || ' ' || COALESCE(patient_number,'')))
  WHERE deleted_at IS NULL;

-- File: 0007_create_hospitals.sql

CREATE TABLE public.hospitals (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  hospital_number TEXT UNIQUE NOT NULL,            -- MWS-H-NNNNNN
  name TEXT NOT NULL,
  license_number TEXT,
  address TEXT NOT NULL,
  city TEXT NOT NULL,
  district TEXT NOT NULL,
  province TEXT NOT NULL,
  contact_phone TEXT NOT NULL,
  contact_email CITEXT,
  contact_person_name TEXT NOT NULL,
  contact_person_designation TEXT,
  partnership_status TEXT NOT NULL DEFAULT 'PENDING'
    CHECK (partnership_status IN ('PENDING', 'ACTIVE', 'SUSPENDED', 'TERMINATED')),
  partnership_started_at DATE,
  partnership_ended_at DATE,
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES public.user_profiles(id),
  updated_by UUID REFERENCES public.user_profiles(id),
  deleted_at TIMESTAMPTZ
);

CREATE INDEX idx_hospitals_status ON public.hospitals(partnership_status) WHERE deleted_at IS NULL;
CREATE INDEX idx_hospitals_city ON public.hospitals(city) WHERE deleted_at IS NULL;

-- hospital_staff: links user_profiles to a hospital
CREATE TABLE public.hospital_staff (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  hospital_id UUID NOT NULL REFERENCES public.hospitals(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES public.user_profiles(id) ON DELETE CASCADE,
  role TEXT NOT NULL DEFAULT 'STAFF' CHECK (role IN ('STAFF', 'ADMIN')),  -- within hospital
  designation TEXT,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (hospital_id, user_id)
);

CREATE INDEX idx_hospital_staff_hospital ON public.hospital_staff(hospital_id) WHERE is_active = TRUE;
CREATE INDEX idx_hospital_staff_user ON public.hospital_staff(user_id) WHERE is_active = TRUE;

-- patient_documents: file uploads for patients
CREATE TABLE public.patient_documents (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  patient_id UUID NOT NULL REFERENCES public.patients(id) ON DELETE CASCADE,
  document_type TEXT NOT NULL CHECK (document_type IN ('prescription', 'hospital_letter', 'lab_report', 'consent_form', 'other')),
  file_path TEXT NOT NULL,                          -- Supabase Storage path
  file_name TEXT NOT NULL,
  file_size_bytes BIGINT NOT NULL,
  mime_type TEXT NOT NULL,
  virus_scan_status TEXT NOT NULL DEFAULT 'PENDING'
    CHECK (virus_scan_status IN ('PENDING', 'CLEAN', 'INFECTED', 'ERROR')),
  virus_scanned_at TIMESTAMPTZ,
  uploaded_by UUID REFERENCES public.user_profiles(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  deleted_at TIMESTAMPTZ
);

CREATE INDEX idx_patient_documents_patient ON public.patient_documents(patient_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_patient_documents_scan ON public.patient_documents(virus_scan_status) WHERE deleted_at IS NULL;
```

### 3.6 Requests, Allocations, Deliveries

```sql
-- File: 0008_create_requests.sql

CREATE TABLE public.blood_requests (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  request_number TEXT UNIQUE NOT NULL,             -- MWS-R-YYYYMMDD-NNNN
  requester_type TEXT NOT NULL CHECK (requester_type IN ('hospital', 'guardian', 'admin')),
  requester_user_id UUID REFERENCES public.user_profiles(id),
  requester_hospital_id UUID REFERENCES public.hospitals(id),
  patient_id UUID NOT NULL REFERENCES public.patients(id) ON DELETE RESTRICT,
  blood_type_needed TEXT NOT NULL CHECK (blood_type_needed IN ('A+','A-','B+','B-','AB+','AB-','O+','O-')),
  component_type_needed TEXT NOT NULL CHECK (component_type_needed IN ('whole_blood', 'plasma', 'platelets', 'cryoprecipitate', 'double_red_cells')),
  quantity_units INTEGER NOT NULL CHECK (quantity_units > 0),
  units_allocated INTEGER NOT NULL DEFAULT 0,
  units_issued INTEGER NOT NULL DEFAULT 0,
  units_delivered INTEGER NOT NULL DEFAULT 0,
  urgency TEXT NOT NULL CHECK (urgency IN ('routine', 'urgent', 'emergency')),
  clinical_justification TEXT NOT NULL,
  required_by_date TIMESTAMPTZ NOT NULL,
  delivery_location TEXT NOT NULL,
  delivery_address TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'SUBMITTED'
    CHECK (status IN ('SUBMITTED', 'VERIFIED', 'MATCHING', 'ALLOCATED', 'DISPATCHED', 'DELIVERED', 'RECEIVED', 'CLOSED', 'REJECTED', 'CANCELLED')),
  status_changed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  status_changed_by UUID REFERENCES public.user_profiles(id),
  sla_deadline TIMESTAMPTZ,                        -- computed from urgency
  sla_breached BOOLEAN NOT NULL DEFAULT FALSE,
  rejection_reason TEXT,
  cancellation_reason TEXT,
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES public.user_profiles(id),
  updated_by UUID REFERENCES public.user_profiles(id),
  deleted_at TIMESTAMPTZ
);

CREATE INDEX idx_requests_status ON public.blood_requests(status, urgency) WHERE deleted_at IS NULL;
CREATE INDEX idx_requests_hospital ON public.blood_requests(requester_hospital_id, status) WHERE deleted_at IS NULL;
CREATE INDEX idx_requests_patient ON public.blood_requests(patient_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_requests_blood_type ON public.blood_requests(blood_type_needed, component_type_needed, status) WHERE deleted_at IS NULL;
CREATE INDEX idx_requests_sla ON public.blood_requests(sla_deadline) WHERE deleted_at IS NULL AND status NOT IN ('CLOSED', 'REJECTED', 'CANCELLED');
CREATE INDEX idx_requests_created ON public.blood_requests(created_at DESC) WHERE deleted_at IS NULL;

-- File: 0009_create_allocations.sql

CREATE TABLE public.allocations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  request_id UUID NOT NULL REFERENCES public.blood_requests(id) ON DELETE RESTRICT,
  bag_id UUID NOT NULL REFERENCES public.blood_bags(id) ON DELETE RESTRICT,
  allocated_by UUID NOT NULL REFERENCES public.user_profiles(id),
  allocated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  status TEXT NOT NULL DEFAULT 'ALLOCATED'
    CHECK (status IN ('ALLOCATED', 'ISSUED', 'DELIVERED', 'RETURNED', 'CANCELLED')),
  cancelled_at TIMESTAMPTZ,
  cancelled_by UUID REFERENCES public.user_profiles(id),
  cancel_reason TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_allocations_bag_active ON public.allocations(bag_id) WHERE status IN ('ALLOCATED', 'ISSUED');
CREATE INDEX idx_allocations_request ON public.allocations(request_id);

-- File: 0010_create_deliveries.sql

CREATE TABLE public.deliveries (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  delivery_number TEXT UNIQUE NOT NULL,
  request_id UUID NOT NULL REFERENCES public.blood_requests(id) ON DELETE RESTRICT,
  allocation_ids UUID[] NOT NULL,                   -- multiple bags
  dispatch_type TEXT NOT NULL CHECK (dispatch_type IN ('courier', 'pickup', 'volunteer', 'staff')),
  dispatched_by UUID REFERENCES public.user_profiles(id),
  dispatched_at TIMESTAMPTZ,
  courier_name TEXT,
  courier_contact TEXT,
  vehicle_number TEXT,
  expected_arrival_at TIMESTAMPTZ,
  delivered_at TIMESTAMPTZ,
  received_by_name TEXT,
  received_by_signature_path TEXT,                  -- storage path
  received_by_otp_verified BOOLEAN NOT NULL DEFAULT FALSE,
  status TEXT NOT NULL DEFAULT 'PENDING'
    CHECK (status IN ('PENDING', 'DISPATCHED', 'IN_TRANSIT', 'DELIVERED', 'RETURNED', 'CANCELLED')),
  cold_chain_logs JSONB DEFAULT '[]',               -- temperature readings
  cold_chain_breached BOOLEAN NOT NULL DEFAULT FALSE,
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES public.user_profiles(id),
  updated_by UUID REFERENCES public.user_profiles(id)
);

CREATE INDEX idx_deliveries_request ON public.deliveries(request_id);
CREATE INDEX idx_deliveries_status ON public.deliveries(status);
CREATE INDEX idx_deliveries_dispatched ON public.deliveries(dispatched_at DESC) WHERE dispatched_at IS NOT NULL;
```

### 3.7 Volunteers, Tasks, Camps

```sql
-- File: 0011_create_volunteers.sql

CREATE TABLE public.volunteers (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES public.user_profiles(id) ON DELETE CASCADE,
  volunteer_number TEXT UNIQUE NOT NULL,            -- MWS-V-NNNNNN
  skills JSONB NOT NULL DEFAULT '[]',
  preferred_task_types JSONB NOT NULL DEFAULT '[]',
  availability_calendar JSONB NOT NULL DEFAULT '{}',  -- per-day availability
  geographic_area TEXT,
  city TEXT NOT NULL,
  district TEXT NOT NULL,
  emergency_contact_name TEXT NOT NULL,
  emergency_contact_phone TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'ACTIVE' CHECK (status IN ('ACTIVE', 'INACTIVE', 'SUSPENDED')),
  total_hours NUMERIC(7,2) NOT NULL DEFAULT 0,
  total_tasks_completed INTEGER NOT NULL DEFAULT 0,
  total_donors_registered INTEGER NOT NULL DEFAULT 0,
  joined_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  last_active_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES public.user_profiles(id),
  updated_by UUID REFERENCES public.user_profiles(id),
  deleted_at TIMESTAMPTZ
);

CREATE UNIQUE INDEX idx_volunteers_user_active ON public.volunteers(user_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_volunteers_status ON public.volunteers(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_volunteers_district ON public.volunteers(district) WHERE deleted_at IS NULL AND status = 'ACTIVE';

-- File: 0012_create_tasks.sql

CREATE TABLE public.tasks (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  task_number TEXT UNIQUE NOT NULL,
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  task_type TEXT NOT NULL CHECK (task_type IN (
    'camp_setup', 'donor_registration', 'donation_recording', 'delivery',
    'patient_followup', 'inventory_management', 'administrative', 'other'
  )),
  assigned_to UUID REFERENCES public.volunteers(id) ON DELETE SET NULL,
  assigned_by UUID NOT NULL REFERENCES public.user_profiles(id),
  assigned_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  accepted_at TIMESTAMPTZ,
  declined_at TIMESTAMPTZ,
  decline_reason TEXT,
  location_name TEXT,
  location_address TEXT,
  location_lat NUMERIC(10,7),
  location_lng NUMERIC(10,7),
  scheduled_start_at TIMESTAMPTZ NOT NULL,
  scheduled_end_at TIMESTAMPTZ NOT NULL,
  actual_start_at TIMESTAMPTZ,                     -- check-in
  actual_end_at TIMESTAMPTZ,                       -- check-out
  actual_hours NUMERIC(5,2),
  status TEXT NOT NULL DEFAULT 'ASSIGNED'
    CHECK (status IN ('ASSIGNED', 'ACCEPTED', 'DECLINED', 'CHECKED_IN', 'COMPLETED', 'NO_SHOW', 'CANCELLED')),
  related_camp_id UUID REFERENCES public.camps(id),
  related_request_id UUID REFERENCES public.blood_requests(id),
  outcome_notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES public.user_profiles(id),
  updated_by UUID REFERENCES public.user_profiles(id),
  deleted_at TIMESTAMPTZ
);

CREATE INDEX idx_tasks_assignee ON public.tasks(assigned_to, status) WHERE deleted_at IS NULL;
CREATE INDEX idx_tasks_scheduled ON public.tasks(scheduled_start_at) WHERE deleted_at IS NULL;
CREATE INDEX idx_tasks_status ON public.tasks(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_tasks_camp ON public.tasks(related_camp_id) WHERE deleted_at IS NULL AND related_camp_id IS NOT NULL;

-- File: 0013_create_camps.sql

CREATE TABLE public.camps (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  camp_number TEXT UNIQUE NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  partner_institution TEXT,
  start_at TIMESTAMPTZ NOT NULL,
  end_at TIMESTAMPTZ NOT NULL,
  location_name TEXT NOT NULL,
  location_address TEXT NOT NULL,
  location_lat NUMERIC(10,7),
  location_lng NUMERIC(10,7),
  city TEXT NOT NULL,
  district TEXT NOT NULL,
  target_collections INTEGER NOT NULL DEFAULT 50,
  actual_collections INTEGER NOT NULL DEFAULT 0,
  assigned_volunteers UUID[] DEFAULT '{}',
  coordinator_id UUID REFERENCES public.user_profiles(id),
  status TEXT NOT NULL DEFAULT 'SCHEDULED'
    CHECK (status IN ('SCHEDULED', 'IN_PROGRESS', 'COMPLETED', 'CANCELLED')),
  pre_registration_enabled BOOLEAN NOT NULL DEFAULT TRUE,
  pre_registered_count INTEGER NOT NULL DEFAULT 0,
  report_generated_at TIMESTAMPTZ,
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES public.user_profiles(id),
  updated_by UUID REFERENCES public.user_profiles(id),
  deleted_at TIMESTAMPTZ
);

CREATE INDEX idx_camps_status_start ON public.camps(status, start_at) WHERE deleted_at IS NULL;
CREATE INDEX idx_camps_city ON public.camps(city) WHERE deleted_at IS NULL;
CREATE INDEX idx_camps_coordinator ON public.camps(coordinator_id) WHERE deleted_at IS NULL;
```

### 3.8 Notifications, Reports, Knowledge Base

```sql
-- File: 0014_create_notifications.sql

CREATE TABLE public.notification_templates (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  code TEXT UNIQUE NOT NULL,                       -- e.g., 'donor_thank_you'
  name TEXT NOT NULL,
  description TEXT,
  channel TEXT NOT NULL CHECK (channel IN ('email', 'sms', 'whatsapp', 'push', 'in_app')),
  subject_template TEXT,                           -- for email
  body_template TEXT NOT NULL,                     -- with {{variables}}
  variables JSONB NOT NULL DEFAULT '[]',           -- expected variable names
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  version INTEGER NOT NULL DEFAULT 1,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE public.notifications (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  template_id UUID REFERENCES public.notification_templates(id),
  recipient_user_id UUID REFERENCES public.user_profiles(id) ON DELETE CASCADE,
  recipient_address TEXT,                          -- email/phone/etc
  channel TEXT NOT NULL CHECK (channel IN ('email', 'sms', 'whatsapp', 'push', 'in_app')),
  subject TEXT,
  body TEXT NOT NULL,
  variables JSONB DEFAULT '{}',
  status TEXT NOT NULL DEFAULT 'PENDING'
    CHECK (status IN ('PENDING', 'SENT', 'DELIVERED', 'FAILED', 'SUPPRESSED')),
  priority TEXT NOT NULL DEFAULT 'NORMAL' CHECK (priority IN ('LOW', 'NORMAL', 'HIGH', 'EMERGENCY')),
  scheduled_for TIMESTAMPTZ,
  sent_at TIMESTAMPTZ,
  delivered_at TIMESTAMPTZ,
  failed_at TIMESTAMPTZ,
  failure_reason TEXT,
  retry_count INTEGER NOT NULL DEFAULT 0,
  external_id TEXT,                                -- provider message ID
  related_entity_type TEXT,
  related_entity_id TEXT,
  correlation_id UUID,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_notifications_recipient ON public.notifications(recipient_user_id, status) WHERE deleted_at IS NULL;
CREATE INDEX idx_notifications_status_scheduled ON public.notifications(status, scheduled_for) WHERE status = 'PENDING';
CREATE INDEX idx_notifications_priority ON public.notifications(priority, status);

-- File: 0015_create_reports.sql

CREATE TABLE public.reports (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  report_type TEXT NOT NULL CHECK (report_type IN (
    'daily_digest', 'monthly_impact', 'annual', 'camp_summary',
    'inventory_reconciliation', 'donor_analytics', 'request_analytics',
    'volunteer_analytics', 'custom'
  )),
  title TEXT NOT NULL,
  period_start TIMESTAMPTZ NOT NULL,
  period_end TIMESTAMPTZ NOT NULL,
  parameters JSONB DEFAULT '{}',
  generated_content JSONB,                         -- structured data
  narrative_content TEXT,                          -- AI-drafted narrative
  ai_generated BOOLEAN NOT NULL DEFAULT FALSE,
  ai_model TEXT,
  ai_prompt TEXT,
  file_path TEXT,                                  -- generated PDF path
  status TEXT NOT NULL DEFAULT 'DRAFT'
    CHECK (status IN ('DRAFT', 'IN_REVIEW', 'APPROVED', 'PUBLISHED', 'REJECTED')),
  generated_by UUID,                               -- system or user
  reviewed_by UUID REFERENCES public.user_profiles(id),
  reviewed_at TIMESTAMPTZ,
  approved_by UUID REFERENCES public.user_profiles(id),
  approved_at TIMESTAMPTZ,
  published_at TIMESTAMPTZ,
  is_public BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_reports_type_period ON public.reports(report_type, period_start DESC);
CREATE INDEX idx_reports_status ON public.reports(status);
CREATE INDEX idx_reports_public ON public.reports(is_public, published_at DESC) WHERE is_public = TRUE;

-- File: 0018_create_kb_articles.sql

CREATE TABLE public.kb_articles (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  slug TEXT UNIQUE NOT NULL,
  title TEXT NOT NULL,
  excerpt TEXT,
  content TEXT NOT NULL,                           -- Markdown
  category TEXT NOT NULL,
  tags TEXT[] DEFAULT '{}',
  author_id UUID REFERENCES public.user_profiles(id),
  status TEXT NOT NULL DEFAULT 'DRAFT'
    CHECK (status IN ('DRAFT', 'PUBLISHED', 'UNPUBLISHED', 'ARCHIVED')),
  published_at TIMESTAMPTZ,
  view_count INTEGER NOT NULL DEFAULT 0,
  is_featured BOOLEAN NOT NULL DEFAULT FALSE,
  search_vector tsvector,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_kb_articles_status_published ON public.kb_articles(status, published_at DESC) WHERE status = 'PUBLISHED';
CREATE INDEX idx_kb_articles_category ON public.kb_articles(category) WHERE status = 'PUBLISHED';
CREATE INDEX idx_kb_articles_search ON public.kb_articles USING gin(search_vector);
```

### 3.9 Settings

```sql
-- File: 0017_create_settings.sql

CREATE TABLE public.settings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  key TEXT UNIQUE NOT NULL,
  value JSONB NOT NULL,
  description TEXT,
  category TEXT NOT NULL,
  is_sensitive BOOLEAN NOT NULL DEFAULT FALSE,
  is_system BOOLEAN NOT NULL DEFAULT FALSE,        -- system-level (not editable in UI)
  requires_super_admin BOOLEAN NOT NULL DEFAULT FALSE,
  version INTEGER NOT NULL DEFAULT 1,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_by UUID REFERENCES public.user_profiles(id)
);

CREATE INDEX idx_settings_category ON public.settings(category);

-- settings_history: append-only settings audit
CREATE TABLE public.settings_history (
  id BIGSERIAL PRIMARY KEY,
  key TEXT NOT NULL,
  old_value JSONB,
  new_value JSONB NOT NULL,
  changed_by UUID REFERENCES public.user_profiles(id),
  changed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  reason TEXT
);

CREATE INDEX idx_settings_history_key ON public.settings_history(key, changed_at DESC);
```

### 3.10 Domain Events

```sql
-- Domain events table for cross-context async communication
CREATE TABLE public.domain_events (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  event_type TEXT NOT NULL,                        -- e.g., 'donation.recorded'
  entity_type TEXT NOT NULL,
  entity_id TEXT NOT NULL,
  payload JSONB NOT NULL,
  occurred_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  processed_at TIMESTAMPTZ,
  processing_error TEXT,
  retry_count INTEGER NOT NULL DEFAULT 0,
  consumed_by TEXT[] DEFAULT '{}'                  -- which consumers have processed
);

CREATE INDEX idx_domain_events_unprocessed ON public.domain_events(occurred_at) WHERE processed_at IS NULL;
CREATE INDEX idx_domain_events_type ON public.domain_events(event_type, occurred_at DESC);
```

---

## 4. Entity-Relationship Diagram

### 4.1 Mermaid ERD

```mermaid
erDiagram
    user_profiles ||--o{ user_roles : "has"
    user_profiles ||--o{ user_sessions : "has"
    user_profiles ||--o| donors : "is"
    user_profiles ||--o| volunteers : "is"
    user_profiles ||--o{ hospital_staff : "member_of"
    user_profiles ||--o{ audit_log : "performs"
    
    donors ||--o{ donations : "makes"
    donors ||--o{ blood_bags : "provides"
    donors }o--|| user_profiles : "account"
    
    donations ||--|| blood_bags : "produces"
    donations }o--o| camps : "at"
    
    blood_bags ||--o{ blood_bag_state_history : "transitions"
    blood_bags }o--|| storage_locations : "stored_in"
    blood_bags ||--o{ allocations : "allocated_in"
    
    hospitals ||--o{ hospital_staff : "employs"
    hospitals ||--o{ patients : "treats"
    hospitals ||--o{ blood_requests : "requests"
    
    patients ||--o{ blood_requests : "needs"
    patients ||--o{ patient_documents : "has"
    patients }o--o| user_profiles : "guardian_account"
    
    blood_requests ||--o{ allocations : "fulfilled_by"
    blood_requests ||--o{ deliveries : "delivered_via"
    
    allocations }o--|| blood_bags : "reserves"
    allocations }o--|| blood_requests : "for"
    
    deliveries }o--|| blood_requests : "for"
    
    volunteers ||--o{ tasks : "assigned"
    volunteers }o--|| user_profiles : "account"
    
    tasks }o--o| camps : "part_of"
    tasks }o--o| blood_requests : "related_to"
    
    camps ||--o{ tasks : "has"
    camps ||--o{ donations : "collects"
    
    user_profiles ||--o{ notifications : "receives"
    notification_templates ||--o{ notifications : "instantiates"
    
    blood_bags ||--o{ audit_log : "audited"
    blood_requests ||--o{ audit_log : "audited"
    donors ||--o{ audit_log : "audited"

    user_profiles {
        uuid id PK
        citext email
        text phone
        text full_name
        text preferred_language
        text status
    }
    
    donors {
        uuid id PK
        uuid user_id FK
        text donor_number
        text blood_type
        date date_of_birth
        text cnic
        date deferral_until
        boolean permanent_deferral
        text emergency_availability_status
        jsonb communication_preferences
    }
    
    blood_bags {
        uuid id PK
        text bag_number
        uuid donor_id FK
        uuid donation_id FK
        text blood_type
        text component_type
        int volume_ml
        timestamptz collection_date
        timestamptz expiry_date
        text status
        text screening_status
    }
    
    blood_requests {
        uuid id PK
        text request_number
        text requester_type
        uuid patient_id FK
        text blood_type_needed
        text component_type_needed
        int quantity_units
        text urgency
        text status
        timestamptz sla_deadline
    }
    
    patients {
        uuid id PK
        text patient_number
        text full_name
        text blood_type
        text disease_condition
        uuid treating_hospital_id FK
        text guardian_name
        uuid guardian_user_id FK
    }
    
    hospitals {
        uuid id PK
        text hospital_number
        text name
        text partnership_status
        text city
    }
```

### 4.2 ASCII ERD (Simplified)

For environments where Mermaid is not rendered:

```
+-------------------+        +-------------------+        +-------------------+
|   user_profiles   |<-------|   user_roles      |        |   user_sessions   |
|-------------------|        |-------------------|        |-------------------|
| id (PK)           |<-------| user_id (FK)      |        | id (PK)           |
| email             |        | role              |        | user_id (FK)      |
| phone             |        | granted_at        |        | session_token_hash|
| full_name         |        | revoked_at        |        | expires_at        |
| preferred_language|        +-------------------+        +-------------------+
| status            |
+---------+---------+
          |
          | 1:1 (when role=DONOR)
          v
+-------------------+        +-------------------+        +-------------------+
|      donors       |        |    donations      |        |    blood_bags     |
|-------------------|        |-------------------|        |-------------------|
| id (PK)           |<-------| donor_id (FK)     |<-------| donor_id (FK)     |
| user_id (FK)      |        | donation_date     |        | donation_id (FK)  |
| donor_number      |        | component_type    |        | bag_number        |
| blood_type        |        | volume_ml         |        | blood_type        |
| date_of_birth     |        | outcome           |        | component_type    |
| cnic              |        | bag_id (FK)------>|--------| collection_date   |
| deferral_until    |        +-------------------+        | expiry_date       |
| emergency_avail.. |                                     | status            |
| consent_*         |                                     | screening_status  |
+-------------------+                                     +---------+---------+
                                                                    |
                                                                    | 1:many
                                                                    v
                                                          +-------------------+
                                                          |  allocations      |
                                                          |-------------------|
                                                          | bag_id (FK)       |
                                                          | request_id (FK)---|------+
                                                          | status            |      |
                                                          +-------------------+      |
                                                                                     |
+-------------------+        +-------------------+        +-------------------+      |
|     patients      |        |  blood_requests   |        |    deliveries    |      |
|-------------------|        |-------------------|        |-------------------|      |
| id (PK)           |<-------| patient_id (FK)   |<-------| request_id (FK)  |<-----+
| patient_number    |        | request_number    |        | delivery_number  |
| full_name         |        | blood_type_needed |        | dispatch_type    |
| blood_type        |        | component_needed  |        | dispatched_at    |
| disease_condition |        | urgency           |        | delivered_at     |
| guardian_user_id  |        | status            |        | status           |
| treating_hosp_id  |        | sla_deadline      |        +-------------------+
+-------------------+        +-------------------+
          |                            ^
          |                            | many:1
          v                            |
+-------------------+                  |
|    hospitals      |------------------+
|-------------------|
| id (PK)           |
| hospital_number   |
| name              |
| partnership_status|
+-------------------+
```

---

## 5. Row-Level Security Policies

RLS is enabled on every table containing user data. This section defines representative policies; the complete set is in `packages/database/rls/`.

### 5.1 Enabling RLS

```sql
-- File: 0020_enable_rls_policies.sql
ALTER TABLE public.user_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.user_roles ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.user_sessions ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.donors ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.donations ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.blood_bags ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.blood_bag_state_history ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.storage_locations ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.patients ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.patient_documents ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.hospitals ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.hospital_staff ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.blood_requests ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.allocations ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.deliveries ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.volunteers ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.tasks ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.camps ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.notifications ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.notification_templates ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.reports ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.kb_articles ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.settings ENABLE ROW LEVEL SECURITY;

-- Audit log is NOT RLS-protected; access is via SECURITY DEFINER function only
-- (SELECT revoked from all roles; SUPER_ADMIN reads via service_role)
```

### 5.2 Helper Function: `has_role`

```sql
CREATE OR REPLACE FUNCTION public.has_role(_user_id UUID, _role TEXT)
RETURNS BOOLEAN
LANGUAGE sql
SECURITY DEFINER
STABLE
SET search_path = public
AS $$
  SELECT EXISTS (
    SELECT 1 FROM public.user_roles
    WHERE user_id = _user_id
      AND role = _role
      AND revoked_at IS NULL
  );
$$;

CREATE OR REPLACE FUNCTION public.current_user_has_role(_role TEXT)
RETURNS BOOLEAN
LANGUAGE sql
STABLE
SECURITY DEFINER
SET search_path = public
AS $$
  SELECT public.has_role(auth.uid(), _role);
$$;
```

### 5.3 Donor RLS Policies

```sql
-- File: rls/donors.sql

-- Donors can read their own record
CREATE POLICY donors_self_read
  ON public.donors FOR SELECT
  TO authenticated
  USING (user_id = auth.uid());

-- Admins can read all donors
CREATE POLICY donors_admin_read
  ON public.donors FOR SELECT
  TO authenticated
  USING (public.current_user_has_role('SUPER_ADMIN') OR public.current_user_has_role('ADMIN'));

-- Volunteers can read limited donor fields at camps (handled via view, not direct table)
-- Donors do NOT get to see other donors

-- Donors can update their own record (limited fields)
CREATE POLICY donors_self_update
  ON public.donors FOR UPDATE
  TO authenticated
  USING (user_id = auth.uid())
  WITH CHECK (user_id = auth.uid());

-- Donors can insert their own record
CREATE POLICY donors_self_insert
  ON public.donors FOR INSERT
  TO authenticated
  WITH CHECK (user_id = auth.uid());

-- Admins can insert/update/delete donors
CREATE POLICY admins_modify_donors
  ON public.donors FOR ALL
  TO authenticated
  USING (public.current_user_has_role('SUPER_ADMIN') OR public.current_user_has_role('ADMIN'))
  WITH CHECK (public.current_user_has_role('SUPER_ADMIN') OR public.current_user_has_role('ADMIN'));
```

### 5.4 Blood Bag RLS Policies

```sql
-- File: rls/blood_bags.sql

-- Admins: full access
CREATE POLICY bags_admin_all
  ON public.blood_bags FOR ALL
  TO authenticated
  USING (public.current_user_has_role('SUPER_ADMIN') OR public.current_user_has_role('ADMIN'))
  WITH CHECK (public.current_user_has_role('SUPER_ADMIN') OR public.current_user_has_role('ADMIN'));

-- Volunteers: can INSERT during camp donation recording, but cannot SELECT/UPDATE/DELETE
CREATE POLICY bags_volunteer_insert
  ON public.blood_bags FOR INSERT
  TO authenticated
  WITH CHECK (public.current_user_has_role('VOLUNTEER'));

-- Hospitals: NO direct access to blood_bags table
-- (They see aggregate inventory via a view: public.inventory_aggregate_view)

-- Donors: can SELECT their own donated bags only
CREATE POLICY bags_donor_self_read
  ON public.blood_bags FOR SELECT
  TO authenticated
  USING (
    donor_id IN (SELECT id FROM public.donors WHERE user_id = auth.uid())
  );
```

### 5.5 Blood Request RLS Policies

```sql
-- File: rls/blood_requests.sql

-- Admins: full access
CREATE POLICY requests_admin_all
  ON public.blood_requests FOR ALL
  TO authenticated
  USING (public.current_user_has_role('SUPER_ADMIN') OR public.current_user_has_role('ADMIN'))
  WITH CHECK (public.current_user_has_role('SUPER_ADMIN') OR public.current_user_has_role('ADMIN'));

-- Hospital staff: full access to their own hospital's requests
CREATE POLICY requests_hospital_own
  ON public.blood_requests FOR ALL
  TO authenticated
  USING (
    requester_hospital_id IN (
      SELECT hospital_id FROM public.hospital_staff
      WHERE user_id = auth.uid() AND is_active = TRUE
    )
  )
  WITH CHECK (
    requester_hospital_id IN (
      SELECT hospital_id FROM public.hospital_staff
      WHERE user_id = auth.uid() AND is_active = TRUE
    )
  );

-- Patient guardians: full access to their own patients' requests
CREATE POLICY requests_guardian_own
  ON public.blood_requests FOR ALL
  TO authenticated
  USING (
    patient_id IN (
      SELECT id FROM public.patients WHERE guardian_user_id = auth.uid()
    )
  )
  WITH CHECK (
    patient_id IN (
      SELECT id FROM public.patients WHERE guardian_user_id = auth.uid()
    )
  );

-- Public: can INSERT new requests (via Edge Function that creates patient record)
-- (Not directly — requests are created via Edge Function with service role)
```

### 5.6 Audit Log Access

Audit log has NO RLS policies; instead, access is via SECURITY DEFINER function:

```sql
CREATE OR REPLACE FUNCTION public.read_audit_log(
  _entity_type TEXT DEFAULT NULL,
  _entity_id TEXT DEFAULT NULL,
  _actor_id UUID DEFAULT NULL,
  _limit INTEGER DEFAULT 100,
  _offset INTEGER DEFAULT 0
)
RETURNS SETOF public.audit_log
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = public
AS $$
BEGIN
  -- Only SUPER_ADMIN can read audit log
  IF NOT public.current_user_has_role('SUPER_ADMIN') THEN
    RAISE EXCEPTION 'Permission denied: audit log access requires SUPER_ADMIN role';
  END IF;
  
  RETURN QUERY
  SELECT * FROM public.audit_log
  WHERE (_entity_type IS NULL OR entity_type = _entity_type)
    AND (_entity_id IS NULL OR entity_id = _entity_id)
    AND (_actor_id IS NULL OR actor_id = _actor_id)
  ORDER BY created_at DESC
  LIMIT _limit OFFSET _offset;
END;
$$;

GRANT EXECUTE ON FUNCTION public.read_audit_log(...) TO authenticated;
```

### 5.7 RLS Testing Strategy

Every RLS policy has a corresponding test in `apps/web/tests/rls/`. Tests:

1. Create test users with each role.
2. Attempt each operation (SELECT, INSERT, UPDATE, DELETE) as each user.
3. Assert that authorized operations succeed and unauthorized operations fail.
4. Run in CI on every PR.

```typescript
// Example RLS test (simplified)
describe('blood_bags RLS', () => {
  it('allows admin to read all bags', async () => {
    const admin = await createTestUser(['ADMIN']);
    const { data, error } = await supabaseAs(admin).from('blood_bags').select('*');
    expect(error).toBeNull();
    expect(data.length).toBeGreaterThan(0);
  });
  
  it('prevents donor from reading other donors\' bags', async () => {
    const donor1 = await createTestUser(['DONOR']);
    const donor2 = await createTestUser(['DONOR']);
    await createBloodBag({ donor: donor2 });
    const { data, error } = await supabaseAs(donor1).from('blood_bags').select('*');
    expect(error).toBeNull();
    expect(data.find(b => b.donor_id === donor2.donorId)).toBeUndefined();
  });
});
```

---

## 6. TypeScript Type System

### 6.1 Generated Database Types

Database types are auto-generated via `supabase gen types typescript` and stored in `packages/database/types/database.types.ts`. Regenerated on every migration.

```typescript
// packages/database/types/database.types.ts (excerpt, generated)
export type Json = string | number | boolean | null | { [key: string]: Json | undefined } | Json[];

export interface Database {
  public: {
    Tables: {
      donors: {
        Row: {
          id: string;
          user_id: string;
          donor_number: string;
          blood_type: BloodType;
          date_of_birth: string;
          // ... all columns
        };
        Insert: {
          id?: string;
          user_id: string;
          donor_number: string;
          blood_type: BloodType;
          // ... required columns
        };
        Update: {
          id?: string;
          user_id?: string;
          // ... partial
        };
      };
      // ... other tables
    };
  };
}
```

### 6.2 Domain Entity Types

Domain types live in `packages/shared/src/types/` and are pure TypeScript (no Supabase dependency):

```typescript
// packages/shared/src/types/enums.ts
export const BloodType = {
  A_POSITIVE: 'A+',
  A_NEGATIVE: 'A-',
  B_POSITIVE: 'B+',
  B_NEGATIVE: 'B-',
  AB_POSITIVE: 'AB+',
  AB_NEGATIVE: 'AB-',
  O_POSITIVE: 'O+',
  O_NEGATIVE: 'O-',
} as const;
export type BloodType = (typeof BloodType)[keyof typeof BloodType];

export const ComponentType = {
  WHOLE_BLOOD: 'whole_blood',
  PLASMA: 'plasma',
  PLATELETS: 'platelets',
  CRYOPRECIPITATE: 'cryoprecipitate',
  DOUBLE_RED_CELLS: 'double_red_cells',
} as const;
export type ComponentType = (typeof ComponentType)[keyof typeof ComponentType];

export const Role = {
  SUPER_ADMIN: 'SUPER_ADMIN',
  ADMIN: 'ADMIN',
  VOLUNTEER: 'VOLUNTEER',
  HOSPITAL: 'HOSPITAL',
  DONOR: 'DONOR',
  PATIENT: 'PATIENT',
} as const;
export type Role = (typeof Role)[keyof typeof Role];

export const BloodBagStatus = {
  COLLECTED: 'COLLECTED',
  SCREENED: 'SCREENED',
  STORED: 'STORED',
  RESERVED: 'RESERVED',
  ISSUED: 'ISSUED',
  DELIVERED: 'DELIVERED',
  RETURNED: 'RETURNED',
  EXPIRED: 'EXPIRED',
  DISCARDED: 'DISCARDED',
} as const;
export type BloodBagStatus = (typeof BloodBagStatus)[keyof typeof BloodBagStatus];

export const RequestStatus = {
  SUBMITTED: 'SUBMITTED',
  VERIFIED: 'VERIFIED',
  MATCHING: 'MATCHING',
  ALLOCATED: 'ALLOCATED',
  DISPATCHED: 'DISPATCHED',
  DELIVERED: 'DELIVERED',
  RECEIVED: 'RECEIVED',
  CLOSED: 'CLOSED',
  REJECTED: 'REJECTED',
  CANCELLED: 'CANCELLED',
} as const;
export type RequestStatus = (typeof RequestStatus)[keyof typeof RequestStatus];

export const Urgency = {
  ROUTINE: 'routine',
  URGENT: 'urgent',
  EMERGENCY: 'emergency',
} as const;
export type Urgency = (typeof Urgency)[keyof typeof Urgency];
```

```typescript
// packages/shared/src/types/entities.ts
import type { BloodType, ComponentType, BloodBagStatus, RequestStatus, Urgency, Role } from './enums';

export interface Donor {
  id: string;
  userId: string;
  donorNumber: string;
  bloodType: BloodType;
  dateOfBirth: string;        // ISO date
  gender: 'male' | 'female' | 'other';
  cnic: string | null;
  city: string;
  district: string;
  province: string;
  emergencyContactName: string;
  emergencyContactPhone: string;
  deferralUntil: string | null;
  permanentDeferral: boolean;
  emergencyAvailabilityStatus: 'available' | 'unavailable' | 'limited';
  emergencyAvailabilityUntil: string | null;
  communicationPreferences: CommunicationPreferences;
  consent: {
    dataProcessing: boolean;
    marketing: boolean;
    aiUse: boolean;
    publicStory: boolean;
    at: string | null;
  };
  createdAt: string;
  updatedAt: string;
  deletedAt: string | null;
}

export interface CommunicationPreferences {
  channels: {
    email: boolean;
    sms: boolean;
    whatsapp: boolean;
    push: boolean;
  };
  topics: {
    requests: boolean;
    reminders: boolean;
    reports: boolean;
    marketing: boolean;
  };
}

export interface BloodBag {
  id: string;
  bagNumber: string;
  qrCodeToken: string;
  donorId: string;
  donationId: string;
  bloodType: BloodType;
  componentType: ComponentType;
  volumeMl: number;
  collectionDate: string;
  expiryDate: string;
  storageLocationId: string | null;
  status: BloodBagStatus;
  screeningStatus: 'PENDING' | 'CLEARED' | 'REJECTED';
  screeningResult: ScreeningResult | null;
  currentStateChangedAt: string;
  notes: string | null;
}

export interface BloodRequest {
  id: string;
  requestNumber: string;
  requesterType: 'hospital' | 'guardian' | 'admin';
  requesterUserId: string | null;
  requesterHospitalId: string | null;
  patientId: string;
  bloodTypeNeeded: BloodType;
  componentTypeNeeded: ComponentType;
  quantityUnits: number;
  unitsAllocated: number;
  unitsIssued: number;
  unitsDelivered: number;
  urgency: Urgency;
  clinicalJustification: string;
  requiredByDate: string;
  status: RequestStatus;
  slaDeadline: string | null;
  slaBreached: boolean;
  createdAt: string;
  updatedAt: string;
}

export interface AuthenticatedUser {
  id: string;
  email: string;
  fullName: string;
  roles: Role[];
  activeRole: Role;
  preferredLanguage: 'en-PK' | 'ur-PK';
}
```

### 6.3 DTO Types

DTOs are the wire format for API responses; they are projections of entity types:

```typescript
// packages/shared/src/types/dto.ts
export interface DonorDTO {
  id: string;
  donorNumber: string;
  bloodType: BloodType;
  fullName: string;            // from user_profiles, denormalized for performance
  city: string;
  donationCount: number;
  lastDonationDate: string | null;
  eligibilityStatus: 'eligible' | 'deferred' | 'permanent_deferral';
  nextEligibleDate: string | null;
  emergencyAvailability: 'available' | 'unavailable' | 'limited';
}

export interface BloodRequestDTO {
  id: string;
  requestNumber: string;
  patientName: string;          // anonymized for non-admin viewers
  bloodTypeNeeded: BloodType;
  componentTypeNeeded: ComponentType;
  quantityUnits: number;
  unitsAllocated: number;
  urgency: Urgency;
  status: RequestStatus;
  slaDeadline: string | null;
  createdAt: string;
  hospitalName?: string;
}

export interface ApiResponse<T> {
  data: T;
  message?: string;
}

export interface ApiListResponse<T> {
  data: T[];
  pagination: {
    page: number;
    pageSize: number;
    total: number;
    totalPages: number;
  };
}

export interface ApiError {
  error: {
    code: string;
    message: string;
    details?: Record<string, unknown>;
    correlationId: string;
  };
}
```

---

## 7. Validation Schemas (Zod)

### 7.1 Common Schemas

```typescript
// packages/shared/src/validation/common.ts
import { z } from 'zod';

export const uuidSchema = z.string().uuid();
export const isoDateSchema = z.string().datetime();
export const phoneSchema = z.string()
  .regex(/^\+92[0-9]{10}$/, 'Phone must be in format +923XXXXXXXXX');
export const cnicSchema = z.string()
  .regex(/^[0-9]{5}-[0-9]{7}-[0-9]$/, 'CNIC must be in format XXXXX-XXXXXXX-X');
export const emailSchema = z.string().email().max(255);

export const paginationSchema = z.object({
  page: z.number().int().min(1).default(1),
  pageSize: z.number().int().min(1).max(100).default(20),
  sort: z.string().optional(),
  order: z.enum(['asc', 'desc']).default('desc'),
  search: z.string().optional(),
});

export const bloodTypeSchema = z.enum(['A+', 'A-', 'B+', 'B-', 'AB+', 'AB-', 'O+', 'O-']);
export const componentTypeSchema = z.enum([
  'whole_blood', 'plasma', 'platelets', 'cryoprecipitate', 'double_red_cells'
]);
export const urgencySchema = z.enum(['routine', 'urgent', 'emergency']);
```

### 7.2 Donor Schemas

```typescript
// apps/web/src/features/donors/validation/create-donor.ts
import { z } from 'zod';
import { bloodTypeSchema, phoneSchema, cnicSchema, emailSchema } from '@marwat/shared/validation';

export const createDonorSchema = z.object({
  userId: z.string().uuid(),
  bloodType: bloodTypeSchema,
  dateOfBirth: z.string().date(),
  gender: z.enum(['male', 'female', 'other']),
  cnic: cnicSchema.optional(),
  passport: z.string().optional(),
  address: z.string().min(5).max(500),
  city: z.string().min(2).max(100),
  district: z.string().min(2).max(100),
  province: z.enum(['Punjab', 'Sindh', 'KPK', 'Balochistan', 'Islamabad', 'GB', 'AJK']),
  postalCode: z.string().optional(),
  emergencyContactName: z.string().min(2).max(100),
  emergencyContactPhone: phoneSchema,
  emergencyContactRelation: z.string().optional(),
  heightCm: z.number().int().min(100).max(250).optional(),
  weightKg: z.number().min(30).max(200).optional(),
  medicalScreening: z.object({
    hivPositive: z.boolean(),
    hepatitisB: z.boolean(),
    hepatitisC: z.boolean(),
    syphilis: z.boolean(),
    malaria: z.boolean(),
    diabetes: z.boolean(),
    hypertension: z.boolean(),
    heartDisease: z.boolean(),
    currentMedications: z.array(z.string()),
    recentTravelMalariaRegion: z.boolean(),
    recentTattooOrPiercing: z.boolean(),
    recentSurgery: z.boolean(),
    pregnantOrBreastfeeding: z.boolean().optional(),
  }),
  communicationPreferences: z.object({
    channels: z.object({
      email: z.boolean().default(true),
      sms: z.boolean().default(false),
      whatsapp: z.boolean().default(true),
      push: z.boolean().default(true),
    }),
    topics: z.object({
      requests: z.boolean().default(true),
      reminders: z.boolean().default(true),
      reports: z.boolean().default(true),
      marketing: z.boolean().default(false),
    }),
  }),
  consent: z.object({
    dataProcessing: z.literal(true, { message: 'Data processing consent is required' }),
    marketing: z.boolean().default(false),
    aiUse: z.boolean().default(false),
    publicStory: z.boolean().default(false),
  }),
}).refine(
  (data) => data.cnic || data.passport,
  { message: 'Either CNIC or passport is required' }
);

export type CreateDonorInput = z.infer<typeof createDonorSchema>;
```

### 7.3 Blood Request Schema

```typescript
// apps/web/src/features/requests/validation/create-request.ts
import { z } from 'zod';
import { bloodTypeSchema, componentTypeSchema, urgencySchema, phoneSchema } from '@marwat/shared/validation';

export const createBloodRequestSchema = z.object({
  patient: z.object({
    fullName: z.string().min(2).max(200),
    dateOfBirth: z.string().date().optional(),
    age: z.number().int().min(0).max(150).optional(),
    gender: z.enum(['male', 'female', 'other']).optional(),
    bloodType: bloodTypeSchema,
    cnic: z.string().optional(),
    diseaseCondition: z.string().min(2).max(500),
    diseaseDetails: z.string().max(2000).optional(),
    treatingDoctorName: z.string().optional(),
    treatingDoctorContact: z.string().optional(),
    guardianName: z.string().min(2).max(200),
    guardianRelation: z.string().optional(),
    guardianContact: phoneSchema,
  }),
  request: z.object({
    bloodTypeNeeded: bloodTypeSchema,
    componentTypeNeeded: componentTypeSchema,
    quantityUnits: z.number().int().min(1).max(20),
    urgency: urgencySchema,
    clinicalJustification: z.string().min(10).max(2000),
    requiredByDate: z.string().datetime(),
    deliveryLocation: z.string().min(2).max(200),
    deliveryAddress: z.string().min(5).max(500),
  }),
  documents: z.array(z.object({
    documentType: z.enum(['prescription', 'hospital_letter', 'lab_report', 'consent_form', 'other']),
    filePath: z.string(),
    fileName: z.string(),
    fileSizeBytes: z.number().int().positive(),
    mimeType: z.string(),
  })).default([]),
  consent: z.object({
    dataProcessing: z.literal(true),
    publicStory: z.boolean().default(false),
  }),
});

export type CreateBloodRequestInput = z.infer<typeof createBloodRequestSchema>;
```

---

## 8. API Design

### 8.1 API Style

The platform uses two API styles:

1. **PostgREST API (Supabase)** — for standard CRUD operations, accessed via the Supabase JS SDK. RLS enforces authorization. Used for ~80% of operations.
2. **Edge Functions** — for complex operations, webhook receivers, and operations requiring service-role privileges. Used for ~20% of operations.

There is no separate REST API layer; the Supabase PostgREST API is the primary data API. Edge Functions handle operations that cannot be expressed as direct table queries.

### 8.2 PostgREST Patterns

#### 8.2.1 Querying

```typescript
// List donors with filters and pagination
const { data, error } = await supabase
  .from('donors')
  .select(`
    id,
    donor_number,
    blood_type,
    user_profiles(full_name),
    donations(count)
  `)
  .eq('blood_type', 'O+')
  .is('deleted_at', null)
  .order('created_at', { ascending: false })
  .range(0, 19);  // first 20
```

#### 8.2.2 Inserting

```typescript
const { data, error } = await supabase
  .from('donors')
  .insert({
    user_id: userId,
    donor_number: await generateDonorNumber(),
    blood_type: input.bloodType,
    // ...
  })
  .select()
  .single();
```

#### 8.2.3 Updating

```typescript
const { data, error } = await supabase
  .from('donors')
  .update({
    emergency_availability_status: 'available',
    emergency_availability_until: input.until,
    updated_by: userId,
  })
  .eq('id', donorId)
  .select()
  .single();
```

#### 8.2.4 Soft Delete

```typescript
const { error } = await supabase
  .from('donors')
  .update({
    deleted_at: new Date().toISOString(),
    deleted_by: userId,
  })
  .eq('id', donorId);
```

### 8.3 Edge Function Endpoints

Edge Functions are named and documented. Each has a corresponding TypeScript client wrapper in the feature module.

| Function | Path | Method | Purpose | Auth |
|---|---|---|---|---|
| `webhook-receiver` | `/functions/v1/webhook-receiver` | POST | Receive webhooks from n8n | HMAC |
| `report-generator` | `/functions/v1/report-generator` | POST | Generate a report | JWT (ADMIN+) |
| `report-download` | `/functions/v1/report-download` | GET | Download report PDF | JWT (per report) |
| `virus-scan` | `/functions/v1/virus-scan` | POST | Trigger virus scan for uploaded file | Service role |
| `ai-content` | `/functions/v1/ai-content` | POST | Generate AI content (draft) | JWT (ADMIN+) |
| `bulk-export` | `/functions/v1/bulk-export` | POST | Export data (CSV/Excel) | JWT (ADMIN+) |
| `public-request-create` | `/functions/v1/public-request-create` | POST | Public blood request submission | None (rate-limited) |
| `donor-qr-verify` | `/functions/v1/donor-qr-verify` | GET | Verify a bag via QR code | None (rate-limited) |
| `inventory-aggregate` | `/functions/v1/inventory-aggregate` | GET | Public aggregate inventory view | None (cached) |
| `notification-send` | `/functions/v1/notification-send` | POST | Send a notification (called by n8n) | HMAC |

### 8.4 Edge Function Template

```typescript
// infrastructure/supabase/functions/_shared/template.ts
import { serve } from 'https://deno.land/std/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';
import { verifyJWT } from './auth.ts';
import { verifyHmac } from './hmac.ts';
import { corsHeaders, handleOptions } from './cors.ts';
import { errorResponse, successResponse } from './response.ts';

interface FunctionConfig {
  auth: 'jwt' | 'hmac' | 'none';
  requiredRole?: Role[];
  rateLimit?: { requests: number; window: string };
}

export function createHandler(
  handler: (req: Request, ctx: HandlerContext) => Promise<Response>,
  config: FunctionConfig
) {
  return serve(async (req: Request) => {
    if (req.method === 'OPTIONS') return handleOptions();
    
    try {
      // Authentication
      let userId: string | null = null;
      if (config.auth === 'jwt') {
        const jwt = req.headers.get('Authorization')?.replace('Bearer ', '');
        if (!jwt) return errorResponse(401, 'Missing authorization');
        const payload = await verifyJWT(jwt);
        if (!payload) return errorResponse(401, 'Invalid token');
        userId = payload.sub;
        
        if (config.requiredRole) {
          const supabase = createSupabaseClient(jwt);
          const hasRole = await checkRole(supabase, userId, config.requiredRole);
          if (!hasRole) return errorResponse(403, 'Insufficient permissions');
        }
      } else if (config.auth === 'hmac') {
        const signature = req.headers.get('X-Marwat-Signature');
        const body = await req.text();
        if (!verifyHmac(body, signature)) {
          return errorResponse(401, 'Invalid signature');
        }
      }
      
      // Rate limiting (post-v1; for v1, rely on Supabase built-in)
      
      // Call handler
      const ctx: HandlerContext = { userId, req };
      const response = await handler(req, ctx);
      response.headers.set('Access-Control-Allow-Origin', '*');
      response.headers.set('Access-Control-Allow-Headers', '*');
      return response;
    } catch (error) {
      console.error('Edge function error:', error);
      return errorResponse(500, 'Internal error', error.message);
    }
  });
}
```

### 8.5 Example Edge Function: `public-request-create`

```typescript
// infrastructure/supabase/functions/public-request-create/index.ts
import { createHandler } from '../_shared/template.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';
import { createBloodRequestSchema } from './schema.ts';  // Zod schema (Deno-compatible)

export const handler = createHandler(async (req, ctx) => {
  const body = await req.json();
  
  // Validate input
  const result = createBloodRequestSchema.safeParse(body);
  if (!result.success) {
    return errorResponse(400, 'Validation failed', result.error.flatten());
  }
  const input = result.data;
  
  // Use service role to create patient + request (no public auth)
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!,
    { auth: { persistSession: false } }
  );
  
  // Create patient
  const { data: patient, error: patientError } = await supabase
    .from('patients')
    .insert({
      patient_number: await generatePatientNumber(supabase),
      full_name: input.patient.fullName,
      blood_type: input.patient.bloodType,
      disease_condition: input.patient.diseaseCondition,
      guardian_name: input.patient.guardianName,
      guardian_contact: input.patient.guardianContact,
      consent_data_processing: input.consent.dataProcessing,
      consent_public_story: input.consent.publicStory,
      consent_at: new Date().toISOString(),
    })
    .select()
    .single();
  
  if (patientError) throw patientError;
  
  // Create request
  const slaDeadline = calculateSLADeadline(input.request.urgency);
  const { data: request, error: requestError } = await supabase
    .from('blood_requests')
    .insert({
      request_number: await generateRequestNumber(supabase),
      requester_type: 'guardian',
      patient_id: patient.id,
      blood_type_needed: input.request.bloodTypeNeeded,
      component_type_needed: input.request.componentTypeNeeded,
      quantity_units: input.request.quantityUnits,
      urgency: input.request.urgency,
      clinical_justification: input.request.clinicalJustification,
      required_by_date: input.request.requiredByDate,
      delivery_location: input.request.deliveryLocation,
      delivery_address: input.request.deliveryAddress,
      sla_deadline: slaDeadline,
    })
    .select()
    .single();
  
  if (requestError) throw requestError;
  
  // Emit domain event (triggers n8n workflow for notifications)
  await supabase.from('domain_events').insert({
    event_type: 'request.submitted',
    entity_type: 'blood_request',
    entity_id: request.id,
    payload: { requestId: request.id, urgency: request.urgency },
  });
  
  return successResponse({ request, patient });
}, { auth: 'none' });
```

### 8.6 API Error Response Standard

```typescript
// All error responses follow this format:
{
  "error": {
    "code": "VALIDATION_ERROR" | "UNAUTHORIZED" | "FORBIDDEN" | "NOT_FOUND" | "CONFLICT" | "RATE_LIMITED" | "INTERNAL_ERROR",
    "message": "Human-readable message",
    "details": { /* field-specific errors for validation */ },
    "correlationId": "uuid"
  }
}
```

HTTP status codes:
- 200 OK — successful read
- 201 Created — successful create
- 204 No Content — successful update/delete
- 400 Bad Request — validation error
- 401 Unauthorized — not authenticated
- 403 Forbidden — not authorized
- 404 Not Found — resource doesn't exist
- 409 Conflict — duplicate or state conflict
- 422 Unprocessable Entity — business rule violation
- 429 Too Many Requests — rate limited
- 500 Internal Server Error — server error

### 8.7 Service Interface Implementations (Phase 1 In-Process → Phase 3 n8n)

> **This section is the concrete realization of Architecture §10.1 Service Interface Pattern.** The founder codes against the interfaces; the implementations are swapped via environment variables.

#### 8.7.1 NotificationService

**Interface** (`packages/domain/src/services/notification-service.ts`):

```typescript
export interface NotificationService {
  send(notification: NotificationInput): Promise<NotificationResult>;
}

export interface NotificationInput {
  recipientUserId: string;
  templateCode: string;          // e.g., 'donor_thank_you'
  variables: Record<string, string | number>;
  channels?: NotificationChannel[]; // override user preference; default = user preference
  priority?: 'LOW' | 'NORMAL' | 'HIGH' | 'EMERGENCY';
  scheduledFor?: string;          // ISO datetime; if set, queue for later
  relatedEntityType?: string;
  relatedEntityId?: string;
  correlationId?: string;
}

export interface NotificationResult {
  notificationId: string;
  status: 'SENT' | 'QUEUED' | 'FAILED' | 'SUPPRESSED';
  channelsAttempted: NotificationChannel[];
  externalIds?: Record<NotificationChannel, string>;
}
```

**Phase 1 implementation** (`infrastructure/services/in-process/in-process-notification-service.ts`):

```typescript
export class InProcessNotificationService implements NotificationService {
  constructor(
    private readonly emailService: EmailService,   // Resend adapter; no-op if not configured
    private readonly supabase: SupabaseClient,
    private readonly logger: Logger,
  ) {}

  async send(input: NotificationInput): Promise<NotificationResult> {
    // 1. Insert into notifications table (always, for audit)
    const { data, error } = await this.supabase
      .from('notifications')
      .insert({ /* ... */ })
      .select()
      .single();
    if (error) throw error;

    // 2. Resolve channels (input override > user preference > template default)
    const channels = input.channels ?? await this.resolveUserChannels(input.recipientUserId);

    // 3. For each channel, dispatch via the corresponding sub-service
    for (const channel of channels) {
      switch (channel) {
        case 'email':
          await this.emailService.send({ /* ... */ });
          break;
        case 'in_app':
          // Already inserted into notifications table; nothing more to do
          break;
        case 'sms':
          // Phase 1: not implemented; log + skip
          this.logger.warn('SMS not implemented in Phase 1', { notificationId: data.id });
          break;
        case 'whatsapp':
          // Phase 3: not implemented in Phase 1; log + skip
          this.logger.warn('WhatsApp not implemented in Phase 1', { notificationId: data.id });
          break;
        case 'push':
          // Phase 4: not implemented in Phase 1; log + skip
          this.logger.warn('Push not implemented in Phase 1', { notificationId: data.id });
          break;
      }
    }

    return { notificationId: data.id, status: 'SENT', channelsAttempted: channels };
  }
}
```

**Phase 3 implementation** (`infrastructure/services/n8n/n8n-notification-service.ts`):

```typescript
export class N8nNotificationService implements NotificationService {
  constructor(
    private readonly n8nWebhookUrl: string,
    private readonly hmacSecret: string,
    private readonly fallback: NotificationService,  // InProcessNotificationService
    private readonly logger: Logger,
  ) {}

  async send(input: NotificationInput): Promise<NotificationResult> {
    try {
      const payload = JSON.stringify(input);
      const signature = computeHmac(payload, this.hmacSecret);

      const response = await fetch(this.n8nWebhookUrl, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'X-Marwat-Signature': signature,
        },
        body: payload,
        signal: AbortSignal.timeout(10_000),  // 10s timeout
      });

      if (!response.ok) throw new Error(`n8n returned ${response.status}`);
      return await response.json();
    } catch (error) {
      this.logger.error('n8n notification failed, falling back to in-process', {
        error: String(error),
        input,
      });
      return this.fallback.send(input);  // graceful degradation
    }
  }
}
```

**Selection** (`infrastructure/services/index.ts`):

```typescript
export function createNotificationService(deps: ServiceDeps): NotificationService {
  const inProcess = new InProcessNotificationService(deps.email, deps.supabase, deps.logger);

  const provider = process.env.NOTIFICATION_PROVIDER ?? 'in-process';
  if (provider === 'in-process') return inProcess;
  if (provider === 'n8n') {
    return new N8nNotificationService(
      process.env.N8N_NOTIFICATION_WEBHOOK_URL!,
      process.env.N8N_WEBHOOK_SECRET!,
      inProcess,  // fallback
      deps.logger,
    );
  }
  throw new Error(`Unknown NOTIFICATION_PROVIDER: ${provider}`);
}
```

#### 8.7.2 BackupService (Phase 1 In-Process → Phase 3 n8n)

Same pattern. Phase 1 runs `pg_dump` via `child_process.exec` and stores locally. Phase 3 delegates to an n8n workflow that does pg_dump + off-site copy to Cloudflare R2, with fallback to the in-process version on n8n failure.

#### 8.7.3 ReportService (Phase 1 In-Process → Phase 3 n8n + AI)

Same pattern. Phase 1 generates reports synchronously in-process (database queries + a Markdown templating engine). Phase 3 delegates to an n8n workflow that adds AI-assisted narrative (subject to admin review).

#### 8.7.4 AIContentService (Phase 3+ Only)

```typescript
export interface AIContentService {
  draftContent(spec: ContentSpec): Promise<ContentDraft>;
}

export class UnavailableAIContentService implements AIContentService {
  async draftContent(_spec: ContentSpec): Promise<ContentDraft> {
    throw new ServiceUnavailableError('AI content generation is not available in this phase');
  }
}

// Phase 3 implementation
export class GroqAIContentService implements AIContentService { /* ... */ }
export class LocalOllamaAIContentService implements AIContentService { /* ... */ }
```

In Phase 1–2, `createAIContentService()` returns `UnavailableAIContentService`. Feature code that calls it MUST catch `ServiceUnavailableError` and fall back to template-based content or skip the AI feature.

---

## 9. Edge Function Specifications

> **PHASE NOTE.** Edge Functions are **Phase 2+**. Phase 1 has **zero Edge Functions** — all operations are done client-side via the Supabase JS SDK with RLS enforcement, or via Next.js API routes (which run in the same process as the web app and use the service-role key for sensitive operations). Edge Functions are introduced in Phase 2 when capabilities that cannot be done client-side (virus scanning, public request submission without auth, complex reports) are needed.

(Detailed in `infrastructure/supabase/functions/<name>/README.md` for each. Summary table in §8.3.)

### 9.1 `report-generator`

**Trigger:** POST request from admin dashboard or n8n workflow.

**Input:**
```typescript
{
  reportType: 'daily_digest' | 'monthly_impact' | 'annual' | 'camp_summary' | ...;
  periodStart: string;       // ISO datetime
  periodEnd: string;
  parameters?: Record<string, unknown>;
  generateNarrative?: boolean;  // use AI
}
```

**Output:**
```typescript
{
  reportId: string;
  status: 'DRAFT' | 'COMPLETED';
  generatedContent: Record<string, unknown>;
  narrativeContent: string | null;
  filePath: string | null;   // PDF path in storage
}
```

**Implementation:**
1. Validate input (Zod).
2. Query data for the report type and period.
3. Compile structured content.
4. If `generateNarrative`, call AI service with anonymized data.
5. Generate PDF (using `pdf-lib` or Puppeteer).
6. Save PDF to storage.
7. Insert report record.
8. Return report ID and content.

### 9.2 `virus-scan`

**Trigger:** POST from Supabase Storage webhook (file uploaded) or n8n workflow.

**Input:** `{ filePath: string; entityType: 'patient_document' | 'donor_medical' | ... }`

**Output:** `{ status: 'CLEAN' | 'INFECTED' | 'ERROR'; details?: string }`

**Implementation:**
1. Download file from storage to temp.
2. Run ClamAV scan (ClamAV daemon running in Docker alongside n8n, or external service).
3. Update `virus_scan_status` on the relevant table.
4. If infected: quarantine file (move to quarantine bucket), alert admin via notification.
5. Return result.

---

## 10. n8n Workflow Specifications

> **PHASE NOTE.** n8n workflows are **Phase 3 only**. Phase 1 and Phase 2 have **no n8n** — the equivalent behavior is implemented as in-process TypeScript services (see §8.7). This entire section is reference material for Phase 3 implementation; the founder can skip it during Phase 1–2.

### 10.1 Standard Workflow JSON Structure

Every n8n workflow JSON follows this structure (simplified; full files in `automations/n8n/workflows/`):

```json
{
  "name": "AUT-003: Post-Donation Thank You",
  "nodes": [
    {
      "parameters": { "path": "donation-recorded", "httpMethod": "POST" },
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [0, 0]
    },
    {
      "parameters": { "functionCode": "/* HMAC verification */" },
      "name": "Verify HMAC",
      "type": "n8n-nodes-base.function",
      "typeVersion": 1,
      "position": [200, 0]
    },
    {
      "parameters": { /* idempotency check */ },
      "name": "Idempotency Check",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 1,
      "position": [400, 0]
    },
    {
      "parameters": { /* fetch donor + donation */ },
      "name": "Fetch Context",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 1,
      "position": [600, 0]
    },
    {
      "parameters": { /* determine channel from preferences */ },
      "name": "Select Channel",
      "type": "n8n-nodes-base.switch",
      "typeVersion": 1,
      "position": [800, 0]
    },
    {
      "parameters": { /* send via Resend/WhatsApp/SMS */ },
      "name": "Send Notification",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 1,
      "position": [1000, 0]
    },
    {
      "parameters": { /* insert into notifications table */ },
      "name": "Log",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 1,
      "position": [1200, 0]
    }
  ],
  "connections": { /* node connections */ },
  "settings": {
    "executionOrder": "v1"
  }
}
```

### 10.2 Workflow Specification Format

Each workflow has a companion spec file `automations/n8n/specs/AUT-NNN.md`:

```markdown
# AUT-003: Post-Donation Thank You

## Purpose
Send a thank-you message to a donor after a successful donation, via their preferred channel.

## Trigger
- **Type:** Database webhook
- **Table:** `donations`
- **Event:** INSERT
- **Condition:** `outcome = 'successful'`

## Input Schema
```typescript
{
  event: 'INSERT';
  table: 'donations';
  record: { id: string; donor_id: string; donation_date: string; ... }
}
```

## Output
- One notification record inserted into `notifications` table.
- One notification sent via donor's preferred channel.

## Failure Handling
- **HMAC verification failure:** Return 401, do not process.
- **Donor not found:** Log error, alert admin, do not retry (data integrity issue).
- **Notification send failure:** Retry 3× with exponential backoff; on persistent failure, log + alert admin.
- **Idempotency:** Skip if `event_id` already in `workflow_executions` table.

## Related
- PRD: FR-DON-034
- Architecture: §10.2, AUT-003
- n8n workflow: `workflows/AUT-003-post-donation-thank-you.json`
```

---

## 11. Authentication & Session Management

### 11.1 Supabase Client Setup

```typescript
// apps/web/src/lib/supabase/server.ts
import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';
import { Database } from '@marwat/database/types';

export async function createSupabaseServerClient() {
  const cookieStore = await cookies();
  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll: () => cookieStore.getAll(),
        setAll: (cookies) => cookies.forEach(({ name, value, options }) => 
          cookieStore.set(name, value, options)
        ),
      },
    }
  );
}

// apps/web/src/lib/supabase/client.ts
'use client';
import { createBrowserClient } from '@supabase/ssr';
import { Database } from '@marwat/database/types';

export function createSupabaseBrowserClient() {
  return createBrowserClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  );
}
```

### 11.2 Cookie Configuration

```typescript
// Cookie settings (httpOnly, secure, sameSite=lax)
{
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'lax' as const,
  path: '/',
  maxAge: 60 * 60 * 24 * 30,  // 30 days for refresh token
}
```

### 11.3 Session Refresh Middleware

```typescript
// apps/web/src/middleware.ts
import { createServerClient } from '@supabase/ssr';
import { NextResponse, type NextRequest } from 'next/server';
import { Database } from '@marwat/database/types';

const PUBLIC_ROUTES = ['/', '/about', '/request-blood', '/donate-blood', '/volunteer', '/knowledge-base', '/impact', '/contact', '/privacy', '/terms', '/login', '/register', '/forgot-password'];
const AUTH_ROUTES = ['/login', '/register', '/forgot-password'];

export async function middleware(request: NextRequest) {
  const response = NextResponse.next({ request });
  
  const supabase = createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll: () => request.cookies.getAll(),
        setAll: (cookies) => cookies.forEach(({ name, value, options }) => {
          request.cookies.set(name, value);
          response.cookies.set(name, value, options);
        }),
      },
    }
  );
  
  // Refresh session (rotates tokens)
  const { data: { session } } = await supabase.auth.getSession();
  
  const path = request.nextUrl.pathname;
  const isPublic = PUBLIC_ROUTES.some(r => path === r || path.startsWith(r + '/'));
  const isAuthRoute = AUTH_ROUTES.some(r => path === r || path.startsWith(r + '/'));
  
  if (!session && !isPublic && !isAuthRoute) {
    const redirectUrl = request.nextUrl.clone();
    redirectUrl.pathname = '/login';
    redirectUrl.searchParams.set('redirect', path);
    return NextResponse.redirect(redirectUrl);
  }
  
  if (session && isAuthRoute) {
    const redirectUrl = request.nextUrl.clone();
    redirectUrl.pathname = '/dashboard';
    return NextResponse.redirect(redirectUrl);
  }
  
  // Role-based route protection (deeper checks via RLS)
  if (session && path.startsWith('/admin')) {
    const { data: roles } = await supabase
      .from('user_roles')
      .select('role')
      .eq('user_id', session.user.id)
      .is('revoked_at', null);
    
    const hasAdminRole = roles?.some(r => r.role === 'SUPER_ADMIN' || r.role === 'ADMIN');
    if (!hasAdminRole) {
      const redirectUrl = request.nextUrl.clone();
      redirectUrl.pathname = '/403';
      return NextResponse.redirect(redirectUrl);
    }
  }
  
  // Apply security headers
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  response.headers.set('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');
  // CSP applied via next.config.mjs
  
  return response;
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|api/health).*)'],
};
```

---

## 12. Authorization — Permission Matrix

### 12.1 Permission Helper

```typescript
// apps/web/src/lib/auth/permissions.ts
import type { Role } from '@marwat/shared/types';
import type { AuthenticatedUser } from '@marwat/shared/types';

export type Action = 
  | 'create' | 'read' | 'update' | 'delete' | 'list'
  | 'verify' | 'allocate' | 'issue' | 'dispatch' | 'deliver' | 'cancel'
  | 'publish' | 'approve' | 'export' | 'configure';

export type Resource =
  | 'donor' | 'donation' | 'blood_bag' | 'blood_request'
  | 'allocation' | 'delivery' | 'patient' | 'patient_document'
  | 'hospital' | 'hospital_staff' | 'volunteer' | 'task' | 'camp'
  | 'notification' | 'notification_template' | 'report' | 'kb_article'
  | 'setting' | 'audit_log' | 'user' | 'user_role';

const PERMISSIONS: Record<Role, Partial<Record<Action, Resource[]>>> = {
  SUPER_ADMIN: {
    // All resources, all actions — handled separately
  },
  ADMIN: {
    create: ['donor', 'donation', 'blood_bag', 'blood_request', 'allocation', 'delivery', 'patient', 'hospital', 'volunteer', 'task', 'camp', 'notification', 'report', 'kb_article'],
    read: ['donor', 'donation', 'blood_bag', 'blood_request', 'allocation', 'delivery', 'patient', 'patient_document', 'hospital', 'hospital_staff', 'volunteer', 'task', 'camp', 'notification', 'report', 'kb_article', 'setting'],
    update: ['donor', 'donation', 'blood_bag', 'blood_request', 'allocation', 'delivery', 'patient', 'hospital', 'volunteer', 'task', 'camp', 'notification', 'report', 'kb_article', 'setting'],
    delete: ['donor', 'donation', 'blood_bag', 'blood_request', 'volunteer', 'task', 'camp', 'notification', 'report', 'kb_article'],
    verify: ['blood_request', 'patient_document'],
    allocate: ['blood_bag', 'blood_request'],
    issue: ['blood_bag', 'allocation'],
    dispatch: ['delivery'],
    cancel: ['blood_request', 'task', 'delivery'],
    publish: ['report', 'kb_article'],
    export: ['donor', 'donation', 'blood_bag', 'blood_request', 'volunteer', 'task', 'camp'],
  },
  VOLUNTEER: {
    create: ['donor', 'donation', 'blood_bag', 'task'],
    read: ['task', 'camp'],  // own tasks only (RLS)
    update: ['task'],        // own tasks (RLS)
  },
  HOSPITAL: {
    create: ['blood_request', 'patient'],
    read: ['blood_request', 'delivery'],  // own hospital only (RLS)
    update: ['blood_request'],            // own only (RLS)
    cancel: ['blood_request'],            // own only (RLS)
  },
  DONOR: {
    read: ['donor', 'donation', 'blood_bag'],  // own only (RLS)
    update: ['donor'],                          // own only (RLS)
  },
  PATIENT: {
    create: ['blood_request'],                  // for own patients (RLS)
    read: ['blood_request', 'patient', 'patient_document'],  // own only (RLS)
    cancel: ['blood_request'],                  // own only (RLS)
  },
};

export function can(user: AuthenticatedUser, action: Action, resource: Resource): boolean {
  if (user.roles.includes('SUPER_ADMIN')) return true;
  
  for (const role of user.roles) {
    const allowedResources = PERMISSIONS[role]?.[action] ?? [];
    if (allowedResources.includes(resource)) return true;
  }
  return false;
}
```

### 12.2 Authorization Wrapper Hook

```typescript
// apps/web/src/hooks/use-auth.ts
'use client';
import { useSupabase } from './use-supabase';
import { useEffect, useState } from 'react';
import { useRouter } from 'next/navigation';
import type { AuthenticatedUser, Action, Resource } from '@marwat/shared/types';
import { can } from '@/lib/auth/permissions';

export function useAuth() {
  const supabase = useSupabase();
  const [user, setUser] = useState<AuthenticatedUser | null>(null);
  const [loading, setLoading] = useState(true);
  const router = useRouter();
  
  useEffect(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
      if (session) {
        loadUser(session.user.id);
      } else {
        setLoading(false);
      }
    });
    
    const { data: { subscription } } = supabase.auth.onAuthStateChange((_event, session) => {
      if (session) {
        loadUser(session.user.id);
      } else {
        setUser(null);
        setLoading(false);
      }
    });
    
    return () => subscription.unsubscribe();
    
    async function loadUser(userId: string) {
      const { data: profile } = await supabase
        .from('user_profiles')
        .select('id, email, full_name, preferred_language')
        .eq('id', userId)
        .single();
      
      const { data: roles } = await supabase
        .from('user_roles')
        .select('role')
        .eq('user_id', userId)
        .is('revoked_at', null);
      
      if (profile && roles) {
        setUser({
          ...profile,
          roles: roles.map(r => r.role),
          activeRole: roles[0]?.role ?? 'PUBLIC',
        });
      }
      setLoading(false);
    }
  }, [supabase, router]);
  
  return {
    user,
    loading,
    can: (action: Action, resource: Resource) => 
      user ? can(user, action, resource) : false,
    signOut: () => supabase.auth.signOut(),
  };
}
```

---

## 13. State Machines

### 13.1 Blood Bag State Machine

```typescript
// packages/domain/src/inventory/state-machine.ts
import { BloodBagStatus } from '@marwat/shared/types';

export const BLOOD_BAG_TRANSITIONS: Record<BloodBagStatus, BloodBagStatus[]> = {
  COLLECTED: ['SCREENED', 'DISCARDED'],
  SCREENED: ['STORED', 'DISCARDED'],           // REJECTED screening → DISCARDED
  STORED: ['RESERVED', 'EXPIRED', 'DISCARDED'],
  RESERVED: ['ISSUED', 'STORED', 'EXPIRED', 'DISCARDED'],  // reservation can be cancelled
  ISSUED: ['DELIVERED', 'RETURNED', 'DISCARDED'],
  DELIVERED: ['RETURNED'],                     // unused unit returned
  RETURNED: ['STORED', 'DISCARDED'],
  EXPIRED: ['DISCARDED'],
  DISCARDED: [],                               // terminal
};

export function canTransition(from: BloodBagStatus, to: BloodBagStatus): boolean {
  return BLOOD_BAG_TRANSITIONS[from]?.includes(to) ?? false;
}

export function transition(
  current: BloodBagStatus,
  target: BloodBagStatus,
  actor: { id: string; role: string }
): { success: true; newStatus: BloodBagStatus } | { success: false; error: string } {
  if (!canTransition(current, target)) {
    return { 
      success: false, 
      error: `Invalid transition: ${current} → ${target}` 
    };
  }
  // Authorization check
  if (target === 'DISCARDED' && actor.role !== 'ADMIN' && actor.role !== 'SUPER_ADMIN') {
    return { success: false, error: 'Only admins can discard blood bags' };
  }
  return { success: true, newStatus: target };
}
```

### 13.2 Blood Request State Machine

```typescript
// packages/domain/src/requests/state-machine.ts
import { RequestStatus } from '@marwat/shared/types';

export const REQUEST_TRANSITIONS: Record<RequestStatus, RequestStatus[]> = {
  SUBMITTED: ['VERIFIED', 'REJECTED', 'CANCELLED'],
  VERIFIED: ['MATCHING', 'CANCELLED'],
  MATCHING: ['ALLOCATED', 'CANCELLED'],        // matching can fail and re-match
  ALLOCATED: ['DISPATCHED', 'CANCELLED'],
  DISPATCHED: ['DELIVERED', 'CANCELLED'],
  DELIVERED: ['RECEIVED', 'CLOSED'],           // received = confirmed by recipient
  RECEIVED: ['CLOSED'],
  CLOSED: [],
  REJECTED: [],
  CANCELLED: [],
};

export function canTransitionRequest(from: RequestStatus, to: RequestStatus): boolean {
  return REQUEST_TRANSITIONS[from]?.includes(to) ?? false;
}
```

### 13.3 SLA Calculation

```typescript
// packages/domain/src/requests/sla.ts
import { Urgency } from '@marwat/shared/types';

const SLA_HOURS: Record<Urgency, number> = {
  routine: 24,
  urgent: 4,
  emergency: 0.5,
};

export function calculateSLADeadline(
  submittedAt: Date,
  urgency: Urgency
): Date {
  const hours = SLA_HOURS[urgency];
  return new Date(submittedAt.getTime() + hours * 60 * 60 * 1000);
}

export function isSLABreached(
  slaDeadline: Date,
  currentStatus: string
): boolean {
  if (['CLOSED', 'REJECTED', 'CANCELLED'].includes(currentStatus)) return false;
  return new Date() > slaDeadline;
}
```

### 13.4 Donor Eligibility

```typescript
// packages/domain/src/donors/eligibility.ts
import { ComponentType } from '@marwat/shared/types';

const DEFERRAL_WINDOWS_DAYS: Record<ComponentType, number> = {
  whole_blood: 56,
  plasma: 14,
  platelets: 7,
  cryoprecipitate: 14,
  double_red_cells: 112,
};

export interface EligibilityInput {
  lastDonationDate: Date | null;
  lastDonationComponent: ComponentType | null;
  deferralUntil: Date | null;
  permanentDeferral: boolean;
  medicalScreening: {
    hivPositive: boolean;
    hepatitisB: boolean;
    hepatitisC: boolean;
    syphilis: boolean;
    malaria: boolean;
    recentTravelMalariaRegion: boolean;
    recentTattooOrPiercing: boolean;
    recentSurgery: boolean;
    pregnantOrBreastfeeding: boolean;
  };
  gender: 'male' | 'female' | 'other';
}

export interface EligibilityResult {
  eligible: boolean;
  reason: string | null;
  nextEligibleDate: Date | null;
}

export function calculateEligibility(input: EligibilityInput, now: Date = new Date()): EligibilityResult {
  if (input.permanentDeferral) {
    return { eligible: false, reason: 'Permanently deferred', nextEligibleDate: null };
  }
  
  if (input.deferralUntil && input.deferralUntil > now) {
    return { 
      eligible: false, 
      reason: 'Temporarily deferred', 
      nextEligibleDate: input.deferralUntil 
    };
  }
  
  // Medical deferrals
  const m = input.medicalScreening;
  if (m.hivPositive || m.hepatitisB || m.hepatitisC || m.syphilis) {
    return { eligible: false, reason: 'Permanent deferral: medical condition', nextEligibleDate: null };
  }
  if (m.malaria) {
    return { eligible: false, reason: 'Deferred: malaria', nextEligibleDate: null };
  }
  if (m.pregnantOrBreastfeeding) {
    return { eligible: false, reason: 'Deferred: pregnancy/breastfeeding', nextEligibleDate: null };
  }
  
  // Donation deferral window
  if (input.lastDonationDate && input.lastDonationComponent) {
    const deferralDays = DEFERRAL_WINDOWS_DAYS[input.lastDonationComponent];
    const nextEligible = new Date(input.lastDonationDate);
    nextEligible.setDate(nextEligible.getDate() + deferralDays);
    if (nextEligible > now) {
      return { 
        eligible: false, 
        reason: `Deferral window: ${deferralDays} days since last ${input.lastDonationComponent} donation`, 
        nextEligibleDate: nextEligible 
      };
    }
  }
  
  // Recent procedures (12-month deferral)
  if (m.recentTattooOrPiercing) {
    return { 
      eligible: false, 
      reason: 'Deferred: recent tattoo/piercing (12-month deferral)', 
      nextEligibleDate: new Date(now.getFullYear() + 1, now.getMonth(), now.getDate()) 
    };
  }
  
  return { eligible: true, reason: null, nextEligibleDate: null };
}
```

---

## 14. Frontend Routing & Component Architecture

### 14.1 Route Guard Component

```tsx
// apps/web/src/components/auth/route-guard.tsx
'use client';
import { useAuth } from '@/hooks/use-auth';
import { Role } from '@marwat/shared/types';
import { LoadingScreen } from '@/components/feedback/loading-screen';
import { UnauthorizedPage } from '@/components/feedback/unauthorized';

interface RouteGuardProps {
  children: React.ReactNode;
  allowedRoles: Role[];
}

export function RouteGuard({ children, allowedRoles }: RouteGuardProps) {
  const { user, loading } = useAuth();
  
  if (loading) return <LoadingScreen />;
  if (!user) return <UnauthorizedPage reason="not_authenticated" />;
  
  const hasAllowedRole = user.roles.some(r => allowedRoles.includes(r));
  if (!hasAllowedRole) return <UnauthorizedPage reason="insufficient_role" />;
  
  return <>{children}</>;
}
```

### 14.2 Dashboard Layout

```tsx
// apps/web/src/app/(dashboard)/layout.tsx
import { DashboardSidebar } from '@/components/layout/dashboard-sidebar';
import { DashboardHeader } from '@/components/layout/dashboard-header';
import { NotificationProvider } from '@/components/notifications/notification-provider';

export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-screen overflow-hidden">
      <DashboardSidebar />
      <div className="flex flex-1 flex-col overflow-hidden">
        <DashboardHeader />
        <main className="flex-1 overflow-y-auto bg-muted/10 p-6">
          <NotificationProvider>
            {children}
          </NotificationProvider>
        </main>
      </div>
    </div>
  );
}
```

### 14.3 Data Table Pattern

```tsx
// apps/web/src/components/data/data-table.tsx (simplified)
'use client';
import { ColumnDef, flexRender, getCoreRowModel, useReactTable } from '@tanstack/react-table';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';

interface DataTableProps<T> {
  data: T[];
  columns: ColumnDef<T, any>[];
  loading?: boolean;
  emptyMessage?: string;
}

export function DataTable<T>({ data, columns, loading, emptyMessage }: DataTableProps<T>) {
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
  });
  
  if (loading) return <DataTableSkeleton />;
  
  return (
    <Table>
      <TableHeader>
        {table.getHeaderGroups().map(group => (
          <TableRow key={group.id}>
            {group.headers.map(header => (
              <TableHead key={header.id}>
                {flexRender(header.column.columnDef.header, header.getContext())}
              </TableHead>
            ))}
          </TableRow>
        ))}
      </TableHeader>
      <TableBody>
        {table.getRowModel().rows.length === 0 ? (
          <TableRow>
            <TableCell colSpan={columns.length} className="text-center text-muted-foreground py-8">
              {emptyMessage ?? 'No records found'}
            </TableCell>
          </TableRow>
        ) : (
          table.getRowModel().rows.map(row => (
            <TableRow key={row.id}>
              {row.getVisibleCells().map(cell => (
                <TableCell key={cell.id}>
                  {flexRender(cell.column.columnDef.cell, cell.getContext())}
                </TableCell>
              ))}
            </TableRow>
          ))
        )}
      </TableBody>
    </Table>
  );
}
```

### 14.4 Realtime Hook

```typescript
// apps/web/src/hooks/use-realtime.ts
'use client';
import { useEffect } from 'react';
import { useSupabase } from './use-supabase';
import type { Database } from '@marwat/database/types';

interface UseRealtimeOptions {
  table: keyof Database['public']['Tables'];
  filter?: string;
  onInsert?: (row: any) => void;
  onUpdate?: (row: any) => void;
  onDelete?: (row: any) => void;
}

export function useRealtime({ table, filter, onInsert, onUpdate, onDelete }: UseRealtimeOptions) {
  const supabase = useSupabase();
  
  useEffect(() => {
    const channel = supabase
      .channel(`${table}-changes`)
      .on('postgres_changes', 
        { event: 'INSERT', schema: 'public', table, filter },
        (payload) => onInsert?.(payload.new)
      )
      .on('postgres_changes',
        { event: 'UPDATE', schema: 'public', table, filter },
        (payload) => onUpdate?.(payload.new)
      )
      .on('postgres_changes',
        { event: 'DELETE', schema: 'public', table, filter },
        (payload) => onDelete?.(payload.old)
      )
      .subscribe();
    
    return () => { supabase.removeChannel(channel); };
  }, [table, filter, onInsert, onUpdate, onDelete, supabase]);
}
```

---

## 15. Mobile App Architecture

### 15.1 Expo Router Structure

```
src/app/
├── _layout.tsx                      # Root: auth gate, theme provider
├── (auth)/
│   ├── login.tsx
│   └── register.tsx
├── (donor)/
│   ├── _layout.tsx                  # Tab navigator
│   ├── home.tsx                     # Dashboard
│   ├── donations.tsx                # History
│   ├── impact.tsx                   # Impact summary
│   └── profile.tsx
├── (volunteer)/
│   ├── _layout.tsx                  # Tab navigator
│   ├── tasks.tsx                    # Assigned tasks
│   ├── tasks/[id].tsx               # Task detail
│   ├── check-in.tsx                 # Check-in/out
│   ├── history.tsx
│   └── profile.tsx
└── scan.tsx                         # QR scanner for donor check-in at camp
```

### 15.2 Push Notifications

```typescript
// apps/mobile/src/lib/notifications.ts
import * as Notifications from 'expo-notifications';
import { Platform } from 'react-native';
import { supabase } from './supabase';

export async function registerForPushNotifications() {
  const { status } = await Notifications.requestPermissionsAsync();
  if (status !== 'granted') return null;
  
  const token = (await Notifications.getExpoPushTokenAsync()).data;
  
  // Save token to user profile for n8n to send to
  const { data: { user } } = await supabase.auth.getUser();
  if (user) {
    await supabase
      .from('user_profiles')
      .update({ push_token: token })
      .eq('id', user.id);
  }
  
  if (Platform.OS === 'android') {
    Notifications.setNotificationChannelAsync('default', {
      name: 'Marwat Welfare',
      importance: Notifications.AndroidImportance.HIGH,
    });
    Notifications.setNotificationChannelAsync('emergency', {
      name: 'Emergency Blood Requests',
      importance: Notifications.AndroidImportance.MAX,
    });
  }
  
  return token;
}
```

### 15.3 Offline Queue

```typescript
// apps/mobile/src/lib/offline-queue.ts
import AsyncStorage from '@react-native-async-storage/async-storage';

interface QueuedAction {
  id: string;
  endpoint: string;
  method: 'POST' | 'PATCH' | 'DELETE';
  body: any;
  queuedAt: string;
}

const QUEUE_KEY = '@marwat/offline-queue';

export async function queueAction(action: Omit<QueuedAction, 'id' | 'queuedAt'>) {
  const queue = await getQueue();
  queue.push({
    ...action,
    id: Math.random().toString(36).substr(2, 9),
    queuedAt: new Date().toISOString(),
  });
  await AsyncStorage.setItem(QUEUE_KEY, JSON.stringify(queue));
}

export async function processQueue() {
  const queue = await getQueue();
  const failed: QueuedAction[] = [];
  
  for (const action of queue) {
    try {
      await sendAction(action);
    } catch (e) {
      failed.push(action);
    }
  }
  
  await AsyncStorage.setItem(QUEUE_KEY, JSON.stringify(failed));
}

async function sendAction(action: QueuedAction) {
  // Use supabase client to perform the action
  // Throw on failure so it stays in queue
}
```

---

## 16. Testing Strategy

### 16.1 Test Pyramid

```
                    ┌─────────┐
                    │   E2E   │  10% — Playwright, full user journeys
                    └─────────┘
                ┌─────────────────┐
                │  Integration    │  20% — Supabase local + feature tests
                └─────────────────┘
            ┌──────────────────────────┐
            │       Unit Tests         │  70% — pure functions, components
            └──────────────────────────┘
```

### 16.2 Tools

| Test Type | Tool | Location |
|---|---|---|
| Unit (pure functions) | Vitest | `packages/*/tests/`, `apps/web/src/features/*/tests/` |
| Component | Vitest + Testing Library | `apps/web/src/components/tests/` |
| Integration | Vitest + Supabase local | `apps/web/tests/integration/` |
| RLS | Vitest + Supabase local | `apps/web/tests/rls/` |
| E2E | Playwright | `apps/web/tests/e2e/` |
| Mobile | Jest + React Native Testing Library | `apps/mobile/src/**/tests/` |

### 16.3 Coverage Requirements

- **Domain logic (`packages/domain/`):** 100% coverage (pure functions, easy to test).
- **Feature services (`apps/web/src/features/*/services/`):** ≥ 90% coverage.
- **Components:** ≥ 80% coverage (UI testing is harder, focus on critical paths).
- **RLS policies:** 100% coverage (every policy has positive and negative tests).
- **Edge Functions:** ≥ 80% coverage (integration tests).

Coverage enforced in CI via `vitest --coverage` with thresholds.

### 16.4 E2E Test Example

```typescript
// apps/web/tests/e2e/donor-registration.spec.ts
import { test, expect } from '@playwright/test';

test('donor can register via public website', async ({ page }) => {
  await page.goto('/donate-blood');
  
  await page.fill('[name="fullName"]', 'Test Donor');
  await page.fill('[name="email"]', `test-${Date.now()}@example.com`);
  await page.fill('[name="phone"]', '+923001234567');
  await page.fill('[name="password"]', 'SecurePass123!');
  await page.selectOption('[name="bloodType"]', 'O+');
  await page.fill('[name="dateOfBirth"]', '1995-01-15');
  await page.click('[data-testid="consent-checkbox"]');
  await page.click('button[type="submit"]');
  
  // Verify OTP screen
  await expect(page).toHaveURL(/\/verify-otp/);
  
  // Mock OTP verification (test environment)
  const otp = await getTestOTP(page);
  await page.fill('[name="otp"]', otp);
  await page.click('button[type="submit"]');
  
  // Verify dashboard
  await expect(page).toHaveURL(/\/donor/);
  await expect(page.locator('h1')).toContainText('Welcome, Test Donor');
});
```

### 16.5 RLS Test Example

```typescript
// apps/web/tests/rls/donors.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { createClient } from '@supabase/supabase-js';
import { createTestUser, cleanupTestUser, supabaseAdmin } from './helpers';

describe('donors RLS', () => {
  let donor1, donor2, admin;
  
  beforeAll(async () => {
    donor1 = await createTestUser(['DONOR']);
    donor2 = await createTestUser(['DONOR']);
    admin = await createTestUser(['ADMIN']);
  });
  
  afterAll(async () => {
    await cleanupTestUser(donor1.id);
    await cleanupTestUser(donor2.id);
    await cleanupTestUser(admin.id);
  });
  
  it('donor can read own record', async () => {
    const { data, error } = await donor1.client
      .from('donors')
      .select('*')
      .eq('user_id', donor1.id);
    expect(error).toBeNull();
    expect(data).toHaveLength(1);
  });
  
  it('donor cannot read other donor records', async () => {
    const { data, error } = await donor1.client
      .from('donors')
      .select('*')
      .eq('user_id', donor2.id);
    expect(error).toBeNull();
    expect(data).toHaveLength(0);
  });
  
  it('admin can read all donor records', async () => {
    const { data, error } = await admin.client
      .from('donors')
      .select('*');
    expect(error).toBeNull();
    expect(data.length).toBeGreaterThanOrEqual(2);
  });
  
  it('unauthenticated user cannot read donor records', async () => {
    const anonClient = createClient(process.env.SUPABASE_URL!, process.env.SUPABASE_ANON_KEY!);
    const { data, error } = await anonClient.from('donors').select('*');
    expect(data).toHaveLength(0);
  });
});
```

---

## 17. CI/CD Pipeline Specification

### 17.1 GitHub Actions Workflow (Phase 1 — Minimal)

Phase 1 ships with a **single, minimal CI workflow**. The goal is to catch obvious errors on every push without slowing the founder down. The workflow runs lint, typecheck, unit tests, and RLS tests. No build, no E2E, no security scan, no deployment — those are added in Phase 2+.

```yaml
# .github/workflows/ci.yml (Phase 1)
name: CI

on: [push, pull_request]

jobs:
  lint-typecheck-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env: { POSTGRES_PASSWORD: postgres }
        ports: ['5432:5432']
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm typecheck
      - run: pnpm format:check
      - run: pnpm db:migrate
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/postgres
      - run: pnpm test:unit
      - run: pnpm test:rls
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/postgres
```

**Phase 1 deployment** is manual: the founder runs `git pull && docker compose up -d && pnpm db:migrate` on the office PC. No Vercel preview, no auto-deploy. This is acceptable for a solo-founder internal-use Phase 1.

### 17.2 GitHub Actions Workflow (Phase 2+ — Expanded)

Phase 2 adds build, integration tests, E2E tests, security scan, and Vercel preview deployment. The full workflow below replaces the Phase 1 minimal workflow when Phase 2 begins.

```yaml
# .github/workflows/ci.yml (Phase 2+ — replaces Phase 1 minimal)
name: CI

on:
  push:
    branches: [main, staging]
  pull_request:
    branches: [main, staging]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint-typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm typecheck
      - run: pnpm format:check

  unit-tests:
    runs-on: ubuntu-latest
    needs: lint-typecheck
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm test:unit -- --coverage
      - uses: codecov/codecov-action@v3

  rls-tests:
    runs-on: ubuntu-latest
    needs: lint-typecheck
    services:
      supabase:
        image: supabase/postgres:15.1.0.117
        env:
          POSTGRES_PASSWORD: postgres
        ports: ['5432:5432']
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm db:migrate
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/postgres
      - run: pnpm test:rls
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/postgres

  integration-tests:
    runs-on: ubuntu-latest
    needs: rls-tests
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm test:integration

  security-scan:
    runs-on: ubuntu-latest
    needs: lint-typecheck
    steps:
      - uses: actions/checkout@v4
      - uses: snyk/actions/node@master
        with:
          command: test
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      - name: Run CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript-typescript
      - uses: github/codeql-action/analyze@v3
      - name: Secret scan
        uses: trufflesecurity/trufflehog@main

  e2e-tests:
    runs-on: ubuntu-latest
    needs: [unit-tests, rls-tests, integration-tests]
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm exec playwright install --with-deps
      - run: pnpm test:e2e

  build:
    runs-on: ubuntu-latest
    needs: [unit-tests, rls-tests, integration-tests, security-scan]
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm build
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: |
            apps/web/.next
            apps/mobile/dist

  deploy-preview:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-args: '--prod=false'
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}

  deploy-staging:
    if: github.ref == 'refs/heads/staging' && github.event_name == 'push'
    runs-on: ubuntu-latest
    needs: build
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-args: '--prod'
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-project-name: marwat-staging

  deploy-production:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: production
      url: https://marwatwelfare.org
    steps:
      - uses: actions/checkout@v4
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-args: '--prod'
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-project-name: marwat-production
```

### 17.2 Pre-commit Hooks

```yaml
# .husky/pre-commit
- pnpm lint:staged
- pnpm typecheck
- pnpm test:affected
```

---

## 18. Environment Variables Reference

> **PHASE NOTE.** Each variable below is tagged with the phase in which it is first required. Variables tagged `P1` MUST be set for Phase 1 to run. Variables tagged `P2+` are optional in Phase 1 (the platform runs with sensible defaults or skips the feature). The `.env.example` file in the repo includes these phase tags as comments so the founder knows exactly what to set when.

```bash
# .env.example
# Phase tags: [P1] required for Phase 1, [P2+] arrives in later phases.

# ===== App =====
NODE_ENV=development|production                                    # [P1]
NEXT_PUBLIC_APP_URL=http://localhost:3000                          # [P1] set to LAN IP for office deployment
NEXT_PUBLIC_APP_NAME="Marwat Welfare Society"                      # [P1]

# ===== Supabase =====
# [P1] For local/LAN deployment, these point at the local Docker Supabase stack.
# [P2+] For cloud deployment, these point at the Supabase cloud project.
NEXT_PUBLIC_SUPABASE_URL=http://localhost:8000                     # [P1]
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key                        # [P1]
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key                    # [P1] server only
SUPABASE_DB_URL=postgresql://postgres:postgres@localhost:5432/postgres  # [P1]

# ===== Service Providers (Phase 1 defaults to in-process) =====
NOTIFICATION_PROVIDER=in-process|n8n                              # [P1] default: in-process; [P3] set to n8n
REPORT_PROVIDER=in-process|n8n                                    # [P1] default: in-process; [P3] set to n8n
BACKUP_PROVIDER=in-process|n8n                                    # [P1] default: in-process; [P3] set to n8n
AI_CONTENT_PROVIDER=unavailable|groq|ollama|openai                # [P1] default: unavailable; [P3] set to a real provider

# ===== Auth =====
NEXT_PUBLIC_MFA_REQUIRED_ROLES=ADMIN,SUPER_ADMIN                  # [P1]
JWT_SIGNING_SECRET=                                                # [P1] Supabase managed, not set directly

# ===== n8n (Phase 3 only — leave unset in Phase 1-2) =====
N8N_WEBHOOK_URL=https://n8n.marwatwelfare.org/webhook             # [P3]
N8N_WEBHOOK_SECRET=                                                # [P3] HMAC secret, shared with n8n
N8N_API_KEY=                                                       # [P3] for deploying workflows
N8N_NOTIFICATION_WEBHOOK_URL=                                      # [P3] if NOTIFICATION_PROVIDER=n8n
N8N_REPORT_WEBHOOK_URL=                                            # [P3] if REPORT_PROVIDER=n8n
N8N_BACKUP_WEBHOOK_URL=                                            # [P3] if BACKUP_PROVIDER=n8n

# ===== Email (Resend — Phase 2; in Phase 1 email is no-op) =====
RESEND_API_KEY=                                                    # [P2]
RESEND_FROM_EMAIL=noreply@marwatwelfare.org                        # [P2]
RESEND_FROM_NAME="Marwat Welfare Society"                          # [P2]

# ===== WhatsApp (Phase 3 only) =====
WHATSAPP_API_TOKEN=                                                # [P3]
WHATSAPP_PHONE_NUMBER_ID=                                          # [P3]
WHATSAPP_BUSINESS_ACCOUNT_ID=                                      # [P3]

# ===== SMS (optional, paid — Phase 3+) =====
SMS_GATEWAY_URL=                                                   # [P3+]
SMS_GATEWAY_API_KEY=                                               # [P3+]

# ===== AI (Phase 3 only) =====
AI_PROVIDER=groq|openai|local                                      # [P3]
AI_API_KEY=                                                        # [P3]
AI_MODEL=llama-3.1-70b-versatile                                   # [P3]

# ===== Storage (Phase 1; local MinIO or Supabase Storage) =====
SUPABASE_STORAGE_BUCKET_PATIENT_DOCS=patient-documents             # [P1]
SUPABASE_STORAGE_BUCKET_DONOR_MEDICAL=donor-medical                # [P1]
SUPABASE_STORAGE_BUCKET_CAMP_PHOTOS=camp-photos                    # [P2]
SUPABASE_STORAGE_BUCKET_PUBLIC=public-assets                       # [P2]
SUPABASE_STORAGE_BUCKET_REPORTS=reports                            # [P1]
SUPABASE_STORAGE_BUCKET_AVATARS=avatars                            # [P1]

# ===== Cloudflare (off-site backups — Phase 3 only) =====
CLOUDFLARE_R2_ACCOUNT_ID=                                          # [P3]
CLOUDFLARE_R2_ACCESS_KEY_ID=                                       # [P3]
CLOUDFLARE_R2_SECRET_ACCESS_KEY=                                   # [P3]
CLOUDFLARE_R2_BUCKET=marwat-backups                                # [P3]

# ===== Monitoring (Phase 2+) =====
SENTRY_DSN=                                                        # [P2]
SENTRY_ORG=                                                        # [P2]
SENTRY_PROJECT=                                                    # [P2]
SENTRY_AUTH_TOKEN=                                                 # [P2]
UPTIMEROBOT_API_KEY=                                               # [P2]

# ===== Mobile (Expo — Phase 4 only) =====
EXPO_PROJECT_ID=                                                   # [P4]
EXPO_PUSH_NOTIFICATIONS_ENABLED=true                               # [P4]

# ===== Feature Flags (all default to off in Phase 1) =====
NEXT_PUBLIC_FEATURE_SMS=false                                      # [P3+]
NEXT_PUBLIC_FEATURE_WHATSAPP=false                                 # [P3]
NEXT_PUBLIC_FEATURE_AI_CONTENT=false                               # [P3]
NEXT_PUBLIC_FEATURE_OFFLINE_MOBILE=false                           # [P4]
NEXT_PUBLIC_FEATURE_PUBLIC_WEBSITE=false                           # [P2]
NEXT_PUBLIC_FEATURE_DONOR_PORTAL=false                             # [P2]
NEXT_PUBLIC_FEATURE_PATIENT_PORTAL=false                           # [P2]
NEXT_PUBLIC_FEATURE_HOSPITAL_PORTAL=false                          # [P2]
NEXT_PUBLIC_FEATURE_BLOOD_CAMPS=false                              # [P2]
NEXT_PUBLIC_FEATURE_N8N_AUTOMATION=false                           # [P3]

# ===== Security =====
CSP_REPORT_URI=                                                    # [P2] optional, for CSP violation reports
HSTS_PRELOAD=true                                                  # [P2+] (no HTTPS in Phase 1 LAN)
```

---

## 19. Migration & Seeding Strategy

### 19.1 Migration File Naming

```
packages/database/migrations/
├── 0001_create_extensions.sql
├── 0002_create_user_roles.sql
├── ...
├── 0023_add_patient_consent_v2.sql
└── README.md
```

Files are numbered sequentially with 4-digit prefixes. Each file is idempotent (safe to run multiple times — uses `IF NOT EXISTS` / `CREATE OR REPLACE`).

### 19.2 Migration Application

```bash
# Development
pnpm db:migrate                       # applies pending migrations to local Supabase

# Staging / Production
pnpm db:migrate:staging
pnpm db:migrate:production
```

Migration scripts use `supabase migration up` (Supabase CLI) with the appropriate environment's database URL.

### 19.3 Seed Data

```bash
# Development / Staging only (NEVER production)
pnpm db:seed
```

Seed files in `packages/database/seed/` create:

- 1 SUPER_ADMIN (test credentials documented in README, NOT used in production).
- 3 ADMIN users.
- 50 DONOR users with varied blood types, locations, donation histories.
- 10 VOLUNTEER users.
- 5 HOSPITAL users.
- 20 PATIENT records.
- 100 BLOOD_BAG records in various states.
- 30 BLOOD_REQUEST records in various states.
- 10 CAMPS (some completed, some scheduled).
- Sample NOTIFICATIONS, REPORTS, KB_ARTICLES.

All seed data uses synthetic names, phone numbers, and addresses. Real-looking but clearly fake (e.g., phone numbers all start with +9230000XXXXX).

### 19.4 Migration Safety Rules

1. **Never delete a migration file.** Once applied to production, it's immutable. Corrections are new migrations.
2. **Never modify a migration file after merging.** Same reason.
3. **Down migrations** are best-effort. Schema changes are reversible; data changes are not. Test down migrations on staging before relying on them.
4. **Long-running migrations** (e.g., backfilling a new column on a large table) must be done in three steps: (1) add column nullable, (2) backfill in batches, (3) add NOT NULL constraint. This avoids table locks.
5. **Migration review** in PR: include the SQL, the expected schema diff, and any data migration impact.

---

## 20. Glossary

| Term | Definition |
|---|---|
| **Aggregate** | A cluster of domain objects treated as a single unit for data changes. |
| **Barrel** | An `index.ts` file that re-exports the public API of a module. |
| **DTO** | Data Transfer Object — the wire format for API responses. |
| **Domain Event** | A significant event in the domain that other parts of the system react to. |
| **Edge Function** | A serverless function (Deno) hosted on Supabase. |
| **Feature Module** | A vertical slice of the application containing UI, logic, types, and validation for one feature. |
| **HMAC** | Hash-based Message Authentication Code — used for webhook signature verification. |
| **In-Process Service** | A service implementation that runs inside the Next.js process (Phase 1–2 default). |
| **Local-First** | Architecture principle: the platform runs on a single laptop, office LAN, or cloud without code change. |
| **Phase Tag (P1–P4)** | Implementation phase marker: P1 Core MVP, P2 Operational Improvements, P3 Automation, P4 Scale. |
| **PHI** | Protected Health Information. |
| **PostgREST** | The REST API layer Supabase exposes over PostgreSQL. |
| **RLS** | Row-Level Security. |
| **Service Interface Pattern** | Application calls interfaces; concrete implementations (in-process or n8n-backed) are selected via environment variables. |
| **SLA** | Service Level Agreement — committed response time. |
| **Soft Delete** | Marking a record as deleted without physically removing it. |
| **State Machine** | A model where an entity transitions between defined states via legal transitions. |

---

## 21. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0.0 | 2026-07-06 | Principal Architect | Initial baseline approved by Board. |
| 1.1.0 | 2026-07-06 | Founder / Principal Architect | **Startup-first refactor.** Added §1.2 Phase Tags convention (`P1`–`P4`) applied throughout the document. Added §8.7 Service Interface Implementations with full TypeScript code for `NotificationService` (in-process Phase 1 + n8n Phase 3 + graceful fallback), `BackupService`, `ReportService`, and `AIContentService` (`UnavailableAIContentService` for Phase 1–2). Added phase notes to §9 Edge Functions (Phase 2+) and §10 n8n Workflows (Phase 3 only). Rewrote §17 CI/CD into §17.1 Phase 1 Minimal (single-job lint+typecheck+test+RLS) and §17.2 Phase 2+ Expanded (full pipeline). Added phase tags `[P1]`/`[P2]`/`[P3]`/`[P4]` to every environment variable in §18. Added service provider env vars (`NOTIFICATION_PROVIDER`, `REPORT_PROVIDER`, `BACKUP_PROVIDER`, `AI_CONTENT_PROVIDER`). Added feature flags for portals, camps, n8n, AI, mobile. All schemas, types, RLS policies, validation schemas, and state machines from v1.0.0 are preserved unchanged — Phase 1 simply implements a subset of them. |

---

> **End of Technical Design Specification.** The next document is the **AI Development Rulebook** (`04-AI_DEVELOPMENT_RULEBOOK.md`), which defines the coding standards and conventions that every future AI-assisted coding session must follow.
