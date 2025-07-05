# **VPN et Torrent : AirVPN vs Gluetun, Intégration & Isolement réseau**

  

## **Contexte et enjeux de la protection VPN pour le torrent**

  

Télécharger via BitTorrent sur un homelab  _self-hosted_  présente des risques en termes de confidentialité et de sécurité. Un VPN de qualité permet de chiffrer le trafic P2P et de masquer l’adresse IP publique réelle, évitant ainsi d’exposer son identité aux pairs du torrent et aux tiers malveillants. De plus, un bon VPN aide à contourner d’éventuelles restrictions de FAI sur le trafic P2P. Cependant, tous les services VPN ne se valent pas pour cet usage : il faut examiner la  **performance**(vitesse, stabilité), la  **sécurité**  (chiffrement, politique de logs), la  **compatibilité P2P**  (autorisation du torrent, support du port forwarding) et les moyens d’intégration technique (clients, conteneurs, etc.)  . Nous comparons ci-dessous AirVPN, Gluetun et d’autres services similaires, puis abordons l’intégration avec des clients BitTorrent et les bonnes pratiques d’isolement réseau (VLAN, pare-feu), sans oublier la prévention des fuites DNS, l’usage d’un  _kill switch_  et la question des logs.

  

## **Comparatif : AirVPN vs Gluetun et autres VPN adaptés au torrent**

  

**AirVPN**  est un fournisseur VPN réputé dans la communauté pour son sérieux technique et son respect de la vie privée. Basé en Italie, AirVPN applique une politique stricte de  **no-log**, ne conservant  **aucune donnée**  permettant de relier une IP et un horaire à un utilisateur  . Il autorise  **BitTorrent sur tous ses serveurs**  sans discrimination et fournit un service de  **port forwarding (transmission de port entrant)**  indispensable aux utilisateurs torrent  . En pratique, AirVPN permet à chaque client d’ouvrir plusieurs ports via son interface (par exemple pour améliorer le  _seeding_). Côté sécurité, son application  _Eddie_  (libre et multiplateforme) inclut la fonctionnalité  **Network Lock**, un  _kill switch_  qui bloque tout trafic hors VPN en cas de déconnexion et prévient les fuites IPv6/DNS ou WebRTC  . AirVPN supporte OpenVPN (chiffrements AES-256-GCM et CHACHA20-POLY1305) et WireGuard pour un meilleur débit  . En termes de  **performance**, AirVPN offre des débits corrects mais son réseau, plus restreint que certains concurrents, peut être légèrement moins rapide que les poids lourds du marché sur des tests globaux. Cependant, avec WireGuard et un serveur bien choisi, de nombreux utilisateurs parviennent à saturer leur bande passante sans problème dans un usage torrent normal. AirVPN se distingue enfin par un support de la communauté actif (forums techniques) et l’acceptation de nombreux moyens de paiement anonymes (cryptomonnaies comme Monero, etc.)  .

  

**Gluetun**  n’est pas un fournisseur VPN à proprement parler, mais un  **client VPN léger en conteneur Docker**  qui sert de passerelle pour d’autres applications. Il supporte de nombreux fournisseurs VPN en OpenVPN ou WireGuard (AirVPN, Mullvad, ProtonVPN, PIA, etc. sont pris en charge)  . L’idée est qu’au lieu d’utiliser l’application native du VPN, on utilise Gluetun comme point d’accès VPN dans un environnement auto-hébergé.  **Performance**  : Gluetun ajoute très peu de surcoût, car il se base sur OpenVPN/WireGuard en arrière-plan. Avec WireGuard (supporté pour la plupart des grands VPN dans Gluetun), on obtient généralement de meilleurs débits et une utilisation CPU réduite par rapport à OpenVPN, ce qui est idéal sur des matériels modestes (NAS, mini-PC)  .  **Sécurité**  : Gluetun intègre d’office un  **pare-feu interne (kill switch)**  ne laissant passer que le trafic nécessaire vers les serveurs VPN et éventuellement le LAN autorisé  . Ainsi, en cas de chute du VPN, les conteneurs connectés à Gluetun n’ont pas d’accès Internet direct, évitant les fuites. Gluetun configure également le  **DNS sur TLS**  (DNS chiffré) pour les conteneurs connectés, empêchant les fuites DNS en forçant les requêtes DNS à passer par le VPN  .  **Compatibilité torrent**  : Gluetun étant compatible avec AirVPN et consorts, on bénéficie des mêmes avantages (ex: port forwarding) selon le fournisseur choisi. Par exemple, si on utilise AirVPN via Gluetun, il suffit de renseigner les ports attribués par AirVPN dans la configuration (variable d’environnement  FIREWALL_VPN_INPUT_PORTS  de Gluetun et configuration du port dans le client torrent) afin d’accepter les connexions entrantes  . Gluetun facilite aussi le  **partage de la connexion VPN**  entre plusieurs applications : un seul conteneur Gluetun peut protéger plusieurs clients BitTorrent ou autres services (Sonarr, Radarr, etc.) en les joignant au réseau Docker de Gluetun  . En somme, Gluetun offre une solution  **flexible**  et  **open source**  pour intégrer un VPN dans un homelab, avec un contrôle fin (logs locaux minimalistes, paramètres modulables) – à condition bien sûr de disposer d’un compte chez un bon fournisseur VPN sous-jacent.

  

