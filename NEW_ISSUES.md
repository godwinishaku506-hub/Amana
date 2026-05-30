# New Issues — Loose Ends Analysis

Date: 2026-05-30
Context: Systematic analysis of Amana monorepo against README project description and codebase health. Identifies gaps not already covered in SMART_CONTRACT_ISSUES.md, FRONTEND_REFACTORING_ISSUES.md, AMANA_GITHUB_ISSUES.md, or ISSUE_CONTEXT.md.

---

## INFRA-001 — Orphaned Root-Level `src/` Directory Duplicates Frontend Code

**Description:**
A root-level `src/` directory (`/src`) contains duplicate copies of frontend files:
- `src/app/mediator/disputes/[id]/page.tsx` — duplicates `frontend/src/app/mediator/disputes/[id]/page.tsx`
- `src/app/mediator/disputes/[id]/MediatorPanelClient.tsx` — duplicates `frontend/src/...`
- `src/components/ui/VideoUploadCard.tsx` — duplicates `frontend/src/components/ui/VideoUploadCard.tsx`

This is orphaned/stale code that shadows actual frontend files. It creates confusion about which file is canonical and could cause maintenance divergence.

**Location:** `/src/` (repository root)

**Recommendation:** Remove the entire root `src/` directory. All content is duplicated in `frontend/src/`.

**Priority:** High

---

## INFRA-002 — Conflicting Jest Config Files in Backend

**Description:**
Backend has both `jest.config.js` AND `jest.config.ts`. The `package.json` `test` script references `jest.config.js`, but the `.ts` variant exists alongside. This is ambiguous: are tests supposed to use the JS or TS config? If they differ, behavior is undefined.

**Location:** `backend/jest.config.js`, `backend/jest.config.ts`

**Recommendation:** Remove the unused config file and consolidate to a single source of truth.

**Priority:** Medium

---

## INFRA-003 — Two Competing Error Handlers in Backend

**Description:**
Backend has two error handler implementations:
1. `backend/src/errors/errorHandler.ts` — AppError-aware with Zod handling
2. `backend/src/middleware/errorHandler.ts` — Simpler handler with correlation ID support

`app.ts` imports from `./middleware/errorHandler`, so #2 is the active one. But #1 still exists with its own logic, potentially causing confusion about which handler patterns to follow when adding new routes.

**Location:** `backend/src/errors/errorHandler.ts`, `backend/src/middleware/errorHandler.ts`

**Recommendation:** Consolidate into a single canonical error handler and remove the unused one. Move any unique capabilities (Zod handling, correlation IDs) into the canonical version.

**Priority:** Medium

---

## INFRA-004 — 130 Pre-existing Backend Test Failures Make CI Unreliable

**Description:**
COMPLETION_SUMMARY.md reports 485 total backend tests with 130 failures across 22 test suites. Root causes include:
- Vitest/Jest mismatch imports (`vi` vs `jest`, `describe`/`it` conflicts)
- Migration discovery path resolution issues
- Stellar config mocking issues
- Trade route validation failures
- Auth middleware test failures

These pre-existing failures mean CI's backend test gate is effectively broken — failures are expected and ignored, reducing signal quality.

**Location:** `backend/src/**/__tests__/` (22 failing test suites)

**Recommendation:** Triage and fix pre-existing test failures. Prioritize fixing the runner mismatch (Jest vs Vitest), then address the remaining failures.

**Priority:** Critical

---

## INFRA-005 — Working Tree Has 28 Uncommitted Modified Files

**Description:**
Git status shows 28 modified files across backend, frontend, and mobile — plus 6 untracked files. These are uncommitted changes that clutter the working tree and make it impossible to distinguish between intentional work and accidental modifications.

**Affected areas:** backend (2 files), frontend (25 files), mobile (7 files incl untracked)

**Recommendation:** Either commit these changes to a feature branch or restore them. The working tree should be clean on `main`.

**Priority:** High

---

## INFRA-006 — Stale Tracking Documents Clutter Repository Root

**Description:**
The repository root has 15+ markdown tracking documents, many of which describe completed work or historical PRs:
- `TODO.md` — Wallet auth feature (all steps completed)
- `TODO-Bootstrap.md` — Backend bootstrap (stale, partially done)
- `ISSUE_CONTEXT.md` — Copilot prompt for 4 resolved issues
- `COMPLETION_SUMMARY.md` — Backend hardening done report
- `BACKEND_HARDENING_PR.md` — PR description for merged hardening work
- `PR_DESCRIPTION.md` — PR description for 4 issues
- `issue.md` — Single issue template for #351
- `pr.md` — Single PR description for #351
- `COMPLETED` — Empty flag file
- `TEST_COVERAGE_MATRIX.md` — Partially overlaps with TESTING.md

