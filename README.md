# n8n_IA_Agent 
Ce workflow n8n automatise les tâches DevOps courantes via des commandes Slack, en utilisant l'IA pour interpréter les demandes et exécuter les actions appropriées.
Vue d'ensemble
Le workflow fonctionne comme un agent DevOps intelligent qui peut :
Créer des branches Git
Lancer des pipelines CI/CD Jenkins
Créer des merge/pull requests
Générer des rapports d'erreurs de pipeline
**Architecture
Déclencheur principal: Slack Trigger : Écoute les messages sur le canal Slack et capture tous les événements de messages du cana
Intelligence artificielle
AI Agent : Utilise un modèle LLM (Mistral via OpenRouter) pour analyser et comprendre les commandes
Prompt système : Définit 4 catégories d'actions possibles avec des exemples d'extraction
Code parser : Parse et structure la réponse de l'IA en JSON exploitable

**Flux de traitement: 
1. Réception et analyse: Slack Message → AI Agent → Code Parser → Switch (routage)
2. Routage par type d'action: le noeud switch dirige vers 4 sous workflow selon l'intention detectée:
   
A. Création de branche Git (create_branch):
Vérifie si la branche existe déjà
Extrait le SHA de la branche principale
Crée la nouvelle branche via l'API GitHub
Génère des noms de branche au format feature/description-kebab-case

B. Création de Merge Request (create_merge_request):
Vérifie l'existence des branches source et cible
Compare les branches pour détecter les différences
Crée la pull request sur GitHub
Envoie une notification email au superviseur

C. Lancement de Pipeline (run_pipeline): dans cette partie les pipeline supporter sont: test, build, deploy et full ( ces pipeline à changer selon le choix)
Classification intelligente du type de pipeline selon le message
Gestion des jetons CSRF Jenkins
Suivi en temps réel de l'exécution
Récupération et analyse des logs
Formatage des résultats avec gestion d'erreurs

D. Rapport d'erreurs (generate_error_report)
Récupère l'historique des builds Jenkins
Filtre les builds échoués
Analyse intelligente des logs d'erreur
Génère un rapport structuré avec liens et détails

**Credentials nécessaires à configurer: 
Slack API : Token pour accéder au workspace Slack
GitHub API : Token avec permissions sur les repositories
Jenkins Basic Auth : Authentification pour l'API Jenkins
Gmail OAuth2 : Pour l'envoi d'emails de notification
OpenRouter API : Clé API pour le modèle LLM
PostgreSQL : Base de données pour tracer les commandes

**Variables d'environnement
GITHUB_TOKEN : Token GitHub pour l'authentification
URLs Jenkins configurées dans les nœuds HTTP (mettre votre URL jenkins)

**Utilisation:
Commandes slack: (expl)
Creation de branche git: créer une nouvelle branche nommée "XYZ" dans le repo "X" (vous donner l'url de repo) et mentionner la branche principal (main, master....)
Lancement de merge request: fais une merge request de la branche "source" vers la branche "cible" et informer le superviseur de projet git "on indique son adresse email "
Lancement pipeline: lance le pipeline de test sur la repo "URL repo git" pour la branche "mentionnez la branche surlaquelle travailler" 
Génération rapport d'erreur d'exécution des pipeline: génére un rapport d'erreur pour le pipeline "indiquer le nom de pipeline"

**Structure de la base de données: pour stocker les informations de commande envoyée en local( ou votre instance n8n tourne):
CREATE TABLE user_n8n_commands (
    id SERIAL PRIMARY KEY,
    user_id VARCHAR NOT NULL,
    channel_id VARCHAR,
    command_text TEXT NOT NULL,
    workflow_name VARCHAR,
    timestamp TIMESTAMP DEFAULT NOW()
);

**Format des réponses
Toutes les réponses sont formatées en Markdown avec :
Statuts visuels (✅❌⚠️)
Liens cliquables vers GitHub/Jenkins
Informations détaillées sur les actions effectuées
Horodatage des opérations