**Autres services VPN à considérer**  : plusieurs fournisseurs concurrents d’AirVPN sont appréciés des utilisateurs P2P, chacun avec ses atouts et limites. Parmi les plus recommandés figurent :

-   **Mullvad**  (Suède) – Énorme réputation de confidentialité (audits publics, incident où aucun log n’a pu être saisi lors d’un mandat  ). Forfait simple à 5 €/mois, anonymat possible (compte numéro, paiement en liquide envoyé par courrier).  **Inconvénient**  en 2023 : Mullvad a supprimé le port forwarding  . Cela limite la connectivité entrante pour le torrent (pas idéal pour le  _seeding_  intensif ou des usages type serveur). Mullvad reste excellent en sécurité et a de bons débits en WireGuard, mais sans ouverture de ports, un torrent sera en mode passif (connectivité un peu réduite). À prendre en compte selon vos besoins.
    
-   **ProtonVPN**  (Suisse) – Service moderne orienté confidentialité (code client open source, audits), bénéficiant d’une infrastructure performante. ProtonVPN  **autorise le P2P**  sur des serveurs dédiés et propose du  **port forwarding**  (sur les offres payantes  _Plus/Visionary_)  . Leurs vitesses sont très élevées sur les serveurs récents (10 Gbit)  , souvent supérieures à Mullvad  . En contrepartie, ProtonVPN est un peu plus cher et son interface est moins technique qu’AirVPN. C’est un bon choix si la vitesse est prioritaire et que l’on veut un écosystème (ils offrent aussi mail sécurisé, etc.), tout en ayant les ports ouverts pour le torrent.
    
-   **Private Internet Access (PIA)**  – Fournisseur historique (désormais basé aux États-Unis, propriété du groupe Kape). PIA propose des tarifs attractifs, un large réseau de serveurs et  **supporte le port forwarding**  sur certains serveurs  . Ses clients intègrent un kill switch et sont open source. PIA a prouvé à plusieurs reprises en justice sa politique  _no-log_. Cependant, son appartenance à une entreprise ayant racheté de nombreux VPN (et anciennement associée à des adwares) fait que certains utilisateurs  _privacy_  restent méfiants. Niveau torrent, PIA fonctionne bien et autorise P2P sur tous les serveurs ne se trouvant pas dans des pays restrictifs. C’est une option « grand public » performante si l’on est à l’aise avec l’idée de faire confiance à sa politique de confidentialité.
    
-   **IVPN**  (Gibraltar) – VPN très éthique et transparent (membre de l’initiative Privacy Guides), axé sur la minimisation des données. Toutefois, IVPN est en train d’**abandonner le port forwarding**  également (décision similaire à Mullvad)  . Sans ports ouverts, IVPN est parfait pour la navigation anonyme, mais moins optimisé pour torrent (sauf usage purement en téléchargement sans seeding public). À noter qu’IVPN offre une fonctionnalité  _Multi-hop_  et des clients minimalistes orientés confidentialité maximale.
    
-   **Windscribe**  (Canada) – Service au positionnement un peu différent : il propose une version gratuite limitée et une version Pro flexible. Windscribe  **peut fournir du port forwarding**  mais seulement via l’achat d’une IP dédiée (_Static IP_  dans leurs options) ou l’utilisation de serveurs dits  _legacy_. L’avantage est qu’il intègre un pare-feu/kill switch permanent appelé  _Firewall_  côté client. Windscribe a une approche un peu moins formelle, mais la communauté tech le considère comme relativement fiable (pas de preuve de no-log publiée néanmoins). Utile pour ceux qui voudraient un plan très modulable ou des fonctions comme le proxy intégré.
    

  

