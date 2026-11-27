# Congelator

AI-powered freezer management assistant for households.

Congelator aims to make freezer inventory easier to maintain, reduce food waste, and help households plan meals using what they already have. Users will be able to manage stock manually or through natural-language conversations.

**Status:** Product and architecture design complete; implementation has not started. The features and technologies below describe the intended architecture, not functionality already delivered.

## Planned capabilities

- Shared household inventories and configurable freezer locations.
- Product lots, packaging, quantities and auditable stock movements.
- Conversational stock management with clarification, confirmation and undo.
- Preservation reminders and meal suggestions based on actual inventory.
- Mobile-first progressive web app, initially in French.

## Technical direction

| Area | Planned technology |
| --- | --- |
| Monorepo | pnpm, Nx, strict TypeScript |
| Backend | NestJS, DDD, hexagonal architecture, CQRS |
| Frontend | React, Vite, TanStack, Tailwind CSS, shadcn/ui |
| Persistence | PostgreSQL, MikroORM, row-level security |
| Authentication | Better Auth |
| AI | OpenAI Agents SDK, application-owned conversation memory |
| Background jobs | Redis, BullMQ, dedicated worker |
| Quality | Vitest, Testcontainers, Playwright, AI evaluations |
| Operations | Docker Compose, GitHub Actions, OpenTelemetry |

Business rules and authorization remain deterministic and independent of the language model. Infrastructure and dependencies will be introduced incrementally.

## Documentation

The initial design documents were written in French. New public documentation is written in English.

- [Product specification](docs/PRODUCT.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Architecture decision records](docs/adr/README.md)
- [Domain model](docs/DATA_MODEL.md)
- [AI design](docs/AI.md)
- [Testing strategy](docs/TESTING.md)
- [Infrastructure](docs/INFRASTRUCTURE.md)
- [Security and observability](docs/SECURITY.md)
- [Roadmap](docs/ROADMAP.md)

## Development

`BOOT-001` is the first implementation ticket and covers initializing and verifying the monorepo. Setup commands will be documented after verification. See the [contribution guide](CONTRIBUTING.md) for the development workflow, [AGENTS.md](AGENTS.md) for coding-agent rules, and the [bootstrap prompt](docs/PROMPT_BOOTSTRAP.md) for the ticket scope.

This project uses AI-assisted development with human review, explicit architectural decisions and automated verification.

No production credentials, deployment-specific configuration or real user data belong in this repository.
