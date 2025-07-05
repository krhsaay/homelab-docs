# n8n — Automatisation no-code

n8n (prononcé « en-eight-en ») est une plateforme d’automatisation de flux de travail (workflow)  **no-code/low-code**  et  _source-available_, conçue en Node.js pour être exécutée sur site ou dans le cloud. Elle sert de « colle digitale » très flexible, permettant de connecter des applications, services, bases de données et APIs pour automatiser presque tout ce qu’on peut imaginer. L’interface web drag-and-drop de n8n facilite la création de workflows sans coder, tout en offrant la possibilité d’ajouter du code JavaScript ou Python personnalisé via un nœud  _Code_. La version gratuite (« Community Edition ») est disponible en auto-hébergement ; une licence dite  _Sustainable Use License_(fair-code, depuis 2022) autorise un usage interne ou personnel gratuit mais limite la revente commerciale du service. n8n propose aussi une offre SaaS (n8n Cloud) ainsi qu’une édition Entreprise avec fonctionnalités avancées (authentification SSO, audit, environnements). Son modèle « fair code » garantit toutefois la gratuité pour usage privé ou éducatif.

## Architecture générale

n8n est une application Node.js qui s’exécute typiquement dans un environnement conteneurisé (Docker/LXC) ou sur une VM/Linux. Par défaut, un déploiement de base consiste en un seul service n8n (UI et moteur d’exécution) avec une base SQLite embarquée. Cet  **environnement monolithique**  est facile à lancer (par exemple via l’image Docker officielle) et convient aux tests ou homelabs légers. Dans ce mode, n8n stocke toutes les données (credentials, logs d’exécution, définitions de workflows) dans un fichier SQLite (par défaut  `~/.n8n/database.sqlite`). Pour un usage plus robuste, on privilégie une base de données dédiée (PostgreSQL) pour la persistance, ainsi qu’un  _mode file d’attente_  distribué.
En  **mode distribué**  (« queue mode »), l’application n8n se découple en plusieurs composants : les instances  _main_(UI/API) gèrent l’interface utilisateur et ajoutent les exécutions à la file, un broker Redis sert de file de messages, et des instances  _workers_  récupèrent les jobs pour les exécuter. Chaque nœud  _Main_  traite les webhooks et appels API, réplique les workflows, puis pousse les tâches en file d’attente. Les nœuds  _Worker_  écoutent Redis, extraient les workflows et les exécutent en parallèle. Les données persistantes (définition des workflows, résultats, logs, credentials) sont stockées dans PostgreSQL (ou autre SGBD), partagé entre les instances. Cette architecture supporte la haute disponibilité (réplication des pods/UI) et la montée en charge horizontale (ajout de workers selon la charge).

### Évolutivité et modes d’exécution

En pratique, on commence souvent par un seul conteneur Docker n8n (SQLite) pour prototyper rapidement. Quand la charge augmente, on active le mode  _queue_: il faut alors configurer  `EXECUTIONS_MODE=queue`, définir les connexions Redis et PostgreSQL, et déployer plusieurs pods n8n (main) et workers. Sous Kubernetes, on utilise typiquement un Deployment (ou StatefulSet) pour n8n, un Deployment Redis et un StatefulSet PostgreSQL (ou service managé). Il faut aussi définir la clé d’encryption globale  `N8N_ENCRYPTION_KEY`  identique pour tous les pods – c’est indispensable pour chiffrer/déchiffrer les credentials utilisateurs. Cette clé peut être stockée dans un secret K8s.

Dans un homelab classique (ex. un seul serveur ou LXC), on peut se contenter d’un conteneur Docker n8n « all-in-one », ou d’installer n8n via Node.js (voir ci-dessous). Sur des machines plus puissantes, un petit cluster n8n (avec 2+ nœuds main et quelques workers) et une base Postgres permettent de traiter plusieurs workflows concurrents sans goulot d’étranglement. En résumé, n8n peut être déployé en mode  _monolite_  (pour commencer) ou en mode  _cluster_  (queue mode) pour évoluer avec la charge.

