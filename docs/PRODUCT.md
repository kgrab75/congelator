# Product Specification — Congelator

- Version : 1.1 (cadrage consolidé)
- Date : 27 novembre 2026
- Statut : décisions de Product Discovery validées ; spécification à relire avant implémentation
- Application name: Congelator
- Cible : MVP public V1.0, précédé de versions privées utilisables

## 1. Objet du document

Ce document constitue la référence fonctionnelle du projet pour le porteur du produit et Codex. Il décrit les besoins et comportements attendus, sans imposer de framework, de fournisseur IA ou de modèle de données physique. Les décisions techniques seront consignées dans `ARCHITECTURE.md`, les conventions de travail dans `AGENTS.md` et le découpage opérationnel dans `ROADMAP.md`.

Les décisions DP-001 à DP-013 ont été explicitement validées. Les règles détaillées ci-dessous consolident les propositions discutées et constituent une base de spécification à valider à la relecture. Les points ouverts sont regroupés en fin de document. En cas de conflit, demander un arbitrage plutôt que choisir silencieusement une nouvelle règle produit.

## 2. Vision et positionnement

Créer un assistant personnel et collaboratif qui connaît le contenu des congélateurs d'un foyer, permet de tenir l'inventaire à jour en langage naturel, aide à limiter le gaspillage et propose des repas adaptés aux stocks réels.

L'application est expérimentée dans un premier foyer avant d'être ouverte au grand public. L'authentification, les permissions et l'isolation des foyers sont nécessaires dès les premières versions. Aucun modèle de monétisation n'est imposé pour le MVP.

### 2.1 Problèmes à résoudre

- Oubli des aliments stockés et de leur emplacement.
- Manque de visibilité sur les quantités et dates de conservation.
- Friction excessive des inventaires manuels.
- Achats redondants et gaspillage évitable.
- Difficulté à trouver des idées de repas à partir des aliments disponibles.
- Désynchronisation des inventaires partagés entre plusieurs personnes.

### 2.2 Principes produit

- Langage naturel comme moyen privilégié de gestion ; interface manuelle toujours disponible.
- Précision facultative lorsque l'information n'est pas essentielle à l'action.
- Actions fiables, traçables et corrigibles.
- Données métier persistées comme source de vérité ; l'IA interprète et propose, elle ne remplace pas les règles métier.
- Distinction entre recommandation de qualité et sécurité alimentaire ; aucune garantie de salubrité fondée sur une date seule.
- Une suggestion de repas ou une intention future ne déclenche pas de sortie de stock.
- Expérience mobile-first, accessible et utilisable par des personnes non techniciennes.

## 3. Registre des décisions

| ID | Sujet | Décision validée |
|---|---|---|
| DP-001 | Positionnement | Approche progressive ; authentification dès le MVP |
| DP-002 | Foyers | Plusieurs foyers par utilisateur |
| DP-003 | Rôles | Propriétaire et membre |
| DP-004 | Congélateurs | Hiérarchie libre avec modèles prédéfinis |
| DP-005 | Stocks | Produits, lots et conditionnements facultatifs |
| DP-006 | Quantités | Exactes, estimées ou inconnues |
| DP-007 | Catalogue | Global et privé dès le MVP |
| DP-008 | Conservation | Gestion hybride : fabricant, référentiel, personnalisation |
| DP-009 | IA | Autonomie adaptative |
| DP-010 | Alertes | Notifications intelligentes |
| DP-011 | Repas | Assistant culinaire intelligent |
| DP-012 | UX | Hybride, mobile-first, PWA |
| DP-013 | Périmètre | MVP complet avec dictée vocale |

## 4. Comptes, foyers et permissions

### FR-01 — Comptes et foyers

Un utilisateur authentifié peut créer plusieurs foyers, rejoindre un foyer sur invitation et passer d'un foyer à l'autre. Chaque foyer contient ses membres, ses congélateurs, son catalogue privé, son inventaire, ses mouvements et ses préférences communes. Le contexte du foyer actif doit être visible et utilisé par défaut pour les opérations et conversations. Un changement de foyer ne doit pas réutiliser une action en attente visant l'ancien foyer sans nouvelle vérification.

### FR-02 — Rôles

