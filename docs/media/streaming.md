# **Comparatif Plex vs Jellyfin vs Emby pour un Homelab Sécurisé**

  

Dans un homelab auto-hébergé, Plex, Jellyfin et Emby sont les trois serveurs multimédia majeurs. Chacun offre un ensemble de fonctionnalités similaires (gestion de collections vidéo/musique/photos, transcodage, clients multiples), mais diffère sur des points clés (licence, performance, flexibilité, coût). Nous comparons ci-dessous ces solutions selon plusieurs critères :  **transcodage et performances**,  **clients multi-appareils**,  **gestion multi-utilisateurs et contrôle parental**,  **interface utilisateur**,  **modèle économique**,  **intégration avec Overseerr/Sonarr/Radarr**,  **aspects légaux**, ainsi que  **installation/optimisation techniques**.

  

## **Transcodage et performances**

  

Le transcodage (conversion à la volée) est très exigeant en ressources.  **Plex**  utilise principalement  _Intel QuickSync_(présent sur les CPU Intel récents) pour l’encodage/décodage matériel  . L’accélération matérielle dans Plex nécessite un abonnement Plex Pass (car activée uniquement sur les versions récentes du serveur)  . Plex peut théoriquement utiliser une carte NVIDIA/AMD dédiée, mais le support est « as is » et peu testé  .  **Emby**  prend en charge un large éventail d’accélération : Intel QuickSync, NVIDIA NVENC/NVDEC, AMD AMF (Windows) et VA-API (Linux)  .  **Jellyfin**, étant un fork open-source d’Emby, supporte aussi QuickSync, NVENC et VA-API. Sa documentation recommande fortement d’utiliser un GPU (Intel Arc ou NVIDIA RTX) pour le transcodage, car le CPU seul peine (un Ryzen 9 5950X ne peut même pas transcoder un flux 4K sans GPU)  . De fait, la qualité d’encodage matériel varie selon le matériel : Jellyfin note  _Apple ≥ Intel ≥ Nvidia >>> AMD_  (ex. Apple M1/2 excellent, AMD détériore plus la qualité)  . En résumé :

-   **CPU**  : tous peuvent transcoder par CPU seul, mais c’est très lent et lourd. Jellyfin déconseille les systèmes  _sans_GPU (les CPUs, même hauts de gamme, sont insuffisants)  .
    
-   **Intel QuickSync**  : supporté par Plex (nécessite Plex Pass)  , Emby  et Jellyfin (fortement recommandé)  .
    
-   **NVIDIA NVENC/NVDEC**  : supporté nativement par Emby  et Jellyfin (avec quelques patchs pour lever la limite de flux) ; Plex n’a pas de support officiel mature (les GPU NVIDIA sous Plex sont « pas officiellement testés »  ).
    
-   **AMD (VCE/AMF)**  : support partiel dans Emby (Windows)  ; Jellyfin le supporte via VA-API mais la qualité de l’encodeur AMD est très inférieure  . Plex ne le recommande pas.
    

  

Globalement, Plex privilégie QuickSync et déconseille AMD/NVIDIA, Emby est plus polyvalent (mais sans abonnement, pas de transcodage matériel), et Jellyfin fonctionne mieux sur Intel/NVIDIA qu’AMD  . Les sorties encodées en matériel peuvent être moins nettes qu’en logiciel (blocages ou flou sous faible débit)  . La configuration réseau et stockage est aussi critique : Jellyfin recommande un  **SSD rapide**  (~100 Go) pour le système et le cache de transcodage et une connexion  **Ethernet Gigabit**  minimum (le Wi-Fi ou le 100 Mb/s sont insuffisants)  . Pour un accès distant fluide, prévoir >20 Mb/s d’upload  . L’accélération matérielle (Intel/NVENC) est essentielle pour du 1080p/4K multiple.

  

## **Compatibilité multi-appareils**

  

Tous trois offrent des clients web et de nombreuses apps natives, mais leur disponibilité et coût diffèrent :

-   **Plex**  dispose d’applications officielles sur pratiquement toutes les plateformes grand public (web, iOS, Android, Apple TV, Roku, Amazon Fire TV, Android TV, consoles Xbox/PlayStation, Smart TV Samsung/LG, etc.). Ces apps sont globalement  **gratuites**, même si certaines fonctionnalités avancées (offline, synchronisation) requièrent Plex Pass. La compatibilité est très étendue et bien supportée.
    
