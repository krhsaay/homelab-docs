# **Gestionnaires de mots de passe self-hosted pour homelab**

  

## **Vaultwarden (Bitwarden_RS)**

Vaultwarden (anciennement  _bitwarden_rs_) est une implémentation en Rust du serveur Bitwarden, allégée et optimisée pour l’auto-hébergement  . Elle reproduit l’essentiel des fonctionnalités de Bitwarden (mots de passe, cartes, notes sécurisées) avec une empreinte mémoire minimale. Vaultwarden propose notamment la gestion multi-utilisateurs via des organisations et collections partagées, la génération et sauvegarde de mots de passe, ainsi que des clients officiels Bitwarden (extensions de navigateur et applications mobiles) compatibles  . L’installation se fait typiquement avec Docker, par exemple :

```bash
docker run -d --name vaultwarden \
  -e DOMAIN="https://vault.mondomaine.tld" \
  -v /srv/vaultwarden-data:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```
Ce conteneur utilise SQLite par défaut, mais peut aussi se connecter à une base MariaDB/MySQL pour de meilleures performances. La documentation recommande d’utiliser un proxy HTTPS (par exemple NGINX Proxy Manager) car les navigateurs n’autorisent pas le chiffrement  _Web Crypto_  hors contexte sécurisé  .

  

### **Sécurité et authentification**

  

Vaultwarden offre du chiffrement de bout en bout (E2E) comme Bitwarden : les données sont chiffrées côté client avec AES-256 avant d’être envoyées au serveur (modèle  _zero-knowledge_)  . Les options d’authentification à deux facteurs sont riches : codes OTP via une application (Google Authenticator, Authy), clé de sécurité FIDO2/WebAuthn (YubiKey), authentification par email et Duo MFA  . La politique de mots de passe est gérée côté client (génération aléatoire et longueur minimale dans les paramètres utilisateur). Vaultwarden ne supporte pas nativement LDAP/SAML ; c’est une solution plutôt autonome. Comme tous les gestionnaires, il est vivement recommandé de faire des sauvegardes régulières de la base et des fichiers joints – la documentation met en garde contre les pertes de données en cas de défaillance  .

  

### **Installation et maintenance**

  

Vaultwarden se déploie facilement via Docker ou Podman  . Un reverse-proxy (Traefik, NGINX Proxy Manager, Caddy…) est conseillé pour fournir un certificat SSL/TLS (Let’s Encrypt, mkcert, etc.) et exposer le service sur un nom de domaine  . La maintenance se limite à mettre à jour l’image Docker (et migrer la base de données si besoin). L’interface web est simple et fonctionnelle, et l’application Android/iOS de Bitwarden fonctionne sans modification. La communauté est active, mais contrairement au serveur officiel Bitwarden, Vaultwarden n’a pas de bilan public d’audit tiers  . Toutefois, le code est open source (AGPL v3  ) et régulièrement mis à jour.

  

## **Passbolt**

  

Passbolt est un gestionnaire de mots de passe libre conçu pour les équipes et organisations. Il utilise un modèle de chiffrement  _end-to-end_  avec clés publiques-privées : chaque utilisateur génère une paire de clés, et seule la clé privée (protégée par passphrase) reste sur son appareil  . Cette architecture garantit que seul le destinataire peut déchiffrer les mots de passe partagés. Passbolt Community Edition (CE) est entièrement open source (AGPL v3  ) et auto-hébergeable. Il dispose d’une interface web conviviale, d’extensions navigateur (Chrome/Firefox/Edge) et d’applications mobiles hybrides (iOS/Android) et desktop  .

  

### **Fonctions clés**

  