| Permission | Propriétaire | Membre |
|---|---|---|
| Consulter et rechercher le stock | Oui | Oui |
| Utiliser l'assistant et les recettes | Oui | Oui |
| Ajouter, consommer, corriger, déplacer | Oui | Oui |
| Configurer les congélateurs | Oui | Oui |
| Consulter l'historique | Oui | Oui |
| Configurer le foyer | Oui | Non |
| Inviter ou exclure des membres | Oui | Non |
| Transférer la propriété ou supprimer le foyer | Oui | Non |
| Quitter le foyer | Après transfert si le foyer est conservé | Oui |

Chaque foyer possède exactement un propriétaire. Le créateur devient propriétaire. Un utilisateur peut détenir des rôles différents selon les foyers. Le départ d'un membre ne supprime pas l'historique des mouvements ; la conservation des identifiants et leur éventuelle anonymisation doivent respecter les règles de protection des données. L'assistant dispose au maximum des permissions de l'utilisateur courant. Toute lecture ou mutation est autorisée côté serveur, y compris les appels d'outils IA.

Critères d'acceptation : un membre ne peut ni exclure un autre membre ni supprimer le foyer ; un utilisateur sans appartenance ne peut consulter ou modifier aucun stock du foyer ; le transfert de propriété ne laisse jamais un foyer actif sans propriétaire.

## 5. Congélateurs et emplacements

### FR-03 — Équipements et structure

Un foyer peut gérer plusieurs congélateurs nommés librement. Des modèles prédéfinis facilitent la création : armoire, coffre, compartiment de réfrigérateur, configuration personnalisée (et autres modèles ultérieurs). L'utilisateur peut adapter les noms, le nombre et l'organisation des emplacements. Le type d'équipement n'impose pas de structure permanente.

Un congélateur peut ne contenir aucun compartiment. Chaque emplacement peut contenir à la fois du stock et des sous-emplacements. La hiérarchie ne doit pas comporter de cycle. L'utilisateur peut renommer et déplacer les emplacements. Supprimer un emplacement ou un équipement contenant des stocks requiert une réaffectation ou une opération explicite de sortie des stocks ; aucun stock ne doit disparaître implicitement. La configuration peut se faire manuellement ou en langage naturel, avec aperçu avant une création complexe.

Exemple : Cuisine > Bac du haut > Panier gauche. Une référence comme « en haut » est résolue dans le contexte du foyer ; si plusieurs emplacements correspondent, l'assistant demande une précision.

Critères d'acceptation : créer un congélateur armoire avec trois tiroirs ; ajouter un sous-emplacement ; déplacer un emplacement sans perdre ses stocks ; refuser une suppression qui laisserait des stocks orphelins.

## 6. Catalogue de produits

### FR-04 — Types et informations

Le catalogue prend en charge les surgelés industriels, les aliments congelés à domicile et les préparations maison. Les informations sont adaptées au type : nom, catégorie, marque et code-barres si disponibles, état cru/cuit, conditionnement commercial, ingrédients, allergènes et recette facultatifs. Les informations absentes ne sont pas inventées.

### FR-05 — Catalogue hybride

- Catalogue global : références commerciales et informations génériques non personnelles, réutilisables par plusieurs foyers.
- Catalogue privé : références propres au foyer, préparations maison, personnalisations, équivalences et synonymes locaux.
- Une référence locale peut être associée à une référence globale sans exposer les données privées.
- Une correction locale ne modifie pas les données des autres foyers.
- Les références sans stock restent disponibles pour un futur ajout.
- Les produits distincts ne sont pas fusionnés automatiquement sur la seule similarité de leur nom.

Le catalogue global sera alimenté progressivement. La procédure de création, de validation, de correction et de dédoublonnage des références globales reste à définir avant son implémentation. L'intégration Open Food Facts, le scan et la reconnaissance d'emballages ne sont pas requis en V1.0.

### FR-06 — Identification conversationnelle

L'assistant recherche les produits connus avant de proposer une nouvelle référence. Il peut exploiter des synonymes et habitudes modifiables du foyer. En cas de correspondances plausibles multiples, il présente les candidats ou pose une question ciblée. Une référence commerciale, un code-barres ou une composition non connus ne doivent pas être présentés comme vérifiés.

Critères d'acceptation : deux foyers peuvent utiliser une même référence globale avec des stocks et alias indépendants ; une préparation maison reste privée ; un produit épuisé peut être ajouté à nouveau sans doublon local inutile.

## 7. Inventaire et mouvements

