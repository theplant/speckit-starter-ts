<!--
Sync Impact Report:
- Version change: 1.4.0 → 1.5.0
- Modified principles:
  - V. E2E Testing Discipline → I. E2E Testing Discipline (moved to first position)
  - I. API-First Contract + II. Type-Safe Client → II. API-First Contract & Type-Safe Client (merged)
  - III. Component-Driven UI → III. Component-Driven UI (renumbered)
  - IV. Query-Centric Data → IV. Query-Centric Data (renumbered)
  - VII. Simplicity → V. Simplicity (renumbered)
- Added sections: None
- Removed sections: None (Type-Safe Client merged into API-First Contract)
- Templates requiring updates: ✅ plan-template.md, ✅ spec-template.md, ✅ tasks-template.md (no changes needed)
- Follow-up TODOs: None
-->

# Frontend Console Constitution

## Core Principles

### I. E2E Testing Discipline (NON-NEGOTIABLE)

All testing MUST be done exclusively with Playwright end-to-end tests against the real backend API.

**Testing Approach**
- Every feature MUST have corresponding E2E tests before it is considered complete
- Tests MUST simulate real user behavior through the browser
- Tests MUST cover the full user journey, not isolated components
- No unit tests, no integration tests in isolation

**Coverage Requirements**
- All routes (public, authenticated, error) MUST have E2E test coverage
- All user interactions (buttons, forms, modals, navigation, CRUD) MUST be tested
- All loading states and error states MUST be verified
- Route parameters and query strings MUST be tested for edge cases

**Real Backend Only**
- Tests MUST run against the real backend API - mocking is strictly prohibited
- Tests MUST NOT use `page.route()` or any request interception to fake responses
- If the backend is not running, tests MUST fail clearly

**Test Independence**
- Tests MUST NOT depend on execution order of other tests
- Tests MUST clean up any data they create (or use isolated test data)
- Tests MUST be able to run in parallel without conflicts

**AI-Driven Execution**
- Tests MUST run non-interactively without waiting for human review
- The test-fix cycle MUST be autonomous: run tests → read errors → fix code → re-run
- Playwright MUST use only the `'list'` reporter for clean, parseable AI output

**Error Debugging Protocol**
- On test failure, capture HTTP request as curl command: `tests/debug/{test-name}-{timestamp}-request.sh`
- On test failure, capture HTTP response: `tests/debug/{test-name}-{timestamp}-response.txt`
- Debug files MUST be gitignored but directory structure MUST exist

**Console Error Capture (NON-NEGOTIABLE)**
- All browser console errors MUST be captured and output during test execution
- React/JavaScript errors (e.g., context errors, runtime exceptions) MUST be visible in test output
- Tests MUST listen to `page.on('console')` and `page.on('pageerror')` events
- Console errors MUST be printed immediately when they occur, not just on test failure
- This ensures application bugs (like missing context providers) are immediately visible

**Configuration**
- Playwright MUST only include the `chromium` project (no Firefox/WebKit)
- Test files MUST be in `tests/e2e/` with naming `{route-or-feature}.spec.ts`
- Page objects MUST be used for reusable page interactions
- Test utilities MUST be in `tests/e2e/utils/`

**Quality Gates**
- All E2E tests MUST pass before merge
- New routes/interactions MUST have corresponding E2E tests
- No flaky tests allowed - tests MUST be deterministic

### II. API-First Contract & Type-Safe Client

The backend API is the product. The frontend MUST treat the API as the single source of truth, with full type safety from contract to component.

**API Contract Rules**
- Frontend MUST NOT implement business logic that belongs in the backend
- All data mutations MUST go through API endpoints
- Frontend MUST handle API errors gracefully with user-friendly messages
- API contract changes MUST be coordinated between frontend and backend teams
- Use `Content-Type: application/json` for all API communication (protojson encoding)
- All endpoints versioned under `/api/v1/`

**Type Safety Requirements**
- Generate TypeScript types from backend protobuf using `protoc-gen-ts` (ts-proto)
- Types MUST be generated BEFORE any implementation begins
- Generated types live in `src/api/generated/` directory
- NEVER use `any` type for API responses or requests
- NEVER write manual type definitions that duplicate protobuf contracts
- API client functions MUST return typed responses using generated types
- Use discriminated unions for API response states (loading, error, success)
- Type generation MUST be automated via `pnpm run generate:types` script

**Type Generation Command:**
```bash
# Generate types from backend proto files
protoc --plugin=./node_modules/.bin/protoc-gen-ts_proto \
  --ts_proto_out=./src/api/generated \
  --ts_proto_opt=outputServices=false \
  --ts_proto_opt=esModuleInterop=true \
  --ts_proto_opt=snakeToCamel=true \
  -I ./backend/api/proto \
  ./backend/api/proto/[PROJECT]/v1/*.proto
```