This makes it hard for new contributors to find relevant documentation.

**Recommendation:** Archive stale/completed tracking docs into an `archive/` directory. Keep only `README.md`, `DISTRIBUTED_TRACING_GUIDE.md`, `TESTING.md`, and the active issue documents (`FRONTEND_REFACTORING_ISSUES.md`, `SMART_CONTRACT_ISSUES.md`, `AMANA_GITHUB_ISSUES.md`) at root.

**Priority:** Medium

---

## BACKEND-001 — Auth Service Has Zero Test Coverage (Critical Path)

**Description:**
Per TEST_COVERAGE_MATRIX.md, `auth.service.ts`, `auth.routes.ts`, and `auth.middleware.ts` have zero test coverage. The auth service handles Stellar wallet verification, challenge/response, and JWT issuance — it is the authentication backbone of the entire platform. Any regression here breaks all authenticated flows.

**Location:**
- `backend/src/services/auth.service.ts`
- `backend/src/routes/auth.routes.ts`
- `backend/src/middleware/auth.middleware.ts`

**Priority:** Critical

---

## BACKEND-002 — Backend Bootstrap Incomplete (TODO-Bootstrap.md Steps 3-8 Pending)

**Description:**
TODO-Bootstrap.md tracks 8 bootstrap steps. Steps 3–8 remain pending:
- Step 3: Create `middleware/logger.ts` (pino-http)
- Step 4: Create `middleware/errorHandler.ts`
- Step 5: Edit `app.ts` (add middlewares)
- Step 6: Update `.env.example`
- Step 7: Create `__tests__/app.test.ts`
- Step 8: Update `index.ts` (env validation)

Some of these may already exist in partial form.

**Location:** `backend/src/`

**Priority:** High

---

## BACKEND-003 — User Controller and Routes Have Zero Test Coverage

**Description:**
`user.service.ts`, `user.controller.ts`, `user.routes.ts`, and `user.validators.ts` have zero test coverage. These handle user profile management, a core domain entity.

**Location:** `backend/src/controllers/user.controller.ts`, `backend/src/services/user.service.ts`, `backend/src/routes/user.routes.ts`, `backend/src/validators/user.validators.ts`

**Priority:** High

---

## BACKEND-004 — Contract Service Has Zero Test Coverage

**Description:**
`contract.service.ts` has zero test coverage. This service mediates between the backend API and the Soroban smart contract — it's the bridge between off-chain and on-chain logic.

**Location:** `backend/src/services/contract.service.ts`

**Priority:** High

---

## FRONTEND-001 — No E2E Tests Between Frontend and Backend

**Description:**
Frontend has unit tests (Jest), visual regression tests (Playwright), and 1 e2e test file (`evidence.e2e.test.tsx`). There are no end-to-end tests that exercise the full frontend-to-backend flow (auth → create trade → fund → confirm delivery → complete). Critical user journeys have no integration safety net.

**Location:** `frontend/src/__tests__/` (1 incomplete e2e test)

**Recommendation:** Add Playwright e2e tests covering critical user journeys with the backend running in test mode.

**Priority:** High

---

## FRONTEND-002 — Root Landing Page Uses Template/Demo Content

**Description:**
The root page (`/`) at `frontend/src/app/page.tsx` still uses template/demo content from the Next.js scaffold rather than product-aligned content. This contradicts FRONTEND_REFACTORING_ISSUES.md FE-REF-007 which calls for replacing this content.

**Note:** This is partially addressed by existing issue FE-REF-007 but flagged here as a critical visible gap for any visitor.

**Location:** `frontend/src/app/page.tsx`

**Priority:** High

---

## MOBILE-001 — Mobile App Is a Skeleton With Only 1 Screen

**Description:**
The mobile app (React Native/Expo) consists of only:
- `App.tsx` — Stack navigator with 1 screen
- `HomeScreen.tsx` — Single screen with basic placeholder text
- `api/auth.ts` + `api/client.ts` — API scaffolding
- `stores/authStore.ts` — Auth state store

README describes mobile features including: trade discovery, wallet connection, evidence capture, push notifications, and trade status updates. **None of these are implemented.**

**Location:** `mobile/`

**Priority:** High

---

## MOBILE-002 — Mobile App Has No Test Framework Configured

**Description:**
The mobile stack has:
- No test framework installed (no Jest, no React Native Testing Library)
- No test files anywhere
- No test script in `package.json`

Despite CI having a mobile test gate placeholder, no actual test execution happens.