### FR-07 — Produit, lot et conditionnement

Le produit décrit la référence alimentaire. Le lot représente un ensemble de stock partageant des caractéristiques de conservation ; il ne désigne pas nécessairement le numéro de lot industriel. Le conditionnement est un contenant physique facultativement suivi : paquet, boîte, sachet, pot, barquette, etc.

Plusieurs lots peuvent correspondre au même produit. Un lot peut être réparti sur plusieurs emplacements et comporter plusieurs conditionnements, avec une quantité restante propre à chacun. Un stock sans suivi individuel de contenant reste localisable. La représentation technique devra éviter de compter à la fois une quantité agrégée et ses conditionnements. Il doit être possible de détailler un stock initialement non détaillé sans perdre son historique ni augmenter artificiellement sa quantité.

### FR-08 — Quantités et conversions

| Famille | Exemples |
|---|---|
| Masse | g, kg |
| Volume | ml, cl, L |
| Comptage | pièce, unité |
| Portion | portion |
| Conditionnement | paquet, boîte, sachet, pot |

La quantité est exacte, estimée ou inconnue. Une quantité inconnue signifie que le produit est présent sans quantité exploitable ; elle ne vaut ni zéro ni une unité. Une quantité estimée conserve son caractère approximatif lors des calculs. Des expressions relatives sont comprises : moitié du paquet initial, moitié du restant, quart, tout le reste. Le moteur métier réalise les conversions et calculs ; les conversions entre familles différentes ou conditionnements exigent une équivalence connue, spécifique au produit ou contenant.

Les quantités ne deviennent jamais négatives. Une consommation supérieure au stock connu est bloquée et donne lieu à clarification ou correction explicite. Une consommation partielle d'un stock inconnu ne doit pas inventer le solde. Les totaux ne mélangent pas des unités incompatibles ; l'inventaire indique si une part du stock est inconnue. Les conditionnements dénombrés et pièces indivisibles ne doivent pas produire de fractions incohérentes.

### FR-09 — Opérations de stock

Opérations accessibles par interface manuelle et assistant : ajouter, consommer, déplacer, rechercher, consulter, corriger, jeter, annuler et réconcilier l'inventaire. Les changements d'emplacement préservent les dates de conservation. Une consommation partielle sans déplacement exprimé conserve l'emplacement du reste. Les produits épuisés sont retirés de l'inventaire actif mais conservés dans le catalogue et l'historique.

La sortie pour décongélation est distincte d'une consommation effective : si elle est enregistrée, le stock congelé est décrémenté une fois ; une consommation ultérieure ne doit pas le décrémenter à nouveau. Le suivi avancé, la réservation et les rappels de décongélation sont hors MVP ; la portée exacte du suivi minimal est à préciser.

### FR-10 — Historique, concurrence et annulation

Chaque mouvement est traçable : acteur, instant, nature de l'opération, stock concerné, variation ou correction, emplacement et origine manuelle ou conversationnelle. Les annulations utilisent des opérations compensatoires et ne détruisent pas l'historique. Une correction doit vérifier l'état courant et les éventuelles opérations intervenues entre-temps. En cas de conflit, demander une résolution plutôt qu'écraser une modification légitime. Les reprises réseau et appels répétés ne doivent pas déclencher des mouvements en double. Les règles de cohérence sont identiques quel que soit le point d'entrée.

Critères d'acceptation :
- Deux paquets de six glaces dans deux bacs donnent douze unités ; une glace consommée dans le premier laisse cinq unités dans ce paquet et onze au total.
- Après retrait de 250 g d'un paquet de 1 kg, la moitié du reste représente 375 g, et non 500 g.
- Deux boîtes de deux portions du même lot peuvent être rangées dans deux emplacements différents.
- Une quantité inconnue ne bloque pas l'ajout et n'est pas affichée comme zéro.
- Une annulation restaure la variation si elle reste cohérente, sans écraser un changement concurrent.
- Deux consommations concurrentes ne peuvent pas rendre le stock négatif.

## 8. Conservation et sécurité alimentaire

### FR-11 — Dates et provenance

Distinguer date d'ajout (connue), achat, préparation, congélation, ouverture, DDM, DLC, date cible calculée, date cible personnalisée et date de sortie/décongélation si renseignée. Les dates peuvent être exactes, partielles, estimées ou inconnues. L'application préserve la précision et la provenance de l'information, par exemple « juin 2027 » sans inventer un jour exact. L'ajout d'un produit n'implique pas que sa congélation a eu lieu aujourd'hui.

