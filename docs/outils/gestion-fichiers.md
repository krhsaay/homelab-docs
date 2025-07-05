# **Gestion de fichiers auto-hébergée**

  

Les solutions de gestion de fichiers self-hosted permettent d’accéder et partager ses documents via une interface web en conservant ses données chez soi. Parmi les plus populaires,  **File Browser**  (un gestionnaire de fichiers web léger) et  **Nextcloud Files**  (une suite complète de sync & partage) se distinguent. D’autres alternatives existent (Seafile, Pydio Cells, FileRun, Filestash, etc.) offrant chacune des fonctionnalités particulières. L’objectif est de comparer ces solutions selon leur interface, permissions, synchronisation, accès distant et protocoles supportés, en tenant compte de l’usage en « homelab » (stockage perso, partage local/externe, sauvegardes, multi-utilisateur) et des aspects de sécurité (chiffrement, audit, SSO, HTTPS, droits granulaires).

  

## **Comparaison technique des solutions**

-   **File Browser**  : c’est un gestionnaire de fichiers web simple et rapide  . On l’installe sur un serveur en pointant un répertoire racine (filebrowser -r /chemin/vers/fichiers), et on accède aux fichiers via une interface web épurée. Il permet de  _téléverser, supprimer, prévisualiser, renommer et modifier_  les fichiers, avec gestion multi-utilisateurs  (avec rôles Admin/User). Il n’inclut pas de clients de synchronisation automatique : tout passe par l’interface HTTP. File Browser prend en charge l’édition de fichiers en ligne et des commandes personnalisées, mais reste limité à ses propres fonctionnalités. Il ne propose pas de chiffrement côté serveur ni d’audit avancé (log minimal). 
    
-   **Nextcloud Files**  : c’est une plateforme cloud complète (sync & share) qui propose une  _interface web moderne_, des clients de synchronisation (Windows/Mac/Linux) et des applis mobiles  . Elle offre la  **collaboration en temps réel**(édition de documents, notes, etc.), la recherche unifiée et le partage sécurisé. Nextcloud supporte de nombreux protocoles et intégrations : on peut accéder aux fichiers via WebDAV, FTP, SMB/CIFS, NFS, SharePoint, ou en stockant les données sur des stockages externes (S3, OpenStack Swift, etc.)  . En termes de permissions, Nextcloud permet des partages publics avec mot de passe/expiration, ou privés aux utilisateurs/groupes, avec droits (lecture/écriture/commentaire) très granulaires  . Côté sécurité, Nextcloud inclut des  **fonctionnalités de cryptage puissant**(chiffrement côté serveur et chiffrement de bout-en-bout pour les partages sensibles)  , ainsi que des contrôles d’accès aux fichiers basés sur des règles avancées et des protections anti-ransomware/brute-force  . Des apps Nextcloud permettent d’ajouter SSO/SAML, LDAP/AD et MFA pour l’authentification.
    
-   **Autres solutions**  : on trouve notamment  **Pydio Cells**  (plateforme de partage de fichiers orientée entreprise) et  **Seafile**. Pydio se veut « privacy-friendly » et permet aux équipes de travailler ensemble sur un cloud interne  . Seafile est axé synchronisation et partage fiable, avec chiffrement des bibliothèques et travail en groupe  . Ces solutions incluent généralement leurs propres clients desktop/mobiles et protocoles (par ex. WebDAV pour Pydio).  **Filestash**  (anciennement Nuage) est un client web multi-protocoles (FTP/SFTP/S3/WebDAV/Git, etc.) qui sert à monter diverses sources de fichiers.  **FileRun**  est un gestionnaire Web (type Google Drive) avec base de données, partage de liens et visionneuse intégrée. Chacune diffère par la richesse fonctionnelle et l’ergonomie : Nextcloud propose le plus de fonctionnalités (agenda, mail, chat, bureautique en ligne…), File Browser offre la plus grande légèreté et simplicité, Pydio et Seafile se positionnent sur la collaboration d’équipe. L’utilisation de l’une ou l’autre dépendra du besoin : simple partage de fichiers (File Browser), cloud personnel complet (Nextcloud), synchronisation de dossiers collaboratifs (Seafile) ou intégration multi-protocoles (Filestash).
    

  

## **Intégration avec différents NAS**

  

