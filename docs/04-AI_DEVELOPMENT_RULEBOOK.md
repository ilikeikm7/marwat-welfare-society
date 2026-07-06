# Marwat Welfare Society — AI Development Rulebook

| Field | Value |
|---|---|
| **Document Title** | AI Development Rulebook |
| **Project Name** | Marwat Welfare Society — Blood Donation & Distribution Platform |
| **Document Version** | 1.1.0 |
| **Status** | Approved Baseline |
| **Classification** | Internal — Project Constitution |
| **Last Updated** | 2026-07-06 |
| **Document Owner** | Founder / Principal Architect |
| **Approval Authority** | Founder (solo-founder stage); Board Tech Committee (post-Phase 4) |
| **Depends On** | `01-PRD.md`, `02-SYSTEM_ARCHITECTURE.md`, `03-TECHNICAL_DESIGN_SPECIFICATION.md` (all v1.1.0+) |
| **Audience** | Every AI coding assistant (Claude, GPT, Cursor, Copilot, GLM, etc.), every human contributor, every code reviewer |

> **CONSTITUTION CLAUSE.** This document is the binding rulebook for every line of code written for the Marwat Welfare Society platform, whether by a human or an AI. Every AI coding session MUST begin by reading this document in full. Every pull request MUST conform to these rules. Every code review MUST enforce them. Violations are merge-blockers — except during the solo-founder stage (Phases 1–3), where the founder may self-merge with a documented justification in the commit message. Where a rule appears overly strict, propose an ADR (encouraged but optional during solo-founder stage) to amend this document rather than silently deviating.
>
> **v1.1.0 REFACTOR NOTE.** This version simplifies the **pre-session protocol** (§2) for solo-founder practicality — the confirmation block is shorter and self-merges are acknowledged. It reframes **Git & PR rules** (§16) for a solo founder: one approver is enough (including self-merge with justification). It marks **AI features as Phase 3+** throughout. It explicitly preserves every technical rule (TypeScript, React, database, security, validation, testing, performance, accessibility, i18n, documentation) unchanged — security and code quality are not reduced, only governance bureaucracy is simplified.

---

## Table of Contents

