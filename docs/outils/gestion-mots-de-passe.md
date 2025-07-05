# **Gestionnaires de mots de passe auto-hébergés**

  

Dans un homelab, on privilégie souvent des gestionnaires auto-hébergés pour garder le contrôle total des données sensibles. Voici un comparatif des solutions les plus populaires :

-   **Vaultwarden (Bitwarden RS)**  : alternative légère à Bitwarden écrite en Rust, offrant les mêmes fonctionnalités (application web, extensions navigateur, apps mobiles) tout en étant très économe en ressources  . 100 % compatible avec les clients officiels Bitwarden, il permet de partager des mots de passe via des « organisations » (famille/équipe) et propose de nombreuses fonctions premium gratuitement  . En revanche, Vaultwarden n’a pas fait l’objet d’audits de sécurité officiels (seul Bitwarden officiel publie des rapports d’audit)  , et dépend du support communautaire pour les mises à jour et la sécurité.
    
-   **Passbolt**  : conçu pour les équipes et la collaboration, Passbolt permet de partager finement les secrets entre utilisateurs (permissions par mot de passe ou dossier, équipes, etc.) et possède une interface web très intuitive  . Sa sécurité est renforcée par un chiffrement bout-en-bout basé sur OpenPGP : chaque mot de passe est chiffré avec la clé publique de chacun des utilisateurs autorisés, la clé privée ne quittant jamais le client  . Passbolt est régulièrement audité par des tiers (tests d’intrusion, SOC2, etc.)  , ce qui lui confère un haut niveau de confiance. En contrepartie, l’usage quotidien repose sur l’extension de navigateur (obligatoire) et certaines fonctionnalités avancées (LDAP, rapports) ne sont disponibles que dans la version Pro payante.
    
-   **KeePassXC + Syncthing**  : KeePassXC est un gestionnaire local multiplateforme (Windows/Linux/macOS) qui stocke les mots de passe dans un fichier chiffré (.kdbx)  . La base peut être synchronisée entre appareils via Syncthing ou tout autre service (Nextcloud, Dropbox, etc.)  . On protège ce fichier par un mot de passe maître et, en option, un fichier-clé ou un dispositif matériel (par exemple YubiKey)  . Cette solution « old-school » est très sûre (AES-256, certifié ANSSI) et totalement hors-ligne, mais elle est plus adaptée à un usage individuel ou familial – le partage simultané d’un même fichier par plusieurs utilisateurs peut entraîner des conflits et ne gère pas les rôles utilisateurs.
    
-   **Autres solutions**  : Par exemple, si vous utilisez Nextcloud, l’application  **Passwords**  est un gestionnaire intégré qui offre une interface web moderne et le partage de mots de passe entre utilisateurs Nextcloud  . D’autres projets open source existent (Psono, Teampass, Bitwarden Server officiel en auto-hébergement, etc.), mais Vaultwarden, Passbolt et KeePassXC couvrent déjà la plupart des besoins courants dans un homelab.
    

  

## **Sécurité et chiffrement**

  

Tous ces gestionnaires chiffrent vos données  _client-side_  (zéro connaissance). Vaultwarden/Bitwarden utilise AES-256 pour le coffre et dérive la clé depuis le mot de passe maître (PBKDF2 ou Argon2)  . Passbolt repose sur OpenPGP (RSA 2048) pour chiffrer chaque secret avec la clé publique des utilisateurs concernés, la clé privée n’étant jamais envoyée au serveur  . KeePassXC chiffre aussi la base de données par AES-256 (avec chiffrement ChaCha20 optionnel)  . Ainsi, dans tous les cas, seul l’utilisateur possédant le mot de passe maître (et la clé privée dans le cas de Passbolt) peut déchiffrer les données  .

  

**Authentification à deux facteurs (2FA)**  : Bitwarden/Vaultwarden supporte plusieurs méthodes 2FA – applications TOTP (Google Authenticator, Authy, etc.), clés matérielles FIDO2/WebAuthn (YubiKey, Titan, etc.) et email/Duo pour les comptes premium  . Passbolt exige déjà à la base deux facteurs (la clé privée + la passphrase) et peut être configuré pour accepter des facteurs supplémentaires (YubiKey, TOTP, etc.)  . KeePassXC ne gère pas directement le 2FA, mais on peut considérer un fichier-clé ou un périphérique YubiKey comme second facteur en plus du mot de passe.

  

**Audit & conformité**  : Bitwarden (et donc Vaultwarden par héritage) fait l’objet d’audits annuels par Cure53 et d’autres laboratoires  , avec certifications (ISO27001, SOC2) et bug bounty. Vaultwarden, en tant que projet communautaire, ne publie pas d’audits formels  . Passbolt a passé avec succès plusieurs audits de sécurité et conformité récents  , et son code est open source et 100 % auditable  . KeePassXC (et KeePass) ont une longue histoire de robustesse (KeePass 2.10 a été certifié par l’ANSSI)  , même s’ils n’ont pas d’audit de troisième partie régulier comme les précédents.

  

**Stockage local vs cloud**  : Tous ces gestionnaires sont conçus pour être auto-hébergés, sans dépendre d’un tiers cloud. Par exemple, Passbolt peut être installé sur votre propre serveur (de la Raspberry Pi à un cluster HA)  , et il fonctionne de manière autonome derrière votre firewall. Vaultwarden se déploie aussi sur votre serveur perso ou VPS. KeePassXC garde le fichier en local et on recommande de le sauvegarder/synchroniser via un service de confiance (Syncthing, Nextcloud). L’idée commune est que  _vos mots de passe restent sous votre contrôle_, sur votre infrastructure privée  , assurant ainsi un niveau maximal de confidentialité.

  