_(Bien sûr, d’autres VPN existent – par exemple_ **_NordVPN_** _ou_ **_Surfshark_** _offrent de très bons débits – mais ils n’autorisent pas le port forwarding et sont donc moins intéressants pour le torrent. Des services spécialisés comme_ **_TorGuard_** _ou_ **_Hide.me_** _proposent aussi le port forwarding et ciblent les power users, mais ils sont moins grand public. Dans ce comparatif, on se concentre sur les solutions éprouvées par la communauté auto-hébergement et confidentialité.)_

  

## **Intégration des clients BitTorrent avec Gluetun**

  

Une des forces de Gluetun est de simplifier l’intégration des applications de téléchargement en conteneur avec le VPN. Le principe est de faire tourner le client BitTorrent (par ex.  **qBittorrent**,  **Transmission**  ou autres) dans un conteneur Docker séparé, mais  **connecté au réseau de Gluetun**  plutôt qu’au réseau par défaut. Grâce à Docker Compose, on peut définir que le service qBittorrent utilise  network_mode: "service:gluetun"  afin que  **tout son trafic passe par le conteneur VPN**  . Dans ce scénario, qBittorrent n’a  **aucune connectivité directe**  hors VPN : il obtient une IP locale virtuelle du conteneur Gluetun et sort uniquement à travers le tunnel chiffré. C’est idéal pour garantir qu’aucun téléchargement ne fuite hors VPN.

  

Concrètement, il faut d’abord lancer Gluetun avec les bonnes variables (identifiants VPN, choix du serveur, activation éventuelle du WireGuard, etc. comme vu précédemment). Ensuite, on lance le conteneur du client torrent en dépendance de Gluetun. Par exemple, voici un extrait de configuration Docker Compose typique :

```yaml
services:
  gluetun:
    image: qmcgaw/gluetun
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - VPN_SERVICE_PROVIDER=AIRVPN
      - VPN_TYPE=wireguard
      - WG_PRIVATE_KEY=<clé privée WireGuard>
      - SERVER_COUNTRIES=FR # ex: choisir un pays/serveur
      - FIREWALL_VPN_INPUT_PORTS=51820 # ex: port entrant attribué par le VPN
      - FIREWALL_OUTBOUND_SUBNETS=192.168.0.0/24 # accès LAN (à adapter ou vider pour isoler)
    # ... autres variables (PUID, PGID, TZ, etc.) ...
    restart: unless-stopped

  qbittorrent:
    image: linuxserver/qbittorrent
    network_mode: "service:gluetun"  # utilise le réseau du VPN
    depends_on:
      - gluetun
    ports:
      - "8080:8080"  # port web UI, exposé via VPN
    environment:
      - WEBUI_PORT=8080
      # ... PUID/PGID ...
    volumes:
      - /data/torrents:/downloads  # stockage des torrents
    restart: unless-stopped
```

Dans cet exemple, qBittorrent utilisera l’interface réseau de Gluetun (tun0) et son adresse IP VPN. On peut accéder à son interface Web via le port 8080 redirigé, et ce trafic passera par le VPN. Si le VPN tombe, le port ne répondra plus (kill switch de Gluetun actif).  **Intégration avec port forwarding**  : si votre VPN (ex: AirVPN, PIA, ProtonVPN) vous fournit un port entrant, il faut le configurer côté VPN/routeur  **et**  dans le client torrent. Avec Gluetun, on utilise  FIREWALL_VPN_INPUT_PORTS  pour ouvrir le port dans son pare-feu interne  , puis dans qBittorrent on fixe ce même port comme port d’écoute. Ainsi, le trafic entrant depuis le serveur VPN sera transmis au client torrent  . Plusieurs guides et scripts existent pour automatiser cette synchronisation (par ex. mise à jour du port de qBittorrent via l’API de Gluetun sur Unraid  ), mais manuellement cela fonctionne bien une fois le port configuré.

  

**Clients torrent compatibles**  : qBittorrent est très répandu et se marie bien avec Gluetun. Transmission fonctionne également en conteneur, de même que Deluge ou d’autres – il suffit de les mettre sur le même  _service network_  que Gluetun. À noter que certains conteneurs “tout-en-un” existent (ex: l’image Docker  _haugene/transmission-openvpn_  qui combine un client torrent et un client OpenVPN). Cependant, l’approche modulaire avec Gluetun + qBittorrent est plus flexible : on peut y ajouter d’autres applications (Jackett, Sonarr, Radarr…) en les faisant passer par le même VPN. Il suffit de connecter ces conteneurs au réseau de Gluetun (option  network_mode: service:gluetun  ou via un réseau Docker partagé)  . Cela mutualise la connexion VPN de façon propre.

  