-   **Emby**  propose aussi des clients sur mobiles (iOS/Android), PC (Windows, macOS, Linux), TV (Roku, Android TV, Samsung Tizen, LG WebOS), consoles, etc. Beaucoup de ces apps sont gratuites à installer, mais certaines fonctions ou la vidéo complète peuvent être bloquées derrière l’achat de l’application ou Emby Premiere (par exemple, l’app mobile gratuite ne joue qu’une minute de chaque vidéo sans la licence Premiere  ). Certains clients officiels Emby sur TV ou mobile coûtent un paiement unique.
    
-   **Jellyfin**  a des clients first-party (web, Android, iOS, Windows, etc.) entièrement gratuits, ainsi que des app tiers (Kodi, Infuse, etc.). La portée est grande (Android, Fire TV, Apple TV via TestFlight, Roku, Samsung TV via sideload, etc.), mais certains clients officiels manquent encore ou nécessitent un patch non officiel. En pratique, Jellyfin est « largely free » mais l’installation sur certains appareils demande parfois des solutions tierces  .
    

  

D’après un comparatif,  **Plex**  est « le plus simple et convivial », accessible « presque partout » avec peu d’applications payantes  .  **Emby**  offre un bon compromis (surtout si on paie l’app ou Premiere), et  **Jellyfin**  reste gratuit mais avec parfois plus de bricolage client nécessaire  . En résumé : Plex gagne en disponibilité native, Emby suit de près (souvent via achèvement d’app), Jellyfin couvre l’essentiel mais dépend de la communauté pour certains clients.

  

## **Multi-utilisateurs, contrôle parental et métadonnées**

-   **Plex**  : prend en charge plusieurs  **profils utilisateur**  via « Plex Home ». Un serveur peut partager ses bibliothèques jusqu’à ~100 comptes utilisateurs (amis/famille)  . En revanche, pour créer des  **comptes gérés**  dans le Home (permettant une alternance sans re-login) et appliquer des restrictions, il faut un Plex Pass  . Plex propose des profils pré-définis (Jeune/enfant, Adolescent) qui limitent l’accès selon les classifications de contenu  . Les parents peuvent aussi désactiver les sources de contenus en ligne (films/séries Plex) pour ces comptes. Des restrictions plus fines (boutons manuels, filtres personnalisés) demandent également Plex Pass.
    
-   **Emby**  : autorise un nombre illimité de profils d’utilisateurs. Chaque profil peut recevoir des  **contrôles parentaux**sophistiqués. On peut fixer un classement maximum de film/série, bloquer ou autoriser des contenus via des  _tags_, et même créer un planning d’accès (ex. interdiction en dehors d’un horaire)  . Ces options (classements, tags, planning) sont disponibles en gratuit, bien qu’Emby Premiere active des fonctions supplémentaires (synchro mobile, téléchargements offline, etc.).
    
-   **Jellyfin**  : gère aussi plusieurs utilisateurs et bibliothèques distinctes, mais ne fournit pas (encore) de système de contrôle parental prédéfini. Pour restreindre du contenu, on utilise le tagging manuel et les  **bibliothèques séparées**. Par exemple, on peut marquer les films « adultes » avec un tag et interdire ce tag au profil enfant, ou bien créer deux bibliothèques (« Familial » vs « Adulte ») et ne donner accès qu’à la bibliothèque correspondante  . C’est une approche manuelle (assez puissante, mais fastidieuse à paramétrer). Notez que Jellyfin étant open-source, il repose sur des sources de métadonnées ouvertes (TheMovieDB, TheTVDB, etc.), tout comme Emby/Plex. Les trois extraient affiche, synopsis, cast, etc. automatiquement (via TMDb/IMDb), et permettent la modification manuelle des métadonnées dans l’interface.
    

  

**Tableau – Multi-utilisateurs et contrôle parental :**