## **Cas d’usage en homelab**

  

Un homelab regroupe souvent plusieurs utilisateurs (famille, petit bureau). Voici quelques scénarios typiques :

-   **Partage multi-utilisateurs**  : Passbolt est taillé pour le travail d’équipe. Il permet de créer des groupes, gérer des permissions fines et partager facilement des mots de passe ou des dossiers entre plusieurs utilisateurs (avec des rôles d’accès)  . Vaultwarden/Bitwarden offre aussi des fonctionnalités « organisations » pour partager des coffres communs (idéal pour une famille ou un petit groupe), mais les fonctions avancées (rapports, provisioning) sont réservées à la version payante. KeePassXC n’a pas de fonction de partage en natif : on se contente généralement de partager le fichier  .kdbx  via Syncthing, ce qui reste manuel et sans contrôle d’accès.
    
-   **Partage familial/équipe**  : Par exemple, dans une famille, on peut créer une organisation Vaultwarden où chacun a son compte personnel et on y ajoute les identifiants communs (Wi-Fi, compte Netflix familial, etc.). En milieu professionnel ou dans un bureau collaboratif, Passbolt facilite le partage organisé de comptes (serveurs, bases de données, applications) avec historique et journaux, garantissant que seuls les membres d’une équipe autorisée y ont accès. L’application Nextcloud Passwords (si vous utilisez Nextcloud) permet également le partage de mots de passe entre utilisateurs Nextcloud de manière transparente  .
    
-   **Intégration LDAP / annuaire**  : Dans un homelab ou une PME, on peut vouloir synchroniser les utilisateurs depuis un annuaire central (AD/LDAP). Passbolt Pro (version commerciale) propose un connecteur LDAP/AD qui importe automatiquement les utilisateurs et groupes de l’annuaire dans Passbolt  , simplifiant la gestion des comptes. En revanche, Vaultwarden/Bitwarden et KeePassXC n’intègrent pas nativement LDAP (on doit créer manuellement chaque compte Bitwarden). Notez qu’il existe des moyens détournés (SSO ou script de provisioning) pour Bitwarden, mais rien de intégré comme chez Passbolt.
    

  

## **Installation et intégration (Docker, reverse proxy, sauvegardes)**

  

Ces gestionnaires se déploient généralement en conteneurs Docker pour la facilité. Par exemple,  **Vaultwarden**  s’installe en quelques commandes Docker Compose : on utilise l’image  vaultwarden/server  et on monte un volume local (par ex.  ./vw-data) pour stocker la base de données chiffrée, les pièces jointes et les logs  . Le guide recommande d’utiliser un tag de version fixe plutôt que  latest  pour garder le contrôle des mises à jour  .  **Passbolt CE**  fournit également un fichier  docker-compose.yml  d’exemple à télécharger  : il faut le configurer (URL publique  APP_FULL_BASE_URL, paramètres SMTP, etc.) puis lancer  docker compose up -d. Un conteneur pour la base de données (MariaDB), un autre pour PHP/Redis, etc., seront démarrés automatiquement  . KeePassXC est une application client (Linux, Windows, Mac) – on l’installe via les paquets/flatpak habituels sur chaque poste, et on utilise Syncthing pour synchroniser le fichier  .kdbx  entre machines si nécessaire.

  

Il est recommandé de placer ces services derrière un  **reverse proxy**  sécurisé (Caddy, Nginx, Traefik, etc.) pour gérer HTTPS. Par exemple, on peut configurer Caddy avec un hôte  password.votredomaine.com  qui fait  reverse_proxy vaultwarden:80  vers le conteneur Vaultwarden, en ajoutant l’en-tête  X-Real-IP  pour que l’application voie l’IP client réelle (utile pour les logs et Fail2Ban)  . De même, Passbolt peut être exposé en HTTPS sur son propre domaine, avec certificat Let’s Encrypt automatique.

  

**Sauvegardes**  : c’est un point critique. Pour Vaultwarden, il faut copier régulièrement tout le dossier de données (vw-data), car il contient la base chiffrée, les attachements et les configurations  . Il faut aussi sauvegarder le fichier  docker-compose.yml  et la configuration du proxy (Caddyfile ou Nginx). Pour Passbolt, la procédure est de faire un  mysqldumpde la base de données et de copier les clés GPG serveur (serverkey.asc  et  serverkey_private.asc) depuis le conteneur  . Ne pas oublier non plus les fichiers de configuration/d’environnement (.env). Pour KeePassXC, on sauvegarde simplement le fichier  .kdbx  (et le fichier-clé si utilisé). Ces sauvegardes doivent être stockées hors-site (NAS, autre cloud privé) et testées régulièrement  .

  

En résumé, un déploiement sécurisé comprend un conteneur pour chaque composant (moteur de base de données, application, proxy), la configuration HTTPS/2FA, et un plan de sauvegarde rigoureux. En suivant ces bonnes pratiques (chiffrement fort, double authentification, isolation réseau, audits, etc.), vous obtiendrez un gestionnaire de mots de passe auto-hébergé fiable et adapté à votre homelab  .
