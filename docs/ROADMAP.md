# Roadmap — livraisons vérifiables

Pas de date promise. La sécurité, l’accessibilité et les tests commencent dès V0.1. Ne pas implémenter les fonctionnalités futures par anticipation sans besoin concret.

| Jalon | Livrables | Critère de sortie |
|---|---|---|
| V0.1 — Foundations | Nx/pnpm, stack POCs, CI, authentication, households, invitations, ownership transfer and membership lifecycle, RBAC/RLS, hierarchical freezers, PWA shell, and baseline observability | `SEC-001` verifies reproducible startup, authentication, household and freezer journeys, tenant isolation, and baseline observability |
| V0.2 inventory | Private and global catalog, `StockLot` and positions, quantities, transactional stock operations, immutable movements, idempotency, domain events, correction, undo, and search | Daily manual use, with concurrency and history verified. First personal deployment if ready |
| V0.3 assistant | Agent unique, tools, mémoire locale, clarification/confirmation, UI SSE, Web Speech API | E2E manuel/IA, évaluations, reprise/idempotence, quotas ; repli texte pour navigateurs non compatibles |
| V0.4 food waste prevention | Validated preservation reference, dates, alerts, Transactional Outbox, BullMQ, Redis, independent worker, push, email, and preferences | Approved sources, revalidated and deduplicated reminders, and tested retries |
| V0.5 culinaire | Repas à partir des stocks, restrictions, favoris, récapitulatif | Suggestions sans mutation et sans faits inventés ; contrôles restrictions |
| V0.6 préparation | Audit sécurité, restauration, privacy, accessibilité, charge, quotas, supervision, admin minimal | Critères de production et FR-01..23 vérifiés ; matériel et réseau validés |
| V1.0 | Ouverture progressive | Invités puis inscription publique selon risque/capacité ; production personnelle possible |
| V1.x | Dictée backend OpenAI puis Whisper, scan, Open Food Facts, photos | Priorités à arbitrer ; pas de calendrier imposé |

## V0.1 backlog

The roadmap records milestone objectives, exit criteria, and open decisions. The [V0.1 — Foundations milestone](https://github.com/kgrab75/congelator/milestone/1) and its issues track scope, dependencies, acceptance criteria, and validation evidence. Link changes in delivery scope or milestone assignments back to the corresponding roadmap entry. Use the product specification and ADRs for requirements and architecture decisions.

| Ticket | Summary |
|---|---|
| [BOOT-000](https://github.com/kgrab75/congelator/issues/2) | Align project documentation and the development workflow. |
| [REPO-001](https://github.com/kgrab75/congelator/issues/3) | Establish repository governance and publish the backlog. |
| [BOOT-001](https://github.com/kgrab75/congelator/issues/4) | Bootstrap and verify the Nx workspace and baseline CI. |
| [BOOT-002](https://github.com/kgrab75/congelator/issues/5) | Verify PostgreSQL roles, migrations, and row-level security. |
| [BOOT-003](https://github.com/kgrab75/congelator/issues/6) | Verify Better Auth integration and pooled tenant context. |
| [AUTH-001](https://github.com/kgrab75/congelator/issues/7) | Deliver production-ready authentication for V0.1. |
| [HH-001](https://github.com/kgrab75/congelator/issues/8) | Create households and establish membership access. |
| [HH-002](https://github.com/kgrab75/congelator/issues/9) | Invite and enroll household members. |
| [HH-003](https://github.com/kgrab75/congelator/issues/10) | Transfer household ownership atomically. |
| [HH-004](https://github.com/kgrab75/congelator/issues/11) | Manage household membership lifecycle. |
| [FZ-001](https://github.com/kgrab75/congelator/issues/12) | Manage freezers and hierarchical locations, including empty location removal. |
| [UI-001](https://github.com/kgrab75/congelator/issues/13) | Deliver the responsive household and PWA shell. |
| [OBS-001](https://github.com/kgrab75/congelator/issues/14) | Establish baseline observability. |
| [SEC-001](https://github.com/kgrab75/congelator/issues/15) | Run the final integrated V0.1 verification gate. |

Review the workspace bootstrap results before starting dependent work. Verify versions and compatibility with official documentation during each POC. Each relevant implementation ticket includes its own authorization, tenant isolation, and security tests. The final gate verifies the delivered journeys together.

## Outstanding decisions and target milestones

Milestones show when an answer is needed for implementation or release.

| Target | Open decisions and validation |
|---|---|
| `BOOT-001` | Compatible toolchain versions and pins. [Architecture](ARCHITECTURE.md#foundation-pocs-and-open-decisions) |
| `BOOT-002` / `BOOT-003` | Physical schema and role grants for the POC, Auth identifiers and account linking, and verification of authenticated tenant isolation with a connection pool. [Data model](DATA_MODEL.md#isolation-postgresql) · [Security](SECURITY.md) |
| V0.1 | RLS policies for household creation, invitations, transfer, and membership lifecycle. Confirm enrollment and account linking, the invitation flow and lifetime, and the app shell's visual identity. Select and validate a real email provider before the first deployment requiring external email delivery. [Data model](DATA_MODEL.md#isolation-postgresql) · [Product questions](PRODUCT.md#product-and-operational-questions) · [Infrastructure](INFRASTRUCTURE.md) |
| V0.2 | Catalog governance, quantity precision and rounding, multi-lot selection, stock breakdown into tracked packages, corrections and undo, and minimal thaw tracking. Decide inventory retention and deletion policies before introducing stock operations. Verify deployment readiness if the household pilot goes live. [Product questions](PRODUCT.md#product-and-operational-questions) · [Data model](DATA_MODEL.md#open-data-model-questions) · [Infrastructure](INFRASTRUCTURE.md) |
| V0.3 | Sensitive-action thresholds, preference learning, crash and SSE recovery, dictation support matrix, model selection, and initial quotas and evaluation targets. [Product questions](PRODUCT.md#product-and-operational-questions) · [AI implementation questions](AI.md#open-implementation-questions) · [Security](SECURITY.md) · [Testing](TESTING.md) |
| V0.4 | Preservation sources and partial dates, alert defaults, worker execution context, and notification email delivery. [Product questions](PRODUCT.md#product-and-operational-questions) · [Architecture](ARCHITECTURE.md#foundation-pocs-and-open-decisions) |
| V0.5 | Multi-diner restrictions, unverified composition, and privacy of allergy data. [Product questions](PRODUCT.md#product-and-operational-questions) |
| V0.6, before V1.0 | Trademark and domain availability, account deletion and household continuity, conversation and audit retention, production budget and availability targets, capacity and recovery, and measurable product success criteria. [Product questions](PRODUCT.md#product-and-operational-questions) · [Security](SECURITY.md) · [Infrastructure](INFRASTRUCTURE.md) |
