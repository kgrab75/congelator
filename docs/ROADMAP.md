# Roadmap — livraisons vérifiables

Pas de date promise. La sécurité, l’accessibilité et les tests commencent dès V0.1. Ne pas implémenter les fonctionnalités futures par anticipation sans besoin concret.

| Jalon | Livrables | Critère de sortie |
|---|---|---|
| V0.1 foundations | Nx/pnpm, stack POCs, CI, auth, households, invitations, RBAC/RLS, hierarchical freezers, PWA shell, and baseline observability (`OBS-001`) | Reproducible startup, tenant isolation and auth/household/freezer flows tested, and baseline observability verified |
| V0.2 inventory | Private and global catalog, `StockLot` and positions, quantities, transactional stock operations, immutable movements, idempotency, domain events, correction, undo, and search | Daily manual use, with concurrency and history verified. First personal deployment if ready |
| V0.3 assistant | Agent unique, tools, mémoire locale, clarification/confirmation, UI SSE, Web Speech API | E2E manuel/IA, évaluations, reprise/idempotence, quotas ; repli texte pour navigateurs non compatibles |
| V0.4 food waste prevention | Validated preservation reference, dates, alerts, Transactional Outbox, BullMQ, Redis, independent worker, push, email, and preferences | Approved sources, revalidated and deduplicated reminders, and tested retries |
| V0.5 culinaire | Repas à partir des stocks, restrictions, favoris, récapitulatif | Suggestions sans mutation et sans faits inventés ; contrôles restrictions |
| V0.6 préparation | Audit sécurité, restauration, privacy, accessibilité, charge, quotas, supervision, admin minimal | Critères de production et FR-01..23 vérifiés ; matériel et réseau validés |
| V1.0 | Ouverture progressive | Invités puis inscription publique selon risque/capacité ; production personnelle possible |
| V1.x | Dictée backend OpenAI puis Whisper, scan, Open Food Facts, photos | Priorités à arbitrer ; pas de calendrier imposé |

## V0.1 backlog

1. `BOOT-000` — Align project documentation and record the approved workflow in the [contribution guide](../CONTRIBUTING.md).
2. `BOOT-001` — Verify the version matrix and initialize the Nx/pnpm workspace, minimal web and API apps, architectural lint rules, tests, typecheck, build, and CI. Verify setup from a frozen lockfile and document local startup. See the [bootstrap prompt](PROMPT_BOOTSTRAP.md) for the ticket scope.
3. `BOOT-002` — Set up local PostgreSQL with Compose, database health checks, and environment validation. Establish migrations, separate migration and runtime roles, and minimal RLS policies. Verify policy enforcement with Testcontainers under the runtime role.
4. `BOOT-003` — Verify Better Auth and NestJS integration on the same origin, including session cookies and CSRF controls. Propagate the authenticated execution context into RLS within each transaction, and test that tenant context is isolated across pooled connections and requests. Verify integration compatibility, including Vitest/SWC and OpenAPI/Orval generation.
5. `HH-001` — Start after tenant isolation is verified in `BOOT-002` and `BOOT-003`. Create the household and its first `OWNER` atomically and test household-level isolation. Invitations and ownership transfer belong in separate tickets. Both creation and transfer must preserve exactly one owner atomically.
6. `FZ-001` — Implement the freezer root, location hierarchy, and invariant tests.
7. `UI-001` — Deliver the responsive app shell, household routes, components and stories, auth flows, and PWA shell.

`OBS-001` is a separate V0.1 ticket after `BOOT-001`. Deliver Pino structured logging, request correlation identifiers, health and readiness endpoints, initial OpenTelemetry instrumentation, and sensitive data redaction. Verify that the endpoints and logs expose no secrets. The full Grafana stack is outside this ticket. See [Security and observability](SECURITY.md) for rollout details.

Review `BOOT-000` before starting `BOOT-001`, then review `BOOT-001` results before starting the next ticket. Verify versions and compatibility with official documentation during each POC.

## Outstanding decisions and target milestones

Milestones show when an answer is needed for implementation or release.

| Target | Open decisions and validation |
|---|---|
| `BOOT-001` | Compatible toolchain versions and pins. [Architecture](ARCHITECTURE.md#foundation-pocs-and-open-decisions) |
| `BOOT-002` / `BOOT-003` | Physical schema and role grants for the POC, Auth identifiers and account linking, and verification of authenticated tenant isolation with a connection pool. [Data model](DATA_MODEL.md#isolation-postgresql) · [Security](SECURITY.md) |
| V0.1 | RLS policies for household creation, invitations, and transfer, invitation lifetime, and visual identity for the app shell. [Data model](DATA_MODEL.md#isolation-postgresql) · [Product questions](PRODUCT.md#product-and-operational-questions) |
| V0.2 | Catalog governance, quantity precision and rounding, multi-lot selection, stock breakdown into tracked packages, corrections and undo, and minimal thaw tracking. Verify deployment readiness if the household pilot goes live. [Product questions](PRODUCT.md#product-and-operational-questions) · [Data model](DATA_MODEL.md#open-data-model-questions) · [Infrastructure](INFRASTRUCTURE.md) |
| V0.3 | Sensitive-action thresholds, preference learning, crash and SSE recovery, dictation support matrix, model selection, and initial quotas and evaluation targets. [Product questions](PRODUCT.md#product-and-operational-questions) · [AI implementation questions](AI.md#open-implementation-questions) · [Security](SECURITY.md) · [Testing](TESTING.md) |
| V0.4 | Preservation sources and partial dates, alert defaults, worker execution context, and email provider. [Product questions](PRODUCT.md#product-and-operational-questions) · [Architecture](ARCHITECTURE.md#foundation-pocs-and-open-decisions) |
| V0.5 | Multi-diner restrictions, unverified composition, and privacy of allergy data. [Product questions](PRODUCT.md#product-and-operational-questions) |
| V0.6, before V1.0 | Trademark and domain availability, account deletion and household continuity, conversation and audit retention, production budget and availability targets, capacity and recovery, and measurable product success criteria. [Product questions](PRODUCT.md#product-and-operational-questions) · [Security](SECURITY.md) · [Infrastructure](INFRASTRUCTURE.md) |
