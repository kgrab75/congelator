# Architecture — Congelator

Statut : cadrage validé le 27 novembre 2026 ; détails d’implémentation proposés. Nom retenu : Congelator. Voir [PRODUCT](PRODUCT.md) pour le comportement et [ADR](adr/README.md) pour les décisions.

## Registre

| ID | Sujet | Décision |
|---|---|---|
| TA-001 | Stack | React + Vite + TanStack ; NestJS ; TypeScript strict ; REST/OpenAPI et SSE. |
| TA-002 | Monorepo | pnpm + Nx ; projets et tags ; frontières lint ; client généré. |
| TA-003 | Architecture | Monolithe modulaire, DDD, hexagonale, CQRS logique ; Domain Events, Outbox si nécessaire ; pas d’Event Sourcing. |
| TA-004 | Persistance | PostgreSQL + MikroORM ; mapping séparé pour agrégats complexes ; transactions et migrations. |
| TA-005 | Identité | Better Auth, sessions cookies, RBAC par foyer, RLS PostgreSQL. |
| TA-006 | Stock | StockLot racine ; StockPosition porte les quantités ; StockMovement historique immuable. |
| TA-007 | Agent IA | OpenAI Agents SDK, agent unique évolutif ; tools métier ; règles déterministes ; SSE. |
| TA-008 | Mémoire | PostgreSQL source de vérité ; Session adapter ; contexte borné ; préférences structurées ; PendingAction. |
| TA-009 | Dictée | Web Speech API en V0.3 ; OpenAI puis Whisper après V1.0 ; sélection progressive. |
| TA-010 | Jobs | BullMQ + Redis + worker NestJS indépendant ; Outbox ; notifications internes, push et e-mail. |
| TA-011 | Frontend | Feature-based ; Tailwind, shadcn/ui, Lucide, Storybook, Orval, MSW, i18next, Workbox. |
| TA-012 | Tests | Quality Engineering + TDD pragmatique ; Vitest, Testcontainers, Playwright, fast-check, évaluations IA. |
| TA-013 | Infrastructure | Production personnelle initiale, Docker Compose, Nginx existant, Actions/GHCR ; Ansible/OpenTofu progressifs. |
| TA-014 | Sécurité et observabilité | Security & Observability by Design progressive ; Pino, OpenTelemetry, Grafana progressif ; quotas IA. |

## Topologie cible

```mermaid
flowchart TD
  U[Utilisateur PWA] --> P[Nginx HTTPS]
  P --> W[Web React statique]
  P --> A[API NestJS REST SSE]
  A --> DB[(PostgreSQL)]
  A --> AI[Adaptateur OpenAI Agents SDK]
  A --> R[(Redis BullMQ)]
  J[Worker NestJS] --> R
  J --> DB
  J --> N[Push et email]
  DB --> B[Sauvegarde externe chiffrée]
```

API et worker partagent les mêmes bibliothèques métier. Redis est une infrastructure de jobs, pas la source de vérité des stocks. Aucune transaction SQL ne traverse un appel réseau IA. Pas de microservices métier, pas de Kubernetes obligatoire.

## Frontières et organisation Nx

Structure cible (créer les projets à mesure de leur utilité) :

```text
apps/{api,web,worker}/
libs/backend/<context>/{domain,application,infrastructure,feature}/
libs/frontend/<feature>/{feature,ui,data-access,util}/
libs/frontend/design-system/
libs/shared/{api-client,assistant-contracts}/
libs/testing/
tests/{e2e,security,performance,ai-evals}/
infrastructure/
docs/adr/
```

Contexts : Identity, Households, Freezers, Catalog, Inventory, Preservation, Assistant, Notifications, Meals, Preferences. Transcription est une capacité technique. Utiliser des APIs applicatives publiques ou événements explicites entre contextes, pas de dépendance circulaire ni d’accès aux repositories privés d’un autre contexte.

- Domain : classes TypeScript pures, invariants, événements, VO ; aucun NestJS, ORM ou SDK IA.
- Application : use cases, ports, politiques, handlers ; dépend du domaine. Les décorateurs CQRS peuvent être confinés à des wrappers d’intégration si nécessaires.
- Infrastructure : implémente les ports, mapping, DB, IA, transports.
- Feature : composition NestJS, contrôleurs et DTO. Les rôles techniques et la vérification d’appartenance sont injectés côté serveur.
- Tags Nx `platform`, `scope`, `type` ; ESLint restreint aussi les dépendances externes. Lint et vérification de cycles bloquants.
- Les queries utilisent des projections SQL typées ; les commands passent par le domaine. Le code de lecture ne modifie pas le stock.

## Frontend

Routes `/households/$householdId/...` ; query keys avec identité/contexte ; purger les caches privés à la déconnexion et au changement de compte, invalider après mutation. TanStack Form pour formulaires ; état local React, store global uniquement si besoin justifié. Design tokens, composants shadcn, stories et états vide/erreur/chargement. Orval génère types/hooks/mocks à partir du contrat NestJS ; CI vérifie leur fraîcheur. SSE utilise un contrat versionné et un client dédié. Aucun import de DTO internes, entités ou services backend. i18next et Intl ; français initial. PWA : cache statique seulement par défaut, pas de données privées ou mutations offline. UI IA via registre fermé, jamais HTML/JS arbitraire.

## Exécution des jobs

Worker dédié introduit lorsque nécessaire. Outbox écrite atomiquement avec mutation et audit ; relay avec verrouillage et reprise, livraison au moins une fois ; consommateurs idempotents, retries bornés, échecs inspectables. Identifiants de déduplication persistés ; pas de données personnelles inutiles dans Redis. Vérification des dates déterministe, notifications revalidées à l’envoi, fuseaux horaires et heures silencieuses.

## POC V0.1 et décisions ouvertes

Valider versions stables compatibles Node/pnpm/Nx/NestJS/Vite/MikroORM/Vitest/SWC/Better Auth ; authentification sur même origine, migrations séparées, RLS transactionnel avec pool et worker ; génération OpenAPI ; compilation et tests de frontières ; PWA installable. Épingler les versions et images après vérification, pas de numéros inventés. Le détail des schémas, bibliothèques Decimal, fournisseur mail, métriques, et identité visuelle reste ouvert. Toute divergence structurante nécessite ADR et validation humaine.