**Location:** `mobile/package.json`

**Priority:** Medium

---

## MOBILE-003 — Duplicate ESLint Configurations in Mobile

**Description:**
Mobile has both an existing `.eslintrc.js` (legacy format) AND an untracked `eslint.config.mjs` (flat config format v9) that was added but not yet tracked. Having two competing ESLint configs means linting behavior is ambiguous.

**Location:** `mobile/.eslintrc.js`, `mobile/eslint.config.mjs`

**Priority:** Low

---

## DOCS-001 — README Roadmap Checkboxes Are All Unchecked Despite Many Features Implemented

**Description:**
README.md Phase 1–4 roadmap checkboxes are all `[ ]` (unchecked), even though:
- Phase 1: Core Soroban contract logic (`deposit`, `release`, `refund`) IS implemented in `lib.rs`
- Phase 1: Next.js UI for trade creation DOES exist
- Phase 2: `Loss_Ratio` variables ARE integrated into the contract
- Phase 2: Mediator dashboard DOES exist
- Phase 3: IPFS integration IS implemented
- Phase 3: Driver manifest logging EXISTS

This is misleading for contributors trying to understand project maturity.

**Location:** `README.md` (lines 119-137)

**Priority:** Medium

---

## DOCS-002 — README Claims "Mandatory" Video Verification But Contract Doesn't Enforce It

**Description:**
README states: "Proof-of-Delivery (PoD): A mandatory video-based verification protocol involving the buyer and the driver to confirm the state of goods."

However, the contract's `confirm_delivery()` function can be called without requiring `submit_video_proof()` first. The video proof is optional in the actual implementation.

**Note:** This is partially covered by SMART_CONTRACT_ISSUES.md SC-001 but flagged here as a documentation gap too.

**Location:** `README.md` (line 21), `contracts/amana_escrow/src/lib.rs`

**Priority:** Medium

---

## DOCS-003 — Mobile README Promises Features That Don't Exist

**Description:**
`mobile/README.md` likely describes features (trade discovery, evidence capture, push notifications) that have no implementation. This misleads mobile contributors about the app's maturity.

**Location:** `mobile/README.md`

**Priority:** Medium

---

## TEST-001 — Critical User Flow Has No E2E Coverage

**Description:**
The entire happy path flow — authenticate wallet → create trade → fund trade → confirm delivery → complete settlement — has zero end-to-end test coverage. Only isolated unit tests exist per service. This means regressions in cross-service orchestration are undetectable until production.

**Affected flows:**
- Wallet auth → JWT issue → authenticated API calls
- Trade create → fund → status transitions
- Evidence upload → IPFS → delivery confirmation
- Dispute → mediator resolution → settlement

**Priority:** Critical

---

## TEST-002 — Stellar Service and Path Payment Service Untested

**Description:**
`stellar.service.ts` and `pathPayment.service.ts` both have zero test coverage. These handle Stellar blockchain interactions (path payments, account management) — core financial operations where bugs cost real money.

**Location:** `backend/src/services/stellar.service.ts`, `backend/src/services/pathPayment.service.ts`

**Priority:** Critical

---

## Prioritization Summary

| Priority | Issue | Title |
|----------|-------|-------|
| Critical | INFRA-004 | 130 Pre-existing Backend Test Failures |
| Critical | BACKEND-001 | Auth Service Zero Test Coverage |
| Critical | TEST-001 | No E2E Coverage for Critical User Flows |
| Critical | TEST-002 | Stellar and PathPayment Services Untested |
| High | INFRA-001 | Orphaned Root-Level src/ Directory |
| High | INFRA-005 | 28 Uncommitted Modified Files |
| High | BACKEND-002 | Backend Bootstrap Incomplete |
| High | BACKEND-003 | User Controller Untested |
| High | BACKEND-004 | Contract Service Untested |
| High | FRONTEND-001 | No Frontend-Backend E2E Tests |
| High | FRONTEND-002 | Root Page Uses Template Content |
| High | MOBILE-001 | Mobile App Is a Skeleton |
| Medium | INFRA-002 | Conflicting Jest Configs |
| Medium | INFRA-003 | Two Competing Error Handlers |
| Medium | INFRA-006 | Stale Tracking Documents Clutter Root |
| Medium | MOBILE-002 | Mobile Has No Test Framework |
| Medium | DOCS-001 | README Roadmap Checkboxes All Unchecked |
| Medium | DOCS-002 | README Claims Mandatory Video Verification |
| Medium | DOCS-003 | Mobile README Overpromises |
| Low | MOBILE-003 | Duplicate ESLint Configs in Mobile |