## Nœuds, connecteurs et déclencheurs

n8n repose sur le concept de  **flux de données**  à travers des  _nœuds_  (nodes) connectés visuellement. L’éditeur graphique permet de faire glisser des nœuds sur la toile et de les relier par des flèches pour définir l’ordre d’exécution. Il existe deux grandes familles de nœuds :

-   **Nœuds déclencheurs (Trigger nodes)**  : ils initient un workflow lorsqu’un événement survient. Par exemple, un  _Webhook_  peut recevoir un appel HTTP externe, un nœud  _Cron/Schedule_  se déclenche à intervalles réguliers, un nœud  _IMAP Email_  surveille une boîte mail, etc. Lorsqu’un trigger se produit, il active le début du workflow correspondant.
    
-   **Nœuds d’action (Action nodes)**  : ils réalisent les opérations après le déclenchement. Il en existe des centaines pour interfacer des services variés : envoi d’email (SMTP), messages Slack/Telegram/Discord, requêtes API HTTP, lectures/écritures en base de données (MySQL, PostgreSQL, MongoDB, etc.), manipulation de fichiers (CSV, JSON, Excel), et bien d’autres. Il y a aussi des nœuds  _prêt-à-l’emploi_  pour des applications SaaS (Gmail, Google Sheets, Notion, Salesforce, etc.). L’éditeur facilite la configuration de chaque nœud sans code.
    
-   **Nœud Code (Function node)**  : ce nœud spécial permet d’écrire du code JavaScript (ou Python en auto-hébergement) pour traiter des données complexes ou définir des logiques personnalisées dans le flux. C’est utile si les nœuds standards ne suffisent pas.
    

n8n dispose de plus de 500 intégrations prêtes à l’emploi couvrant l’essentiel des services et apps (API tierces, bases de données, messageries, réseaux sociaux, etc.), ainsi que plus de 1 250 modèles de workflows partageables. Par exemple, on trouve des nœuds dédiés pour Slack, Telegram, Discord, GitHub, Jira, Twitter, Stripe, Zapier, Trello, Matrix, et même des outils d’IA (OpenAI, LangChain, etc.). L’abondance de nœuds intégrés et le support de la  **Code node**  rendent n8n extrêmement polyvalent. Les nœuds sont configurés via l’interface : on saisit les paramètres (url, clés API, requêtes, conditions, etc.) dans des champs, sans écrire de script.

## Interface et supervision

n8n s’utilise principalement via une  **interface web**  unifiée (port 5678 par défaut) qui permet de concevoir, exécuter et surveiller les workflows. Le tableau de bord principal affiche la liste des workflows, leur état (activé/désactivé), ainsi que les journaux d’exécution (succès ou erreurs). Depuis l’UI, on peut lancer des workflows manuellement pour les tester, et consulter l’historique de chaque exécution (valeurs entrées/sorties). Un mode  _Debug_  permet de retracer étape par étape un workflow. L’interface gère aussi les  _crédentials_  (clés API, logins) avec chiffrement et partage entre utilisateurs, ainsi que la gestion multi-utilisateurs et rôles (RBAC) en version Entreprise.

Côté supervision technique, n8n ne propose pas de tableaux de bord analytiques intégrés (comme Kibana ou Graylog). En revanche, il expose quelques endpoints REST pour la santé du service :

-   `/healthz`  (200 si le service est en ligne),
    
-   `/healthz/readiness`  (200 si la DB est accessible et migrée),
    
-   `/metrics`  (informations détaillées type Prometheus).
    

Ces endpoints (désactivés par défaut pour la version open source) peuvent être activés via des variables d’environnement et utilisés pour surveiller n8n avec des outils externes (Prometheus/Grafana, Zabbix, etc.). Plusieurs utilisateurs créent aussi des dashboards personnalisés pour suivre le nombre d’exécutions, les temps de réponse ou les erreurs, en extrayant ces données des métriques ou des logs stockés en base. Enfin, on peut intégrer dans n8n même des notifications d’alerte : par exemple un workflow peut envoyer un mail ou un message Slack lorsqu’un autre workflow échoue plusieurs fois.

