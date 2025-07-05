# **Gestion de photos auto-hébergées : Immich et alternatives**

  

Les solutions open-source de gestion de photos auto-hébergées se multiplient. Parmi les plus abouties on trouve  **Immich**,  **PhotoPrism**  et  **LibrePhotos**  (fork actif d’OwnPhotos). Chacune offre des interfaces et des fonctionnalités différentes. Immich vise une expérience proche de Google Photos : interface  **timeline**  chronologique, tris automatiques («  _souvenirs_  » par années), applications mobiles iOS/Android et outils de sauvegarde (CLI, apps)  . PhotoPrism, en revanche, est une PWA web au style plus « liste », sans appli mobile native  . LibrePhotos propose elle aussi une vue chronologique et supporte le multi-utilisateur  .

  

**Fonctionnalités IA :**  Immich intègre reconnaissance faciale, détection d’objets et recherche sémantique avancées. Ses algorithmes (DBSCAN pour les visages) détectent les visages et les regroupent par « personnes » que l’on peut nommer  . La recherche libre (smart search) utilise des modèles CLIP dans Postgres, permettant de chercher par mots-clés sans métadonnées explicites  . Un système de géocodage inverse local (GeoNames) traduit les coordonnées GPS EXIF en villes/états/pays pour les afficher et filtrer  . LibrePhotos offre aussi reconnaissance faciale, détection d’objets/scènes et recherche sémantique  . Photoprism promet des fonctions IA («  _AI-driven_  »), mais en pratique la reconnaissance est jugée médiocre (“visages rarement reconnus, objets mal identifiés”  ).

TABLEAU

Les retours d’expérience montrent qu’**Immich**  est très rapide (jobs de ML en parallèle, interface fluide) et bénéficie d’un développement actif  . PhotoPrism, bien que mature, demande plus de ressources (GPU matériel accélère les conversions)  et sa documentation est moins axée « utilisateurs débutants ». LibrePhotos offre un bon compromis (multi-utilisateur, timeline), mais certaines fonctionnalités sont encore en développement. En résumé, Immich tend à surpasser PhotoPrism côté fonctionnalités et ergonomie  , tandis que LibrePhotos se rapproche de Google Photos (avec reconnaissance faciale et recherche avancée)  .

  

## **Installation d’Immich en homelab**

  

**Prérequis système :**  Immich s’installe idéalement via Docker Compose (le plugin  docker compose  est requis  ). Recommandations matérielles : Linux (Ubuntu/Debian) de préférence, minimum 4 Go de RAM (6 Go recommandé) et 2 cœurs CPU (4 conseillés)  . La base de données PostgreSQL doit résider sur un disque local rapide (SSD),  _pas_  sur un partage réseau  . Prévoyez ~1–3 Go d’espace disque pour le  _DB_data_, et assurez-vous d’allouer au moins 2 Go RAM à PostgreSQL si vous limitez les ressources Docker  . Les fichiers photos/vidéos eux peuvent être stockés sur NAS/partage (DB_DATA_LOCATION  et  STORAGE_LOCATION  dans le  .env).

  

**Installation Docker Compose :**  Téléchargez les images Docker officielles (immich-server,  immich-machine-learning,  immich-...) et configurez  docker-compose.yml  avec les variables d’environnement nécessaires (source externe, ports, volumes). Exemple de procédure :

1.  Cloner la template Docker Compose d’Immich (disponible sur le site officiel).
    
2.  Éditer le fichier  .env  : définir  DB_DATA_LOCATION  (volume local PostgreSQL),  UPLOAD_LOCATION(chemin de stockage des médias), et autres paramètres (ports, quotas).
    
3.  Lancer  docker compose up -d. Le service s’expose par défaut sur le port 2283.
    

  

**Options de stockage :**  On peut configurer Immich pour copier ou relier une « bibliothèque externe » via des  _storage templates_  (voir la doc d’Immich  _Storage Template_). En général, on crée un volume Docker pour PostgreSQL sur SSD local, et un volume pour les données media (ce peut être un partage NAS monté ou un disque dédié). Pour les RAW, notez qu’Immich gère la plupart des formats (y compris RAW), mais certains (ex. CR2 Fuji) peuvent apparaître brisés faute de conversion réussie  . Photoprism, en comparaison, génère des sidecars RAW automatiquement (ce qui facilite la gestion des RAW)  , ce qu’Immich ne fait pas (au contraire, il recommande plutôt d’utiliser  _External Library_  pour référencer les originaux sans copier).

  

**Reverse proxy et sécurité :**  En production, on place typiquement Immich derrière un proxy (NGINX, Traefik ou Caddy) pour gérer TLS et authentification.  _Important_  : le proxy doit transmettre les en-têtes  Host,  X-Real-IP,  X-Forwarded-For  et  X-Forwarded-Proto  vers Immich  . L’exemple NGINX suivant (à adapter à votre domaine) montre l’idée :

