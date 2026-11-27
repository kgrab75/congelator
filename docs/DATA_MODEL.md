# Data model — modèle conceptuel, pas encore DDL

Références : PRODUCT, TA-003 à TA-006, TA-008. Les champs et contraintes ci-dessous sont une proposition détaillée à valider avant migrations.

| Entité | Identité et attributs essentiels |
|---|---|
| Auth User | Identifiant Better Auth opaque ; email, sessions gérés par le fournisseur |
| UserProfile | userId, displayName, locale, timezone |
| Household | id, name, createdAt |
| Membership | householdId, userId, role OWNER/MEMBER, joinedAt ; unicité couple |
| Invitation | id, householdId, email, token hash, expiry, status |
| Freezer | id, householdId, name, templateRef facultatif |
| StorageLocation | id, householdId, freezerId, parentId, name, order ; racine par freezer |
| GlobalProduct | id, name, brand, barcode facultatif, category, provenance, commercial size |
| HouseholdProduct | id, householdId, globalProductId facultatif, name, aliases, category, conversions |
| StockLot | id, householdId, productId, origin, frozenDate, preservation metadata, version |
| StockPosition | id, householdId, lotId, locationId, quantity, packaging facultatif, initialQuantity, openedAt |
| StockMovement | id, householdId, lotId, actor, operation, before/after, source, occurredAt, commandId, reverseOf |
| PreservationRule | id, version, category, conditions, duration, source, type, effective date |
| Conversation | id, householdId, userId, title, createdAt, updatedAt |
| ConversationItem / Message | conversationId, sequence, role/type, payload, timestamp ; décider modèle unique ou projection |
| AssistantRun / ToolExecution | runId, actor scope, state, model/prompt version, idempotency, result, usage |
| PendingAction | id, owner scope, immutable validated payload, state, expiresAt, expected version |
| Preference / Habit | scope user/household, type, value, provenance, consent si pertinent |
| Notification / Delivery | householdId, userId, source, channel, state, dedupeKey, scheduledAt |
| Recipe / Favorite | scope, ingredients, instructions, provenance, userId selon visibilité |
| OutboxEvent | id, type, scope, payload, timestamp, attempts, publishedAt |
| UsageLedger | runId, provider, model, actor scope, token usage, rate version, estimated cost |

## Cardinalités et invariants

`Household` has many memberships, freezers, and products. Users can belong to many households. Creating a household and transferring ownership must each preserve exactly one `OWNER` atomically. A partial unique index can prevent two owners. The write path must also guarantee that an owner exists. Test the invariant on success and rollback.

Freezer 1:N locations, une racine ; parent et enfant même freezer et foyer, aucun cycle ; verrouiller/coordonner les mutations de structure ; empêcher suppression si stock présent (y compris sous-arbre concerné) avant réaffectation. GlobalProduct 0..1 pour HouseholdProduct. Catalogue global lisible par utilisateurs autorisés, écriture/modération à définir ; ne jamais publier de données privées par défaut.

HouseholdProduct 1:N StockLot, StockLot 1:N StockPosition, StorageLocation 1:N StockPosition. La quantité restante est dans la position, total calculé par unité compatible ; si quantité inconnue, ne pas afficher un total exact trompeur. Conditionnement individuel facultatif ; quantité initiale distincte du restant. Un déplacement conserve quantité, provenance et conservation. Consommation multi-lots : sélection explicite/clarification si ambiguë ; FEFO pour allocation appropriée selon règles produit, pas de supposition sur lot réellement utilisé.

Quantity : union EXACT/ESTIMATED avec Decimal et unité, UNKNOWN sans valeur numérique ; aucune quantité négative, conversions compatibles uniquement, fraction initiale/restante explicite, pas de division par zéro. Quantités en NUMERIC (échelle à fixer après cas métier), décimal string dans les contrats. Conservation : DATE pour jour civil, précision et source (étiquette/règle versionnée/utilisateur/inconnue), type DLC/DDM/cible qualité distinct, date de congélation distincte de l’entrée. Aucun chiffre alimentaire inventé.

## Transactions et audit

`StockLot` is the root, and changes go through domain methods. Verify optimistic version checks, explicit locks, or both, and use a stable lock order when multiple lots are involved. Persist current state, append-only movements, and idempotency records in one transaction. When the [Transactional Outbox](ARCHITECTURE.md#stock-transactions-and-job-delivery) is in use, that transaction includes applicable outbox entries. The audit trail is separate from aggregate loading and does not serve as an Event Store. Undo uses a compensating movement, revalidates state, permissions, and the undo window, and prevents repeated undo. Define deletion and anonymization rules that preserve privacy.

## Isolation PostgreSQL

Schémas `auth` et `app` proposés ; migrations distinctes, rôles migration et runtime séparés. UUID v7 proposé pour identifiants métier, sans supposer que Better Auth émet ce format. Pour tables tenant, `household_id NOT NULL`, références composites `(household_id,id)` lorsque pertinent, contraintes et index de recherche adaptés. RLS USING + WITH CHECK, défaut interdit, FORCE RLS lorsque approprié ; rôle applicatif ni propriétaire, ni superuser, ni BYPASSRLS. Contexte transactionnel initialisé sur la connexion effective avec `set_config(..., true)` et contrôlé par backend ; ne jamais utiliser une variable de session durable avec pool. Autorisation métier et validation des memberships avant actions ; politiques d’accès aux memberships et bootstrap création/invitation à concevoir/tester sans échappatoire générale. Conversations et préférences privées filtrées également par userId. Workers disposent d’un contexte limité et explicite, pas de bypass global. Tests négatifs SQL avec rôle runtime et vérification de fuite entre requêtes.

## Open data model questions

- **V0.1** — Validate the physical schema and indexes, role grants, Better Auth identifier handling, and RLS policies for household creation, invitation acceptance, and ownership transfer. The [POC sequence](ROADMAP.md#v01-backlog) is a prerequisite for household implementation. Define business migrations and physical diagrams after the relevant POC and contract review.
- **V0.2** — Choose the decimal library, storage precision and scale, and rounding behavior for conversions and fractions after validating the inventory cases.
- **V0.6, before V1.0** — Decide how deletion, anonymization, archival, and retention apply to shared movement history. See the [product questions](PRODUCT.md#product-and-operational-questions) for the associated account and ownership scenarios.