| **Fonctionnalité** | **Plex** | **Emby** | **Jellyfin** |
|-----------|-----------|-----------|-----------|
|   Profils (utilisateurs)        |    Jusqu’à 100 comptes partagés, 15 dans « Home » (avec Plex Pass)       |      Illimité     |     Illimité      |
|     Comptes gérés (« Home »)      |    Avec Plex Pass (admin crée les profils)       |      Natif, paramétrage individuel     |     Natif (via utilisateurs séparés)      |           |           |           |
|    Contrôle parental (classement)       |    Classes prédéfinies (Jeune, Ados…)  ; avancé avec Plex Pass       |     Classement par note max, tags, planning      |     Tagging manuellement + bibliothèques séparées      |           |
|     Restrictions fines      |  Plex Pass requis pour filtres avancés         |     Sans limite (tags, horaires)      |     Via tags/bibliothèques (pas d’interface dédiée)	     |           |           |        |
|   Métadonnées & organisation        |    Puissant, axé Netflix/Kodi (Pinterest, critiques…); édition possible       |      Complète (avec plug-ins)     |  Complète (open-source, modifiable)         |           |           |           |
|           |           |           |           |           |           |           |

## **Interface utilisateur et expérience**

  

**Plex**  est souvent perçu comme le plus « polishé » et le plus simple d’emploi, avec une interface épurée et comparable aux services de streaming commerciaux  . La navigation est intuitive (grandes images de couvertures, menus clairs) et les apps officielles offrent une expérience homogène.  **Emby**  propose une interface moderne (basée sur Bootstrap) assez claire, un bon compromis entre Plex et Jellyfin. Les possibilités de personnalisation GUI y sont un peu plus nombreuses (thèmes, plugins).  **Jellyfin**  a une interface fonctionnelle et de plus en plus complète (surtout en version 10+), mais certains utilisateurs trouvent l’UX un peu moins intuitive (par exemple, la nouvelle navigation au scroll horizontal a surpris certains)  . En pratique, Plex séduira ceux qui veulent une expérience « Netflix maison » clé-en-main, tandis qu’Emby et Jellyfin attireront les amateurs prêtes à ajuster les paramètres pour plus de flexibilité  .

  

## **Modèle économique (licences, Open Source, Freemium)**

-   **Plex**  est un logiciel  **propriétaire**. Le serveur de base et la plupart des apps sont gratuits, mais de nombreuses fonctions avancées requièrent un abonnement  _Plex Pass_  (mensuel ou annuel, ~5–6 € par mois). Le  _hardware transcoding_  en est un : il est désactivé sans Plex Pass  . La gestion des comptes « Home » avancée (profils multiples sans re-login) nécessite aussi Plex Pass  .
    
-   **Emby**  est en mode  **freemium**. Le serveur et la quasi-totalité des fonctionnalités de base sont gratuites. Emby propose une clé « Premiere » (à tarif raisonnable unique) pour débloquer certaines fonctions : transcodage matériel (GPU), synchronisation mobile, extensions de plugins, etc. Sans cette clé, le transcodage est strictement logiciel (ou inexistant sur Android TV)  . De plus, certaines apps mobiles ou de bureau ne permettent que la lecture limitée d’une minute sans Emby Premiere  .
    
-   **Jellyfin**  est  **100% libre et gratuit**  (licence GNU GPL)  . Serveur et clients officiels ne coûtent rien, sans aucune restriction payante  . Il n’y a ni abonnement ni frais cachés. Tout est open-source, et l’équipe refuse la publicité ou la collecte de données.
    

  

**Tableau – Modèle économique :**

| **Critère** | **Plex** | **Emby** | **Jellyfin** |
|-----------|-----------|-----------|-----------|
|   Licence        |    Propriétaire       |      Propriétaire (partiellement)     |     Open-source (GPL)     |
|    Hébergement     |    SaaS + serveur local       |      Serveur local uniquement    |   Serveur local uniquement     |           |           |           |
|   Coût de base      |    Gratuit       |     Gratuit     |     Gratuit    |           |
|     Abonnement      |  _Plex Pass_  (facultatif mais requis pour HW transcoding et Home avancé)      |     _Emby Premiere_  (un paiement unique pour accélération matérielle, etc.)      |    Aucun (pas d’abonnement)     |           |           |        |
|  Clients payants       |    Non (applications gratuites sur stores)     |    Certaines apps payantes (ex. mobile, TV)    |  Non (clients officiels gratuits)       |           |           |           |
|   Support        |     Communauté et support Plex      |      Communauté et support Emby     |    Communauté open-source (forum, GitHub)       |           |           |           |

## **Intégration Overseerr, Sonarr, Radarr**

  