## Cas d’usage typiques

n8n est par essence transversal et trouve des applications dans de nombreux domaines. En homelab ou en entreprise, on l’utilise souvent pour :

-   **Notifications et alertes automatisées**  : envoyer des emails/SMS/Slack lorsqu’un événement survient (un fichier est ajouté sur un serveur, un capteur détecte un incident, un seuil de métrique est dépassé, etc.). Par exemple, on peut déclencher une alerte Slack dès qu’un serveur Linux génère une erreur critique.
    
-   **Synchronisation de données**  : tenir à jour automatiquement des données entre plusieurs systèmes. Par exemple, copier des leads d’un formulaire web vers un CRM, mettre à jour une base de données interne à partir de Google Sheets, répliquer des tickets Jira sur un autre outil de ticketing, etc. n8n peut relier Salesforce, MySQL, Airtable, Notion, etc., en utilisant des déclencheurs de type webhook ou planification.
    
-   **Processus métier récurrents**  : automatiser les tâches répétitives. Par exemple, générer et envoyer automatiquement des factures clients (en prenant les données dans une DB), effectuer des paiements réguliers via Stripe, envoyer des rappels par email avant une échéance, ou encore compiler des rapports hebdomadaires (collecte de données, filtrage, envoi de graphique).
    
-   **Collecte et agrégation de données**  : réaliser du web scraping ou appeler des APIs externes pour consolider des données. Par exemple, récupérer quotidiennement des données financières via une API, agréger des posts réseaux sociaux, ou importer des fichiers CSV depuis un dépôt. n8n peut ainsi alimenter automatiquement des dashboards ou journaux.
    
-   **Domotique et IoT**  : piloter des équipements intelligents. Par exemple, changer l’état de systèmes d’éclairage selon l’heure ou la météo (via MQTT), ou déclencher des actions si une caméra détecte du mouvement.
    
-   **Workflows IA**  : intégrer des modèles d’IA. Par exemple, appeler ChatGPT ou LangChain pour générer automatiquement du texte (email, résumé, code) à partir de triggers, ou analyser des images/textes, etc. n8n propose des nœuds pour OpenAI, Hugging Face, et peut orchestrer des chaînes d’outils AI.
    
-   **Orchestration DevOps/SecOps**  : automatiser des tâches IT. Par exemple, créer des tickets Jira lors de pull requests Git, déployer automatiquement une application après un commit, ou encore enrichir des alertes de sécurité (SIEM) en croisant avec MITRE ATT&CK via un agent IA. n8n peut servir de « SOAR » simple, en cascadant appels API, traitements de données et notifications pour fluidifier le travail des équipes IT/Sécurité.
    

Chaque cas d’usage illustre la flexibilité de n8n. Les nombreux  _templates_  fournis (sur le site n8n et la communauté) permettent souvent de démarrer un cas courant en modifiant simplement quelques paramètres.

## Licence et modèle de déploiement

La version open n8n est distribuée sous la  _Sustainable Use License_  (licence fair-code) depuis 2022. Cette licence accorde les droits d’utilisation et de modification en interne ou personnel sans paiement, mais interdit expressément de fournir n8n sous forme de service payant ou de le redistribuer commercialement. En pratique, cela garantit que l’outil reste gratuit pour un usage personnel/homelab, tout en protégeant le modèle économique du fournisseur.

Pour déployer, n8n propose plusieurs options :

-   **Auto-hébergement**  : via Docker (image  `n8nio/n8n`  officielle) ou installation directe sur Linux. Les conteneurs Docker sont les plus simples : un seul  `docker run`  suffit à lancer n8n avec un volume persistant (`/home/node/.n8n`) sur le port 5678. Des fichiers Docker Compose préconfigurés (dans le dépôt “n8n-hosting” officiel) facilitent le passage à Postgres, Redis, etc. Pour ceux qui préfèrent une VM ou LXC, il suffit d’installer Node.js (version 16+), puis d’installer n8n en global (`npm i -g n8n`), ou d’utiliser la version Desktop pour tester localement. n8n recommande le self-hosting pour les utilisateurs avancés; les débutants peu à l’aise avec l’infra peuvent préférer l’offre cloud n8n Cloud.
    