Enfin, pour une sécurité renforcée, il est possible de  **lier le client BitTorrent à l’interface VPN**  même en dehors de Docker. Par exemple, si l’on exécute qBittorrent sur l’hôte directement, on peut spécifier dans ses options avancées de n’écouter que sur l’interface  tun0. Ainsi, s’il n’y a pas de VPN actif, qBittorrent ne pourra pas se connecter du tout. Cette précaution s’ajoute au kill switch du VPN pour éviter toute fuite accidentelle si un jour le client VPN était désactivé involontairement.

  

## **Schéma d’isolement réseau : VLAN et règles de pare-feu**

  

Pour un ingénieur en sécurité, il est conseillé d’**isoler le trafic torrent du reste du réseau local**. L’idée générale est de créer un sous-réseau dédié (par exemple un VLAN distinct) pour la machine ou le conteneur qui télécharge, et d’appliquer des règles de pare-feu strictes entre ce VLAN et le LAN principal.

-   **VLAN dédié aux téléchargements**  : On peut configurer un VLAN (par exemple VLAN 20) sur le routeur/commutateur, auquel sera connecté le serveur ou la VM/conteneur du client torrent. Ce VLAN n’aura accès qu’à Internet (de préférence, uniquement via le VPN). Les autres appareils du LAN résident sur VLAN 1 (par exemple) et n’ont pas de route vers VLAN 20. Ainsi, même si le client torrent est compromis, il ne pourra pas scanner ou atteindre les partages sensibles du LAN  .
    
-   **Règles de pare-feu inter-VLAN**  : Sur le routeur ou firewall, on met en place une règle bloquant  **toute connexion initiée depuis le VLAN torrent vers le LAN principal**. Seules les connexions nécessaires vers Internet sont autorisées. Par exemple, on autorisera VLAN20 -> Internet  _UNIQUEMENT_  vers les adresses/IP des serveurs VPN et en passant par les ports du protocole VPN (UDP 1194/443 si OpenVPN, UDP 51820 si WireGuard, etc.). On peut également autoriser le trafic DNS du VLAN torrent, soit vers le résolveur VPN de Gluetun (DNS sur TLS), soit vers un DNS local selon le cas, mais généralement il vaut mieux que ces requêtes passent dans le tunnel chiffré. Toutes les autres tentatives (VLAN torrent vers 192.168.0.0/24, etc.) doivent être bloquées par défaut  . Inversement, on peut éventuellement autoriser depuis le LAN principal  **l’accès entrant**  à l’interface web du client torrent ou au partage de fichiers du serveur de téléchargement, si l’admin souhaite y accéder depuis son PC. Ces accès spécifiques doivent être finement filtrés (par exemple, autoriser seulement l’IP de l’admin du LAN vers l’IP du torrent box sur le port 8080).
    
-   **Exemple d’architecture**  : imaginez un NAS ou un mini-PC faisant office de  _seedbox_  sur VLAN 20. Il se connecte au VPN (via Gluetun) et télécharge des torrents. Le routeur/pfSense ne laisse passer que le trafic chiffré VPN entre ce NAS et le WAN, tout autre trafic est bloqué. Le reste du réseau (VLAN 1) ignore complètement VLAN 20 (pas de communication possible). Dès lors, les téléchargements ne peuvent pas révéler l’adresse IP du foyer (ils montrent l’IP VPN) et le NAS torrent ne peut pas joindre les appareils sensibles. Ce  **compartimentage**  renforce la sécurité globale du homelab.
    

  

_(À défaut de VLAN, on peut obtenir un résultat similaire en plaçant le client torrent sur un sous-réseau différent avec une interface séparée ou en le connectant directement derrière un second routeur. L’important est la_ **_segmentation_** _: réduire au minimum les passerelles entre l’environnement torrent et le LAN principal. Même en conteneur local, pensez à désactiver l’accès au réseau local dans Gluetun en ne renseignant pas_ _FIREWALL_OUTBOUND_SUBNETS_ _(ou en le limitant strictement)_ _.)_

  

## **Prévention des fuites DNS, kill switch et gestion des logs pour l’anonymat**

  

