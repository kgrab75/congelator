# Sécurité, observabilité et maîtrise des coûts

## Sécurité by design

Better Auth sessions serveur, cookies Secure/HttpOnly/SameSite adaptés, TLS, contrôles CSRF/origine y compris routes auth, rate limiting par identité/IP appropriée, validation Zod et métier, CSP/Helmet, contenu échappé. Google et email/password avec vérification et reset ; vérifier intégration réelle et flux d’association des comptes. Rôles foyer OWNER/MEMBER, admin technique distinct et moindre privilège. RLS + autorisation applicative (DATA_MODEL). Protéger secrets ; aucun secret dans repo, logs, frontend ou prompts. Rotation, accès minimaux et séparation environnements. Audit des dépendances, Gitleaks, Trivy, CodeQL si disponible selon plan/licence, Renovate ; ZAP progressivement, autorisé uniquement sur environnement de test. SOPS/age facultatif. Aucun contrôle de sécurité fondé uniquement sur le modèle.

## Confidentialité

Données minimales, conversations personnelles par foyer, export/suppression avant ouverture, consentement et traitements sensibles à préciser (allergies notamment), catalogue global sans données privées. Politique de rétention à valider : traces détaillées 30 j, agrégats usage 12 mois proposés, audio brut supprimé après traitement interne, conversations selon politique et suppression ; audit partagé soumis aux règles de confidentialité. Ne pas promettre absence de rétention fournisseur sur la seule base de `store:false`. Rédiger politique et vérifier obligations applicables avant public. Pas de données personnelles dans métriques à cardinalité élevée ; agrégats et identifiants techniques minimisés.

## Observabilité progressive

V0.1 Pino JSON + corrélation request/run/job, health/readiness, OpenTelemetry instrumenté avec export configurable ; aucune collecte intégrale des prompts/tool args en prod par défaut. Première prod : métriques Prometheus/Grafana selon ressources ; uptime monitor externe au serveur (Uptime Kuma local ne suffit pas pour coupure). Ensuite Alloy, Loki, Tempo et éventuellement Sentry. Suivre latence p50/p95/p99, erreurs, DB/queue lag, retries, CPU/RAM/disque, backups, IA usage. Health checks sans secrets. Limiter rétention et stockage ; accès dashboards privé. Corréler trace et run sans données sensibles.

## Quotas IA

Ledger par run/appel, modèle, input/output/cached tokens, prix versionné, devise, estimation et réconciliation facture. Limites initiales proposées : 8 tours, 15 tools, 60s, 30 interactions/user/j, 2 EUR/user/mois et 10 EUR/global/mois, alertes à 50/80/100 %. Configurables ; pas des tarifs fournisseur. Réserver atomiquement un budget avant appel selon coût maximal estimable, plafonner tokens et tours, réconcilier usage réel, traiter timeout/retry et réservations abandonnées, empêcher doubles appels ; limites fournisseur complémentaires. Une estimation locale ne garantit pas plafonnement exact de la facture. Si quota épuisé, interface manuelle toujours utilisable. Budget des évaluations séparé.

## Administration et release gates

Admin technique distinct des owners, accès à métriques, quotas, flags et échecs sans accès automatique aux inventaires/conversations ; audit des actions privilégiées et authentification renforcée à définir. Aucun endpoint admin public non protégé. Avant ouverture : tests sécurité inter-tenant/CSRF/idempotence, vérification réseau, restauration, monitoring externe, gestion incidents, export/suppression, notices et consentements requis, limites IA, dépendances corrigées. Les seuils de rétention/budget et objectifs de disponibilité doivent être confirmés après mesures.