-   **n8n Cloud / Entreprise**  : pour un déploiement en mode SaaS sans maintenance, n8n Cloud (service managé) offre un niveau gratuit et des plans payants. L’édition Entreprise d’auto-hébergement ajoute des fonctions (audits, SSO, archivage avancé) adaptées aux organisations.
    

Dans un environnement domestique ou homelab, on privilégiera généralement Docker ou LXC. Par exemple, on peut créer un conteneur Debian/Ubuntu, y installer Docker Compose, et démarrer un stack n8n complet (nœud n8n, Redis, Postgres) d’un coup. Certains utilisateurs partagent même des templates Proxmox pour n8n. L’interface web de n8n (et ses autres ports de webhook) est ensuite accessible via un reverse-proxy (Nginx, Traefik, Caddy) pour gérer SSL et noms de domaine.

## Performances et ressources

n8n n’est pas très gourmand en CPU si l’on traite peu de workflows, mais son débit dépendra du type de nœuds utilisés. En revanche, la mémoire et l’I/O disque sont les facteurs critiques : chaque instance n8n et la base de données doivent avoir suffisamment de RAM. La documentation indique qu’une instance idle n8n Cloud consomme ~100 Mo, mais les besoins peuvent monter si les workflows manipulent de gros volumes de données (par ex. nœud  _Code_  dupliquant de larges variables).

**RAM**  – Pour un usage léger, quelques gigaoctets suffisent (une configuration de base comme  _2 vCPU + 4–8 Go_  permet déjà de lancer n8n avec quelques workflows). En production ou cas de charge plus élevée, on prévoira davantage :  _4 vCPU + 8–16 Go_  pour des opérations concurrentes importantes. Des utilisateurs notent même 16 Go+ pour un trafic très important. L’optimisation JavaScript repose sur du  _heap_, donc sur des VMs allouant la moitié (en général) de la RAM à Node.js. Dans tous les cas, privilégier la RAM – n8n recommande au moins 8 Go si possible (Ubuntu/Debian) pour de petits clusters.

**CPU**  – n8n utilise Node.js et peut profiter de plusieurs cœurs lorsqu’il y a plusieurs processus (un nœud worker par cœur par exemple). On rencontre peu de tâches massivement parallèles spécifiques, mais on peut exécuter plusieurs workflows à la fois. Un CPU moderne 4 cœurs est donc souvent un bon début; pour la version cluster, chaque service (main, worker, DB) peut tourner sur un cœur dédié. Selon la charge, multiplier les cœurs pour les workers améliore la réactivité.

**Stockage**  – On stocke les données (workflows, exécutions, credentials) dans la base (SQLite ou Postgres) et dans le volume n8n (`/home/node/.n8n`). Il est fortement conseillé d’utiliser des disques rapides (SSD/NVMe) car les logs et la DB subissent beaucoup d’écriture. Le guide recommande explicitement un stockage SSD et un volume persistant monté sur Docker/LXC. Pour dimensionner l’espace, on part du principe  _[volume de logs journaliers] × [jours de rétention]_. n8n stocke aussi tous les logs d’exécution en base : sans purge, la taille peut croître. Il faut donc configurer la rotation/purge automatique via  `EXECUTIONS_DATA_PRUNE=true`  et  `EXECUTIONS_DATA_MAX_AGE`  (pour effacer les exécutions anciennes). Sur SQLite, après purge il faut éventuellement lancer un  `VACUUM`  manuel pour compacter le fichier. En Postgres, la purge libère l’espace automatiquement.

**Scalabilité**  – Toutes les composantes peuvent se mettre en cluster. n8n peut déployer plusieurs  _nœuds Main_  derrière un load-balancer, et plusieurs  _Workers_  connectés à un Redis commun. PostgreSQL peut être en cluster primaire/standby pour la résilience. Ce déploiement distribué offre haute-disponibilité (HA) et montée en charge horizontale aisées. En pratique, on commence sur un nœud, puis on passe à 3+ nœuds pour le back-end (Postgres), autant de workers que nécessaire, et éventuellement un nœud master d’API n8n répliqué.