```nginx
server {
    listen 80;
    server_name immich.example.com;
    client_max_body_size 50G;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_read_timeout 600s;
    proxy_send_timeout 600s;
    location / {
        proxy_pass http://127.0.0.1:2283;
    }
    # Assurez-vous de proxyfier aussi /.well-known/immich pour l’app mobile
    location = /.well-known/immich {
        proxy_pass http://127.0.0.1:2283;
    }
}
```

Cette configuration autorise les uploads volumineux (client_max_body_size  > taille max) et augmente les timeouts (important pour les longues requêtes)  . Un bloc similaire existe pour Traefik (il faut notamment ajouter les labels  _traefik.enable_,  _traefik.http.routers_, etc., et étendre les délais d’entrée de 60s à plusieurs minutes  ). Quel que soit le proxy, utiliser HTTPS/TLS est impératif en frontal.

  

Côté authentification, Immich intègre nativement  **OIDC**  (OpenID Connect). On peut le connecter à un fournisseur d’identité externe (Auth0, Keycloak, Authentik, Authelia, Google…) pour le login  . Par exemple, un utilisateur a configuré Authentik et a pu désactiver complètement l’authentification locale par mot de passe  . Ainsi, les profils avancés utiliseront l’OIDC + 2FA de leur domaine, tandis que les novices peuvent rester sur l’authent local simple. D’autres bonnes pratiques : lancer Docker en utilisateur non-root, mettre à jour régulièrement (Immich évolue très vite  ) et sauvegarder la base de données (Postgres).

  

**Configuration minimale :**  Une fois le proxy en place, accéder à  https://<votre-domaine>  permet de créer le premier compte (utilisateur  _admin_)  . Dans le panneau d’administration, on pourra ensuite ajouter d’autres utilisateurs, définir des quotas par utilisateur, régler les paramètres ML (nombre minimal de faces pour le clustering, etc.). Au minimum, prévoyez 6–8 Go de RAM allouée à Docker, 2 CPU, et un disque SSD pour la base de données. L’appareil hôte doit accepter les montages Docker sur un système de fichiers Unix (ext4, ZFS, APFS… pas NTFS/exFAT  ).

  

## **IA dans Immich : reconnaissance, détection et recherche**

  

Immich regroupe plusieurs fonctions IA avancées :

-   **Reconnaissance faciale :**  Immich détecte automatiquement les visages dans les photos/vidéos et les regroupe par « personnes »  . Chaque personne peut se voir attribuer un nom, et on peut rechercher une personne pour afficher tous les clichés la contenant. L’algorithme utilise la détection et le clustering (DBSCAN) en tâche de fond  . En pratique, la reconnaissance faciale d’Immich est jugée très bonne, proche des standards Google/Apple  .
    
-   **Détection d’objets et recherche sémantique :**  Immich analyse chaque image avec des modèles ML (ex. CLIP) pour générer des descriptions contextuelles  . Il en résulte une  **recherche intelligente**  (« Smart Search ») où l’on peut saisir n’importe quel mot (par ex. « plage », « chat ») et obtenir les images correspondantes, même sans tag manuel. Le système de filtres de recherche avancée permet en outre de combiner : personnes, localisation, appareil photo, date, type (image/vidéo), album, etc.  . Cette recherche par  _Texte libre_  (CLIP) est très puissante, mais consomme de la mémoire. On peut choisir des modèles plus petits pour gagner en rapidité ou en mémoire aux dépens de précision  .
    
-   **Géolocalisation (reverse-geocoding) :**  Lors de l’import, si une image contient des données GPS EXIF, Immich effectue un géocodage inverse local (base GeoNames embarquée) pour déterminer la ville, l’état et le pays  . Ces informations sont affichées dans les détails de l’image et utilisables pour filtrer/rechercher par lieu. L’avantage est qu’aucune API externe n’est requise (tout est en local).
    
-   **Accélération matérielle :**  Le traitement IA (détection des visages, objets, embeddings CLIP) est intensif. Immich peut tirer parti d’un GPU pour accélérer ces tâches (CUDA pour NVIDIA, ROCm pour AMD, ARM NN, OpenVINO…)  . L’accélération GPU est expérimentale mais peut considérablement réduire la charge CPU et la durée des jobs ML. Sur des serveurs puissants, on activera donc le support GPU dans  docker-compose  pour la machine learning. Sans GPU, les tâches IA tourneront sur CPU, ce qui reste acceptable mais plus lent.
    

  

Chaque fonction IA consomme des ressources (RAM, CPU/GPU) lors de l’indexation initiale. En usage continu, Immich crée des jobs asynchrones («  _worker_  ») pour traiter les nouvelles images en arrière-plan, ce qui rend l’interface réactive pour l’utilisateur  . En résumé, Immich propose reconnaissance faciale, détection de scènes/objets, recherche CLIP et géolocalisation embarquée, configurables dans le panneau d’administration, mais ces options peuvent rallonger le temps de traitement selon le hardware. Le tableau suivant récapitule l’impact estimé :

TABLEAU

## **Cas d’usage concrets**

