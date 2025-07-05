# **Karakeep – gestionnaire de favoris tout-en-un**

Karakeep (anciennement  _Hoarder_) est un gestionnaire de favoris  **open source**  auto-hébergé (« bookmark‑everything app ») qui stocke non seulement des liens, mais aussi des notes, des images et des PDF  . Son interface web moderne (ci-dessus) permet d’ajouter facilement du contenu et de l’organiser dans des listes. Karakeep automatise la récupération des métadonnées (titres, descriptions, images) de chaque lien  , propose un  **moteur de recherche plein texte**  sur tout le contenu stocké  , et inclut une fonction d’**archivage complet de pages web**  (via l’outil  _Monolith_  pour contrer la disparition des liens)  . Des  **extensions navigateur**  (Chrome, Firefox) et des applications mobiles (iOS, Android) sont disponibles pour capturer rapidement du contenu  . Enfin, Karakeep intègre des fonctions basées sur l’**IA**  : génération automatique de mots-clés (tags) et résumés par ChatGPT ou modèles locaux (via Ollama)  . Ces fonctionnalités (archivage de pages, recherche full-text, extensions et IA) permettent d’exploiter Karakeep comme un hub personnel de veille ou de documentation  .

  

## **Comparatif avec d’autres solutions open-source**

  

Plusieurs autres gestionnaires de favoris auto-hébergés existent, chacun avec ses points forts :

-   **Linkding**  : outil léger et minimaliste, optimisé pour la rapidité  . Il récupère automatiquement titre, description et icônes des sites  . Il offre un  **archivage de pages**  (sauvegarde au format HTML local ou sur Internet Archive)  , des  **extensions Chrome/Firefox**  pour ajouter/rechercher des liens  , et une API REST. Par contre, il n’a pas d’intégration IA ni d’app mobile dédiée.
    
-   **Shaarli**  : gestionnaire très simple mono-utilisateur (pas de base de données, tout est stocké dans un fichier)  . Il sert à « partager, commenter et sauvegarder » des liens  , et peut se transformer en blog léger ou wiki personnel. Shaarli inclut recherche textuelle sur les titres/descriptions  et prise en charge des  **tags**  , mais n’offre pas d’archivage de pages ni de fonctionnalités avancées (pas d’extensions officielles, juste un bookmarklet  ).
    
-   **Omnivore**  : application orientée « read-it-later » (pour lire plus tard) et centrée sur le texte  . Elle propose surlignage, prise de notes et partage social, ainsi que la sauvegarde de position de lecture ou de newsletters via email  . Omnivore supporte les  **extensions navigateurs**  (Chrome, Safari, Firefox, Edge) et le hors-ligne  . Elle permet la gestion de contenus web et PDF, mais ne gère pas nativement d’images ou de médias divers comme Karakeep. Omnivore inclut aussi des  **tags**  (étiquettes) pour organiser les articles  .
    

  

Ainsi, Karakeep se distingue par son périmètre plus large (liens + notes + images + flux RSS) et ses fonctions avancées (archivage complet, IA), tandis que Linkding et Shaarli visent la simplicité, et Omnivore la lecture optimisée de contenus textuels.

  

## **Fonctionnalités clés de Karakeep**

-   **Archivage de pages web**  : Karakeep peut enregistrer la page complète (via Monolith) et même archiver les vidéos (YouTube-DL), garantissant la pérennité du contenu  .
    
-   **Recherche plein-texte**  : grâce à Meilisearch, tous les mots du contenu sauvé sont indexés, permettant des recherches rapides dans les articles, notes et transcriptions  .
    
-   **Extensions et applications**  : un module  **Chrome**  et un  **Firefox addon**  permettent d’ajouter un lien ou un extrait en un clic depuis le navigateur  . Les apps mobiles (Android, iOS) et le support PWA facilitent l’ajout de contenu depuis un smartphone.
    
-   **Intégration IA**  : Karakeep attribue automatiquement des tags et génère des résumés de contenu via ChatGPT ou des modèles locaux (Ollama) pour faciliter la classification et la lecture rapide  .
    