**Sécurité**  – n8n chiffre les données sensibles stockées (avec la clé N8N_ENCRYPTION_KEY) et recommande le TLS pour les connexions UI/API. En version Enterprise, on dispose de contrôles d’accès avancés (RBAC, audit, SSO, etc.). En open-source, on peut quand même activer l’authentification via un proxy et forcer HTTPS. Les communications entre nœuds (Redis, Postgres) doivent être sécurisées si on les distribue sur plusieurs hôtes. En homelab, on peut contenir le tout derrière un VPN interne ou un reverse-proxy sécurisé.

**Maintenabilité**  – Il convient de sauvegarder régulièrement la base de données (Postgres dump) ainsi que le dossier  `~/.n8n`  (contenant la clé d’encryption et les workflows). Les mises à jour se font simplement en remontant une nouvelle image Docker (commande  `docker pull`  ou  `docker compose pull`) puis en redémarrant le conteneur. Un bon automatisme est d’utiliser des scripts d’orchestration (docker-compose, Ansible, etc.) pour limiter l’effort opérationnel. Il faut aussi purger périodiquement les exécutions anciennes (`EXECUTIONS_DATA_PRUNE`) pour éviter de saturer le disque.

## Recommandations par profil d’usage

-   **Débutant / homelab simple**  : n8n est agréable pour démarrer grâce à son interface intuitive. Pour un débutant on peut lancer rapidement un container Docker n8n (voire utiliser n8n Desktop) et explorer quelques modèles. L’offre cloud gratuite (n8n Cloud) ou la documentation officielle (quickstarts) permettent de tester sans config complexe. Pas besoin d’optimiser les ressources au départ : un serveur 2 vCPU, 4 Go RAM (comme un petit VPS ou Raspberry Pi 4) fera fonctionner une instance basique. Il vaut mieux se familiariser avec la logique « flow-based » avant de répartir en clusters.
    
-   **Utilisateur avancé**  : si on maîtrise l’infra ou a des besoins plus exigeants, n8n excelle. On peut intégrer du code personnalisé (JS/Python), gérer des connexions API complexes, et scripter le déploiement (via Docker Compose, Terraform, etc.). Les experts tireront profit du « queue mode » pour assurer la stabilité à grande échelle, et pourront lier n8n à d’autres outils (par exemple envoyer ses événements vers un SIEM/Elastic ou un autre système de logs pour corréler). On peut aussi coupler n8n avec des outils de BI/monitoring (Prometheus, ELK) pour aller plus loin.
    
-   **Usage professionnel / gros volume**  : en production, on industrialise n8n sur un cluster : plusieurs instances n8n (UI/API), une base Postgres haute-dispo, Redis en cluster, et de nombreux workers. N8n Entreprise (ou Cloud Enterprise) apporte des fonctions essentielles (gestion multi-environnements, archivage, support) pour les équipes IT/Security. Les pipelines de CI/CD peuvent déployer n8n automatiquement, et les devs intègrent facilement les workflows dans des outils de gestion de version. Dans ces cas, des machines dédiées (8+ cœurs, 16+ Go) sont courantes, tout en gardant des disques SSD et des sauvegardes régulières.
    

En résumé, n8n est un outil très polyvalent pour l’automatisation low-code. Pour un homelab débutant, on privilégiera la simplicité : Docker-compose officiel ou n8n Cloud, interface web, et quelques workflows d’essai. Avec l’expérience, on peut étendre le système (cluster, code personnalisé) et combiner n8n à d’autres plateformes (par exemple l’utiliser en front-end d’un SIEM ou comme orchestrateur pour des scripts devops). L’essentiel est d’avoir une collecte fiable d’événements et d’actions, et n8n fournit ce cadre flexible. Les utilisateurs avancés peuvent bénéficier des templates existants ou créer les leurs (y compris via Sigma ou autres standards) pour répondre à n’importe quel cas d’usage.