### FR-12 — Référentiel et calcul

Le référentiel interne doit être documenté, versionné et sourcé. Une règle de conservation identifie une catégorie, les conditions d'application (dont état cru/cuit et température de référence lorsqu'ils sont pertinents), une durée ou recommandation, sa nature (qualité ou sécurité), sa source et sa version. Le moteur métier sélectionne une règle applicable et conserve sa provenance. Sans données suffisantes ou règle applicable, il ne produit pas de date prétendument certaine.

La date fabricant n'est utilisée que dans les conditions auxquelles elle s'applique ; une DLC d'un aliment frais ne doit pas être interprétée comme date limite du produit congelé sans tenir compte de son historique et des règles applicables. Une date personnalisée de priorité ne peut effacer ou prolonger une limite sanitaire d'origine. Une ouverture ne change la date recommandée que si une règle pertinente existe. Le LLM ne crée pas de durées de conservation.

### FR-13 — Indicateurs et priorisation

Afficher les dates pertinentes, leur précision et leur origine, les produits à privilégier, les dates dépassées et les informations manquantes. La priorité s'appuie sur le principe FEFO (date pertinente la plus proche), lorsque les dates sont comparables. Les dates inconnues ne doivent pas être interprétées comme une absence de risque. Aucun retrait automatique du stock à échéance. Un dépassement de date de qualité ne signifie pas automatiquement que l'aliment est impropre, et une date non dépassée ne garantit pas sa sécurité. Les alertes sanitaires non résolues doivent empêcher les recommandations de repas concernées.

Critères d'acceptation : une date partielle reste partielle ; une règle calculée est traçable à sa version ; une modification de la date cible personnelle ne masque pas une alerte sanitaire ; un déplacement ne prolonge pas la conservation ; une date inconnue est explicitement affichée comme telle.

Les sources et durées effectives ne sont pas définies par ce document : elles devront être vérifiées et validées séparément avant utilisation.

## 9. Assistant conversationnel

### FR-14 — Intentions et contexte

L'assistant comprend en français les opérations d'inventaire, la configuration, les recherches, les corrections, les consultations de conservation et les demandes culinaires. Il comprend les références aux échanges précédents, peut gérer plusieurs intentions dans un même message et distingue un fait accompli, une instruction explicite et une intention future. Le foyer actif et l'identité de l'acteur restent explicites dans le contexte métier. Le contexte conversationnel ne remplace pas une nouvelle lecture des stocks lorsque nécessaire.

### FR-15 — Autonomie adaptative

| Situation | Comportement |
|---|---|
| Lecture autorisée | Réponse directe à partir des données réelles |
| Action simple, non ambiguë et autorisée | Exécution directe, retour sur le résultat et possibilité de correction |
| Information bloquante ou ambiguïté significative | Clarification avant l'action concernée |
| Information recommandée manquante | Question ciblée si utile ; possibilité de poursuivre sans réponse lorsque cohérent |
| Information facultative manquante | Ne pas bloquer inutilement |
| Action sensible, destructive ou de masse | Confirmation explicite ; confirmation renforcée pour suppression du foyer |
| Intention future ou suggestion | Ne pas modifier le stock sans action explicite |
| Échec de l'opération | Expliquer l'échec, ne jamais prétendre à un succès |

Le niveau d'autonomie ne repose pas sur une simple confiance déclarée par le modèle : les préconditions, droits, ambiguïtés et confirmations nécessaires doivent être vérifiés. Pour les actions multiples, les opérations indépendantes résolues peuvent être effectuées et les autres maintenues en attente ; le résultat liste clairement les succès, échecs et questions restantes. Une réponse de clarification ne doit pas rejouer les opérations déjà effectuées.

### FR-16 — Habitudes et réponses enrichies

Les préférences personnelles et communes au foyer sont distinguées, consultables, modifiables et supprimables. Elles facilitent la résolution des références, les emplacements habituels, les portions et les suggestions culinaires, sans remplacer un fait explicite contradictoire. L'assistant affiche des fiches de stock, changements avant/après, recettes et actions interactives lorsque pertinent. Les opérations sensibles restent soumises aux mêmes règles par bouton ou par message.

