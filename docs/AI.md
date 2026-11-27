# Assistant IA et dictée

## Architecture

Agents SDK TypeScript, un agent avec tools par domaine. Adapter infrastructure derrière un port applicatif ; domaine sans dépendance SDK. Prompt versionné et profils FAST/REASONING/CULINARY configurables ; modèles sélectionnés après mesures qualité/coût/latence. Pas de RAG initial. Le LLM interprète et propose ; les cas d’utilisation déterminent droits, calculs, confirmations et mutation. Pas d’accès SQL direct pour le LLM.

## Tools et contrats

Lectures : search_products, get_inventory, get_freezer_structure, get_expiring_stock, get_stock_history. Écritures : add_stock, consume_stock, move_stock, correct_stock, discard_stock, undo_stock_movement, create_freezer, save_recipe, remember_preference. Outils culinaires consultent stock réel et restrictions. Exposer uniquement tools nécessaires au périmètre et aux autorisations ; Zod strict et validation serveur, identité et foyer injectés hors arguments IA. Les paramètres quantités distinguent ABSOLUTE, FRACTION (INITIAL/REMAINING), ALL_REMAINING. Identifiants résolus sur données autorisées, jamais inventés.

Résultats typés SUCCESS, CLARIFICATION_REQUIRED, CONFIRMATION_REQUIRED, REJECTED, ERROR. Succès uniquement sur commit réel. Actions indépendantes peuvent réussir partiellement, et les éléments résolus ne sont pas rejoués. Actions simples explicites et réversibles : directes ; ambiguïté bloquante : question ; destruction ou masse : confirmation serveur ; intention future : pas de mutation. Annulation compensatoire. Un échec IA ne rend pas le stock manuel indisponible.

## Contexte et reprise

PostgreSQL source de vérité, Session adapter du SDK. Conversations personnelles par foyer ; un run actif par conversation (contrainte atomique et récupération des runs abandonnés). Fenêtre bornée en tokens + résumé, préserver blocs tool call/result. Ne jamais utiliser un ancien résumé comme inventaire actuel. Requêtes fraîches via tools. Préférences structurées privées ou partagées ; inférences soumises à confirmation, modifiables/supprimables. Minimisation de données. `store:false` lorsque compatible ; vérifier politique de rétention fournisseur et désactiver capture sensible du tracing. Aucun envoi de secrets.

PendingAction : PENDING → EXECUTING → COMPLETED/FAILED, ou CANCELLED/EXPIRED. L’action confirmée est immuable, liée à l’auteur et au foyer, expiration et état attendus ; revalidation des permissions, version et stock avant exécution. Identifiant de commande stable ; si crash entre commit et réponse, relire résultat idempotent et ne pas rejouer. RunState SDK facultatif, versionné si persisté ; état métier indépendant. Pas de transaction DB ouverte durant appel IA.

## Streaming et UI

REST crée un run avec clé d’idempotence ; flux SSE associé, événements ordonnés typés (text.delta, tool.started/completed, component, clarification, confirmation, usage, completed, error). Reconnexion ne recrée pas de run ; reprise/lecture de statut ou résultat persisté. Les composants de mutation proviennent de résultats serveur, registre React fermé et validé ; aucune exécution de code produit par modèle. Les boutons appellent des endpoints authentifiés.

## Dictée progressive

V0.3/V1.0 : BrowserSpeechRecognitionProvider via Web Speech API, vérifier support et fonctionnement effectif, texte corrigeable avant envoi, arrêt/annulation, libération microphone, repli texte. Informer que le navigateur peut utiliser un service distant. Ne pas promettre fonctionnement sur tous navigateurs. V1.x : RemoteSpeechRecognitionProvider via MediaRecorder, backend TranscriptionPort, OpenAI adapter puis Whisper auto-hébergé (Python/FastAPI éventuel). A prioritaire en auto, puis B et C selon disponibilité et préférence ; changement de fournisseur transparent sur la politique, pas de renvoi audio silencieux ; nouvelle dictée possiblement nécessaire. Aucun audio brut conservé après traitement côté application. Durée 120 s et taille 10 Mo proposées pour futur backend, quotas et validation média. Pas de conversation vocale temps réel.

## Évaluation et coûts

Datasets synthétiques versionnés : compréhension, ambiguïtés, références, quantité, lots, multi-actions, intents futurs, confirmations, prompt injection, panne fournisseur, non-invention de dates. Évaluer résultat métier, pas seulement séquence d’appels. Tests fake indépendants, évaluations réelles plafonnées ; voir TESTING et SECURITY. Aucun fait alimentaire ou résultat de mutation généré sans source métier. Mesurer tokens, coûts, latence, version de prompt et modèle sans journaliser inutilement le contenu.
