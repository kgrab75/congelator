# Instructions for coding agents

## Scope and language

This repository is Congelator, a freezer-management assistant. Use `congelator` for the repository and Docker Compose project, `@congelator` for internal package names, and `congelator` for the PostgreSQL database in isolated environments. Do not publish packages or images without explicit approval. Code, identifiers, tests and new public documentation in English; user-facing UI initially French via i18n. Read `docs/PRODUCT.md`, `docs/ARCHITECTURE.md`, `docs/ROADMAP.md` and the relevant specialized docs before implementing. Decisions in `docs/adr/` are accepted; detailed proposals and unresolved values require validation when consequential. Do not silently change product behavior or accepted architecture; report conflicts and ask for an ADR if necessary.

## Working rules

- Implement only the requested ticket. Inspect existing files first; preserve unrelated changes. Do not add speculative libraries or placeholder modules for later milestones.
- Use pnpm + Nx, TypeScript strict. Pin compatible versions and commit lockfile. This repository is public: never commit private hostnames, public IP addresses of private infrastructure, deployment credentials, real user data or production-specific configuration. No secrets, real personal data or paid API calls in default tests. Do not deploy, migrate a live DB, publish images or provision resources without explicit approval.
- Backend modular DDD/hexagonal/CQRS. Domain pure TypeScript, no NestJS/ORM/AI imports. Application owns use cases and ports; infra implements; feature composes. Use Nx tags and ESLint. Do not bypass boundaries or invent global shared abstractions.
- Frontend uses generated API contracts, never backend entities. Do not edit generated client. Tenant-scoped queries and clear private cache on logout/identity change. No unsafe dynamic rendering or cache of private API data by service worker.
- All tenant operations authorize server-side; never trust tenant/user/role from AI or client. RLS defense in depth; no bypass runtime. Keep migrations separate. Stock mutations atomic with audit and outbox when necessary; idempotent, nonnegative, unknown not zero. No live LLM call inside DB transaction.
- LLM output untrusted. No invented stock, dates, permissions or successful actions. Confirm sensitive mutations server-side; future intent does not mutate. Always retain manual alternative.
- TDD on domain/use cases: establish failing tests for the expected reason, implement minimum, refactor. Run relevant lint/typecheck/tests/build; use real PostgreSQL in integration. Do not weaken assertions, disable tests, hide failures or claim unrun checks passed.
- Report files changed, decisions made, commands and actual outcomes, known risks, remaining work. Update relevant docs when contracts or conventions change. Never imply a POC or deployment was completed unless verified.

## Documentation map

`PRODUCT.md` (functional truth), `ARCHITECTURE.md` (technical choices), `DATA_MODEL.md` (conceptual model), `AI.md`, `TESTING.md`, `INFRASTRUCTURE.md`, `SECURITY.md`, `ROADMAP.md`, `adr/` (decision log), all under `docs/`.
