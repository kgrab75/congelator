# Quality Engineering et TDD pragmatique

TDD sur invariants et use cases complexes : red vérifié, green, refactor, lint/typecheck. Ne jamais désactiver un test, affaiblir une assertion ou modifier les critères pour faire passer la CI. Tests orientés comportements, horloges/identifiants contrôlés, fakes de ports réutilisables.

| Niveau | Outils et critères |
|---|---|
| Domain | Vitest pur ; fast-check conservation de quantité, conversions, cycles, non-négativité |
| Application | Vitest + fakes ; autorisations, ambiguïtés, idempotence, historique, allocation |
| Integration | Testcontainers PostgreSQL/Redis ; migrations, rollback, verrous, Outbox, RLS vrai rôle runtime |
| HTTP | Supertest ; sessions, erreurs, CSRF, accès refusés, contrats |
| UI | Testing Library, Storybook, MSW, axe-core ; comportements, états, clavier |
| E2E | Playwright Chromium/Firefox/WebKit ; vrais appareils PWA et voix en complément |
| Architecture | Nx + ESLint, imports externes interdits, cycles, génération API fraîche |
| IA | tools et orchestrateur fake en CI ; dataset réel versionné, budget et résultats métier |
| Avancé | StrykerJS progressif sur Inventory ; k6 avant ouverture publique |

## Cas critiques minimum

```typescript
it('should prevent negative stock');
it('should preserve unknown quantity after partial consumption');
it('should preserve total quantity after moving stock');
it('should prevent concurrent overconsumption');
it('should persist stock state and movement atomically');
it('should not execute the same command twice');
it('should reject an expired confirmation');
it('should revalidate permissions before execution');
it('should prevent cross-tenant references');
it('should not leak tenant context between pooled connections');
it('should not modify stock for a future meal');
it('should not claim success after a failed tool execution');
```

When [the Transactional Outbox is introduced](ARCHITECTURE.md#stock-transactions-and-job-delivery), verify that relevant outbox entries commit or roll back with the corresponding business changes and movements.

Parcours E2E : inscription, foyer, invitation, changement foyer, congélateur, ajout/consommation/déplacement/annulation, historique, assistant ambigu et confirmation, reprise, alertes, repas, suppression compte. Ajouter tests de migration à partir de base vide et version précédente, restauration, worker retries et doublons. IA réelle : données synthétiques, aucune donnée utilisateur, budget limité. Les tests de sécurité backend restent déterministes.

## CI et seuils

PR : lint architecture, typecheck, unit/app, intégration, HTTP, E2E critiques, build, contrôle OpenAPI et scans. Nx affected avec base/head corrects et exécution complète périodique ; tests sécurité critiques indépendants de filtres fragiles. Nightly : suites complètes, multi-navigateurs, évaluations IA selon budget, scans. Release : smoke tests, vérification migrations et sauvegardes, sécurité, charge lorsque pertinent.

Objectifs proposés de couverture branches : domain 95 %, application 90 %, infrastructure/frontend 80 %. Évaluer et ajuster sans masquer régressions. Cibles IA proposées : 95 % intentions/résultats/clarifications, tous les scénarios critiques réussis ; mesurer variance, répétitions, tokens et latence. Ces chiffres ne sont pas des résultats ni une garantie de sécurité. POC Vitest/NestJS décorateurs SWC et Nx avant généralisation.
