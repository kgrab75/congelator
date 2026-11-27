# Sécurité, observabilité et maîtrise des coûts

## Sécurité by design

Better Auth sessions serveur, cookies Secure/HttpOnly/SameSite adaptés, TLS, contrôles CSRF/origine y compris routes auth, rate limiting par identité/IP appropriée, validation Zod et métier, CSP/Helmet, contenu échappé. Google et email/password avec vérification et reset ; vérifier intégration réelle et flux d’association des comptes. Rôles foyer OWNER/MEMBER, admin technique distinct et moindre privilège. RLS + autorisation applicative (DATA_MODEL). Protéger secrets ; aucun secret dans repo, logs, frontend ou prompts. Rotation, accès minimaux et séparation environnements. Audit des dépendances, Gitleaks, Trivy, CodeQL si disponible selon plan/licence, Renovate ; ZAP progressivement, autorisé uniquement sur environnement de test. SOPS/age facultatif. Aucun contrôle de sécurité fondé uniquement sur le modèle.

## Confidentialité

Données minimales, conversations personnelles par foyer, export/suppression avant ouverture, consentement et traitements sensibles à préciser (allergies notamment), catalogue global sans données privées. Politique de rétention à valider : traces détaillées 30 j, agrégats usage 12 mois proposés, audio brut supprimé après traitement interne, conversations selon politique et suppression ; audit partagé soumis aux règles de confidentialité. Ne pas promettre absence de rétention fournisseur sur la seule base de `store:false`. Rédiger politique et vérifier obligations applicables avant public. Pas de données personnelles dans métriques à cardinalité élevée ; agrégats et identifiants techniques minimisés.

## Observabilité progressive

The V0.1 baseline is specified in [`OBS-001`](ROADMAP.md#v01-backlog). Use Pino JSON logs, a configurable OpenTelemetry exporter, and redaction for sensitive fields. Correlate requests and extend correlation to AI runs and jobs when those features exist. Production defaults exclude full prompts and tool arguments. Keep health and readiness responses free of secrets.

For the first production deployment, add Prometheus and Grafana as resources permit and monitor uptime from outside the host. Alloy, Loki, Tempo, and optionally Sentry can follow. Track latency percentiles, errors, database and queue lag, retries, resource use, backups, and AI usage. Metric thresholds and retention need validation before production. Keep dashboards private and exclude sensitive data from correlation identifiers and telemetry.

## Quotas IA

Ledger par run/appel, modèle, input/output/cached tokens, prix versionné, devise, estimation et réconciliation facture. Limites initiales proposées : 8 tours, 15 tools, 60s, 30 interactions/user/j, 2 EUR/user/mois et 10 EUR/global/mois, alertes à 50/80/100 %. Configurables ; pas des tarifs fournisseur. Réserver atomiquement un budget avant appel selon coût maximal estimable, plafonner tokens et tours, réconcilier usage réel, traiter timeout/retry et réservations abandonnées, empêcher doubles appels ; limites fournisseur complémentaires. Une estimation locale ne garantit pas plafonnement exact de la facture. Si quota épuisé, interface manuelle toujours utilisable. Budget des évaluations séparé.

## Administration et release gates

Admin technique distinct des owners, accès à métriques, quotas, flags et échecs sans accès automatique aux inventaires/conversations ; audit des actions privilégiées et authentification renforcée à définir. Aucun endpoint admin public non protégé. Avant ouverture : tests sécurité inter-tenant/CSRF/idempotence, vérification réseau, restauration, monitoring externe, gestion incidents, export/suppression, notices et consentements requis, limites IA, dépendances corrigées. Les seuils de rétention/budget et objectifs de disponibilité doivent être confirmés après mesures.