1. [Purpose & Authority](#1-purpose--authority)
2. [Mandatory Pre-Session Protocol for AI Assistants](#2-mandatory-pre-session-protocol-for-ai-assistants)
3. [Repository & Workspace Conventions](#3-repository--workspace-conventions)
4. [Naming Conventions](#4-naming-conventions)
5. [TypeScript Rules](#5-typescript-rules)
6. [React & Next.js Rules](#6-react--nextjs-rules)
7. [React Native & Expo Rules](#7-react-native--expo-rules)
8. [Database & SQL Rules](#8-database--sql-rules)
9. [Security Rules](#9-security-rules)
10. [Validation & Error Handling Rules](#10-validation--error-handling-rules)
11. [Testing Rules](#11-testing-rules)
12. [Performance Rules](#12-performance-rules)
13. [Accessibility Rules](#13-accessibility-rules)
14. [Internationalization Rules](#14-internationalization-rules)
15. [Documentation Rules](#15-documentation-rules)
16. [Git & Pull Request Rules](#16-git--pull-request-rules)
17. [Forbidden Patterns (Never Do These)](#17-forbidden-patterns-never-do-these)
18. [Required Patterns (Always Do These)](#18-required-patterns-always-do-these)
19. [AI-Specific Rules (Beyond Human Rules)](#19-ai-specific-rules-beyond-human-rules)
20. [Prompt Templates for Future AI Sessions](#20-prompt-templates-for-future-ai-sessions)
21. [Code Review Checklist](#21-code-review-checklist)
22. [Violation Reporting & Amendments](#22-violation-reporting--amendments)
23. [Glossary](#23-glossary)

---

## 1. Purpose & Authority

### 1.1 Why This Document Exists

The Marwat Welfare Society platform is a 10+ year project. Over that time, it will be touched by:

- A solo founder (Phases 1–3) using AI coding assistants as force multipliers.
- Multiple human developers (Phase 4+), each with different experience levels and preferences.
- Multiple AI coding assistants (Claude, GPT, Cursor, Copilot, Gemini, GLM, etc.), each with different default behaviors.
- Multiple code reviewers, each enforcing different standards.

Without a single, authoritative, enforceable rulebook, the codebase will drift toward inconsistency — different naming conventions in different files, duplicated logic, security holes from one-off "quick fixes", and architectural patterns that violate the System Architecture Document. This document prevents that drift by being the **single source of truth** for how code is written in this repository.

### 1.2 Authority & Enforcement

This document is approved by the founder and (post-Phase 4) the Board Tech Committee. It has the highest authority of any document in the repository for code-level decisions. Specifically:

- **Higher than personal preference.** If a developer or AI prefers a different pattern, the rulebook wins.
- **Higher than "it works".** Working code that violates the rulebook is not mergeable (without documented justification during the solo-founder stage).
- **Higher than AI defaults.** If an AI assistant's default behavior violates a rule, the AI MUST be instructed to follow the rule instead.
- **Equal to the PRD, Architecture, and TDS.** When this rulebook and another constitution document appear to conflict, the conflict is resolved by an ADR; until then, the more restrictive rule applies.

Enforcement is via:

1. **ESLint / Prettier / TypeScript strict mode** — automated, in CI, merge-blocking (or self-merge with justification during solo-founder stage).
2. **Pre-commit hooks** — automated, run on every commit.
3. **Code review** — every PR. During the solo-founder stage, this means the founder reviewing their own work with the AI's assistance; from Phase 4 onward, a second human reviewer is required.
4. **AI session discipline** — every AI session begins with this document (see §2).

### 1.3 Solo-Founder Stage (Phases 1–3)

During the solo-founder stage, the founder is the sole human contributor. The following adjustments to enforcement apply:

- **Self-merge is permitted** with a brief justification in the commit message (e.g., `refactor(donors): extract eligibility calc — self-merged, tested locally`).
- **ADRs are encouraged but not required** for most decisions (see §22).
- **PRs are still required** for every change to `main` — even solo founders benefit from the diff review and the audit trail.
- **CI must pass** before merge. If CI is broken, fix it before merging; do not override.
- **The rulebook still applies in full.** Security, naming, architecture, testing — none of these are relaxed. Only the *governance process* (reviewers, approvals, ADRs) is simplified.

From Phase 4 onward, the organization is expected to have multiple contributors, and the full review process applies.

### 1.4 Scope

This rulebook governs:

- All code in `apps/`, `packages/`, `infrastructure/`, `automations/`.
- All database migrations, RLS policies, and Edge Functions.
- All n8n workflow JSON files and supporting scripts.
- All test code.
- All documentation that includes code examples.

This rulebook does **not** govern:

- Project management processes (separate document).
- Operational runbooks (separate document).
- Content of the PRD, Architecture, or TDS (those documents govern themselves).

### 1.5 How to Use This Document

**If you are a human developer:**
- Read this document in full before your first contribution.
- Re-read it whenever you switch to a different part of the codebase.
- Refer to it during code review.
- Propose amendments via ADR when you find a rule that should change.

**If you are an AI coding assistant:**
- Your first action in any session for this project MUST be to read this document (see §2 for the exact protocol).
- You MUST follow every rule in this document, even if your default behavior differs.
- You MUST refuse to generate code that violates these rules, and explain which rule prevents it.
- You MUST cite the relevant rule when making architecture or pattern decisions.

---

## 2. Mandatory Pre-Session Protocol for AI Assistants

Every AI coding session for this project MUST follow this protocol. The first message of every AI session MUST include the protocol's confirmation output. No exceptions.

### 2.1 The Protocol

Before writing any code, generating any file, or proposing any architecture, the AI assistant MUST:

1. **Acknowledge the project and phase.** Confirm the assistant is working on "Marwat Welfare Society — Blood Donation & Distribution Platform" and identify which phase (P1–P4) the task belongs to.
2. **Confirm document load.** Confirm it has read (or will read before writing code) the four constitution documents:
   - `docs/01-PRD.md` (v1.1.0+)
   - `docs/02-SYSTEM_ARCHITECTURE.md` (v1.1.0+)
   - `docs/03-TECHNICAL_DESIGN_SPECIFICATION.md` (v1.1.0+)
   - `docs/04-AI_DEVELOPMENT_RULEBOOK.md` (v1.1.0+ — this document)
3. **State the task.** Restate the task in 1–2 sentences for verification.
4. **Identify the phase and affected components.** State which phase the task ships in and which features, packages, or infrastructure it touches.
5. **Identify relevant rules.** List the specific rule IDs from this document that apply to the task (e.g., R-TS-001, R-DB-005, R-SEC-002).
6. **Propose approach.** Briefly outline the approach in 2–3 sentences, citing architecture patterns from the System Architecture Document and TDS.
7. **Proceed** (in non-interactive batch sessions) or **await confirmation** (in interactive sessions). If the user prompt explicitly says "go ahead" / "just do it" / "不要问问题", skip step 7 and proceed.

### 2.2 Confirmation Template

The AI's first response MUST include a block in this format:

```markdown
## Pre-Session Confirmation

- **Project:** Marwat Welfare Society — Blood Donation & Distribution Platform
- **Phase:** [P1 | P2 | P3 | P4]
- **Constitution documents loaded:** PRD v1.1.0, Architecture v1.1.0, TDS v1.1.0, Rulebook v1.1.0
- **Task:** [restate the task in 1-2 sentences]
- **Affected components:** [list features/packages/infra]
- **Relevant rules:** [list rule IDs, e.g., R-TS-001, R-DB-005, R-SEC-002]
- **Approach:** [2-3 sentence outline citing architecture patterns]
- **Status:** [awaiting confirmation | proceeding per user instruction]
```

### 2.3 Refusal Protocol

If the AI is asked to do something that violates this rulebook, it MUST refuse, citing the specific rule. Example:

> I cannot generate a SQL query that bypasses RLS, because **R-SEC-002** requires all data access to go through RLS-protected tables. If you need this access pattern, the correct approach is [proposed alternative per architecture].

The AI MUST NOT silently comply with a request that violates the rules, even if the user insists. The AI MUST NOT generate the violating code with a comment like `// eslint-disable` to suppress the rule.

**Exception for solo-founder stage:** If the founder explicitly acknowledges the violation and provides a documented justification (e.g., "I know this skips validation — needed for a one-off data import, will add proper validation tomorrow"), the AI MAY proceed **only if the violation does not compromise security** (RLS, auth, secrets, PHI). Security violations are never negotiable. The AI MUST add a `// SOLO-FOUNDER EXCEPTION: <justification> — TODO: fix by <date>` comment in the code.

### 2.4 Session Continuation

If a session is resumed (e.g., via conversation history), the AI MUST re-confirm the protocol at the start of the resumed session, including the version numbers of the constitution documents. If the documents have been updated since the prior session, the AI MUST note the version change and re-read the updated document.

---

## 3. Repository & Workspace Conventions

### R-WS-001: Monorepo Layout

The project is a pnpm workspace monorepo. The top-level structure is defined in TDS §2.1. Do not create new top-level directories without an ADR.

### R-WS-002: Package Naming

All packages use the `@marwat/` scope:

- `@marwat/database` — schema, migrations, types.
- `@marwat/domain` — pure business logic.
- `@marwat/shared` — shared types, validation, utilities.
- `@marwat/ui` — shared web UI components.
- `@marwat/config` — shared ESLint, Prettier, tsconfig, Tailwind config.

Apps do not have a scope; they are `web` and `mobile` under `apps/`.

### R-WS-003: Import Paths

Within `apps/web`, use the `@/` alias for `src/`:

```typescript
// ✅ Correct
import { Button } from '@/components/ui/button';
import { useDonor } from '@/features/donors';

// ❌ Forbidden — relative paths across feature boundaries
import { useDonor } from '../../features/donors/hooks/use-donor';
```

Within packages, use the package name:

```typescript
// ✅ Correct
import { BloodType } from '@marwat/shared/types';
import { calculateEligibility } from '@marwat/domain/donors';

// ❌ Forbidden — deep relative imports across packages
import { BloodType } from '../../../shared/src/types/enums';
```

### R-WS-004: No Circular Dependencies

Circular dependencies between modules, features, or packages are forbidden. Enforced by `madge` in CI.

If a circular dependency is detected, the correct fix is to extract the shared dependency into a lower-level package (typically `@marwat/shared` or `@marwat/domain`), not to add an `// @ts-ignore` or restructure imports to hide the cycle.

### R-WS-005: Feature Module Boundaries

A feature module (`src/features/<name>/`) exports its public API via `index.ts`. Other features import only from the barrel. Importing from a feature's internal files is forbidden.

```typescript
// ✅ Correct
import { useDonor, type Donor, CreateDonorInput } from '@/features/donors';

// ❌ Forbidden
import { useDonor } from '@/features/donors/hooks/use-donor';
import type { Donor } from '@/features/donors/types/donor';
```

Enforced by ESLint `no-restricted-imports` rule.

### R-WS-006: Dependency Direction

Dependencies flow inward: `app` → `features` → `lib` → `@marwat/shared` / `@marwat/domain` / `@marwat/database`. The domain package has zero external infrastructure dependencies. Enforced by ESLint.

### R-WS-007: No `any` Type

The `any` type is forbidden. Use `unknown` and narrow with type guards, or define a proper type. The only exception is third-party library types that genuinely cannot be avoided, and these require an `// eslint-disable-next-line @typescript-eslint/no-explicit-any` with a comment explaining why.

### R-WS-008: No `console.log` in Production Code

Use the structured logger (`@/lib/logger`):

```typescript
// ✅ Correct
import { logger } from '@/lib/logger';
logger.info('Donor registered', { donorId });

// ❌ Forbidden
console.log('Donor registered:', donorId);
```

`console.error` is permitted only in Edge Functions where the logger is not available, and even there, structured JSON output is preferred.

### R-WS-009: File Size Limit

No source file should exceed 300 lines. If a file is approaching this limit, refactor by extracting sub-modules. Configuration files (e.g., `tailwind.config.ts`) are exempt.

### R-WS-010: No Magic Numbers or Strings

Constants must be named and defined in one place:

```typescript
// ✅ Correct
import { DEFERRAL_WINDOW_DAYS } from '@marwat/shared/constants';
const nextEligible = addDays(lastDonation, DEFERRAL_WINDOW_DAYS.WHOLE_BLOOD);

// ❌ Forbidden
const nextEligible = addDays(lastDonation, 56);  // what is 56?
```

---

## 4. Naming Conventions

### R-NM-001: Files

| Type | Convention | Example |
|---|---|---|
| React components (PascalCase) | `DonorList.tsx` | `DonorList.tsx` |
| React hooks (camelCase, `use` prefix) | `useDonor.ts` | `useDonor.ts` |
| Services | `donor-service.ts` | `donor-service.ts` |
| Repositories | `donor-repository.ts` | `donor-repository.ts` |
| Types | `donor.ts` (entity), `donor-dto.ts` (DTO) | `donor.ts` |
| Validation schemas | `create-donor.ts`, `update-donor.ts` | `create-donor.ts` |
| Utilities | `eligibility.ts` | `eligibility.ts` |
| Constants | `blood-types.ts` | `blood-types.ts` |
| Test files | `<name>.test.ts` or `<name>.spec.ts` | `eligibility.test.ts` |
| Migration files | `NNNN_description.sql` | `0024_add_donor_field.sql` |
| Edge Functions | `<name>/index.ts` (folder per function) | `report-generator/index.ts` |
| n8n workflows | `AUT-NNN-name.json` | `AUT-003-post-donation-thank-you.json` |

### R-NM-002: TypeScript Identifiers

| Type | Convention | Example |
|---|---|---|
| Variables, functions | camelCase | `donorId`, `calculateEligibility` |
| Types, interfaces | PascalCase | `Donor`, `BloodRequestDTO` |
| Enums (use `as const` objects, not `enum`) | PascalCase | `BloodType`, `RequestStatus` |
| Enum values | UPPER_SNAKE_CASE | `BLOOD_BAG_STATUS.COLLECTED` |
| Constants | UPPER_SNAKE_CASE | `DEFERRAL_WINDOW_DAYS` |
| React components | PascalCase | `DonorList`, `BloodRequestForm` |
| React hooks | camelCase, `use` prefix | `useDonor`, `useRealtimeInventory` |
| React props interface | PascalCase + `Props` suffix | `DonorListProps` |
| Event handlers | `on` + Event | `onSubmit`, `onDonorSelect` |
| Boolean variables | `is`/`has`/`can`/`should` prefix | `isEligible`, `hasConsent` |
| Async functions | verb + noun | `fetchDonor`, `createRequest` |

### R-NM-003: Database Identifiers

| Type | Convention | Example |
|---|---|---|
| Tables | snake_case, plural | `blood_bags`, `donors` |
| Columns | snake_case | `blood_type`, `created_at` |
| Foreign keys | `<referenced_table_singular>_id` | `donor_id`, `hospital_id` |
| Indexes | `idx_<table>_<columns>` | `idx_donors_blood_type` |
| Unique indexes | `uidx_<table>_<columns>` or `idx_<table>_<columns>_unique` | `uidx_donors_user_active` |
| Constraints | `chk_<table>_<purpose>` (check), `fk_<table>_<ref>` (foreign) | `chk_donors_blood_type`, `fk_donations_donor` |
| Functions | `<verb>_<subject>` | `has_role`, `calculate_eligibility` |
| Triggers | `trg_<table>_<event>` | `trg_donors_updated_at` |

### R-NM-004: API & Route Identifiers

- URL paths: kebab-case. `/admin/blood-requests`, `/donate-blood`.
- Query parameters: camelCase. `?bloodType=O+&page=2`.
- JSON body fields: camelCase (matching TypeScript). `{ "bloodType": "O+", "donorId": "..." }`.
- JSON response fields: camelCase (matching TypeScript DTOs).
- HTTP headers: standard HTTP conventions (`Authorization`, `Content-Type`, custom `X-` prefix).

### R-NM-005: Branch Naming

Git branches follow `<type>/<ticket>-<description>`:

- `feature/MWS-123-donor-registration`
- `bugfix/MWS-456-expiry-calculation`
- `chore/MWS-789-update-dependencies`
- `hotfix/MWS-321-security-patch`

Types: `feature`, `bugfix`, `chore`, `hotfix`, `docs`, `refactor`.

### R-NM-006: Commit Messages

Conventional Commits format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

- `type`: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`.
- `scope`: feature or package name (e.g., `donors`, `inventory`, `database`).
- `subject`: imperative mood, lowercase, no period, max 72 chars.
- `body`: wrap at 80 chars, explain why (not what).
- `footer`: breaking changes (`BREAKING CHANGE:`) and issue references (`Closes #123`).

Example:
```
feat(donors): add eligibility calculation per WHO guidelines

Implements the eligibility algorithm from PRD §6.3.2, using deferral
windows from @marwat/shared/constants. Includes medical screening
checks for HIV, hepatitis, malaria, pregnancy, and recent procedures.

Closes #123
```

---

## 5. TypeScript Rules

### R-TS-001: Strict Mode

`tsconfig.json` MUST have:

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

### R-TS-002: Use `interface` for Object Types, `type` for Unions

```typescript
// ✅ Correct — interface for object shapes
interface Donor {
  id: string;
  bloodType: BloodType;
}

// ✅ Correct — type for unions and intersections
type BloodType = 'A+' | 'A-' | 'B+' | 'B-' | 'AB+' | 'AB-' | 'O+' | 'O-';
type DonorWithUser = Donor & { user: User };

// ❌ Forbidden — type alias for plain object shape
type Donor = { id: string; bloodType: BloodType };
```

### R-TS-003: Use `as const` Objects Instead of `enum`

```typescript
// ✅ Correct
export const BloodType = {
  A_POSITIVE: 'A+',
  A_NEGATIVE: 'A-',
  // ...
} as const;
export type BloodType = (typeof BloodType)[keyof typeof BloodType];

// ❌ Forbidden — runtime overhead, tree-shaking issues
enum BloodType {
  A_POSITIVE = 'A+',
}
```

### R-TS-004: Exhaustive Switch Checks

```typescript
function getBagStatusColor(status: BloodBagStatus): string {
  switch (status) {
    case BloodBagStatus.COLLECTED: return 'blue';
    case BloodBagStatus.STORED: return 'green';
    // ... all cases
    case BloodBagStatus.DISCARDED: return 'gray';
    default:
      // Compile-time exhaustiveness check
      const _exhaustive: never = status;
      throw new Error(`Unknown status: ${_exhaustive}`);
  }
}
```

### R-TS-005: No Non-Null Assertion (`!`)

```typescript
// ✅ Correct — narrow with optional chaining or explicit check
const donorName = donor?.user?.fullName ?? 'Unknown';

// ❌ Forbidden — runtime crash risk
const donorName = donor!.user!.fullName;
```

The only exception is in test setup where the value is guaranteed; even there, prefer explicit assertion functions.

### R-TS-006: Explicit Return Types on Public APIs

Public functions, exported functions, and React components MUST have explicit return types:

```typescript
// ✅ Correct
export function calculateEligibility(input: EligibilityInput): EligibilityResult {
  // ...
}

export function DonorList({ donors }: DonorListProps): JSX.Element {
  // ...
}

// ❌ Forbidden — implicit return type on exported function
export function calculateEligibility(input: EligibilityInput) {
  // ...
}
```

Internal/private helpers may omit return types.

### R-TS-007: Use `unknown` over `any` for Catch Clauses

```typescript
// ✅ Correct
try {
  // ...
} catch (error: unknown) {
  if (error instanceof Error) {
    logger.error('Operation failed', { message: error.message });
  } else {
    logger.error('Unknown error', { error: String(error) });
  }
}

// ❌ Forbidden
try {
  // ...
} catch (error: any) {
  logger.error(error.message);
}
```

### R-TS-008: Prefer `Readonly` and `ReadonlyArray`

```typescript
// ✅ Correct
interface Donor {
  readonly id: string;
  readonly bloodType: BloodType;
  readonly donations: ReadonlyArray<Donation>;
}

// ❌ Forbidden — mutable where not needed
interface Donor {
  id: string;
  bloodType: BloodType;
  donations: Donation[];
}
```

### R-TS-009: Discriminated Unions for State

```typescript
// ✅ Correct
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

// Usage
function renderState<T>(state: AsyncState<T>) {
  switch (state.status) {
    case 'idle': return <Idle />;
    case 'loading': return <Loading />;
    case 'success': return <Data data={state.data} />;
    case 'error': return <Error error={state.error} />;
  }
}
```

### R-TS-010: Branded Types for IDs

To prevent mixing IDs of different entities:

```typescript
// packages/shared/src/types/branded.ts
declare const brand: unique symbol;
export type Brand<T, B> = T & { readonly [brand]: B };

export type DonorId = Brand<string, 'DonorId'>;
export type BloodBagId = Brand<string, 'BloodBagId'>;
export type RequestId = Brand<string, 'RequestId'>;

// Usage
function findDonor(id: DonorId): Donor | null { ... }
findDonor(bagId);  // ❌ Type error — correct
```

### R-TS-011: No `export default`

Named exports only. Default exports make refactoring and IDE navigation harder.

```typescript
// ✅ Correct
export function DonorList() { ... }
export function DonorDetail() { ... }

// ❌ Forbidden
export default function DonorList() { ... }
```

### R-TS-012: Import Ordering

Imports are ordered and grouped, enforced by ESLint:

```typescript
// 1. Node built-ins
import { promises as fs } from 'fs';

// 2. External packages
import { z } from 'zod';
import { useQuery } from '@tanstack/react-query';

// 3. Internal packages
import { BloodType } from '@marwat/shared/types';
import { calculateEligibility } from '@marwat/domain/donors';

// 4. Internal aliases
import { Button } from '@/components/ui/button';
import { useAuth } from '@/hooks/use-auth';

// 5. Relative imports (within same feature)
import { DonorCard } from './donor-card';

// 6. Type imports (separate)
import type { Donor } from '@marwat/shared/types';
```

---

## 6. React & Next.js Rules

### R-RE-001: Server Components by Default

Default to Server Components. Only add `"use client"` when the component needs:

- State (`useState`, `useReducer`).
- Effects (`useEffect`).
- Event handlers (`onClick`, etc.).
- Browser APIs (`window`, `localStorage`).
- Realtime subscriptions.
- Third-party client-only libraries.

```tsx
// ✅ Correct — server component (default)
export default async function DonorListPage() {
  const donors = await fetchDonors();
  return <DonorList donors={donors} />;
}

// ✅ Correct — client component (only when needed)
'use client';
import { useState } from 'react';
export function DonorFilter() {
  const [filter, setFilter] = useState('');
  // ...
}
```

### R-RE-002: Client Components Are Leaf Nodes

Client Components should be as small as possible — leaf nodes that receive data via props and emit events via callbacks. They should not fetch data directly; data fetching happens in Server Components or via React Query hooks that call the typed Supabase client.

### R-RE-003: Use `async/await` over `.then()`

```tsx
// ✅ Correct
async function DonorPage({ params }: { params: { id: string } }) {
  const donor = await fetchDonor(params.id);
  return <DonorDetail donor={donor} />;
}

// ❌ Forbidden
function DonorPage({ params }: { params: { id: string } }) {
  return fetchDonor(params.id).then(donor => <DonorDetail donor={donor} />);
}
```

### R-RE-004: Error Boundaries

Every route segment MUST have an `error.tsx` boundary. Errors propagate to the nearest boundary; do not try/catch in components except for specific recoverable cases.

```tsx
// app/(dashboard)/admin/donors/error.tsx
'use client';
export default function DonorsError({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div>
      <h2>Failed to load donors</h2>
      <Button onClick={reset}>Try again</Button>
    </div>
  );
}
```

### R-RE-005: Loading States

Every route segment SHOULD have a `loading.tsx` for streaming UI:

```tsx
// app/(dashboard)/admin/donors/loading.tsx
import { Skeleton } from '@/components/ui/skeleton';
export default function Loading() {
  return <Skeleton className="h-96 w-full" />;
}
```

### R-RE-006: Form Handling

Use React Hook Form + Zod for all forms:

```tsx
'use client';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { createDonorSchema, type CreateDonorInput } from '@/features/donors';

export function DonorForm() {
  const form = useForm<CreateDonorInput>({
    resolver: zodResolver(createDonorSchema),
    defaultValues: { /* ... */ },
  });
  
  const onSubmit = async (data: CreateDonorInput) => {
    // ...
  };
  
  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      {/* ... */}
    </form>
  );
}
```

### R-RE-007: Server Actions for Mutations (Post-v1)

For v1, mutations go through Edge Functions or direct Supabase client calls. Server Actions are permitted in Next.js 15 but add complexity; evaluate post-v1.

### R-RE-008: Realtime Subscriptions Cleanup

Every realtime subscription MUST be cleaned up in the effect:

```tsx
useEffect(() => {
  const channel = supabase.channel('inventory').on(/* ... */).subscribe();
  return () => { supabase.removeChannel(channel); };
}, [supabase]);
```

Failure to clean up causes memory leaks and stale subscriptions.

### R-RE-009: No Inline Styles

Use Tailwind classes. Inline styles are forbidden except for dynamic values that cannot be expressed as classes (e.g., a CSS variable set from a data attribute):

```tsx
// ✅ Correct
<div className="flex flex-col gap-4 p-6" />

// ❌ Forbidden
<div style={{ display: 'flex', flexDirection: 'column', gap: '1rem', padding: '1.5rem' }} />
```

### R-RE-010: Component Composition

Prefer composition over configuration. Use `children` and slot props:

```tsx
// ✅ Correct — composable
<Card>
  <Card.Header>
    <Card.Title>Donor</Card.Title>
  </Card.Header>
  <Card.Body>
    <DonorInfo donor={donor} />
  </Card.Body>
  <Card.Footer>
    <Button>Save</Button>
  </Card.Footer>
</Card>

// ❌ Avoid — over-configured
<Card title="Donor" body={<DonorInfo donor={donor} />} footer={<Button>Save</Button>} />
```

### R-RE-011: Accessibility in Components

Every interactive component MUST be accessible:

- Buttons use `<button>`, not `<div onClick>`.
- Form inputs have associated `<label>` elements.
- Modals trap focus and restore on close.
- Icons have `aria-label` or `aria-hidden` (if decorative).
- Color is never the only indicator of state (add icon or text).

### R-RE-012: Image Optimization

Use `next/image` for all images:

```tsx
import Image from 'next/image';

<Image
  src={donor.avatarUrl}
  alt={`${donor.fullName} avatar`}
  width={48}
  height={48}
  className="rounded-full"
/>
```

Plain `<img>` is forbidden except in Edge Function-generated HTML (where Next.js Image is unavailable).

---

## 7. React Native & Expo Rules

### R-RN-001: Share Logic, Not UI

The mobile app shares domain logic, types, validation, and constants with the web app via `@marwat/shared` and `@marwat/domain`. UI components are NOT shared — React Native and React DOM have different primitives.

### R-RN-002: Use Expo Router

Use Expo Router (file-based routing, mirroring Next.js conventions). Do not use React Navigation directly.

### R-RN-003: Use NativeWind or React Native Paper

For styling, use NativeWind (Tailwind for React Native) to mirror web styling conventions, or React Native Paper for a Material Design approach. Do not mix.

### R-RN-004: Offline-First for Critical Features

Blood request submission and donation recording at camps MUST work offline. Use the offline queue pattern (TDS §15.3) for mutations when the device is offline.

### R-RN-005: Push Notification Channel Routing

Route push notifications to the correct channel:
- Default channel: routine notifications.
- Emergency channel: emergency blood requests (high importance, bypasses Do Not Disturb on Android).

### R-RN-006: Use Secure Storage for Tokens

JWT tokens and refresh tokens MUST be stored in `expo-secure-store`, not `AsyncStorage`. `AsyncStorage` is for non-sensitive data only.

### R-RN-007: Test on Low-End Devices

The app MUST be tested on a mid-range Android device (e.g., 2GB RAM, Android 10+). Performance that is acceptable on a flagship phone but unusable on a mid-range device is a bug.

---

## 8. Database & SQL Rules

### R-DB-001: All Schema Changes via Migrations

Schema changes are made exclusively via versioned migration files in `packages/database/migrations/`. Direct schema changes via Supabase dashboard are forbidden. Enforced by comparing schema in CI to migrations.

### R-DB-002: Every Table Has Audit Columns

Every table that contains domain data MUST have:

```sql
created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
created_by UUID REFERENCES public.user_profiles(id),
updated_by UUID REFERENCES public.user_profiles(id),
deleted_at TIMESTAMPTZ,  -- for soft delete
deleted_by UUID REFERENCES public.user_profiles(id)
```

Lookup tables (e.g., `storage_locations`) may omit `deleted_*` if soft delete is not applicable.

### R-DB-003: RLS on Every User-Data Table

Every table containing user data MUST have RLS enabled and policies defined. Tables without RLS are forbidden. The only exception is `audit_log`, which is accessed via SECURITY DEFINER function.

### R-DB-004: No `SELECT *` in Application Code

Always specify columns explicitly:

```typescript
// ✅ Correct
const { data } = await supabase
  .from('donors')
  .select('id, donor_number, blood_type, user_profiles(full_name)');

// ❌ Forbidden
const { data } = await supabase.from('donors').select('*');
```

This prevents breaking changes when columns are added and improves performance by not fetching unused columns.

### R-DB-005: Foreign Keys Are Mandatory

Every relationship MUST be enforced with a foreign key. No "soft" foreign keys (just a UUID column with no constraint).

### R-DB-006: Index Every Foreign Key

Every foreign key column MUST have an index. Foreign keys do not automatically get indexed in PostgreSQL.

### R-DB-007: Use `citext` for Email Columns

Email columns use `citext` to avoid case-sensitivity issues:

```sql
email CITEXT UNIQUE NOT NULL
```

### R-DB-008: Use `timestamptz`, Never `timestamp`

All timestamp columns are `timestamptz` (with time zone). Never use `timestamp` (without time zone) — it causes bugs when the server timezone changes.

### R-DB-009: Constraints over Application Logic

Enforce data integrity with database constraints (CHECK, UNIQUE, NOT NULL, FOREIGN KEY) wherever possible. Application-level validation is for user feedback, not data integrity. If the database would reject the data, the constraint should exist.

### R-DB-010: Soft Delete for User Data

User data (donors, patients, hospitals, volunteers, etc.) is soft-deleted (`deleted_at` set), never hard-deleted. This preserves referential integrity for historical records (e.g., a blood bag's donor must remain findable even if the donor deletes their account).

Hard delete is permitted only for:
- Lookup table entries that have never been referenced.
- Test data in non-production environments.
- GDPR/DP-law deletion requests where retention period has expired (logged in audit).

### R-DB-011: No Triggers for Business Logic

Triggers are used only for:
- `updated_at` auto-update.
- Audit log insertion.
- Domain event emission.
- Computed/derived column maintenance (rare).

Triggers MUST NOT implement business logic. Business logic belongs in the application layer (or domain layer), where it is testable and visible.

### R-DB-012: Migration Reversibility

Every migration MUST have a down-migration where possible. Irreversible migrations (e.g., dropping a column) require explicit ADR approval. The down-migration is tested in CI.

### R-DB-013: No ORM

The project uses the Supabase JS client (which uses PostgREST), not an ORM (Prisma, TypeORM, Drizzle). Rationale: PostgREST + RLS is the security model; an ORM would bypass RLS or require duplicate schema definitions. The Supabase client is typed via `supabase gen types typescript`.

### R-DB-014: JSON Columns for Flexible Data, Not for Normalizable Data

JSON columns (JSONB) are used for genuinely flexible data (e.g., `medical_screening`, `communication_preferences`). They are NOT used as a substitute for normalization. If a JSON field's keys become stable, promote them to real columns.

---

## 9. Security Rules

### R-SEC-001: Zero Trust

Every API endpoint, every database query, every Edge Function assumes the caller is hostile. Authentication and authorization are checked at every trust boundary, not just at the perimeter.

### R-SEC-002: RLS Is the Last Line of Defense

Even if the application has a bug, RLS prevents unauthorized data access. Every application-level authorization check is a *convenience* (better UX, fewer errors); RLS is the *security*.

### R-SEC-003: No Secrets in Code

Secrets (API keys, passwords, tokens) are in environment variables only. The repo is regularly scanned by `trufflehog` and GitHub secret scanning. Detected secrets trigger immediate revocation and rotation.

If a secret is accidentally committed, the response is:
1. Rotate the secret immediately (assume it is compromised).
2. Remove from git history (via `git filter-repo` or BFG).
3. Force-push the cleaned history.
4. Notify all contributors to re-clone.
5. Document in `SECURITY.md`.

### R-SEC-004: Input Validation on Every API Boundary

Every input to an Edge Function or API route is validated with a Zod schema before any processing. Unvalidated inputs are forbidden. Validation failures return 400 with field-specific errors.

### R-SEC-005: Output Sanitization

All user-generated content rendered as HTML is sanitized with `dompurify` and a strict allowlist. All user-generated content rendered as text uses React's default escaping (no `dangerouslySetInnerHTML`).

```tsx
// ✅ Correct — React escapes by default
<p>{userComment}</p>

// ✅ Correct — sanitized HTML for rich text
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(content) }} />

// ❌ Forbidden — XSS vulnerability
<div dangerouslySetInnerHTML={{ __html: content }} />
```

### R-SEC-006: SQL Injection Prevention

All database queries use the Supabase client's parameterized queries. Raw SQL with string interpolation is forbidden:

```typescript
// ✅ Correct
const { data } = await supabase.from('donors').select('*').eq('id', donorId);

// ❌ Forbidden — SQL injection risk
const { data } = await supabase.rpc(`get_donor('${donorId}')`);
```

If a raw SQL query is unavoidable (rare), use parameterized queries via the `pg` library with explicit `$1` placeholders.

### R-SEC-007: CSRF Protection

All mutating API routes require CSRF protection. For Supabase Auth, the JWT in the httpOnly cookie is automatically included, and the SameSite=lax cookie attribute provides CSRF protection. For Edge Functions called from the client, a CSRF token is included in the request header.

### R-SEC-008: Rate Limiting

Every public endpoint is rate-limited. Authenticated endpoints are rate-limited per user. Rate limit thresholds are in the Architecture Document §7.2.8.

### R-SEC-009: Security Headers

The middleware sets security headers (CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy) on every response. These are tested in CI via a header-checking script.

### R-SEC-010: File Upload Security

All file uploads:
1. Use signed upload URLs (5-minute expiry).
2. Validate file type (whitelist) and size (per-bucket limit).
3. Generate a UUID filename (original filename stored as metadata only).
4. Are virus-scanned before being marked available.
5. Are stored in private buckets (no public read except for the `public-assets` bucket).

### R-SEC-011: Password Policy

- Minimum 12 characters.
- Must include uppercase, lowercase, digit, and symbol.
- Cannot be in the top 10,000 common passwords list.
- Cannot match the user's email or name.
- Hashed with Argon2id via Supabase Auth.
- Never logged, never transmitted in plain text, never stored in application code.

### R-SEC-012: MFA Enforcement

ADMIN and SUPER_ADMIN roles MUST have MFA (TOTP) enabled within 7 days of role assignment. Failure to enroll suspends the role until enrollment completes.

### R-SEC-013: Audit Everything

Every state-changing operation (create, update, delete, state transition, role change) MUST produce an audit log entry with actor, timestamp, before-state, after-state, IP, and user agent. Audit logs are append-only and retained 7 years.

### R-SEC-014: No PHI in Logs

PHI (patient names, disease conditions, medical reports, donor medical screening) MUST NOT appear in logs. Logs use entity IDs only; if a name is needed for debugging, it is fetched separately with explicit authorization.

### R-SEC-015: No PHI to AI Services

PHI is anonymized before being sent to any AI service. Names are replaced with `Patient #123`, hospital names with `Hospital A`, etc. The mapping is stored locally and never sent to the AI.

### R-SEC-016: Encryption at Rest

Database encryption is managed by Supabase (transparent encryption). Column-level encryption via pgcrypto is applied to PHI fields. Encryption keys are stored in Supabase Vault (or environment variables for v1).

### R-SEC-017: Secure Cookies

All cookies are:
- `httpOnly: true` (no JavaScript access).
- `secure: true` in production (HTTPS only).
- `sameSite: 'lax'` (CSRF protection).
- Path-scoped to the application.

### R-SEC-018: Dependency Security

- `pnpm audit` runs in CI; high-severity vulnerabilities block merge.
- Dependabot opens PRs for updates.
- New dependencies require review (license, maintenance, security history).

---

## 10. Validation & Error Handling Rules

### R-VAL-001: Validate at Every Boundary

Input is validated at every trust boundary:
- Client-side (for UX, can be bypassed).
- API/Edge Function (for security, cannot be bypassed).
- Database (constraints as last resort).

### R-VAL-002: Zod Schemas Are the Single Source of Truth

Zod schemas define both runtime validation and TypeScript types:

```typescript
// ✅ Correct
const createDonorSchema = z.object({ /* ... */ });
type CreateDonorInput = z.infer<typeof createDonorSchema>;

// ❌ Forbidden — manual type separate from validation
interface CreateDonorInput { /* ... */ }
const createDonorSchema = z.object({ /* ... */ });
```

### R-VAL-003: Descriptive Error Messages

Validation errors include the field path and a human-readable message:

```typescript
// ✅ Correct
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": {
      "fieldErrors": {
        "blood_type": ["Blood type must be one of: A+, A-, B+, B-, AB+, AB-, O+, O-"],
        "donor.phone": ["Phone must be in format +923XXXXXXXXX"]
      }
    }
  }
}
```

### R-VAL-004: Never Swallow Errors

```typescript
// ✅ Correct — handle or rethrow
try {
  await saveDonor(donor);
} catch (error) {
  logger.error('Failed to save donor', { donorId: donor.id, error });
  throw new SaveError('Failed to save donor', { cause: error });
}

// ❌ Forbidden — silent failure
try {
  await saveDonor(donor);
} catch (e) { /* ignored */ }
```

### R-VAL-005: Custom Error Classes

Use custom error classes for domain-specific errors:

```typescript
// packages/shared/src/errors.ts
export class NotFoundError extends Error {
  constructor(entity: string, id: string) {
    super(`${entity} not found: ${id}`);
    this.name = 'NotFoundError';
  }
}

export class AuthorizationError extends Error { /* ... */ }
export class ValidationFailedError extends Error { /* ... */ }
export class ConflictError extends Error { /* ... */ }
export class SLABreachError extends Error { /* ... */ }
```

### R-VAL-006: Error Response Standard

All API errors follow the standard format (TDS §8.6). The `correlationId` is propagated to logs for debugging.

### R-VAL-007: Idempotency

Mutating operations that may be retried (webhooks, mobile offline queue) MUST be idempotent. The client sends an idempotency key; the server checks if the operation was already applied.

---

## 11. Testing Rules

### R-TST-001: Test Pyramid

- 70% unit tests (pure functions, components in isolation).
- 20% integration tests (feature with database).
- 10% E2E tests (full user journeys).

### R-TST-002: Coverage Thresholds

- `packages/domain/`: 100% coverage (pure functions).
- `apps/web/src/features/*/services/`: ≥ 90% coverage.
- `apps/web/src/features/*/repositories/`: ≥ 90% coverage.
- `apps/web/src/components/`: ≥ 80% coverage.
- `apps/web/tests/rls/`: 100% policy coverage.
- `infrastructure/supabase/functions/`: ≥ 80% coverage.

Coverage enforced in CI; merge-blocking.

### R-TST-003: Test Naming

Tests use descriptive names, not generic ones:

```typescript
// ✅ Correct
describe('calculateEligibility', () => {
  it('returns ineligible when donor is permanently deferred', () => { ... });
  it('returns ineligible within 56 days of whole blood donation', () => { ... });
  it('returns eligible when all conditions are met', () => { ... });
});

// ❌ Forbidden
describe('calculateEligibility', () => {
  it('test 1', () => { ... });
  it('works', () => { ... });
});
```

### R-TST-004: AAA Pattern

Tests follow Arrange-Act-Assert:

```typescript
it('returns ineligible when donor is permanently deferred', () => {
  // Arrange
  const input: EligibilityInput = {
    permanentDeferral: true,
    // ...
  };
  
  // Act
  const result = calculateEligibility(input);
  
  // Assert
  expect(result.eligible).toBe(false);
  expect(result.reason).toBe('Permanently deferred');
});
```

### R-TST-005: No Real Network Calls in Unit Tests

Unit tests do not make real network calls. Mock the Supabase client:

```typescript
const mockSupabase = {
  from: vi.fn(() => ({
    select: vi.fn(() => ({
      eq: vi.fn(() => ({ data: [mockDonor], error: null })),
    })),
  })),
};
```

Integration tests use a local Supabase instance (Docker) and make real calls.

### R-TST-006: RLS Tests Are Mandatory

Every RLS policy has a test that verifies:
- Authorized users can perform the operation.
- Unauthorized users cannot.
- Unauthenticated users cannot (where applicable).

### R-TST-007: E2E Tests Cover Critical Paths

E2E tests cover the user journeys defined in the PRD §8:
- Donor registration.
- Blood request submission.
- Emergency request fulfillment.
- Volunteer task acceptance and completion.
- Donation recording at a camp.
- Report generation and publication.

### R-TST-008: Tests Are Code

Tests follow the same coding standards as production code: typed, linted, formatted, reviewed. Tests are not exempt from the rules in this document.

### R-TST-009: No `xit` or `xdescribe`

Disabled tests are forbidden. If a test is broken, fix it or delete it. A disabled test is technical debt.

### R-TST-010: Test Data Is Synthetic

Test data uses clearly synthetic names, phone numbers, and addresses. Real personal data in tests is forbidden.

---

## 12. Performance Rules

### R-PERF-001: Lazy Load Below the Fold

Below-the-fold content is lazy-loaded:

```tsx
import { lazy } from 'react';
const HeavyChart = lazy(() => import('./heavy-chart'));
```

### R-PERF-002: Image Optimization

All images use `next/image` with appropriate `sizes` attribute. The `width` and `height` are always specified to prevent layout shift.

### R-PERF-003: Database Query Limits

Every database query has a limit. Unbounded queries are forbidden:

```typescript
// ✅ Correct
const { data } = await supabase.from('donors').select('*').limit(100);

// ❌ Forbidden — unbounded
const { data } = await supabase.from('donors').select('*');
```

### R-PERF-004: N+1 Query Prevention

When fetching related data, use the Supabase client's relation syntax, not multiple queries:

```typescript
// ✅ Correct — single query
const { data } = await supabase
  .from('donors')
  .select('id, donor_number, donations(id, donation_date)');

// ❌ Forbidden — N+1
const { data: donors } = await supabase.from('donors').select('*');
for (const donor of donors) {
  const { data: donations } = await supabase
    .from('donations')
    .select('*')
    .eq('donor_id', donor.id);
}
```

### R-PERF-005: Index Every Query

Every query in the application is backed by an index. CI runs `EXPLAIN ANALYZE` on representative queries and flags sequential scans on large tables.

### R-PERF-006: Debounce Realtime Updates

Realtime updates that fire rapidly (e.g., bulk inventory updates) are debounced before re-rendering:

```typescript
const debouncedData = useDebounce(realtimeData, 300);
```

### R-PERF-007: Bundle Size Monitoring

CI tracks bundle size. A PR that increases the initial bundle by more than 10KB requires justification.

### R-PERF-008: Caching Strategy

- Public website pages: cached at CDN, revalidated via webhook on content change.
- Static assets: cached for 1 year (immutable).
- User-specific data: cached in React Query with appropriate `staleTime`.
- Database query results: not cached (RLS makes caching complex).

---

## 13. Accessibility Rules

### R-A11Y-001: WCAG 2.1 AA Compliance

The platform conforms to WCAG 2.1 AA. This is non-negotiable and tested via automated tools (axe-core) and manual review.

### R-A11Y-002: Semantic HTML

Use semantic HTML elements (`<button>`, `<nav>`, `<main>`, `<article>`, `<header>`, `<footer>`) over `<div>` wherever the semantic element applies.

### R-A11Y-003: Keyboard Navigation

Every interactive element is reachable and operable via keyboard. Tab order is logical. Visible focus indicators are present and have sufficient contrast.

### R-A11Y-004: ARIA When Needed

ARIA attributes are added when semantic HTML is insufficient, but not gratuitously. "No ARIA is better than bad ARIA."

### R-A11Y-005: Color Contrast

Color contrast meets WCAG AA ratios:
- Body text: 4.5:1 minimum.
- Large text (≥18pt or 14pt bold): 3:1 minimum.
- UI components and graphics: 3:1 minimum.

### R-A11Y-006: Color Is Not the Only Signal

State is never indicated by color alone. Always add an icon, text, or pattern:

```tsx
// ✅ Correct
<Badge>
  <span className="h-2 w-2 rounded-full bg-red-500" />
  Expired
</Badge>

// ❌ Forbidden — color only
<div className="bg-red-500" />
```

### R-A11Y-007: Form Labels

Every form input has an associated label:

```tsx
// ✅ Correct
<label htmlFor="blood-type">Blood Type</label>
<select id="blood-type" {...register('bloodType')}>...</select>

// ❌ Forbidden
<select {...register('bloodType')}>...</select>
```

### R-A11Y-008: Error Messages Are Announced

Form errors are announced to screen readers via `aria-invalid` and `aria-describedby`:

```tsx
<input
  id="phone"
  aria-invalid={errors.phone ? 'true' : 'false'}
  aria-describedby={errors.phone ? 'phone-error' : undefined}
  {...register('phone')}
/>
{errors.phone && (
  <p id="phone-error" role="alert">{errors.phone.message}</p>
)}
```

### R-A11Y-009: Skip Links

Every page has a "Skip to content" link as the first focusable element:

```tsx
<a href="#main" className="sr-only focus:not-sr-only">Skip to content</a>
<main id="main">...</main>
```

### R-A11Y-010: Reduced Motion

Respect `prefers-reduced-motion`:

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 14. Internationalization Rules

### R-I18N-001: No Hardcoded Strings

All user-facing strings are externalized via `next-intl` (or equivalent):

```tsx
// ✅ Correct
import { useTranslations } from 'next-intl';
const t = useTranslations('donors');
return <h1>{t('title')}</h1>;

// ❌ Forbidden
return <h1>Donor Management</h1>;
```

### R-I18N-002: Locale-Aware Formatting

Dates, times, and numbers use locale-aware formatting:

```tsx
import { FormattedDate, FormattedNumber } from 'react-intl';

<FormattedDate value={date} year="numeric" month="long" day="numeric" />
<FormattedNumber value={1234.56} style="currency" currency="PKR" />
```

### R-I18N-003: RTL Support

The layout must work in RTL (right-to-left) languages. Use logical properties (`ps-4` not `pl-4`) where possible, in preparation for future Arabic support.

### R-I18N-004: Pluralization

Use proper pluralization rules:

```tsx
// ✅ Correct
t('donations.count', { count: donations.length });
// "1 donation" / "5 donations"

// ❌ Forbidden
`${count} donation(s)`
```

### R-I18N-005: Translation Files

Translation files are JSON, organized by feature:

```
src/messages/
├── en-PK/
│   ├── common.json
│   ├── donors.json
│   ├── inventory.json
│   └── ...
└── ur-PK/
    ├── common.json
    ├── donors.json
    └── ...
```

---

## 15. Documentation Rules

### R-DOC-001: README in Every Package

Every package (`apps/web`, `apps/mobile`, `packages/*`) has a `README.md` documenting:
- Purpose.
- Setup.
- Scripts.
- Architecture (link to relevant docs).
- Owner / maintainer.

### R-DOC-002: JSDoc on Public APIs

Public functions, classes, and types have JSDoc comments:

```typescript
/**
 * Calculates donor eligibility based on WHO guidelines.
 * 
 * @param input - Donor's medical and donation history
 * @returns Eligibility result with reason and next eligible date
 * 
 * @example
 * ```ts
 * const result = calculateEligibility({
 *   lastDonationDate: new Date('2024-01-01'),
 *   lastDonationComponent: 'whole_blood',
 *   // ...
 * });
 * ```
 */
export function calculateEligibility(input: EligibilityInput): EligibilityResult {
```

### R-DOC-003: ADR for Architectural Decisions

Every significant architectural decision is recorded as an ADR in `docs/adr/`. See Architecture Document §17.

### R-DOC-004: Keep Docs in Sync

Code changes that affect documented behavior MUST update the docs in the same PR. CI checks for stale docs (e.g., links to removed files, references to old patterns).

### R-DOC-005: Code Comments Explain Why, Not What

```typescript
// ✅ Correct — explains why
// 56 days per WHO whole blood deferral guideline
const DEFERRAL_DAYS_WHOLE_BLOOD = 56;

// ❌ Forbidden — explains what (obvious from code)
// Set deferral days to 56
const DEFERRAL_DAYS_WHOLE_BLOOD = 56;
```

### R-DOC-006: No Commented-Out Code

Commented-out code is forbidden. Use git history to retrieve old code. The only exception is a brief TODO with context:

```typescript
// TODO(MWS-456): Replace with batch API once available
```

### R-DOC-007: TODO Format

TODOs follow the format `TODO(<ticket>): <description>`:

```typescript
// TODO(MWS-789): Add support for plasma-only donors
```

Bare `// TODO` without a ticket is forbidden — it will never be tracked.

---

## 16. Git & Pull Request Rules

### R-GIT-001: Small, Focused PRs

PRs should be small (under 500 lines of diff) and focused on one concern. Large PRs are harder to review and more likely to introduce bugs. If a change requires a large PR, split it into stacked PRs.

During the solo-founder stage, the 500-line guideline is a target, not a hard limit. The founder may exceed it with a note in the PR description explaining why splitting was not practical.

### R-GIT-002: PR Description Template

```markdown
## Summary
[1-3 sentences: what does this PR do and why?]

## Phase
[P1 | P2 | P3 | P4]

## Type
- [ ] feat: New feature
- [ ] fix: Bug fix
- [ ] refactor: Code restructure
- [ ] docs: Documentation
- [ ] test: Test addition
- [ ] chore: Maintenance

## Affected Documents
- [ ] PRD (docs/01-PRD.md)
- [ ] Architecture (docs/02-SYSTEM_ARCHITECTURE.md)
- [ ] TDS (docs/03-TECHNICAL_DESIGN_SPECIFICATION.md)
- [ ] Rulebook (docs/04-AI_DEVELOPMENT_RULEBOOK.md)

## Testing
- [ ] Unit tests pass
- [ ] RLS tests pass
- [ ] Integration tests pass (Phase 2+)
- [ ] E2E tests pass (Phase 2+, if applicable)
- [ ] Manual testing completed locally

## Checklist
- [ ] Code follows the AI Development Rulebook
- [ ] No `any` types introduced
- [ ] No `console.log` in production code
- [ ] No commented-out code
- [ ] All new functions have JSDoc
- [ ] All new database tables have RLS
- [ ] All new endpoints have validation
- [ ] All new features have tests
- [ ] Documentation updated (if applicable)
- [ ] CHANGELOG.md updated (for user-facing changes)

## Solo-Founder Notes (if applicable)
[Any deviations from the rulebook, with justification. Security deviations are NOT permitted.]

## Screenshots / Recordings
[For UI changes]

## Related Issues
Closes #123
```

### R-GIT-003: Linear History

Use rebase, not merge, to keep history linear:

```bash
git pull --rebase
git rebase main
```

Merge commits are forbidden in feature branches. The main branch uses squash-merge to keep one commit per PR.

### R-GIT-004: Approval Policy

**Solo-founder stage (Phases 1–3):** Self-merge is permitted. The founder reviews their own PR (using the AI as a second pair of eyes) and merges once CI passes. A brief justification in the PR description (`self-merged, tested locally, no security impact`) is required.

**Post-Phase 4 (multiple contributors):** PRs require at least one approval from a human reviewer other than the author. Changes touching security, database schema, or RLS policies require a second approval from a senior maintainer.

In all stages, security-sensitive changes (RLS policies, auth flows, secrets handling, PHI access) deserve extra scrutiny — even during the solo-founder stage, the founder should explicitly call these out in the PR description and consider requesting external review (e.g., from an AI security scan or a trusted advisor).

### R-GIT-005: CI Must Pass

All CI checks must pass before merge. During the solo-founder stage, if CI is broken due to a flaky test or environment issue (not a real code problem), the founder may merge with a `SOLO-FOUNDER CI OVERRIDE: <justification>` note in the PR description, and must file an issue to fix the CI flakiness. This exception does NOT apply to lint, typecheck, or test failures — only to infrastructure-level CI issues.

### R-GIT-006: No Direct Pushes to Main

All changes go through PR. Direct pushes to `main` or `staging` are forbidden (branch protection enforced on GitHub).

### R-GIT-007: Signed Commits

Commits are GPG-signed (or SSH-signed). `git config commit.gpgsign true` is in the contribution guide.

### R-GIT-008: CHANGELOG.md Maintenance

Every PR that affects user-facing behavior or API contracts updates `CHANGELOG.md` under an "Unreleased" section. The format follows Keep a Changelog.

---

## 17. Forbidden Patterns (Never Do These)

This section is an explicit list of patterns that are NEVER acceptable. Encountering any of these in a PR is an immediate request-for-changes, no exceptions.

### F-001: `any` Type

```typescript
// ❌ NEVER
function process(data: any) { ... }
```

### F-002: `eslint-disable` Without Justification

```typescript
// ❌ NEVER
// eslint-disable-next-line
const data = JSON.parse(input);
```

Allowed only with explicit comment: `// eslint-disable-next-line @typescript-eslint/no-explicit-any -- Third-party type is genuinely any; tracking in MWS-123`.

### F-003: `console.log` in Production Code

```typescript
// ❌ NEVER
console.log('donor:', donor);
```

### F-004: Inline Styles (except for dynamic values)

```tsx
// ❌ NEVER
<div style={{ color: 'red', padding: '10px' }} />
```

### F-005: `dangerouslySetInnerHTML` Without Sanitization

```tsx
// ❌ NEVER
<div dangerouslySetInnerHTML={{ __html: userContent }} />
```

### F-006: Raw SQL with String Interpolation

```typescript
// ❌ NEVER
const query = `SELECT * FROM donors WHERE id = '${donorId}'`;
```

### F-007: Secrets in Code

```typescript
// ❌ NEVER
const API_KEY = 'sk_live_abc123';
```

### F-008: Hardcoded URLs

```typescript
// ❌ NEVER
const response = await fetch('https://marwatwelfare.org/api/donors');

// ✅ Use environment variable
const response = await fetch(`${process.env.NEXT_PUBLIC_APP_URL}/api/donors`);
```

### F-009: Magic Numbers

```typescript
// ❌ NEVER
if (daysSinceDonation < 56) { ... }

// ✅ Use named constant
if (daysSinceDonation < DEFERRAL_WINDOW_DAYS.WHOLE_BLOOD) { ... }
```

### F-010: Commented-Out Code

```typescript
// ❌ NEVER
// const oldMethod = () => { ... };
// oldMethod();
```

### F-011: `export default`

```typescript
// ❌ NEVER
export default function DonorList() { ... }
```

### F-012: Non-Null Assertion (`!`)

```typescript
// ❌ NEVER
const donor = donors.find(d => d.id === id)!;
```

### F-013: Direct Database Access Bypassing RLS

```typescript
// ❌ NEVER
const supabase = createClient(url, SERVICE_ROLE_KEY);  // bypasses RLS
const { data } = await supabase.from('donors').select('*');
```

Service-role key is used only in Edge Functions, and only for operations explicitly justified in the function's documentation.

### F-014: `enum` (Use `as const` Object)

```typescript
// ❌ NEVER
enum BloodType { A_POSITIVE = 'A+' }
```

### F-015: Implicit Return Type on Exported Function

```typescript
// ❌ NEVER
export function calculateEligibility(input: EligibilityInput) { ... }
```

### F-016: Unhandled Promise Rejections

```typescript
// ❌ NEVER
someAsyncOperation();  // no await, no .catch
```

### F-017: `useEffect` Without Dependency Array

```tsx
// ❌ NEVER
useEffect(() => {
  fetchData();
});  // runs every render
```

### F-018: Swallowed Errors

```typescript
// ❌ NEVER
try { await save(); } catch (e) {}
```

### F-019: Direct DOM Manipulation

```tsx
// ❌ NEVER
document.getElementById('foo').style.color = 'red';
```

Use React state and props instead.

### F-020: Mutating State

```typescript
// ❌ NEVER
donor.name = 'New Name';
setDonors(donors);

// ✅ Immutable update
setDonors(donors.map(d => d.id === donor.id ? { ...d, name: 'New Name' } : d));
```

### F-021: Cross-Feature Internal Imports

```typescript
// ❌ NEVER
import { useDonor } from '@/features/donors/hooks/use-donor';

// ✅ Use barrel
import { useDonor } from '@/features/donors';
```

### F-022: Business Logic in Components

```tsx
// ❌ NEVER
function DonorCard({ donor }) {
  const isEligible = donor.lastDonation && 
    daysBetween(donor.lastDonation, new Date()) > 56 &&
    !donor.medicalScreening.hivPositive &&
    // ... 30 lines of business logic
  return <div>...</div>;
}

// ✅ Logic in domain layer
function DonorCard({ donor }) {
  const eligibility = calculateEligibility(toEligibilityInput(donor));
  return <div>...</div>;
}
```

### F-023: Business Logic in Database Triggers

```sql
-- ❌ NEVER
CREATE TRIGGER calculate_eligibility BEFORE INSERT ON donations
FOR EACH ROW EXECUTE FUNCTION complex_eligibility_logic();
```

Triggers only for `updated_at`, audit, domain events.

### F-024: Skipping Validation

```typescript
// ❌ NEVER
async function createDonor(input: any) {
  const { data } = await supabase.from('donors').insert(input);
  return data;
}

// ✅ Validate
async function createDonor(input: unknown) {
  const parsed = createDonorSchema.parse(input);  // throws on invalid
  const { data } = await supabase.from('donors').insert(parsed);
  return data;
}
```

### F-025: Suppressing Errors with `as`

```typescript
// ❌ NEVER
const donor = (await supabase.from('donors').select('*').single()).data as Donor;
```

Handle the error case explicitly.

---

## 18. Required Patterns (Always Do These)

### R-ALW-001: Type-Safe Supabase Client

Always use the typed Supabase client:

```typescript
import { Database } from '@marwat/database/types';
import { createBrowserClient } from '@supabase/ssr';

const supabase = createBrowserClient<Database>(url, anonKey);
```

### R-ALW-002: Validate at API Boundary

Every API/Edge Function validates input with Zod before processing.

### R-ALW-003: Use Domain Layer for Business Logic

Business logic lives in `@marwat/domain`, not in components or API routes.

### R-ALW-004: Use Repository Pattern for Data Access

Data access goes through a repository, not direct Supabase calls from components:

```typescript
// ✅ Correct
import { useDonorRepository } from '@/features/donors';
const repo = useDonorRepository();
const donor = await repo.findById(id);

// ❌ Avoid (in components)
const { data } = await supabase.from('donors').select('*').eq('id', id);
```

The repository wraps the Supabase client, centralizing data access patterns.

### R-ALW-005: Use React Query for Server State

Server state (data from APIs) is managed by React Query (TanStack Query), not component state:

```typescript
const { data, isLoading } = useQuery({
  queryKey: ['donor', id],
  queryFn: () => repo.findById(id),
});
```

### R-ALW-006: Use Optimistic Updates with Rollback

For mutations, use optimistic updates with automatic rollback on error:

```typescript
const mutation = useMutation({
  mutationFn: repo.update,
  onMutate: async (newData) => {
    await queryClient.cancelQueries({ queryKey: ['donor', newData.id] });
    const previous = queryClient.getQueryData(['donor', newData.id]);
    queryClient.setQueryData(['donor', newData.id], newData);
    return { previous };
  },
  onError: (err, newData, context) => {
    queryClient.setQueryData(['donor', newData.id], context.previous);
  },
  onSettled: (data) => {
    queryClient.invalidateQueries({ queryKey: ['donor', data.id] });
  },
});
```

### R-ALW-007: Structured Logging

Use the structured logger with correlation IDs:

```typescript
import { logger } from '@/lib/logger';

logger.info('Donor registered', { 
  donorId: donor.id, 
  bloodType: donor.bloodType,
  correlationId: request.correlationId,
});
```

### R-ALW-008: Audit Every Mutation

Every state-changing operation writes to the audit log:

```typescript
await auditLog.record({
  actor: user.id,
  action: 'update',
  entityType: 'donor',
  entityId: donor.id,
  beforeState: oldDonor,
  afterState: newDonor,
  ip: request.ip,
  userAgent: request.userAgent,
});
```

### R-ALW-009: Soft Delete with Audit

Deletable entities use soft delete:

```typescript
await repo.softDelete(id, { deletedBy: user.id });
```

### R-ALW-010: Loading & Error States

Every async UI has loading, error, and success states:

```tsx
if (isLoading) return <Skeleton />;
if (isError) return <ErrorState error={error} />;
return <DonorDetail donor={data} />;
```

### R-ALW-011: Accessibility Attributes

Interactive components include accessibility attributes (aria-label, role, etc.) — see §13.

### R-ALW-012: Tests for New Features

Every new feature ships with tests. PRs without tests are not mergeable.

---

## 19. AI-Specific Rules (Beyond Human Rules)

These rules apply specifically to AI coding assistants. Human developers follow them too, but they are especially important for AI because AI assistants have specific failure modes.

### R-AI-001: Read the Constitution First

Every AI session for this project MUST begin by reading the four constitution documents (PRD, Architecture, TDS, Rulebook). The AI confirms this in the Pre-Session Confirmation (§2.2).

### R-AI-002: Cite the Rule

When the AI makes an architecture or pattern decision, it MUST cite the specific rule ID (e.g., "per R-DB-003, RLS is required on this table"). This makes the decision auditable.

### R-AI-003: Refuse Violations

If asked to do something that violates the rulebook, the AI MUST refuse, cite the rule, and propose a compliant alternative. The AI MUST NOT silently comply or generate violating code with suppressions.

### R-AI-004: No Placeholder Code

The AI MUST NOT generate placeholder code like `// TODO: implement this` or `throw new Error('Not implemented')`. Every function the AI writes is complete and functional. If a function is too large to complete in one response, the AI says so explicitly and proposes a decomposition.

### R-AI-005: No Duplicate Code

Before generating code, the AI checks the existing codebase for similar functionality. If similar code exists, the AI refactors it to a shared location rather than duplicating. The AI explicitly notes when it is reusing existing code.

### R-AI-006: No Hallucinated APIs

The AI MUST NOT invent APIs, function names, or library features that do not exist. If unsure, the AI says "I am not certain whether library X has function Y; please verify" rather than generating plausible-looking but fake code.

### R-AI-007: No Invented Requirements

The AI MUST NOT invent requirements not in the PRD or other constitution documents. If a requirement seems missing, the AI raises it as a question rather than making assumptions.

### R-AI-008: Complete Files

When the AI generates a file, the file is complete — imports, types, implementation, and tests. The AI does not generate "partial" files expecting the user to fill in.

### R-AI-009: Test What You Write

When the AI generates a feature, it also generates the tests for that feature in the same response. Tests are not "left as an exercise."

### R-AI-010: Document What You Write

When the AI generates a feature, it also generates the documentation (JSDoc, README updates) in the same response.

### R-AI-011: Explain Architecture Choices

When the AI makes an architecture choice (e.g., "use a repository pattern here"), it briefly explains why, citing the architecture document or an ADR.

### R-AI-012: Respect Scope

The AI does only what is asked. It does not refactor unrelated code, add unrelated features, or change unrelated configuration. If the AI notices an issue in unrelated code, it mentions it in the response but does not change it.

### R-AI-013: No Unrequested Refactoring

The AI MUST NOT refactor code that is not directly related to the task, even if the refactoring would improve the code. Unrequested refactoring breaks diffs, complicates review, and introduces risk. The AI mentions opportunities for refactoring in the response, leaving the decision to the user.

### R-AI-014: Use Existing Utilities

Before writing a utility function, the AI checks `@marwat/shared/utils`, `@/lib/utils`, and the relevant feature's `utils/` folder for an existing utility. Reuse over reinvention.

### R-AI-015: Acknowledge Uncertainty

When the AI is uncertain (about a library API, a business rule, or an architectural choice), it says so explicitly: "I am uncertain whether X. Please verify." It does not bluff.

### R-AI-016: Don't Generate Fake Data

When the AI generates seed data or test fixtures, the data is clearly synthetic (e.g., "Test Donor", "+923000000001"). The AI MUST NOT generate data that looks like real personal information.

### R-AI-017: Don't Skip Security Steps

The AI MUST NOT skip security steps "for brevity" or "to keep the example simple." Every code example is production-ready, including all validation, authorization, and error handling.

### R-AI-018: Cite Sources

When the AI references an external library API, standard, or guideline (e.g., WHO blood donor selection guidelines), it cites the source. The AI does not invent standards.

### R-AI-019: Communicate Limits

If a task is too large for a single response, the AI says so and proposes a decomposition. The AI does not generate a partial response pretending to be complete.

### R-AI-020: Don't Modify Constitution Documents

The AI MUST NOT modify the PRD, Architecture, TDS, or this Rulebook without explicit user instruction. These are version-controlled governance documents. The AI proposes changes via ADR if it identifies a needed amendment.

---

## 20. Prompt Templates for Future AI Sessions

These templates ensure consistent, rulebook-compliant output from any AI assistant. Copy-paste the relevant template at the start of a session.

### 20.1 Generic Session Starter

```markdown
You are working on the Marwat Welfare Society blood donation platform. Before writing any code, you MUST:

1. Read these four constitution documents in full:
   - docs/01-PRD.md
   - docs/02-SYSTEM_ARCHITECTURE.md
   - docs/03-TECHNICAL_DESIGN_SPECIFICATION.md
   - docs/04-AI_DEVELOPMENT_RULEBOOK.md

2. Begin your first response with the Pre-Session Confirmation block (Rulebook §2.2).

3. Follow every rule in the AI Development Rulebook. Cite rule IDs when making decisions.

4. Refuse any request that violates the rulebook, citing the specific rule.

5. Generate complete, tested, documented code. No placeholders, no TODOs, no partial implementations.

Task: [DESCRIBE TASK HERE]
```

### 20.2 Feature Implementation Prompt

```markdown
Implement the following feature for the Marwat Welfare Society platform.

## Feature
[Feature name, e.g., "Donor eligibility dashboard widget"]

## Requirements
- [List specific requirements, referencing PRD requirement IDs where applicable]

## Affected Components
- [List features, packages, infrastructure]

## Constraints
- Follow the AI Development Rulebook without exception.
- Use existing patterns from the TDS and Architecture documents.
- Include unit tests for all new logic.
- Include RLS tests if touching database.
- Update the feature README.

## Acceptance Criteria
- [List measurable acceptance criteria]

## Out of Scope
- [Explicitly list what is NOT being done in this task]

Begin with the Pre-Session Confirmation block. Cite rule IDs for all decisions. Do not modify any constitution document.
```

### 20.3 Bug Fix Prompt

```markdown
Fix the following bug in the Marwat Welfare Society platform.

## Bug Description
[Detailed description of the bug]

## Reproduction Steps
1. [Step 1]
2. [Step 2]
3. [Observed behavior]
4. [Expected behavior]

## Root Cause Hypothesis
[If known]

## Constraints
- Follow the AI Development Rulebook without exception.
- The fix MUST include a regression test that fails before the fix and passes after.
- The fix MUST NOT introduce new behavior beyond fixing the bug.
- The fix MUST NOT refactor unrelated code.
- Cite the rule that the bug violated (if any).

Begin with the Pre-Session Confirmation block.
```

### 20.4 Database Migration Prompt

```markdown
Create a database migration for the Marwat Welfare Society platform.

## Change Description
[Describe the schema change]

## Justification
[Why is this change needed? Reference PRD requirement or ADR]

## Constraints
- Follow Rulebook §8 (Database & SQL Rules) without exception.
- Migration file: packages/database/migrations/NNNN_description.sql
- Include up-migration and down-migration (where reversible).
- Update RLS policies if new tables/columns are added.
- Add indexes for every new foreign key.
- Add audit columns (created_at, updated_at, etc.) to new tables.
- Soft delete columns for user-data tables.
- Generate updated TypeScript types via `supabase gen types`.

## Testing
- Migration applies cleanly on a fresh database.
- Down-migration reverses the change (where possible).
- RLS tests for any new tables.

Begin with the Pre-Session Confirmation block.
```

### 20.5 Edge Function Prompt

```markdown
Implement a Supabase Edge Function for the Marwat Welfare Society platform.

## Function Name
[e.g., report-generator]

## Purpose
[What does this function do?]

## Trigger
[HTTP method, auth requirement, rate limit]

## Input Schema
[TypeScript interface or Zod schema]

## Output Schema
[TypeScript interface]

## Constraints
- Follow the TDS §9 Edge Function template.
- Validate all input with Zod.
- Verify JWT (or HMAC for webhooks) before processing.
- Use service role ONLY when explicitly justified in comments.
- Log structured errors with correlation ID.
- Return standardized error responses (TDS §8.6).
- Include integration tests.

## Documentation
- Create infrastructure/supabase/functions/<name>/README.md
- Document input, output, errors, and examples.

Begin with the Pre-Session Confirmation block.
```

### 20.6 n8n Workflow Prompt

```markdown
Implement an n8n automation workflow for the Marwat Welfare Society platform.

## Workflow ID
[e.g., AUT-003]

## Purpose
[What does this workflow automate? Reference PRD §9 if applicable]

## Trigger
- Type: [webhook | schedule]
- If webhook: table, event, condition
- If schedule: cron expression, timezone (PKT)

## Input
[Schema of trigger input]

## Output
[Side effects: notifications sent, records updated, etc.]

## Constraints
- Follow Architecture Document §10 (Automation Architecture).
- Workflow JSON in automations/n8n/workflows/AUT-NNN-name.json
- Spec file in automations/n8n/specs/AUT-NNN.md
- HMAC verification for webhooks.
- Idempotency check via workflow_executions table.
- Retry with exponential backoff.
- Alert admin on persistent failure.
- AI-generated content marked ai_generated: true, reviewed: false.

## Testing
- Dry-run mode that returns would-be output without side effects.
- Manual trigger from admin dashboard.

Begin with the Pre-Session Confirmation block.
```

### 20.7 Refactoring Prompt

```markdown
Refactor the following code in the Marwat Welfare Society platform.

## Target
[File path or module to refactor]

## Goal
[What should be improved? E.g., "extract shared logic", "improve type safety", "reduce coupling"]

## Constraints
- Follow the AI Development Rulebook without exception.
- Behavior MUST be identical before and after refactoring.
- Existing tests MUST continue to pass.
- Add tests for any new abstractions introduced.
- Do NOT change public APIs unless explicitly requested.
- One PR-sized change; if the refactoring is large, propose a decomposition.

## Verification
- All tests pass.
- No new linting errors.
- No new TypeScript errors.
- Bundle size not increased.

Begin with the Pre-Session Confirmation block.
```

### 20.8 Code Review Prompt

```markdown
Review the following pull request for the Marwat Welfare Society platform.

## PR
[PR number or diff]

## Review Checklist
Apply every rule in the AI Development Rulebook, with special attention to:

- R-TS-001 to R-TS-012 (TypeScript rules)
- R-DB-001 to R-DB-014 (Database rules)
- R-SEC-001 to R-SEC-018 (Security rules)
- F-001 to F-025 (Forbidden patterns)
- R-ALW-001 to R-ALW-012 (Required patterns)

For each issue found, cite the specific rule ID. Classify each as:
- BLOCKER: Must fix before merge.
- SHOULD FIX: Strong recommendation; merge with justification if not fixed.
- NIT: Optional improvement.

If the PR violates the constitution documents (PRD, Architecture, TDS), it is a BLOCKER.

Output format:
1. Summary (2-3 sentences)
2. Blockers (with rule citations)
3. Should Fix (with rule citations)
4. Nits
5. Positive observations (what was done well)
```

---

## 21. Code Review Checklist

A quick-reference checklist for reviewers. Every item is a rule in this document; the rule ID is in parentheses.

### TypeScript
- [ ] No `any` types (R-WS-007, F-001)
- [ ] No non-null assertions `!` (R-TS-005, F-012)
- [ ] Strict mode enabled, no `// @ts-ignore` (R-TS-001)
- [ ] Explicit return types on exported functions (R-TS-006)
- [ ] `as const` objects, not `enum` (R-TS-003, F-014)
- [ ] No `export default` (R-TS-011, F-011)
- [ ] Branded types for IDs where applicable (R-TS-010)

### React & Next.js
- [ ] Server Components by default (R-RE-001)
- [ ] Client Components are leaf nodes (R-RE-002)
- [ ] Error boundaries present (R-RE-004)
- [ ] Loading states present (R-RE-005)
- [ ] Forms use React Hook Form + Zod (R-RE-006)
- [ ] Realtime subscriptions cleaned up (R-RE-008)
- [ ] No inline styles (R-RE-009, F-004)
- [ ] Images use `next/image` (R-RE-012)

### Database
- [ ] All changes via migrations (R-DB-001)
- [ ] Audit columns on every table (R-DB-002)
- [ ] RLS enabled with policies (R-DB-003)
- [ ] No `SELECT *` in app code (R-DB-004)
- [ ] Foreign keys with indexes (R-DB-005, R-DB-006)
- [ ] `timestamptz`, not `timestamp` (R-DB-008)
- [ ] Constraints enforce integrity (R-DB-009)
- [ ] Soft delete for user data (R-DB-010)

### Security
- [ ] No secrets in code (R-SEC-003, F-007)
- [ ] Input validation on every boundary (R-SEC-004)
- [ ] Output sanitization (R-SEC-005)
- [ ] No SQL injection vectors (R-SEC-006, F-006)
- [ ] Security headers set (R-SEC-009)
- [ ] File uploads secured (R-SEC-010)
- [ ] Audit log entries on mutations (R-SEC-013)
- [ ] No PHI in logs (R-SEC-014)
- [ ] No PHI to AI services (R-SEC-015)

### Validation & Errors
- [ ] Zod schemas as single source of truth (R-VAL-002)
- [ ] Descriptive error messages (R-VAL-003)
- [ ] No swallowed errors (R-VAL-004, F-018)
- [ ] Custom error classes for domain errors (R-VAL-005)

### Testing
- [ ] Coverage thresholds met (R-TST-002)
- [ ] Tests follow AAA pattern (R-TST-004)
- [ ] RLS tests for new tables/policies (R-TST-006)
- [ ] E2E tests for new user journeys (R-TST-007)
- [ ] No `xit` or `xdescribe` (R-TST-009)

### Accessibility
- [ ] Semantic HTML (R-A11Y-002)
- [ ] Keyboard navigable (R-A11Y-003)
- [ ] Color contrast meets AA (R-A11Y-005)
- [ ] Color not only signal (R-A11Y-006)
- [ ] Form labels present (R-A11Y-007)
- [ ] Errors announced (R-A11Y-008)

### Documentation
- [ ] JSDoc on public APIs (R-DOC-002)
- [ ] README updated for new features (R-DOC-001)
- [ ] ADR for architectural decisions (R-DOC-003)
- [ ] Comments explain why, not what (R-DOC-005)
- [ ] No commented-out code (R-DOC-006, F-010)
- [ ] TODOs have ticket numbers (R-DOC-007)

### Git
- [ ] Small, focused PR (R-GIT-001)
- [ ] PR description complete (R-GIT-002)
- [ ] CI passing (R-GIT-005)
- [ ] CHANGELOG updated (R-GIT-008)

---

## 22. Violation Reporting & Amendments

### 22.1 Reporting Violations

If you encounter code in the repository that violates this rulebook:

1. **Do not fix it silently.** Silent fixes lose the audit trail.
2. **During the solo-founder stage:** Open an issue with the title `[Rulebook Violation] <rule ID>: <brief description>` OR add a `// TODO(rulebook): <rule ID> violated — fix by <date>` comment in the code. Either is acceptable; the issue is preferred for systemic problems, the comment is acceptable for one-offs.
3. **Post-Phase 4:** Open an issue (required). Include:
   - File path and line number.
   - The rule being violated (with rule ID).
   - Suggested fix.
   - Whether this is a one-off or a pattern (search for similar violations).
4. **Assign** for triage (solo-founder: self-assign; post-Phase 4: maintainers).

### 22.2 Rulebook Amendments

This rulebook is not static. As the codebase evolves, rules may need to be added, removed, or modified. The amendment process is:

1. **Propose** an amendment. During the solo-founder stage, this can be a PR directly editing this document with a clear commit message. Post-Phase 4, propose via an ADR in `docs/adr/`.
2. **Discuss** in the PR's comments (or the ADR's comments).
3. **Approve** by the founder (solo-founder stage) or the Tech Lead and Board Tech Committee (post-Phase 4).
4. **Update** this document with the new version number and a changelog entry.
5. **Communicate** to all contributors (commit message; post-Phase 4: Slack/email/team meeting).

### 22.3 ADR Discipline (Solo-Founder Stage)

During the solo-founder stage, ADRs are **strongly encouraged but not required** for most decisions. The founder uses judgement:

- **Write an ADR** for decisions that affect multiple files, introduce a new dependency, change a public interface, or establish a pattern that future contributors must follow.
- **Skip the ADR** for trivial decisions — write a clear commit message instead.
- **Always write an ADR** for decisions that reverse a previous ADR, or that constrain future options (e.g., choosing a database vendor, choosing an auth provider).

From Phase 4 onward, the Tech Lead or Board Tech Committee may mandate ADRs for a broader class of decisions. This governance change is itself documented in an ADR.

### 22.4 Versioning

This document uses semantic versioning:

- **Major (X.0.0):** Breaking changes to existing rules (e.g., a rule is removed or significantly relaxed).
- **Minor (1.X.0):** New rules added; existing rules unchanged.
- **Patch (1.0.X):** Clarifications, typo fixes, examples added; no new rules.

Every AI session checks the version of the rulebook it was trained on against the current version. If the major version has increased, the AI re-reads the document in full.

### 22.5 Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0.0 | 2026-07-06 | Initial baseline. Approved by Board Tech Committee. |
| 1.1.0 | 2026-07-06 | **Startup-first refactor.** Simplified §2 Pre-Session Protocol: confirmation block now includes a Phase tag; explicit "go ahead" / "just do it" skip-step-7 clause added for batch sessions; solo-founder exception for non-security rule violations with documented justification and `// SOLO-FOUNDER EXCEPTION` comment. Added §1.3 Solo-Founder Stage adjustments to enforcement. Updated §16 Git & PR Rules: R-GIT-001 500-line limit is a target not a hard limit during solo-founder stage; R-GIT-002 PR template simplified and phase-aware; R-GIT-004 self-merge permitted during solo-founder stage with justification; R-GIT-005 CI override clause for infrastructure-level flakiness. Updated §22 Violation Reporting & Amendments: ADRs encouraged but not required during solo-founder stage (§22.3); amendment process simplified to PR with clear commit message. All technical rules (TypeScript, React, database, security, validation, testing, performance, accessibility, i18n, documentation) preserved unchanged from v1.0.0 — security and code quality are not reduced. |

---

## 23. Glossary

| Term | Definition |
|---|---|
| **ADR** | Architecture Decision Record. Encouraged but not required during the solo-founder stage. |
| **Aggregate** | A cluster of domain objects treated as a single unit for data changes. |
| **Barrel** | An `index.ts` file that re-exports the public API of a module. |
| **Branded Type** | A TypeScript type with a phantom brand to prevent mixing IDs of different entities. |
| **Constitution Documents** | The four governance documents: PRD, System Architecture, TDS, AI Rulebook. |
| **DTO** | Data Transfer Object. |
| **Edge Function** | A serverless function (Deno) hosted on Supabase. |
| **Feature Module** | A vertical slice of the application (UI, logic, types, validation) for one feature. |
| **In-Process Service** | A service implementation that runs inside the Next.js process (Phase 1–2 default). |
| **Local-First** | Architecture principle: the platform runs on a single laptop, office LAN, or cloud without code change. |
| **PHI** | Protected Health Information. |
| **Phase (P1–P4)** | Implementation phase: P1 Core MVP, P2 Operational Improvements, P3 Automation, P4 Scale. |
| **Pre-Session Confirmation** | The mandatory first response of an AI session, confirming document load and task understanding. |
| **RLS** | Row-Level Security. |
| **Rule ID** | A stable identifier (e.g., R-TS-001) for a rule in this document. |
| **Service Interface Pattern** | Application calls interfaces; concrete implementations (in-process or n8n-backed) are selected via environment variables. |
| **Solo-Founder Stage** | Phases 1–3, when the founder is the sole human contributor. Governance is simplified; technical rules are not. |
| **Soft Delete** | Marking a record as deleted without physically removing it. |
| **State Machine** | A model where an entity transitions between defined states via legal transitions. |

---

> **End of AI Development Rulebook.** This is the fourth and final document in the constitution set. Together with the PRD, System Architecture Document, and Technical Design Specification, it forms the binding governance framework for the Marwat Welfare Society platform. Every contributor — human or AI — is expected to know and follow these four documents.