Passbolt se distingue par son focus sur la collaboration : gestion de groupes, dossiers partagés, permissions fines et audit des actions. Toutes les fonctionnalités de partage de mots de passe sont disponibles dans la version gratuite (CE) pour un nombre d’utilisateurs illimité, contrairement à Bitwarden où le partage avancé est réservé aux éditions payantes  . L’interface permet d’organiser des collections hiérarchiques et de partager un mot de passe unique ou tout un dossier vers un autre utilisateur ou une équipe, avec héritage de permissions  . La génération de mots de passe se fait depuis l’extension ou le site, et Passbolt prend en charge la complétion automatique.

  

### **Authentification et sécurité**

  

Passbolt renforce la sécurité par une authentification forte. Les utilisateurs peuvent activer une deuxième couche MFA (généralement un code TOTP) pour se connecter  . Il existe aussi une option LDAP/AD en version Pro : la documentation montre comment configurer LDAPS pour synchroniser les comptes d’entreprise  . Passbolt est audité régulièrement et rend publiques ses conclusions : c’est une application  _security-first_  (modèle Cllé Privée+Chiffrage par mot de passe maître)  . La politique de mots de passe peut être imposée par l’administrateur (longueur, complexité) et les journaux d’audit détaillent toutes les connexions et modifications. Des sauvegardes automatiques de la base (PostgreSQL ou MySQL) sont possibles via des scripts.

  

### **Installation**

  

Passbolt peut être installé via Docker ou directement sur un serveur Linux (packages Debian/CentOS). Un  _docker-compose_  simple ou un script dédié (disponible sur leur site) installe le serveur API, la base de données et l’interface web en une seule commande. Par exemple, leur image Docker regroupe le serveur, le webclient et l’interface d’admin  . Il faut configurer un nom de domaine et fournir un certificat TLS (Let’s Encrypt ou autre). Passbolt propose aussi un connecteur LDAP (cronjob) pour importer les utilisateurs depuis un annuaire d’entreprise.

  

## **KeePassXC (avec synchronisation via Syncthing)**

  

KeePassXC est un gestionnaire de mots de passe multiplateforme (Windows/Mac/Linux) fonctionnant hors ligne. Les mots de passe sont stockés dans un fichier local chiffré (format KDBX, AES-256)  . Son code source est libre (GPLv3  ) et  _on-premise_  : aucun service distant n’est requis  . KeePassXC inclut un générateur de mots de passe et une intégration de navigateur via une application compagnon.

  

### **Usage en homelab**

  

Par défaut, KeePassXC ne gère pas le multi-utilisateur. Pour partager le même fichier de mots de passe entre plusieurs machines (ou utilisateurs), on utilise un outil de synchronisation de fichiers tel que  **Syncthing**. On place le fichier KDBX dans un dossier surveillé par Syncthing sur deux ordinateurs ou plus. Chaque modification est ainsi répliquée cryptée vers les autres appareils. Cette approche élimine le serveur central : tout reste chiffré localement, et vous contrôlez entièrement les sauvegardes. Cependant, il faut veiller à éviter les conflits d’édition simultanée (pas de verrouillage de fichier intégré).

  

### **Sécurité et authentification**

  

KeePassXC offre un chiffrement robuste :  _“Votre base reste toujours chiffrée et aucun serveur distant n’est utilisé”_  . On peut protéger la base par un mot de passe principal seul ou combiné à un fichier-clé ou une clé de sécurité (YubiKey en mode challenge-réponse), apportant une forme de MFA matérielle. KeePassXC ne propose pas d’authentification réseau (pas de SSO ni LDAP) car il est conçu pour une utilisation locale. Les sauvegardes ne sont rien de plus que des copies du fichier KDBX ou des dumps d’urgence exportés dans un fichier chiffré hors ligne. Dans un homelab personnel, c’est idéal : simple à déployer (installez l’Appli ou le paquet depuis votre distribution), 100% privé et auditée (le code est ouvert). En revanche, il n’existe pas d’app mobile officielle KeePassXC – on utilise alors KeePassDX (Android) ou Strongbox (iOS) pour accéder au fichier synchronisé.

  