### FR-17 — Dictée vocale

L'utilisateur peut dicter un message depuis son smartphone. La transcription est visible et corrigeable avant envoi ; les mêmes règles conversationnelles s'appliquent ensuite. Le refus de permission microphone, l'échec de transcription ou l'incompatibilité du navigateur n'empêchent pas la saisie textuelle. La conversation audio temps réel et les réponses vocales ne sont pas requises en V1.0. V0.3 et V1.0 : Web Speech API lorsque disponible et fonctionnelle ; saisie textuelle sinon. L’utilisateur est informé que le navigateur peut transmettre la voix à un service distant. Les moteurs backend OpenAI puis Whisper auto-hébergé sont prévus après V1.0. Aucun audio brut conservé par notre application après traitement. Voir `AI.md`.

## 10. Alertes et notifications

### FR-18 — Alertes de conservation

Le système détecte les échéances proches et dépassées sans appel systématique à un LLM. Les seuils sont configurables (proposition de défaut : J-30, J-7, J-1, J). Les notifications sont regroupées, dédupliquées, annulées lorsqu'elles deviennent obsolètes et désactivées pour les produits épuisés. Les dates inconnues peuvent générer un indicateur interne d'information manquante, sans produire de faux rappel daté.

### FR-19 — Canaux et préférences

Centre interne, push et e-mail. Préférences individuelles et par foyer, heures silencieuses et fuseau horaire. Respecter les permissions et le consentement au canal. L'utilisateur peut consulter les notifications, reporter un rappel sans changer la date du produit, accéder au stock, déclarer une consommation ou solliciter une idée de repas. La notification distingue clairement recommandation de qualité et alerte sanitaire.

### FR-20 — Récapitulatif hebdomadaire

Le récapitulatif présente les stocks à privilégier et peut inclure des idées de repas. Les informations factuelles sont calculées sur l'inventaire réel ; l'enrichissement par IA est optionnel et soumis aux quotas. Aucun aliment n'est réservé ou consommé par l'envoi du récapitulatif.

Critères d'acceptation : un rappel répété ne crée pas de doublon ; consommer tout un lot invalide les rappels futurs correspondants ; un membre peut désactiver son e-mail sans désactiver les préférences des autres ; un report de rappel ne modifie aucune date alimentaire.

## 11. Assistant culinaire

### FR-21 — Suggestions et recettes

À la demande de l'utilisateur, l'assistant propose des préparations déjà disponibles, des associations d'aliments et des recettes utilisant les stocks du foyer actif. Il privilégie les échéances proches, vérifie les quantités connues et adapte les idées au nombre de convives, préférences, temps disponible, équipements et contraintes alimentaires. Une quantité inconnue ou estimée ne garantit pas un repas suffisant. Les ingrédients hors congélateur sont indiqués comme disponibles uniquement s'ils sont déclarés, sinon à vérifier ou acheter.

Les allergies et restrictions strictes sont distinctes des préférences simples. Une composition inconnue ne doit pas être présentée comme compatible avec une allergie ; l'assistant signale l'information manquante et ne recommande pas un aliment comme sûr sans vérification. Il exclut des suggestions les produits présentant une alerte sanitaire non résolue. Les indications de décongélation et préparation doivent être prudentes et cohérentes avec les informations connues.

### FR-22 — Personnalisation et favoris

Permettre d'adapter une recette, ses portions, ses ingrédients et sa durée de préparation, de sauvegarder les favoris et de déclarer explicitement les quantités utilisées. Les habitudes de repas peuvent être propres à l'utilisateur ou au foyer ; les équipements appartiennent au foyer. Les portions restantes d'un plat préparé peuvent être ajoutées comme nouveau stock à la demande. Les suggestions seules ne réservent ni ne décrémentent aucun produit.

Critères d'acceptation : une demande de repas ne modifie pas les stocks ; la suggestion distingue les ingrédients confirmés et ceux à vérifier ; une recette pour plusieurs convives ne présente pas une quantité inconnue comme suffisante ; une allergie connue n'est pas ignorée ; une préparation maison issue de restes peut être ajoutée sans déduire deux fois les ingrédients sortis.

## 12. Interface et parcours

### FR-23 — PWA hybride et mobile-first

Application web responsive, installable, adaptée au smartphone, à la tablette et à l'ordinateur. Français au lancement, conception compatible avec l'internationalisation. L'utilisateur identifie toujours son foyer actif. Les espaces principaux sont :