Dans un homelab on utilise souvent un NAS (Network Attached Storage) pour centraliser les données. Les principales plateformes NAS (Synology, QNAP, TrueNAS, OpenMediaVault, Unraid, etc.) proposent des outils de gestion de fichiers et des protocoles variés. Par exemple,  **Synology DSM**  inclut  _File Station_, un gestionnaire centralisé accessible par navigateur, et  **Synology Drive**  pour la synchronisation. Les fichiers du NAS sont exposés via les protocoles  **SMB, NFS, AFP, FTP et WebDAV**  , rendant l’accès possible depuis Windows, macOS ou mobiles. Synology permet de configurer des partages publics ou privés avec mots de passe, expiration, quotas et ACLs fines  . De même,  **QNAP**  offre File Station avec des fonctionnalités proches.

  

D’autres OS de NAS (« DIY NAS » sur serveurs génériques) supportent aussi ces logiciels : par exemple,  **OpenMediaVault (OMV)**  sous Linux ou  **TrueNAS**  (FreeBSD) permettent d’installer Nextcloud ou File Browser via Docker/jails. On peut héberger un conteneur Nextcloud sur un NAS Synology, QNAP (via Container Station) ou sur un système comme OMV grâce au plugin Docker (« OMV-Extras ») ou via LXC sur Proxmox/TrueNAS SCALE. Les NAS libres offrent souvent des plugins ou images pour Nextcloud/OnlyOffice. Ainsi, on peut soit utiliser le gestionnaire natif du NAS, soit déployer l’application tierce (Nextcloud, FileBrowser, etc.) sur le même matériel. En résumé, quel que soit le type de NAS, les fichiers peuvent être servis directement (partage SMB/NFS traditionnel) et/ou via une solution web auto-hébergée qui profite alors du stockage du NAS.

  

## **Cas d’usage en homelab**

-   **Stockage personnel et synchronisation**  : Nextcloud est idéal pour une « private cloud » maison – chaque appareil peut synchroniser des dossiers (photos, documents) vers le serveur Nextcloud, offrant un backup et une disponibilité multi-appareils  . File Browser peut servir de dépôt central simple, auquel on charge manuellement les fichiers via l’interface. Dans les deux cas, on accède à ses données depuis le réseau local ou Internet (avec des DNS dynamiques ou domaine personnalisé).
    
-   **Partage local et externe**  : on peut partager des fichiers avec la famille ou des collaborateurs. Nextcloud permet de créer des liens publics protégés (mot de passe, expiration) ou de partager directement avec d’autres comptes Nextcloud, très pratique pour échanger des gros fichiers ou dossiers. File Browser offre un partage plus basique (compte utilisateur dédié, liens directs en lecture seule). Dans un réseau local (LAN), on peut simplement mapper le stockage réseau (SMB/NFS) ; pour l’extérieur, on ouvrira le port HTTPS ou via un reverse-proxy (voir sécurité)  .
    
-   **Sauvegardes**  : on peut sauvegarder les données du NAS vers Nextcloud (utiliser des tâches cron, rclone ou l’app «External storage» de Nextcloud pour répliquer un partage réseau vers Nextcloud). Nextcloud propose aussi une app de sauvegarde pair-à-pair (Backups Nextcloud) permettant de dupliquer un Nextcloud chez un ami. File Browser ne gère pas explicitement de sauvegardes, mais on peut sauvegarder le répertoire de fichiers via des outils externes (rsync, BorgBackup, Snapshots ZFS).
    
-   **Multi-utilisateur et collaboration**  : Nextcloud gère nativement plusieurs utilisateurs et groupes avec quotas et droits différents, idéal pour un homelab familial ou associatif. On peut attribuer des permissions sur dossiers partagés, et avoir différents comptes (invités, experts, etc.). File Browser supporte également plusieurs utilisateurs (un admin et des utilisateurs)  , mais sans la granularité de Nextcloud. D’autres solutions comme Seafile ajoutent le concept de « bibliothèques » partagées en groupe.
    

  

## **Sécurité**

-   **Chiffrement**  : Nextcloud propose le chiffrement côté serveur (de bout en bout pour certains partages sensibles via «File Drop»). L’interface web permet d’activer le chiffrement des données au repos (soit en stockant sur serveur chiffré, soit en E2EE pour les fichiers partagés)  . File Browser ne chiffre pas les fichiers, on s’appuie donc sur le chiffrement au niveau du disque ou volume (ex. LUKS sur le NAS) et sur le HTTPS.
    
-   **Accès réseau sécurisé (HTTPS / Reverse proxy)**  : Il est impératif d’utiliser HTTPS pour protéger les échanges. On place typiquement File Browser ou Nextcloud derrière un reverse-proxy (Nginx, Traefik, Caddy…) fournissant TLS/SSL (certificats Let’s Encrypt). Dans la DMZ du homelab (voir diagramme réseau), on autorise seulement le port 443 vers le serveur de fichiers et on filtre selon les VLAN. Grâce au proxy, on peut également configurer une authentification avancée (Auth JWT, mTLS) si nécessaire.
    
