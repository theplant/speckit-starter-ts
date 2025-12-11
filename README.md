# Speckit Starter for TypeScript Frontend

A comprehensive template for TypeScript frontend projects with E2E testing, type-safe API contracts, and modern React patterns.

## Overview

This repository provides a battle-tested constitution and templates for building production-ready TypeScript frontends with:

- ✅ **E2E Testing First** - Playwright tests against real backend, no mocking
- ✅ **Type-Safe API Contracts** - Generated TypeScript types from protobuf
- ✅ **Component-Driven UI** - shadcn/ui + Tailwind CSS
- ✅ **Query-Centric Data** - TanStack Query for server state
- ✅ **AI-Driven Development** - Autonomous test-fix cycles

## Quick Start

### Step 1: Initialize Your Project with Spec-Kit

```bash
# Create and initialize your new project
uvx --from git+https://github.com/github/spec-kit.git specify init my-frontend-app

# Navigate to your project
cd my-frontend-app
```

### Step 2: Overlay the TypeScript Template

```bash
git clone --depth 1 git@github.com:theplant/speckit-starter-ts.git /tmp/speckit-starter-ts-$$ && cp -r /tmp/speckit-starter-ts-$$/.specify . && rm -rf /tmp/speckit-starter-ts-$$
```

### Step 3: Review and Customize

```bash
# Read the constitution for detailed principles
cat .specify/memory/constitution.md

# Review the templates
ls -la .specify/templates/
```

## What You Get

### Constitution (Version 1.5.0)

See [`.specify/memory/constitution.md`](.specify/memory/constitution.md) for the complete set of principles.

### Templates

- `spec-template.md` - Feature specification with user stories
- `plan-template.md` - Implementation planning
- `tasks-template.md` - Task breakdown
- `checklist-template.md` - Development checklist

## Technology Stack

- **Language**: TypeScript (strict mode)
- **Framework**: React with functional components and hooks
- **Build Tool**: Vite
- **Data Layer**: TanStack Query (React Query)
- **UI**: Tailwind CSS + shadcn/ui + Lucide icons
- **Type Generation**: protoc-gen-ts (ts-proto)
- **Testing**: Playwright E2E (real backend only)
- **Routing**: React Router

## Project Structure

```
my-frontend-app/
├── .specify/              # Spec-kit configuration
│   ├── memory/
│   │   └── constitution.md
│   └── templates/
├── src/
│   ├── api/               # Generated types and API client
│   │   └── generated/     # protoc-gen-ts output
│   ├── hooks/             # Query hooks (useUsers, useProjects, etc.)
│   ├── components/        # Reusable UI components
│   ├── pages/             # Route-level components
│   └── lib/               # Utilities and helpers
└── tests/
    └── e2e/               # Playwright E2E tests
        └── utils/         # Test utilities
```

## Development Workflow

1. **Generate Types**: `pnpm run generate:types` from backend proto
2. **Write E2E Tests**: Create failing tests in `tests/e2e/`
3. **Verify Tests Fail**: `pnpm test:e2e` - new tests MUST fail
4. **Implement Hooks**: Create query/mutation hooks using generated types
5. **Implement Components**: Build UI components consuming hooks
6. **Verify Tests Pass**: Run full test suite - all tests MUST pass

## Testing Requirements

All testing is done with Playwright E2E tests against the real backend:

- ✅ Tests run against real backend API - **no mocking**
- ✅ All routes and user interactions covered
- ✅ Console errors captured and visible in output
- ✅ Tests are independent and can run in parallel
- ✅ AI-driven autonomous test-fix cycles

## Contributing

This template is designed to be forked and customized. Feel free to:

- Modify the constitution for your specific frontend stack
- Add organization-specific UI principles
- Customize templates
- Share improvements back to the community

## License

MIT License - see [LICENSE](LICENSE) file for details

## Support

- Constitution: See `.specify/memory/constitution.md`
- Spec-kit base: https://github.com/github/spec-kit
- This template: https://github.com/theplant/speckit-starter-ts

---

**Version**: 1.5.0 | **Last Updated**: 2025-12-11
