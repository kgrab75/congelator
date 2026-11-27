# Infrastructure et exploitation — Congelator

## Décision

Production initiale auto-hébergée sur un serveur existant, Docker Compose et Nginx ; pas de VPS obligatoire à V1.0. Capacité, exposition réseau et sauvegardes non vérifiées : autoriser production extérieure uniquement après audit et validation. Les autres services de l’hôte doivent rester isolés et disponibles. VPS européen ou cloud lorsque mesures ou risque l’exigent, pas selon nombre arbitraire d’utilisateurs.

## Nommage

Projet Compose : `congelator`. Images : `congelator-web`, `congelator-api`, `congelator-worker` (préfixées par le registre et le propriétaire réels au moment de publier). Base PostgreSQL : `congelator`, isolée par environnement.

See [product questions](PRODUCT.md#product-and-operational-questions) for public domain name availability.

## Local et déploiement

Local : Node/pnpm/Nx et hot reload ; Compose pour PostgreSQL, Redis quand requis, Mailpit ; monitoring optionnel. Mode full Compose reproductible. Prod : web statique Nginx, API, worker, PostgreSQL, Redis ; réseaux/volumes/secrets propres à l’application, quotas ressources, log rotation, redémarrage contrôlé, healthchecks. Nginx existant route `/`, `/api`, `/api/auth` sur même origine HTTPS ; SSE sans buffering et timeouts adaptés. Ne publier aucun port DB/Redis ; accès web via réseau privé ou port loopback selon topologie réelle. Vérifier pare-feu Docker et accès depuis Internet. Administration via réseau de confiance/VPN, SSH clés ; isolation LAN et aucun socket Docker monté dans les apps.

## CI/CD

GitHub Actions : PR (contrôles TESTING), merge, build Docker multi-stage reproductible, images non-root, scan, GHCR avec SHA/version/digest, SBOM si disponible. Build once deploy many ; staging si disponible puis production approuvée ; si staging absent, tests éphémères et validation explicite. Secrets distincts et tokens minimaux ; pas d’exposition SSH additionnelle non nécessaire. Ansible progressivement pour config Linux ; OpenTofu seulement pour ressources provisionnées, état protégé. Traefik facultatif après migration. Pas de Kubernetes par défaut.

Déploiement : sauvegarde vérifiée, pull digest, tâche migration unique (rôle distinct), déploiement, health/smoke tests. Expand/contract pour compatibilité rolling/rollback. Redéployer digest précédent seulement si schéma compatible. Ne pas restaurer une base en écrasant des données récentes sans décision d’incident.

## Sauvegarde et reprise

PostgreSQL sauvegardes chiffrées hors machine, rétention à définir, restauration régulièrement testée ; PITR avant production publique si nécessaire pour RPO. Redis AOF/snapshots adaptés, politique noeviction pour file, et réconciliation des jobs critiques depuis état PostgreSQL/outbox. Sauvegarder config/secrets de manière séparée ; documenter restauration et rotation. RPO 1h/RTO 4h proposés pour public : à mesurer et financer, pas SLA. Surveillance externe indispensable pour détecter indisponibilité de la connexion domestique.

## Check avant ouverture

Mesurer CPU/RAM/disque/réseau sous charge combinée ; tester interruption/reboot, restauration, TLS, firewall, RLS, logs, quotas et disponibilité du mode manuel. Vérifier exposition IPv4/IPv6, NAT/CGNAT et DNS ; isoler staging et production (données/secrets). Ouvrir sur invitation d’abord, et réévaluer risque avant inscription publique. Migration VPS si saturation, indisponibilité, besoin d’isolation physique, exigences de données ou de service. Aucun prix hébergeur vérifié ni matériel audité.