## **Bitwarden (serveur officiel)**

  

Bitwarden est le projet d’origine, dont Vaultwarden est un clone léger. Le serveur Bitwarden officiel (techniquement open source AGPL v3) est écrit en .NET Core et déployé via plusieurs conteneurs Docker  . Il fournit les mêmes clients (web, mobile, extensions) et fonctionnalités de base, mais exige un déploiement plus complexe (script  bitwarden.sh  ou  _docker-compose_  orchestrant plusieurs services)  . Bitwarden Cloud propose en plus des éditions Enterprise avec annuaire (LDAP/AD via Directory Connector), SSO (SAML), rapports d’audit avancés et politique organisationnelle.

  

### **Points forts et limites**

  

Bitwarden bénéficie d’une longue trajectoire et d’un suivi professionnel : le code est vérifié publiquement, et des audits par Cure53 sont régulièrement publiés  . Il prend en charge l’authentification à deux facteurs (OTPs, YubiKey, WebAuthn) et la récupération/prise de contrôle (Emergency Access). Les fonctionnalités communautaires gratuites incluent le chiffrement E2E AES-256 “zero-knowledge” et un stockage illimité d’éléments  . En revanche, le partage d’éléments en mode équipe est limité en version libre (plan free n’autorise pas le partage détaillé)  , et l’installation auto-hébergée demande des ressources (RAM, CPU) et une gestion d’infrastructure Docker/SSL plus conséquentes.

  

## **Psono**

  

Psono est un gestionnaire de mots de passe pensé pour les entreprises et équipes, open source et auto-hébergeable. Son architecture combine chiffrement côté client et multiples couches de sécurité  . Les mots de passe sont chiffrés localement avant envoi (« multi encryption » : chiffrement client, TLS, chiffrement de stockage)  . Le système de partage s’effectue via des organisations et groupes, et il existe un tableau de bord administrateur pour gérer les licences et les utilisateurs (en version Enterprise).

  

### **Fonctionnalités et sécurité**

  

Psono propose une interface web, des extensions navigateur (Chrome/Firefox/Edge) et des applications mobiles natives iOS/Android  . Il se veut  _“responsable de la protection des données”_  :  **« il chiffe localement sur le device avant envoi, et ne stocke que les données déjà chiffrées »**  . Les clés privées restent sur l’appareil client et ne passent jamais sur le serveur. Psono supporte le partage de mots de passe au sein d’équipes, avec gestion de droits et répertoire d’utilisateurs. On peut importer des comptes depuis un annuaire ou SSO via des modules (LDAP/SAML sont possibles en entreprise). Il n’est pas clair si la version gratuite supporte LDAP, mais la documentation mentionne l’utilisation de Docker pour déployer facilement le serveur et l’interface  . Des audits externes ont déjà été effectués sur Psono, ce qui renforce la confiance dans son modèle cryptographique.

  

### **Déploiement**

  

Psono propose une image Docker unique (entreprise ou communautaire) qui embarque le serveur, le client web et la console d’administration  . Le déploiement type utilise Docker Compose ou Kubernetes. Pour un homelab, on peut simplement lancer ce conteneur derrière un reverse-proxy avec TLS, puis se connecter via l’interface web. La maintenance implique de faire tourner le conteneur à jour et de sauvegarder la base de données (PostgreSQL) ainsi que les clés de chiffrement. Psono possède aussi une couche de  _fileserver_  pour stocker des fichiers joints aux mots de passe (ex : licences, QR codes).

  

## **Autres options pertinentes**

-   **KeePass (Windows)**  – ancêtre de KeePassXC. Fonctionne localement sur Windows (non multiplateformes). On peut aussi utiliser la version Mono sur Linux. Partage via Syncthing similaire à KeePassXC.
    
-   **Pass (cli)**  – gestionnaire Unix en ligne de commande (GNU Pass, format gpg), adapté aux utilisateurs avancés. Sans interface web ni mobile natif, mais très léger et scriptable.
    