Dans un homelab automatisé,  **Sonarr**  (séries TV) et  **Radarr**  (films) gèrent le téléchargement. Ils placent les fichiers dans les dossiers surveillés par le serveur media, qui peut ensuite les rafraîchir. Les trois serveurs se connectent nativement avec Sonarr/Radarr (via API ou SMB mounts) pour mettre à jour la bibliothèque après chaque ajout. Pour la gestion des requêtes utilisateurs,  **Overseerr**  est très populaire : c’est un front-end de demandes conçu pour Plex (et Emby) qui s’interface avec Sonarr/Radarr  . Une variante,  **Jellyseerr**  (fork d’Overseerr), apporte le support  **Jellyfin**  (ainsi qu’Emby et Plex) tout en conservant l’intégration Sonarr/Radarr  . En pratique, on déploie un conteneur Overseerr/Jellyseerr relié aux serveurs Plex/Jellyfin/Emby pour permettre aux utilisateurs de suggérer des films/séries ; les requêtes sont transférées vers Sonarr/Radarr pour l’automatisation du téléchargement  .

  

## **Aspects éthiques et légaux**

  

Un homelab media presuppose que l’on partage  _son propre contenu_. Sur le plan légal, si le contenu est acquis légalement (DVD originaux, fichiers achetés), la « copie privée » est souvent tolérée. Aux États-Unis, par exemple, diffuser à ses amis un stream privé est considéré comme analogue au partage des fichiers eux-mêmes  : l’acte lui-même n’est pas illégal, c’est le contenu partagé qui doit être légal. En France, la loi sur la copie privée (article L122-5 CPI) autorise les copies personnelles de DVD/CD possédés,  _sans contournement de DRM_,  **à condition de ne pas les diffuser**  . Autrement dit, la copie privée doit être strictement individuelle (pas de prêt ni de partage). Ainsi, créer un flux Plex/Jellyfin pour soi ou sa famille proche relève de l’usage privé et entre dans la tolérance du droit (bien que la loi française précise que les copies ne doivent pas être « prêtées ou partagées »  ). En revanche, ouvrir le serveur à des tiers ou publier le contenu sur Internet « comme un service de streaming » serait hors cadre privé et susceptible d’infraction.  **Conseil pratique :**  limiter l’accès (via VPN, accès local seulement, ou authentification stricte) et n’héberger que du contenu dont on détient légalement les droits. Le simple fait d’utiliser ces logiciels n’est pas illégal, mais le respect du droit d’auteur (pas de médias piratés en libre accès) est impératif  .

  

## **Installation technique : Docker, LXC, VM, reverse-proxy**

  

Ces serveurs peuvent être installés de plusieurs manières. Les deux modes les plus courants en homelab sont :

-   **Docker/Container**  : tous disposent d’images Docker officielles ou communautaires (p.ex.  plexinc/pms-docker,  jellyfin/jellyfin,  emby/embyserver). Un  docker-compose.yml  permet de déclarer les volumes de données (médias, configs, cache transcode) et ports (ex. 32400 pour Plex, 8096/8920 pour Jellyfin, 8096 pour Emby). L’avantage est la portabilité et l’isolation. Les conteneurs doivent avoir accès aux dossiers médias (via bind mounts ou un NAS) et, pour le transcodage, souvent au GPU du système (via runtime NVIDIA ou habilitation QuickSync sur l’hôte Linux).
    
-   **Machine virtuelle ou LXC**  : on peut installer Plex/Jellyfin/Emby sur une VM (Debian/Ubuntu) ou un conteneur LXC (ex. Proxmox). LXC + Docker est aussi courant : on crée un LXC (Debian minimal), on active  iocage devicespour QuickSync, puis on lance Docker à l’intérieur. Cela offre un peu plus de découplage et de sécurité.
    

  

**Reverse proxy :**  en front-end, on place généralement un reverse proxy HTTP(s) (Nginx, Traefik, Caddy) pour gérer les domaines et TLS. Par exemple,  plex.exemple.com,  jellyfin.exemple.com  pointent vers les ports internes respectifs (ex. 32400, 8096). Le proxy fournit un certificat SSL Let’s Encrypt et peut exiger une authentification (HTTP Auth) ou limiter les IPs pour renforcer la sécurité. Cela évite d’exposer directement les ports Plex/Jellyfin au WAN et permet de centraliser l’accès web en HTTPS. En interne, on peut aussi limiter l’accès direct (p.ex. en ne mappant pas le port 1900/32469 en UDP pour Plex, etc.), tout en autorisant la synchro via clients officiellement. Les guides d’installation officielles (Jellyfin Docs, guides Plex/Emby) détaillent ces approches. Par exemple, Jellyfin recommande un accès dédié (ports 8096/8920) et suggère l’utilisation de Nginx/Caddy pour le SSL.

  