-   **Autres fonctionnalités**  : OCR intégré pour extraire le texte d’images, capture d’écrans plein-page, moteur de règles personnalisées, actions groupées, support SSO, mode sombre, etc.
    

  

## **Installation de Karakeep**

  

### **Avec Docker Compose**

  

La méthode recommandée est via Docker. Après avoir installé Docker et Docker Compose, créez un dossier sur votre serveur (par exemple  karakeep-app) et placez-y le fichier  docker-compose.yml  officiel  (disponible sur GitHub). Puis créez un fichier  .env  où vous définissez les variables clés (SECRET, URL, clé Meili, etc)  . Par exemple :

```yaml
KARAKEEP_VERSION=release
NEXTAUTH_SECRET=<clé_sécrète>
MEILI_MASTER_KEY=<clé_meilisearch>
NEXTAUTH_URL=http://localhost:3000
```
Après avoir personnalisé ces valeurs (changer les clés, indiquer votre domaine dans  NEXTAUTH_URL, etc.), lancez Karakeep avec :
```sh
docker compose up -d
```

Vous devriez pouvoir accéder à l’interface sur  http://localhost:3000  (ou votre domaine en prod)  . Le fichier Compose configure déjà le réseau et les volumes nécessaires. Pour activer l’IA, ajoutez votre clé OpenAI dans  .env  (voir la doc)  .

  

### **Installation « bare-metal » (script Debian/Ubuntu)**

  

Karakeep fournit un script d’installation pour Debian 12/Ubuntu 24.04 (adapté d’un module Proxmox)  . Ce script télécharge et installe toutes les dépendances (Karakeep, Meilisearch pour la recherche plein texte, navigateur Chrome sans interface graphique, etc.), crée les services système (karakeep-web,  meilisearch, etc.) et les configure pour démarrer automatiquement  . Il s’exécute en root (bash karakeep-linux.sh install). Cette approche convient pour installer Karakeep directement sur un serveur (ou dans un conteneur LXC Debian/Ubuntu) sans passer par Docker.

  

### **Réseau et Reverse Proxy**

  

En production, il est conseillé de mettre Karakeep derrière un reverse proxy afin de gérer le SSL et les noms de domaine. Par exemple, un conteneur  **Caddy**  peut automatiquement obtenir des certificats Let’s Encrypt et rediriger le trafic HTTPS vers Karakeep  . Dans ce cas, on crée un réseau Docker partagé entre Karakeep et Caddy, on expose Karakeep sur un port interne (par ex.  3000), et Caddy se charge de publier Karakeep sous  https://mon-domaine.com  via  reverse_proxy karakeep_web:3000  . On peut aussi utiliser  **Nginx**+Certbot ou tout autre proxy inverse. L’important est de faire pointer votre nom de domaine vers le serveur et de proxyfier le port 3000 de Karakeep en HTTPS sécurisé.

  

## **Cas d’usage en cybersécurité / homelab**

  

Dans un homelab de cybersécurité, Karakeep peut servir de  **plate-forme de veille et de documentation**. On peut y centraliser des liens vers des CVE, des tutoriels, des articles de blog, des outils de pentest, etc. Karakeep autorise l’import de flux  **RSS**  : en s’abonnant aux blogs de sécurité ou alertes techniques, il « pioche » automatiquement les nouveaux articles  . La fonction de recherche plein texte facilite ensuite de retrouver une information précise (ex. vulnérabilité, commande Linux…). On peut aussi annoter les contenus ou prendre des notes internes pour les réutiliser.

  

De manière générale, Karakeep crée une  **bibliothèque de ressources**  partagée. Comme le souligne l’outil Reminiscence (évoqué sur Awesome-Homelab), un bookmark manager auto-hébergé est idéal pour bâtir une base de connaissances personnelle ou collaborative  . Il permet notamment de regrouper et d’archiver des liens utiles à une équipe (ingénieurs, pentesteurs) ou à un projet de recherche. En somme, Karakeep transforme votre collection de favoris en un véritable centre de documentation et de veille sur mesure.