-   **Buttercup**  – gestionnaire simple open source (Electron) avec vaults chiffrés. Possède des applications desktop/mobile et un mode auto-hébergé via Caddy, mais moins de fonctionnalités d’équipe.
    
-   **Teampass**  – gestionnaire PHP/MySQL pour équipes (GPL v3  ). Offre dossiers, 2FA (Google Auth), rôles, mais l’interface est plus complexe et le projet moins actif récemment. Installation via paquet ou Docker.
    
-   **HashiCorp Vault**  – plutôt orienté secrets (devops) qu’accès utilisateur classique. Très robuste (authentification LDAP, Cloud IAM, etc.), mais moins convivial pour un usage métier de mots de passe de sites web.
    
-   **Padloc**,  **Passky**,  **Padloc**, etc. – quelques autres outils open source existent, mais ils sont souvent plus expérimentaux ou limités (par ex. pas d’app mobile).
    

  

## **Comparatif technique**

TABLEAU

* La licence exacte de Psono n’est pas précisée sur leur site, mais le code est en grande partie ouvert.

  

## **Sécurité et bonnes pratiques**

  

Quel que soit le choix, l’élément clé est le chiffrement de bout en bout : la plupart des solutions citées (Vaultwarden/Bitwarden, Passbolt, Psono) chiffrent localement les données avant envoi  . Ainsi, le serveur ne voit jamais les mots de passe en clair. Par ailleurs, vérifiez que votre instance est protégée par HTTPS et que vous appliquez un fort mot de passe maître initial. Les fonctionnalités MFA (OTP TOTP, clés U2F/WebAuthn) ajoutent une barrière supplémentaire : Vaultwarden/Bitwarden supportent un large panel d’options  , Passbolt est conçu pour l’authentification double-facteur et l’impose aux utilisateurs professionnels  , et même Teampass intègre des 2FA comme Google Authenticator  .

  

Pour les entreprises ou environnements multifamille, l’intégration à un annuaire est aussi importante. Bitwarden Enterprise offre un  _Directory Connector_  pour LDAP/AD (au niveau payant), Passbolt Pro permet de synchroniser via LDAPS  , et Teampass/Psono peuvent être connectés à un LDAP. En auto-hébergement pur, on mettra souvent en place Authelia ou Keycloak en frontal pour un SSO global.

  

Enfin, la gestion des sauvegardes est cruciale : sauvegardez la base de données (SQL ou SQLite) et les fichiers (PJ) régulièrement. Les fournisseurs comme Vaultwarden rappellent qu’ils ne sont pas responsables des pertes de données et conseillent les backups fréquents  . Utilisez aussi un gestionnaire de configuration (Ansible, etc.) pour déployer et restaurer votre service au besoin.

  

## **Scénarios d’usage typiques**

-   **Homelab personnel/familial**  – Une solution légère comme Vaultwarden ou KeePassXC convient souvent. Par exemple, Vaultwarden peut être déployé dans un conteneur Docker derrière NGINX Proxy Manager avec un domaine personnel (ex:  vault.mondomaine.fr) et Let’s Encrypt. Chaque membre de la famille crée un compte utilisateur et partage des dossiers communs (wifi, routeur, etc.) via les organisations Bitwarden. KeePassXC reste utile pour un utilisateur unique : sa base locale stockée sur Syncthing entre PC et smartphone permet d’accéder aux mots de passe même sans connexion internet.
    
-   **Environnement de test/sécurisé**  – Pour un labo dédié (VM/containers), Passbolt ou Psono offrent un cadre de tests multi-utilisateurs. Par exemple, on peut lancer Passbolt sur une VM et créer des utilisateurs « test » pour valider l’intégration SSO (LDAP) ou l’API CLI. Grâce à Docker, on peut détruire/réinstaller facilement en cas d’erreur.
    
