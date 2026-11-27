# Roadmap — livraisons vérifiables

Pas de date promise. La sécurité, l’accessibilité et les tests commencent dès V0.1. Ne pas implémenter les fonctionnalités futures par anticipation sans besoin concret.

| Jalon | Livrables | Critère de sortie |
|---|---|---|
| V0.1 fondations | Nx/pnpm, POC stack, CI, auth, foyers, invitations, RBAC/RLS, congélateurs hiérarchiques, PWA shell | Démarrage reproductible ; tests isolation et parcours auth/foyer/structure ; monitoring de base |
| V0.2 inventaire | Catalogue privé/global, StockLot/positions, quantités, mouvements, correction, annulation, recherche | Usage manuel quotidien, concurrence et historique testés ; premier déploiement personnel si prêt |
| V0.3 assistant | Agent unique, tools, mémoire locale, clarification/confirmation, UI SSE, Web Speech API | E2E manuel/IA, évaluations, reprise/idempotence, quotas ; repli texte pour navigateurs non compatibles |
| V0.4 anti-gaspillage | Référentiel validé, dates, alertes, worker/BullMQ/Redis, Outbox, push/email, préférences | Sources approuvées, rappels dédupliqués/revalidés, retries testés |
| V0.5 culinaire | Repas à partir des stocks, restrictions, favoris, récapitulatif | Suggestions sans mutation et sans faits inventés ; contrôles restrictions |
| V0.6 préparation | Audit sécurité, restauration, privacy, accessibilité, charge, quotas, supervision, admin minimal | Critères de production et FR-01..23 vérifiés ; matériel et réseau validés |
| V1.0 | Ouverture progressive | Invités puis inscription publique selon risque/capacité ; production personnelle possible |
| V1.x | Dictée backend OpenAI puis Whisper, scan, Open Food Facts, photos | Priorités à arbitrer ; pas de calendrier imposé |

## Premier backlog (proposition)

1. BOOT-001 : initialiser repo Nx/pnpm, web et API minimaux, conventions, lint/test/build et CI. Aucun domaine métier ni fournisseur IA requis. POC version matrix. Critères : install frozen-lockfile, commandes Nx, build et tests, contrôle frontières minimal, README local, pas de secret.
2. BOOT-002 : Compose local PostgreSQL, healthchecks, env validation, migrations et Testcontainers avec rôle SQL runtime.
3. BOOT-003 : POC Better Auth + NestJS + same-origin + RLS, test Vitest/SWC, génération OpenAPI + Orval, cookies/CSRF.
4. HH-001 : créer foyer/membership owner atomiquement et tester isolation ; invitations et transfert dans tickets séparés.
5. FZ-001 : congélateur et racine, hiérarchie, invariants et tests.
6. UI-001 : app shell responsive, routes foyer, composants et stories, auth, PWA.

Après BOOT-001, relire les résultats et ajuster les conventions avant de demander le ticket suivant à Codex. Les tickets sont proposés, pas des livraisons effectuées. Vérifier documentation officielle et compatibilité des versions lors des POC.