### III. Component-Driven UI

Build UI as a composition of reusable, isolated components.

- MUST use shadcn/ui as the component library foundation
- Components MUST be stateless where possible; lift state up
- Styling MUST use utility-first CSS (e.g., Tailwind CSS)
- Each component MUST have a single responsibility
- Prefer composition over prop drilling; use context sparingly

### IV. Query-Centric Data

Use a query library (e.g., TanStack Query) as the data layer.

- All API calls MUST go through query hooks (`useQuery`, `useMutation`)
- Cache invalidation MUST be explicit and intentional
- Optimistic updates SHOULD be used for better UX where safe
- Query keys MUST be consistent and predictable
- Avoid global state; prefer server state via query cache
- Minimal client state; use React state for UI-only concerns

### V. Simplicity

Start simple. Add complexity only when proven necessary.

- YAGNI: Do not build features "just in case"
- Prefer fewer dependencies over many
- Avoid premature abstraction; wait for patterns to emerge
- Configuration over code where possible
- Delete code that is not used

## Technology Stack

**Core Framework:**
- TypeScript (strict mode enabled)
- React with functional components and hooks
- Vite for build tooling

**Data Layer:**
- TanStack Query (React Query) for server state
- Minimal global state (React Context or Zustand if needed)

**UI Layer:**
- Tailwind CSS for styling
- shadcn/ui components
- Lucide for icons

**Type Generation:**
- protoc-gen-ts for protobuf contracts

**Testing:**
- Vitest + Testing Library for component tests
- Playwright for E2E tests
- NO unit tests; NO API integration tests (backend tests its own contracts)
- Component tests: fast, mock hooks, test every component
- E2E tests: real backend, test all user journeys

**Routing:**
- React Router
- Route structure mirrors resource hierarchy:
  - `/projects/:id/users`
  - `/projects/:id/segments`
  - `/projects/:id/campaigns`
  - etc.

## Development Workflow

### Implementation Order (MANDATORY)

For each user story, tasks MUST be executed in this exact order:

1. **Generate Types**: Run `pnpm run generate:types` from backend proto
2. **Write Component Tests**: Create failing tests in `tests/components/`
3. **Write E2E Tests**: Create failing tests in `tests/e2e/`
4. **Verify Tests Fail**: Run `pnpm test` and `pnpm test:e2e` - all new tests MUST fail
5. **Implement Hooks**: Create query/mutation hooks using generated types
6. **Implement Components**: Build UI components consuming hooks
7. **Verify Tests Pass**: Run full test suite - all tests MUST pass
8. **Refactor**: Clean up code while keeping tests green

**⚠️ VIOLATION**: Implementing code before tests exist is a constitution violation.

### API Integration Flow

1. **Contract First**: Obtain API contract (protobuf)
2. **Generate Types**: Run `pnpm run generate:types`
3. **Write Tests**: Component tests + E2E tests that FAIL
4. **Create Hook**: Implement query/mutation hook with generated types
5. **Build Component**: Consume hook in UI component
6. **Verify**: Run full test suite - all tests MUST pass

### Code Organization

```
src/
├── api/           # Generated types and API client
├── hooks/         # Query hooks (useUsers, useProjects, etc.)
├── components/    # Reusable UI components
├── pages/         # Route-level components
├── lib/           # Utilities and helpers
└── tests/
    └── e2e/           # End-to-end user journey tests (Playwright)

```

### Quality Gates

- All tests MUST pass before merge
- Type checking MUST pass (`tsc --noEmit`)
- Linting MUST pass (ESLint)
- Format MUST be consistent (Prettier)
- No `any` types without explicit justification
- Component tests required for every new component

## Backend API Reference

### API Conventions

- **Base Path**: `/api/v1/`
- **Content-Type**: `application/json` (protojson encoding)
- **Project Scope**: Include `project_id` header for all project-scoped calls
- **Health Check**: `GET /health` returns `{"status":"ok"}`





## Governance

This constitution supersedes all other development practices for frontend projects.

**Amendment Process:**
1. Propose change with rationale
2. Review impact on existing code
3. Update constitution with version bump
4. Communicate changes to team

**Compliance:**
- All PRs MUST verify compliance with these principles
- Complexity MUST be justified in PR description
- Violations require explicit exception with documented reasoning

**Version Policy:**
- MAJOR: Principle removal or fundamental change
- MINOR: New principle or significant guidance addition
- PATCH: Clarifications and minor refinements

**Version**: 1.5.0 | **Ratified**: 2025-12-09 | **Last Amended**: 2025-12-11