- Accueil : synthèse du stock, dates proches, informations manquantes et accès aux congélateurs.
- Stocks : recherche, filtres, regroupements par produit/lot/emplacement, détails et opérations manuelles.
- Assistant : texte, dictée, historique conversationnel pertinent, questions de clarification, résultats enrichis et annulation.
- Repas : suggestions, recettes et favoris.

Accès complémentaires : foyer et membres, équipements, profil, préférences, notifications, historique. L'assistant est accessible depuis tous les espaces principaux. Les fonctions de stock restent utilisables si le fournisseur IA est indisponible ou un quota est atteint. L'installabilité PWA n'implique pas un mode hors connexion complet ; la synchronisation offline avancée est reportée.

### Parcours de référence

| ID | Parcours | Résultat attendu |
|---|---|---|
| SC-01 | « Je range les frites dans le bac du haut. » | Identifier le produit et l'emplacement ; demander les informations utiles manquantes, accepter une quantité/date inconnue si cohérent ; enregistrer et récapituler. |
| SC-02 | « Ce soir, j'utilise la moitié du paquet. » | Résoudre produit, paquet et sens de la fraction ; distinguer une intention future d'un retrait effectif ; conserver l'emplacement du restant par défaut. |
| SC-03 | « Je congèle quatre portions de bourguignon dans deux boîtes. » | Clarifier la répartition si nécessaire ; créer le lot et ses conditionnements si souhaités. |
| SC-04 | « Où sont les steaks ? » | Fournir les emplacements et quantités réellement connus dans le foyer actif. |
| SC-05 | « Finalement, je n'ai mangé qu'une glace. » | Identifier la consommation antérieure et effectuer une compensation cohérente. |
| SC-06 | « J'ai vérifié le bac du haut : deux glaces et un paquet de frites. » | Comparer au stock, présenter les écarts et demander une confirmation avant une correction de masse. |
| SC-07 | « Qu'est-ce qu'on mange ce soir ? » | Proposer des repas adaptés, privilégiant les dates proches, sans mutation du stock. |
| SC-08 | « J'ai acheté 1 kg de frites, 4 steaks et 6 glaces ; les glaces en haut, le reste en bas. » | Gérer plusieurs actions ; distinguer celles effectuées et celles en attente, sans doublons. |
| SC-09 | Changer de foyer pendant une clarification | Ne jamais appliquer silencieusement la réponse au mauvais foyer. |
| SC-10 | Deux membres consomment simultanément la dernière glace | Une seule consommation effective ; l'autre reçoit un conflit ou une clarification. |

## 13. Exigences transversales

- Sécurité : authentification, autorisation côté serveur, isolation inter-foyers, validation des données et des outils IA, protection des secrets et limites d'usage.
- Confidentialité : collecte minimale, informations claires sur les traitements IA, gestion des conversations et préférences, export et suppression des données avant ouverture publique. Définir les règles de rétention et le traitement des données partagées lorsqu'un compte est supprimé.
- Fiabilité : intégrité des stocks, opérations atomiques lorsque nécessaire, résistance aux doublons et à la concurrence, sauvegarde et restauration testées.
- Résilience : erreurs explicites, repli manuel si IA indisponible, reprise sûre des actions incomplètes.
- Accessibilité : interface clavier, libellés lisibles, contrastes et états non transmis uniquement par couleur, parcours mobiles adaptés.
- Coûts : quotas et suivi des usages IA, traitement des dates et calculs sans modèle lorsque possible, enrichissements à la demande ou activés par préférence.
- Observabilité : erreurs et mouvements traçables, suivi des tâches de notification, métriques d'usage et de fiabilité sans journaliser inutilement des données personnelles.
- Qualité : tests métier, intégration et end-to-end ; évaluations de l'assistant sur ambiguïtés, outils, hallucinations, autorisations, actions futures et mutations concurrentes. Les métriques et objectifs chiffrés seront définis séparément.

## 14. Périmètre et livraison progressive