-   **Contrôles d’accès et permissions granulaires**  : Nextcloud permet d’affiner les droits (lecture/écriture/commentaire) à l’échelle du fichier ou dossier partagé  . On peut mettre des ACL Windows/NFS, limiter certains utilisateurs ou groupes, définir des quotas. File Browser se limite aux droits d’un répertoire global par utilisateur (pas de partage public natif, pas d’ACL complexes). Les NAS eux-mêmes (Synology, TrueNAS…) intègrent des ACLs et peuvent protéger les partages par mot de passe.
    
-   **Authentification centrale (SSO/LDAP)**  : Nextcloud peut s’intégrer avec des annuaires LDAP/Active Directory pour gérer les utilisateurs, et des solutions SSO (SAML, OAuth) via apps externes  . Par exemple, on peut connecter Keycloak ou Authelia à Nextcloud. File Browser supporte l’authentification interne ou externe (OAuth ou LDAP possible via configuration). Ainsi, on peut centraliser les comptes sur un annuaire du homelab.
    
-   **Audit et journaux**  : Nextcloud génère des journaux détaillés (logs de connexion, de partage, etc.) pour vérifier les accès et détecter les anomalies. File Browser journalise de manière plus basique (activités d’upload/suppression). On peut compléter avec un SIEM local : par exemple, collecter les logs (via Filebeat/Logstash) du serveur Nextcloud ou du NAS pour les analyser (architecture ELK/Kibana). Le tout afin de détecter d’éventuels accès suspects ou compromissions.
    

  

## **Installation et intégration**

-   **Docker et conteneurs**  : Pour faciliter le déploiement, la plupart de ces outils fournissent des images Docker officielles. Par exemple,  _Nextcloud All-in-One_  propose un conteneur unique (« AIO ») qui installe et configure Nextcloud ainsi que ses dépendances  . File Browser dispose aussi d’une image Docker prête à l’emploi  . On lancera ces conteneurs dans la DMZ ou sur l’hôte, en montant les volumes de données adéquats. Sous Proxmox ou TrueNAS Scale, on peut utiliser un conteneur LXC/VM ou un plugin intégré.
    
-   **LXC / VM**  : Si l’on utilise Proxmox ou TrueNAS, on peut héberger Nextcloud ou File Browser dans une machine virtuelle (avec Debian/Ubuntu) ou un conteneur LXC. Par exemple, on installe Docker dans la VM ou on déploie Nextcloud directement depuis les paquets officiels. Les images Docker sont pratiques, mais on peut aussi compiler/récupérer l’exécutable File Browser en binaire (Linux)  .
    
-   **Reverse proxy et routage**  : On configure un reverse proxy pour gérer le trafic HTTPS/HTTP. Par exemple, avec Nginx on redirige  https://files.mondomaine.fr  vers l’IP interne de Nextcloud, en gérant les certificats. Un reverse proxy peut également fournir un pare-feu applicatif (WAF) ou limiter les taux de connexion. Dans un script d’installation (comme le guide Nextcloud AIO), on voit souvent la commande  docker run … --publish 80:80 --publish 443:443 --publish 8080:8080, mais derrière il est courant d’inverser la polarité : le proxy frontal (port 443) redirige vers le port interne du conteneur  . Il faut configurer dans Nextcloud l’URL externe correcte et éventuellement les en-têtes de proxy (trusted_proxies).
    
-   **Authentification et utilisateurs**  : On créera d’abord un utilisateur administrateur (Nextcloud/AIO génère un mot de passe par défaut à récupérer dans les logs  ). Ensuite, on peut importer ou synchroniser les utilisateurs depuis LDAP/AD pour un accès SSO unique dans le homelab. L’authentification peut être renforcée (2FA, clé U2F, tokens OTP) surtout pour les accès web externes. Il est recommandé d’activer des mots de passe forts et de limiter les tentatives (Nextcloud gère la protection brute-force en natif  ).
    
-   **Reverse proxy / Firewall (DMZ)**  : Dans l’architecture réseau (voir schéma), on place les serveurs de fichiers dans la DMZ « Services » derrière le firewall (OPNsense). Le firewall ouvre les ports nécessaires (443/TCP, 80 pour redirections, 22 si besoin de SFTP) depuis l’Internet vers la DMZ, en isolant les autres VLAN (LAN perso, Lab). Cette configuration « neutre » limite les risques qu’un service compromis atteigne le LAN interne.
    

