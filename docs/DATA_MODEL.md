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

Household 1:N memberships/freezers/products ; user N:N households. Exactement un propriétaire, transfert atomique. Un index unique partiel peut empêcher deux owners ; il ne garantit pas à lui seul l’existence d’un owner. Implémenter et tester les mutations assurant cette règle.

Freezer 1:N locations, une racine ; parent et enfant même freezer et foyer, aucun cycle ; verrouiller/coordonner les mutations de structure ; empêcher suppression si stock présent (y compris sous-arbre concerné) avant réaffectation. GlobalProduct 0..1 pour HouseholdProduct. Catalogue global lisible par utilisateurs autorisés, écriture/modération à définir ; ne jamais publier de données privées par défaut.

HouseholdProduct 1:N StockLot, StockLot 1:N StockPosition, StorageLocation 1:N StockPosition. La quantité restante est dans la position, total calculé par unité compatible ; si quantité inconnue, ne pas afficher un total exact trompeur. Conditionnement individuel facultatif ; quantité initiale distincte du restant. Un déplacement conserve quantité, provenance et conservation. Consommation multi-lots : sélection explicite/clarification si ambiguë ; FEFO pour allocation appropriée selon règles produit, pas de supposition sur lot réellement utilisé.

Quantity : union EXACT/ESTIMATED avec Decimal et unité, UNKNOWN sans valeur numérique ; aucune quantité négative, conversions compatibles uniquement, fraction initiale/restante explicite, pas de division par zéro. Quantités en NUMERIC (échelle à fixer après cas métier), décimal string dans les contrats. Conservation : DATE pour jour civil, précision et source (étiquette/règle versionnée/utilisateur/inconnue), type DLC/DDM/cible qualité distinct, date de congélation distincte de l’entrée. Aucun chiffre alimentaire inventé.

## Transactions et audit

StockLot racine ; mutations via méthodes métier. Version optimiste et/ou verrous explicites testés. Ordre stable des verrous si plusieurs lots. État courant, mouvements append-only, idempotence et Outbox dans la même transaction ; audit ne constitue pas un Event Store. Annulation par compensation et revalidation de l’état, des droits et du délai ; prévenir annulation répétée. Prévoir règles de suppression/anonymisation compatibles avec la confidentialité. Historique non chargé dans l’agrégat à chaque opération.

## Isolation PostgreSQL

Schémas `auth` et `app` proposés ; migrations distinctes, rôles migration et runtime séparés. UUID v7 proposé pour identifiants métier, sans supposer que Better Auth émet ce format. Pour tables tenant, `household_id NOT NULL`, références composites `(household_id,id)` lorsque pertinent, contraintes et index de recherche adaptés. RLS USING + WITH CHECK, défaut interdit, FORCE RLS lorsque approprié ; rôle applicatif ni propriétaire, ni superuser, ni BYPASSRLS. Contexte transactionnel initialisé sur la connexion effective avec `set_config(..., true)` et contrôlé par backend ; ne jamais utiliser une variable de session durable avec pool. Autorisation métier et validation des memberships avant actions ; politiques d’accès aux memberships et bootstrap création/invitation à concevoir/tester sans échappatoire générale. Conversations et préférences privées filtrées également par userId. Workers disposent d’un contexte limité et explicite, pas de bypass global. Tests négatifs SQL avec rôle runtime et vérification de fuite entre requêtes.

## Restant à décider

DDL, index, taille et précision décimales, seuils et politiques d’annulation, identifiants auth, référentiel alimentaire, modération globale, suppressions/archivage, rétention et règles de création/transfert de foyer. Produire diagrammes physiques et migrations seulement après POC et validation des contrats.