-   **Intégration à un parc existant**  – Dans une infrastructure où l’authentification unique est déjà en place, choisissez un gestionnaire offrant la compatibilité (Bitwarden Enterprise pour SAML, Passbolt LDAP, ou configurer Authelia devant Vaultwarden). Par exemple, en plaçant Vaultwarden derrière Authelia/NGINX Proxy Manager, on force une authentification centrale (par OAuth/LDAP) avant d’atteindre le coffre. L’intégration via reverse proxy permet aussi de simplifier l’accès : NGINX Proxy Manager automatisera la génération des certificats HTTPS pour chaque service web de l’homelab.
    

  

## **Déploiement et exemples pratiques**

-   **Docker Compose (Vaultwarden)**  – Un fichier  docker-compose.yml  minimal pour Vaultwarden est :

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      - DOMAIN=https://vault.mondomaine.fr
    volumes:
      - ./vw-data/:/data/
    ports:
      - "80:80"
```

-   Ce conteneur stocke les données sur  ./vw-data/. On crée ensuite un proxy (Traefik, NPM) pointant sur  vault.mondomaine.fr.
    
-   **Docker Compose (Passbolt)**  – Passbolt propose un  _docker-compose_  complet (base de données + serveur) dans sa documentation. Il suffit de cloner leur repo et d’éditer l’.env  pour indiquer le domaine et le mot de passe GPG serveur. Par exemple :
```bash
git clone https://gitlab.com/passbolt/passbolt_docker.git
cd passbolt_docker
cp envvars.example .env
# modifier .env (DOMAIN, GPG keys, etc.)
docker-compose up -d
```

-   L’interface web de Passbolt sera accessible sur  https://passbolt.mondomaine.fr.
    
-   **LXC / VM**  – On peut déployer ces services dans des conteneurs LXC (Proxmox) ou des VM légères. Par exemple, créer un LXC Debian, installer Docker (apt install docker.io docker-compose), puis suivre les mêmes commandes Docker. Des images prêtes à l’emploi existent souvent pour Bitwarden et Vaultwarden sur Docker Hub. L’avantage du LXC est l’isolation réseau et système tout en économisant des ressources comparé à une VM pleine.
    
-   **NGINX Proxy Manager**  – Intégrez chaque service au NPM en créant une entrée de proxy pour son nom de domaine, en validant le certificat Let’s Encrypt. Par exemple, pointez  vault.mondomaine.fr  vers l’IP du conteneur Vaultwarden (port 80). Répétez pour  passbolt.mondomaine.fr  et d’autres. NPM gère l’HTTPS automatiquement. Un firewall (iptables, UFW) peut limiter l’accès aux seuls ports 80/443 pour ces conteneurs.
    
-   **Intégration Reverse Proxy / SSO**  – Pour un accès unifié, on peut associer Vaultwarden à un serveur SSO (ex : Authelia) monté en frontal. Dans NPM, on configurerait un  _proxy host_  sur  vault.mondomaine.fr  qui redirige vers Authelia pour authentifier l’utilisateur via LDAP, puis vers Vaultwarden. Ainsi, on dispose d’un accès centralisé et authentifié unique à tous les services (Mastodon, Nextcloud, Vaultwarden…).
    

  

En résumé, de nombreuses solutions self-hosted existent pour un homelab.  **Vaultwarden**  et  **KeePassXC**  couvrent le cas des utilisateurs techniques cherchant la simplicité,  **Passbolt**  et  **Psono**  s’adressent aux environnements collaboratifs, tandis que  **Bitwarden (officiel)**  et  **Teampass**  conviennent aux déploiements plus formels. Le choix dépendra de la taille de votre organisation, de la nécessité de gestion multi-utilisateurs et du niveau de sécurité exigé. Quel que soit votre choix, privilégiez toujours le chiffrement end-to-end, les sauvegardes régulières et la mise à jour du système.

