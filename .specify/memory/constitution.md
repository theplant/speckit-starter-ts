<!--
Sync Impact Report:
- Version change: 1.3.0 → 1.4.0
- Modified principles:
  - III. Component-Driven UI: Added MANDATORY shadcn/ui requirement
  - V. Test-First: Replaced integration tests with component tests
  - VI. Integration Testing: REMOVED (backend tests API contracts)
- Added sections:
  - Component Testing Requirements (under V. Test-First)
- Removed sections:
  - VI. Integration Testing (entire section removed)
- Templates requiring updates: ✅ tasks-template.md, ✅ plan-template.md
- Follow-up TODOs: Remove tests/integration/ directory and MSW mocks
-->

# Frontend Console Constitution

## Core Principles

### I. API-First Contract

The backend API is the product. The frontend MUST treat the API as the single source of truth.

- Frontend MUST NOT implement business logic that belongs in the backend
- All data mutations MUST go through API endpoints
- Frontend MUST handle API errors gracefully with user-friendly messages
- API contract changes MUST be coordinated between frontend and backend teams
- Use `Content-Type: application/json` for all API communication (protojson encoding)
- All endpoints versioned under `/api/v1/`

### II. Type-Safe Client

All API communication MUST be fully typed from contract to component.

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

### V. Test-First (NON-NEGOTIABLE)

Tests MUST be written before implementation. This is a BLOCKING requirement.

**Implementation is BLOCKED until:**
1. Component tests exist and FAIL (red phase)
2. E2E tests exist and FAIL (red phase)
3. Generated types from protobuf are available

**Test-First Workflow:**
- Red-Green-Refactor cycle strictly enforced
- Tests MUST fail before implementation begins
- Each user story MUST have acceptance tests defined upfront
- NO unit tests; NO API integration tests (backend tests its own contracts)
- Component tests for every UI component
- E2E tests for all user journeys with real backend

**Component Test Requirements:**
- EVERY component MUST have a corresponding test file
- Test all rendering states (loading, error, success, empty)
- Test user interactions (clicks, form inputs, keyboard)
- Test conditional rendering and edge cases
- Mock query hooks at the hook level, NOT HTTP requests
- Use Testing Library queries (getByRole, getByText) for accessibility
- NO snapshot testing

**Component Test Structure:**
```typescript
// tests/components/UserCard.test.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { UserCard } from '@/components/UserCard'

describe('UserCard', () => {
  it('renders user name', () => { ... })
  it('shows loading state', () => { ... })
  it('handles click action', () => { ... })
})
```

**E2E Test Requirements (per user journey):**
- Test the complete user journey from start to finish
- Test against REAL backend (test environment)
- Test navigation between pages
- Test form submissions and validations
- Test error recovery flows
- ALL user journeys MUST have E2E coverage

### VII. Simplicity

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
    ├── components/    # Component tests (Vitest + Testing Library)
    └── e2e/           # End-to-end user journey tests (Playwright)

backend/                   # Symlinked Go backend
├── api/
│   ├── proto/[PROJECT]/v1/       # Protobuf definitions (source of truth)
├── handlers/
│   ├── routes.go          # All API endpoint definitions
│   └── error_codes.go     # Structured error responses
└── Makefile               # proto, test, run, migrate commands
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

### Error Response Format

All errors return structured JSON:
```json
{
  "code": "ERROR_CODE",
  "message": "Human-readable message"
}
```

Common error codes:
- `INVALID_REQUEST` (400)
- `MISSING_PROJECT_ID` (400)
- `NOT_FOUND` (404)
- `DUPLICATE` (409)
- `INTERNAL_ERROR` (500)



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

**Version**: 1.4.0 | **Ratified**: 2025-12-09 | **Last Amended**: 2025-12-09