-   **Sauvegarde mobile automatisée :**  Les applications mobiles Immich (Android/iOS) supportent la synchronisation automatique des photos/vidéos dès qu’elles sont prises  . À la création d’un album  _Backup_  sur le téléphone, toutes les nouvelles images sont envoyées en tâche de fond vers le serveur. (À noter : PhotoPrism, quant à lui, n’a pas d’app officielle – on utilisait par le passé des apps comme PhotoSync en WebDAV pour ses sauvegardes  .)
    
-   **Archivage familial :**  Immich convient pour stocker la photothèque d’une famille. Chaque membre peut avoir son compte (et quota de stockage)  . On crée des albums partagés (ex.  _Vacances 2023_), on met en favoris, on utilise le timeline pour replonger dans les archives par année. Les photos de famille identifiées par la reconnaissance faciale facilitent la recherche par personne. Immich ne supprime pas automatiquement les doublons, mais l’outil CLI fournit vérifie les  _GUID_  avant upload pour éviter les copies (évite les envois répétés)  .
    
-   **Accès multi-utilisateurs et permissions :**  L’administrateur peut créer plusieurs comptes, définir un quota et un  _storage label_  par utilisateur (pour isoler leurs dossiers)  . Les utilisateurs peuvent partager des albums entre eux. Par exemple, on peut rendre un album accessible à tous les membres de la famille. Il est aussi possible de générer un lien public sur un album/photo (fonction  _public link_) pour envoyer à un ami, comme sur Google Photos  .
    
-   **Tri par album/date/personne :**  Immich organise par date (grâce à la timeline) et permet de créer des albums manuellement ou automatiquement («  _événements_  », p. ex. « Jeudi à Berlin »)  . La recherche par date et par personne (grâce aux visages nommés) est très simple. On peut aussi filtrer par lieu (ville/pays), par appareil (modèle d’appareil), ou encore par média (images vs vidéos). Les tags hiérarchiques sont pris en charge si on veut annoter manuellement  .
    
-   **Déduplication :**  Immich ne fusionne pas automatiquement les images similaires en autant, mais son CLI permet d’éviter l’import de fichiers déjà présents (en comparant leur identifiant unique)  . Les bibliothèques n’auront donc pas de doubles stricts issus du smartphone, à moins d’avoir des transferts indépendants. PhotoPrism, pour sa part, gérait les rafales RAW+JPEG en  _photo stacking_  automatique (inutile d’avoir deux copies)  . Immich ne le fait pas, donc on veillera à ne pas dupliquer manuellement l’envoi des mêmes clichés.
    

  

## **Recommandations selon le profil utilisateur**

-   **Débutant :**  Immich est fortement conseillé. Son installation Docker Compose est simple et sa web‑UI est intuitive. Les apps mobiles et la sauvegarde automatique facilitent la prise en main  . LibrePhotos est aussi convivial (vue chronologique familière), mais ne dispose pour l’instant que d’une app Android (pas encore d’iOS)  . Ces deux solutions requièrent peu de configuration initiale. Photoprism est plus complexe à configurer et son absence d’app native le réserve à des utilisateurs plus expérimentés.
    
-   **Intermédiaire :**  Vous pouvez explorer plusieurs options selon vos besoins. Immich reste un excellent choix (IA avancée, OIDC intégré, dev actif)  . LibrePhotos offre de bonnes fonctions et gestion multi-utilisateur  . Photoprism peut être envisagé si l’on maîtrise bien Docker (et idéalement un GPU pour l’accélération), car il offre des réglages très granulaires et un solide support WebDAV  . Pensez aussi à la sécurité (proxy TLS, firewall) et à l’authentification OIDC si vous avez un annuaire.
    
-   **Avancé :**  Vous disposerez des ressources et compétences pour pousser les limites. Immich, avec l’accélération GPU et ses paramètres ajustables, deviendra très performant pour une grosse collection (ses jobs parallèles restent réactifs  ). Vous pourrez déployer Immich en multi-conteneurs, utiliser les extensions (_external libraries_, backups automatisés via CLI) et tirer profit des nouvelles versions fréquentes  . Certains utilisateurs avancés font tourner Photoprism en parallèle pour comparer ou comme outil d’archivage historique (Photoprism est stable mais moins évolutif, son dev est plus lent  ). Enfin, si vous êtes adepte de la personnalisation extrême, vous pouvez également intégrer Immich à un écosystème plus large (Nextcloud, Strapi, etc.), ou expérimenter d’autres projets open source (Piwigo, Photostructure, etc.) selon vos besoins spécifiques.
    

  

**En synthèse**, pour un usage homelab,  **Immich**  se distingue par son équilibre entre facilité d’usage et fonctionnalités avancées (mobile, IA, sécurité)  . Les profils débutants et intermédiaires y trouveront un outil clé en main, tandis que les utilisateurs avancés apprécieront sa capacité d’évolution (GPU, OIDC, API)  .  **LibrePhotos**  est une bonne alternative « Google-like » multi‑utilisateur, et  **PhotoPrism**  reste pertinent si l’on recherche une solution éprouvée, bien que gourmande en ressources  . Chacun pourra donc choisir selon ses priorités : simplicité d’interface et d’installation (Immich), ou contrôle granularité et indépendance (Photoprism), ou mixte (LibrePhotos).