Un bon VPN pour le torrent doit absolument empêcher les  **fuites DNS**  et inclure un  **kill switch**. Avec AirVPN, l’option  _Network Lock_  veille à ce qu’aucune donnée (y compris les requêtes DNS) ne sorte hors du tunnel en cas de coupure  . De même, Gluetun assure ces fonctions automatiquement : son pare-feu intégré bloque tout trafic non VPN et utilise par défaut un DNS chiffré (modifiable) pour résoudre les noms de domaine  . Il est recommandé de  **désactiver IPv6**  sur l’interface torrent ou dans le VPN si votre configuration ne l’utilise pas, afin d’éviter d’éventuelles fuites IPv6 si le fournisseur VPN ne gère pas bien le dual-stack. La plupart des bons VPN (AirVPN, Mullvad, ProtonVPN…) supportent IPv6 et traitent ce cas proprement (AirVPN fournit même de l’IPv6 aux clients pour éviter tout leak  ). Vérifiez après mise en place en utilisant des outils comme ipleak.net ou doileak.com pour confirmer qu’aucune requête DNS ne part vers votre FAI et que l’IP visible correspond bien au VPN.

  

Concernant les  **journaux (logs) et l’anonymat**  : idéalement, votre fournisseur VPN ne doit pas conserver de logs d’activité. AirVPN, par exemple, n’enregistre aucune donnée de connexion hormis un compteur anonyme de sessions simultanées  . Mullvad et ProtonVPN ont également des politiques strictes (Mullvad a démontré qu’aucune info exploitable n’était stockée suite à une perquisition  ). Pour l’utilisateur, il est aussi judicieux de  **minimiser les traces**  : évitez de créer un compte VPN avec votre email personnel; préférez des emails alias ou anonymes. Payez avec des méthodes discrètes (cryptomonnaie, espèces, bons cadeaux) si vous voulez pousser l’anonymat à fond, afin que votre identité civile ne soit pas liée au compte VPN.

  

Du côté du client BitTorrent, sachez que les torrents publics peuvent exposer l’IP VPN dans les  _swarm_. Cela n’est pas problématique (c’est le but, et elle n’est pas directement reliée à vous),  **mais ne téléchargez pas sans VPN**  même brièvement, sinon votre IP réelle pourrait se retrouver dans la liste des pairs partagée. On veillera par ailleurs à configurer le client torrent pour qu’il ne divulgue pas d’infos inutiles : par exemple, désactiver le  _tracking_  DHT/PEX si cela n’est pas nécessaire, éviter d’utiliser son vrai nom d’utilisateur Windows dans les échanges SMB vers le NAS, etc. Ces aspects restent mineurs comparés aux protections du VPN, mais font partie d’une hygiène globale.

  

Enfin, on peut se demander si conserver des  **logs en local**  pose problème. Les journaux de Gluetun ou du client torrent (par ex. historique des téléchargements) sont stockés sur votre serveur. En cas de compromission du serveur ou de saisie physique, ces données pourraient révéler ce que vous avez téléchargé. Si c’est une préoccupation, vous pouvez configurer le client torrent pour qu’il n’enregistre pas l’historique des tâches une fois terminées, ou monter le dossier de config en  **RAM**  pour qu’il s’efface à chaque redémarrage. Certains vont jusqu’à faire tourner le client dans une VM chiffrée ou sur un disque chiffré, mais pour un usage homelab classique, ce niveau est souvent excessif. L’essentiel est surtout  **côté VPN**: assurez-vous de choisir un fournisseur de confiance qui ne logge rien d’identifiant et qui offre les fonctions de sécurité (kill switch, DNS sécurisé, ports) adaptées au torrent  . Avec AirVPN via Gluetun, par exemple, vous cumulez les avantages : pas de logs côté VPN, trafic cloisonné et chiffré, fuites bloquées, et ports configurables pour rester efficace en P2P.

  

**En résumé**, AirVPN et Gluetun forment une combinaison puissante pour du torrenting anonyme et performant dans un homelab. AirVPN apporte un réseau robuste et respectueux de la vie privée, tandis que Gluetun facilite son utilisation dans un environnement Docker, avec isolation et contrôles précis. D’autres VPN comme ProtonVPN ou Mullvad peuvent être utilisés via Gluetun selon vos priorités (vitesse, port forwarding, tarif), mais il faudra tenir compte des récents changements (abandon du port forwarding chez certains). Quelle que soit la solution retenue, n’oubliez pas de segmenter votre réseau pour contenir les risques, et de tester votre configuration (adresse IP visible, étanchéité DNS/IP) avant de lancer vos téléchargements. De cette façon, vous profiterez du contenu via BitTorrent en toute tranquillité, sans mettre en danger votre réseau personnel ni votre anonymat en ligne