## **Optimisation des performances**

  

Pour un serveur réactif :

-   **Stockage rapide**  : utilisez un SSD (ou NVMe) pour le système d’exploitation, le cache de transcodage et la base de données de métadonnées  . Jellyfin conseille 100 Go de SSD pour le cache et OS, afin d’éviter de saturer les disques lents  . Les médias (vidéos) peuvent rester sur des disques plus grands, mais si plusieurs flux 4K passent simultanément, un RAID SSD ou un NVMe dédié peut améliorer la fluidité.
    
-   **Réseau filaire**  : privilégiez l’Ethernet Gigabit ou supérieur. Jellyfin insiste qu’une carte 1 GbE ou plus est fortement recommandée et déconseille le Wi-Fi ou Powerline  . Pour l’accès distant, prévoyez suffisamment d’upload (>20 Mb/s est un minimum pour du 1080p  ).
    
-   **GPU pour transcodage**  : activez l’accélération matérielle. Par exemple, sur une machine Intel, installez les paquets pour QuickSync (Intel Media Server Studio ou sur Linux le paquet  intel-media-va-driver-non-free), puis activez QuickSync dans Jellyfin/Plex/Emby. Pour NVIDIA, installez les pilotes GPU récents et la runtime Docker NVIDIA, puis activez NVENC dans Jellyfin/Emby ou pointez Plex vers le GPU (ce dernier n’est pas officiellement supporté sous Linux, mieux vaut utiliser un client native ou se reposer sur QuickSync). Ces accélérations déchargent le CPU, permettant plusieurs flux HD/4K à la fois.
    
-   **Configuration réseau**  : limitez les connexions superflues. Par exemple, désactivez la recherche UPnP ou DLNA si inutile. Si vous utilisez un VPN, assurez-vous qu’il gère bien le trafic multicast (certains VPN maison bloquent le DLNA).
    
-   **Optimisations logicielles**  : mettez à jour les serveurs régulièrement (Jellyfin a souvent des correctifs optimisant FFmpeg, Emby/Plex publient des versions stables). Dans Jellyfin, assurez-vous d’avoir FFmpeg 5+ via les packages officiels, et activez  _Tone Mapping HDR_  si vous mixez contenu HDR/SDR. Dans Emby, on peut fixer le nombre max de transcodages simultanés. Dans tous les cas, pré-téléchargez les vidéos en résolution native compatible (direct play) autant que possible pour limiter le transcodage sur le serveur.
    

  

## **Cas d’usage typiques en Homelab**

-   **Streaming familial**  : serveur Plex/Jellyfin accessible sur le LAN (ou via VPN pour les membres autorisés), diffusant sur smart TV, Chromecast, smartphones, etc. Les parents créent un profil « Enfant » avec uniquement les dessins animés et films G/PG, tandis que leur profil « Adulte » débloque tout le contenu.
    
-   **Automatisation média**  : dès qu’un nouvel épisode sort, Sonarr le télécharge et actualise Plex. Un utilisateur peut faire une requête via Overseerr/Jellyseerr (par exemple, « Je veux la S05 de The Mandalorian »), qui lance Radarr/Sonarr et notifie le serveur.
    
-   **Accès distant sécurisé**  : au lieu d’exposer Plex direct sur Internet, on se connecte au homelab via VPN (WireGuard/IPv6 local) ou via un reverse proxy avec authentification forte, garantissant que seuls les utilisateurs légitimes peuvent streamer en mobilité.
    
-   **Enregistrement TV et livres**  : Emby/Jellyfin supportent la TV en direct et l’enregistrement DVR (avec une carte TV ou un tuner réseau). Ils peuvent aussi organiser une collection de musique/ebooks/photos au-delà de la vidéo. Dans un cadre homelab, on peut ainsi centraliser  **tous**  les médias personnels (vidéos, audio, livres) dans une seule plateforme.
    
-   **Sécurité avancée**  : comme pour tout service web hébergé, on surveille les logs, on applique des mises à jour de sécurité (OS et serveurs médias) et on limite les accès (pare-feu, authentification). L’option multi-utilisateur permet de cloisonner la navigation de chacun. Par exemple, Jellyfin est « privacy-focused » et ne phone home jamais  , ce qui rassure les ingénieurs cybersécurité.