| Version | Objectif de livraison |
|---|---|
| V0.1 — Fondations | Authentification, foyers, invitations, rôles, équipements hiérarchiques, isolation, socle responsive/PWA |
| V0.2 — Inventaire | Catalogue hybride, lots et conditionnements facultatifs, unités, mouvements, historique, annulation, recherche et filtres ; premier usage quotidien manuel |
| V0.3 — Assistant | Gestion textuelle, outils métier, contexte, autonomie adaptative, actions multiples, synonymes, habitudes, réponses interactives et dictée vocale (jalon proposé) |
| V0.4 — Anti-gaspillage | Référentiel, dates, priorisation, alertes internes/push/e-mail, préférences, récapitulatif hebdomadaire factuel |
| V0.5 — Culinaire | Suggestions, recettes, préférences, restrictions, favoris, enrichissement culinaire des récapitulatifs |
| V0.6 — Ouverture | Quotas, coûts, monitoring, revue de sécurité, export/suppression, sauvegardes, accessibilité et performance : validation finale et compléments |
| V1.0 — MVP public | Ensemble des fonctionnalités validées, dont dictée vocale sur navigateurs compatibles, sous réserve des critères de qualité et de sécurité |

The [roadmap](ROADMAP.md) defines implementation milestones. Foundation design accounts for the dates and structure that later modules will use, and preservation automation arrives in V0.4. Testing, security, accessibility, and privacy start during the foundations. Each milestone is verifiable and usable within its scope. Public release starts at V1.0.

### Hors MVP V1.0

Moteurs de dictée backend OpenAI et Whisper, conversation vocale temps réel, scan de code-barres, intégration Open Food Facts, reconnaissance photo des emballages, lecture de tickets, étiquettes QR, mode hors connexion complet, applications natives, planning de repas, réservation de stock, liste de courses, rappels de décongélation avancés, statistiques de gaspillage avancées, capteurs de température et intégrations domotiques. Les versions V1.1/V1.2/V2 évoquées sont indicatives ; aucun engagement de calendrier. Aucun abonnement ni facturation requis.

## 15. Critères de passage à la V1.0

- Les exigences FR-01 à FR-23 et les parcours critiques sont vérifiés.
- Le foyer pilote peut gérer ses stocks au quotidien, avec et sans IA.
- Les autorisations inter-foyers et rôles sont vérifiées par tests négatifs.
- Les mutations IA sont fiables, idempotentes, auditées et réversibles lorsque cohérent.
- Les règles de conservation sont sourcées et validées ; les dates et incertitudes sont correctement présentées.
- Les suggestions respectent le stock réel et les restrictions connues, sans garantie alimentaire non justifiée.
- Les notifications sont personnalisables, dédupliquées et pertinentes.
- La dictée fonctionne sur les plateformes prises en charge, avec repli textuel.
- La protection des données, les quotas, les sauvegardes et la restauration sont opérationnels ; les anomalies critiques sont traitées avant ouverture publique.

## 16. Points ouverts et prochaines décisions

### Product and operational questions

- **V0.1** — Decide visual identity and the invitation flow, including invitation lifetime.
- **V0.2** — Define global catalog governance, including who can create, validate, correct, or deduplicate entries and what provenance to record. Define how to break down previously untracked packaging while preserving quantity and history, how to handle ambiguous multi-lot consumption, and what minimal thaw tracking includes. FEFO remains a proposal when the user asks for a selection and the lot actually used can be identified. Define the undo window, conditions for undo, and who can correct another member's movements.
- **V0.3** — Define sensitive and bulk-action thresholds, rules for learning and retaining conversation history, and handling of conversation state when users switch households. Establish the supported dictation platforms and release test matrix.
- **V0.4** — Approve preservation sources, coverage, calculation and partial-date handling, and how rules account for temperature and storage conditions. Decide notification thresholds, frequency, digest time, and handling of imprecise dates for reminders.
- **V0.5** — Define how to combine restrictions across diners, handle unverified ingredient or allergen information, and protect allergy data. The exclusions specified in FR-13 and FR-21 still apply.
- **V0.6, before V1.0** — Check commercial trademark availability and choose an available public domain name. Define account deletion, what happens to a household when its owner's account is deleted, and retention or anonymization of shared history. Set measurable targets for operation time, clarification and correction rates, inventory reliability, AI cost per household, adoption, and reduction in food waste.

The [roadmap](ROADMAP.md#outstanding-decisions-and-target-milestones) indexes the remaining decisions by milestone. See the [ADRs](adr/README.md) for validated technical choices and [infrastructure](INFRASTRUCTURE.md) for hosting and release prerequisites.
