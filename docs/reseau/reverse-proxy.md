# **Solutions de Reverse Proxy pour un Homelab de Cybersécurité**

  

Dans un homelab orienté cybersécurité, les reverse proxies jouent un rôle crucial pour exposer en toute sécurité des services internes vers l’extérieur. Ils permettent de centraliser la gestion des accès, d’appliquer des règles de sécurité (authentification, filtrage) et de segmenter le réseau. Nous allons comparer techniquement  **Nginx Proxy Manager**,  **Traefik**  et  **Authelia**, et examiner des exemples de configuration typiques ainsi que des cas d’usage et fonctionnalités de sécurité avancées adaptés à un homelab.

  

## **Comparaison technique : Nginx Proxy Manager, Traefik et Authelia**

  

**Nginx Proxy Manager (NPM)**  est une interface web conviviale permettant de piloter un serveur Nginx (basé sur OpenResty) en arrière-plan. Il offre une gestion simplifiée des hôtes proxy avec une GUI intuitive : on peut facilement ajouter un  **hôte proxy**  (domaine, adresse cible et port) via une interface web, gérer les certificats SSL Let’s Encrypt en un clic, définir des accès restreints, etc. Par défaut, NPM prend en charge les fonctionnalités de base communes à Traefik, comme la gestion automatique des certificats SSL/TLS (via Let’s Encrypt et divers fournisseurs DNS) et le proxying de multiples protocoles (HTTP, HTTPS, mais aussi TCP/UDP via les streams Nginx)  . Son atout principal est la simplicité : tout se fait via l’UI, sans nécessiter d’éditer des fichiers de configuration. NPM gère les utilisateurs multiples avec rôles et journalisation des modifications (audit logs) intégrés, ce qui permet une administration partagée et tracée – une fonctionnalité que Traefik n’a pas nativement  . Techniquement, NPM stocke sa configuration (hôtes, utilisateurs, certificats…) dans une base de données (SQLite ou MariaDB/MySQL)  . Cela signifie que la configuration persiste et peut être sauvegardée, mais introduit aussi un point d’échec potentiel (corruption de DB). NPM étant un frontal à Nginx, il bénéficie des performances reconnues de Nginx (écrites en C, très optimisées) – on observe en effet que Traefik est  **un peu moins rapide**  que Nginx/OpenResty en proxy HTTP, même si l’écart est mineur dans le contexte d’un homelab  . En contrepartie, NPM est moins flexible sur certains points avancés : par exemple, l’interface ne permet de définir qu’un seul serveur de destination par host (pas de load-balancing multiple en GUI)  , et une erreur de configuration peut rendre Nginx indisponible (toute la configuration Nginx échoue si un host est mal défini), interrompant  **tous**  les sites jusqu’à correction  . NPM convient bien aux débutants ou à ceux qui privilégient la rapidité de mise en place via une interface graphique  .

**Traefik**  est un reverse proxy moderne, pensé pour les environnements dynamiques (Docker, Kubernetes). Contrairement à NPM, Traefik privilégie la configuration  **“as code”**  : pas de véritable GUI d’administration (son tableau de bord web est  **en lecture seule**, servant à visualiser la config active)  . La configuration s’effectue via des fichiers YAML statiques/dynamiques ou via des  **labels Docker**  attachés aux conteneurs – ce qui permet une  **découverte automatique**  des services. En effet, Traefik peut surveiller l’API Docker ou Kubernetes et créer les routes proxy correspondantes à la volée en fonction des labels, sans redémarrage du conteneur Traefik  . Cette capacité d’**auto-discovery**  est un avantage dans un homelab très containerisé : déployer un nouveau service Docker avec les bons labels suffit pour l’exposer sur le domaine voulu, Traefik configurant automatiquement le routage  . Traefik intègre nativement le support de Let’s Encrypt (ACME) : il peut générer et renouveler automatiquement les certificats SSL, y compris via des challenges DNS (il suffit de fournir les tokens API des DNS providers dans la configuration)  . Il supporte pleinement le  **load balancing**  entre plusieurs instances d’un service (round-robin, etc.) et des  **middlewares**  puissants pour modifier ou filtrer les requêtes (réécritures d’URL, redirections, rate limiting, IP whitelisting, authentification basique, injection de headers de sécurité, etc.)  . Traefik ne nécessite pas de base de données – sa configuration vit en mémoire et se met à jour dynamiquement – ce qui le rend moins sujet à des corruptions et permet des changements à chaud sans interruption  . En cas de configuration invalide, seul l’élément concerné échouera tandis que le reste du proxy continue de fonctionner, ce qui améliore la  **robustesse**  de l’ensemble par rapport à Nginx (où une erreur peut bloquer tout le service)  . Côté performances, Traefik (écrit en Go) est légèrement moins performant que Nginx en termes de latence pure ou de débit, mais pour un usage homelab les différences sont généralement négligeables  . Traefik s’adresse plutôt aux utilisateurs avancés ou aux environnements nécessitant une forte automatisation et intégration CI/CD, au prix d’une  **courbe d’apprentissage**  plus élevée (compréhension des routers, services, middlewares, labels)  . En résumé,  _Traefik excelle en environnement containerisé dynamique_, tandis que  _Nginx Proxy Manager brille par sa simplicité et son interface utilisateur_.

  

**Authelia**, quant à lui, n’est pas un reverse proxy mais un service  **d’authentification et d’autorisation**  open-source conçu pour se  **coupler aux reverse proxies**  existants (Nginx, Traefik, Caddy, HAProxy, etc.)  . Authelia agit comme un  **portail d’authentification**  à facteurs multiples (MFA) fournissant du  **Single Sign-On (SSO)**  pour vos applications web auto-hébergées  . En pratique, il s’intègre via le mécanisme de  **“forward authentication”**  : le reverse proxy est configuré pour interroger Authelia à chaque requête entrante non authentifiée. Authelia valide si l’utilisateur a une session active (cookie SSO) ou demande à ce qu’il se connecte via son  **portail web sécurisé**. Une fois l’authentification effectuée (avec mot de passe + second facteur possible), Authelia informe le proxy que la requête peut passer, et l’utilisateur accède alors au service cible  . Authelia dispose d’une interface web pour le login des utilisateurs (portal web) mais pas d’une interface “admin” équivalente à NPM/Traefik – sa configuration se fait via un fichier YAML. Il supporte en backend les utilisateurs définis dans un fichier YAML ou dans un annuaire  **LDAP/Active Directory**, ainsi que la possibilité d’être  **fournisseur OpenID Connect (OIDC)**  . Ce dernier point signifie qu’Authelia peut s’intégrer dans un écosystème OAuth2/OIDC : par exemple, des applications supportant OIDC peuvent déleguer l’authentification à Authelia, ou Authelia peut utiliser des  **en-têtes “trusted”**  pour propager l’identité de l’utilisateur aux applications web  . Authelia se déploie facilement en Docker (image <20 Mo) et consomme peu de ressources (quelques dizaines de Mo de RAM)  – intéressant pour un homelab où on cherche à minimiser l’empreinte. Il nécessite généralement un  **stockage**  pour les données de session (Redis par exemple pour la haute dispo, sinon en mémoire ou SQLite) et peut utiliser une base (SQLite, MySQL…) pour stocker son état (sessions, préférences) si on le souhaite  . Authelia complète donc Traefik ou Nginx Proxy en ajoutant une couche d’authentification forte centralisée.  **À noter**  : Nginx Proxy Manager ne propose pas nativement l’intégration d’Authelia dans son interface, contrairement à Traefik où la documentation officielle fournit des exemples de configuration pour Authelia  . Il est tout de même possible d’utiliser Authelia avec Nginx Proxy Manager en ajoutant des directives personnalisées dans la configuration Nginx (voir plus loin).

### **Principales différences Nginx Proxy Manager vs Traefik**

  

Pour synthétiser la comparaison NPM vs Traefik dans un contexte homelab :

-   **Interface & configuration** : NPM offre une  **interface web complète**  pour tout configurer (domaine, upstream, certificat, authentification…), conviviale pour les débutants. Traefik propose seulement un  **tableau de bord**  de visualisation, la configuration se fait par  **fichiers**  ou  **labels**  dans une approche Infrastructure-as-Code  . Modifier une config Traefik exige donc de connaître sa syntaxe (YAML/labels) mais permet une automatisation et une mise en version du code, là où NPM privilégie la simplicité point&click.
    
-   **Gestion des utilisateurs et des droits** : NPM gère nativement des comptes multiples avec rôles (admin, viewer) et garde un  **journal d’audit**  des changements  . Traefik n’a pas d’utilisateurs multiples ni d’authentification sur son dashboard par défaut (il faut ajouter une auth basique manuellement si on veut protéger l’accès à l’UI)  . Dans un homelab à plusieurs administrateurs ou pour tracer les modifications, NPM marque donc un point.
    
-   **Persistance de la configuration** : NPM stocke sa configuration dans une base de données (fichiers SQLite ou serveur MariaDB)  , ce qui assure une persistance même après redémarrage du conteneur. Traefik n’a pas de DB – les configs statiques/dynamiques sont rechargées en mémoire à chaque démarrage et peuvent être mises à jour à chaud. Moins de risque de corruption mais il faut penser à sauvegarder manuellement les fichiers de configuration Traefik/les commandes de déploiement Docker-Compose (alors que NPM nécessite de sauvegarder son volume ou dump SQL).
    
-   **Routing et découvertes de services** : Traefik est conçu pour la  **découverte automatique**  des services dans Docker/K8s – tout conteneur lancé avec les bons labels est automatiquement pris en charge (sous réserve qu’il soit dans le réseau Docker de Traefik)  . NPM ne découvre rien automatiquement : il faut déclarer chaque host via l’UI (bien qu’il soit possible de scripter l’API ou d’utiliser Nginx Proxy  _Companion_  pour du Docker générique, ce n’est pas intégré dans NPM). Pour un homelab très dynamique (services éphémères, scaling), Traefik se montre plus pratique.
    
-   **Flexibilité des fonctionnalités** : Traefik embarque des  **middlewares modulaires**  activables facilement (ex :  traefik.http.middlewares.rateLimit,  ...redirectscheme,  ...auth  etc.), ainsi qu’un écosystème de  **plugins**  officiel pour étendre ses capacités (ex : plugin Fail2Ban,  **CrowdSec**, ModSecurity WAF, etc.)  . Nginx (et donc NPM) est très puissant aussi, mais ajouter par exemple un WAF ModSecurity ou un système de ban IP nécessite de modifier la configuration avancée ou d’utiliser des conteneurs complémentaires – ce n’est pas “plug and play” via l’interface. En contrepartie, Nginx permet un  **contrôle très fin**  de chaque détail (directives SSL/TLS, tuning performance, modules tiers) en éditant la config, tandis que Traefik fait plus de choses automatiquement mais offre moins de contrôle granulaire que du pur Nginx  .
    
-   **Performances** : Nginx/OpenResty offre en général un meilleur throughput et une latence un peu plus faible que Traefik dans les mêmes conditions  . Pour un homelab avec quelques services, cette différence est  **peu perceptible**  . Nginx excelle sous forte charge ou pour servir du contenu static/cache, tandis que Traefik consomme un peu plus de CPU/RAM pour son moteur Go mais reste tout à fait adapté à des charges modestes. On pourra retenir que si l’on cherche à optimiser chaque pourcent de performance, Nginx a l’avantage, mais Traefik apporte d’autres bénéfices (auto-config) qui peuvent primer en homelab.
    

  

En résumé,  **NPM**  est idéal pour démarrer rapidement et administrer simplement un reverse proxy multi-sites via une interface graphique, avec la robustesse éprouvée de Nginx.  **Traefik**  convient aux setups plus automatisés ou complexes, particulièrement si tout tourne en conteneurs, en offrant une intégration native avec Docker/K8s et des fonctionnalités avancées (middlewares, load balancing) au prix d’une configuration par code. Dans bien des cas, le choix dépendra donc de votre aisance technique et des besoins spécifiques du lab (simplicité vs. flexibilité)  . Notons qu’**Authelia**  n’est pas en concurrence directe avec ces proxies mais vient plutôt les compléter pour la couche authentification/SSO multi-facteurs.

  

## **Exemples de configurations typiques**

  

Passons en revue quelques configurations typiques qu’on peut mettre en place avec ces outils, dans un homelab, pour renforcer la sécurité et le confort d’utilisation.

  

### **Reverse proxy avec authentification des accès**

  

Un usage courant en homelab est de  **protéger l’accès aux services internes par une authentification**. Plusieurs approches existent :

-   **Authentification basique (Basic Auth)**  : C’est la méthode la plus simple, intégrée nativement.  **Nginx Proxy Manager**  permet de créer des  _Access Lists_  contenant des utilisateurs (login/mot de passe HTTP Basic) et de les associer à un host. Par exemple, on peut définir une liste “Admin seulement” avec utilisateur  admin  et mot de passe, puis la lier à l’hôte proxy d’une application sensible – NPM forcera alors la saisie de ce login/mot de passe pour accéder au service.  **Traefik**  offre un middleware de basic auth équivalent : on peut définir dans les labels Docker  traefik.http.middlewares.monauth.basicauth.users=utilisateur:motdepasse_hashé  et attacher ce middleware à un router  . Dans le fichier Compose présenté plus haut, on voit un exemple protégeant le dashboard Traefik par basic auth (utilisateur  _admin_, mot de passe  _password_) via ces labels  . Cette méthode a l’avantage d’être simple mais n’offre pas de SSO (chaque service a ses propres identifiants) ni de second facteur.
    
-   **Authentification avancée avec Authelia (SSO)**  : Pour un homelab cybersécurité, mettre en place Authelia apporte une couche  **SSO à double facteur**  pour tous les services web. La configuration consiste à déployer le service Authelia (Docker) et à le  **déclarer dans le reverse proxy**  en tant que provider d’authentification. Concrètement, avec  **Nginx (Proxy Manager)**  cela se fait en ajoutant des directives custom Nginx dans l’onglet “Advanced” de chaque host à protéger. Par exemple, on insérera dans la config du host :

```nginx
auth_request /authelia;
auth_request_set  $target_url  $scheme://$http_host$request_uri;
error_page  401  =302  https://auth.mondomaine.com/?rd=$target_url;
```


-   _(ainsi que d’autres directives pour passer les en-têtes d’utilisateur)_  . Ces instructions signifient : “pour chaque requête sur ce host, interroger l’URL  /authelia  (endpoint interne qui pointe vers Authelia) – si Authelia renvoie 401 non autorisé, alors rediriger (302) le client vers la page de login Authelia  auth.mondomaine.com  en lui passant l’URL cible dans  rd  (redirect)”. Authelia, une fois l’utilisateur authentifié, redirigera celui-ci vers l’URL initiale, qui cette fois sera acceptée (car munie du cookie de session). Ce mécanisme utilise le  **directive auth_request de Nginx**pour interroger un serveur d’auth externe  .
    
    Du côté  **Traefik**, l’intégration est tout aussi simple via un  **middleware de type ForwardAuth**. On peut déclarer, par exemple dans un fichier dynamique ou via labels, un middleware pointant vers Authelia :

```nginx
traefik.http.middlewares.authelia.forwardAuth.address: "http://authelia:9091/api/authz/forward-auth"
traefik.http.middlewares.authelia.forwardAuth.trustForwardHeader: "true"
traefik.http.middlewares.authelia.forwardAuth.authResponseHeaders: "Remote-User,Remote-Groups,Remote-Email"
```
-   Ensuite, pour chaque service à protéger, on attache ce middleware  _authelia_  au router Traefik concerné (par ex. label  traefik.http.routers.monservice.middlewares=authelia@docker)  . Ainsi, toute requête passera par Authelia qui décidera de la laisser passer ou non. Authelia fournit en plus des en-têtes (Remote-User, Remote-Groups, etc.) que le proxy peut forwarder à l’application cible si besoin  (utile si l’application veut connaître l’utilisateur loggé).
    
    _Dans la pratique_, une configuration complète implique aussi : la définition des règles d’accès dans Authelia (fichier YAML  _access_control_: quels domaines exigent 2FA, lesquels sont en accès libre ou 1FA)  , la création des utilisateurs (fichier ou connexion à un LDAP), et la sécurisation du portail Authelia lui-même via HTTPS. Une fois en place, ce setup offre une  **authentification unifiée**  : l’utilisateur se connecte une fois sur Authelia et accède ensuite à plusieurs services sans se reconnecter (SSO) tant que sa session est valide  . On bénéficie aussi de fonctionnalités de sécurité comme le deuxième facteur (TOTP, Webauthn…) et la protection anti-brute-force fournie par Authelia (voir section Sécurité).
    
-   **Restriction par adresse IP (IP whitelisting)**  : Une méthode plus simple, qui peut compléter les précédentes, est de  **limiter l’accès à certains services par IP source**  (par exemple, accessibles uniquement depuis le LAN du homelab ou via VPN). Nginx Proxy Manager permet de définir dans les Access Lists des plages IP autorisées ou refusées. Traefik propose un middleware  IPWhiteList  où l’on spécifie des subnets autorisés. Dans l’extrait de configuration Traefik précédent, on voit par exemple un middleware  local-ipwhitelist@file  pouvant être appliqué à un router pour  **n’accepter que les IP privées**  . Ce type de filtrage est très utile pour, par exemple, exposer l’interface d’administration d’un NAS ou d’une VM uniquement aux IP locales, tout en ayant d’autres services du proxy accessibles depuis Internet. Cela segmente l’accès sans nécessiter d’authentification pour les services purement internes.
    

  

En combinant ces méthodes, on peut obtenir un niveau de sécurité granulaire adapté à chaque service du homelab. Par exemple, un wiki personnel pourrait être accessible uniquement depuis le LAN (filtrage IP), tandis qu’une application plus critique (ex: interface domotique) serait exposée sur Internet mais protégée par Authelia (login + 2FA), et un service moins sensible pourrait se contenter d’une auth basique.

  

### **Accès sécurisé aux services internes depuis l’extérieur**

  

L’un des buts du reverse proxy dans un homelab est de  **rendre accessibles depuis l’extérieur**  (Internet) certaines applications internes, de manière sécurisée. Voici les bonnes pratiques de configuration pour cet usage :

-   **Placer le proxy en “frontale” sécurisée**  : Le reverse proxy sert de  **point d’entrée unique**. On configure le routeur/pare-feu de la box internet pour ne forward que les ports 80/443 vers la machine hébergeant le proxy (idéalement uniquement 443, voir HTTPS ci-dessous). Ainsi,  _aucun service interne n’est exposé en direct_  – ils sont tous sur des IP privées non routables, seul le proxy (dans une DMZ ou VLAN dédié de préférence) est joignable depuis Internet. Cette configuration limite la surface d’attaque : un attaquant ne peut cibler que le proxy lui-même, et non chaque service individuellement.
    
-   **Noms de domaine et DNS**  : On utilise un (sous-)domaine pour chaque service, géré via un DNS public (par ex. un domaine personnalisé, ou des sous-domaines d’un DDNS). Par exemple,  nextcloud.monlab.fr,  wiki.monlab.fr, etc. Le reverse proxy est configuré pour écouter sur ces noms et acheminer vers le bon service interne. Cela permet de virtualiser sur une seule IP publique plusieurs services web. Nginx Proxy Manager et Traefik gèrent très bien le routage par nom de host, et Traefik peut même définir une règle par wildcard ou par motif d’URL si nécessaire. Veillez à configurer des  **entrées DNS**  pointant vers votre IP publique (et à mettre à jour si IP dynamique, via un service DDNS ou l’API du registrar).
    
-   **HTTPS obligatoire**  :  **Chiffrer les communications**  est indispensable. Le proxy doit présenter des certificats SSL valides pour vos domaines. Heureusement, NPM comme Traefik automatisent Let’s Encrypt : il suffit d’activer l’option (et d’avoir son domaine correctement pointé DNS). Dans NPM, on coche “Request a new SSL Certificate” et “Force SSL”. Dans Traefik, on configure un resolver ACME (email, méthode http-01 ou dns-01) – par exemple via la section  certificatesResolvers  du fichier Traefik  . Une fois les certificats en place, on doit  **rediriger tout le trafic HTTP vers HTTPS**  pour éviter toute fuite en clair. NPM propose le bouton  **Force SSL**  qui fait ajouter une règle de redirection 301 automatique. Avec Traefik, on définit soit un middleware  redirectScheme  global, soit on utilise la directive d’entryPoint : par exemple le fichier de config peut contenir:

```yaml
entryPoints:
  http:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: https
          scheme: https
  https:
    address: ":443"
    # ...
```


-   qui signifie qu’on redirige toute connexion HTTP vers l’entrée HTTPS correspondante  . Après ce réglage, toute tentative en  http://  sera automatiquement renvoyée en  https://, garantissant que même les utilisateurs ou liens non sécurisés finissent chiffrés.
    
-   **Authentification et autorisation**  : Comme discuté plus haut, il est fortement recommandé de protéger par un mécanisme d’authentification tout service ouvert sur Internet (à moins que le service ait déjà sa propre auth robuste). Ainsi, on évite qu’une application vulnérable soit accessible publiquement sans contrôle. Typiquement, on utilisera Authelia ou au moins une authentification basique pour les applications sensibles. Authelia permet en plus de définir des  **règles d’autorisation**  fines (par groupe d’utilisateurs, par adresse IP source, etc.)  , ce qui peut servir à n’autoriser que certains utilisateurs à accéder à une ressource donnée.
    
-   **Durcissement et filtres**  : Le proxy peut également servir de  **première ligne de défense applicative**. On peut activer des  **headers de sécurité**  globaux (HSTS, XSS-Protection, Content-Security-Policy, etc.) sur le proxy. Par exemple, Nginx Proxy Manager permet d’ajouter des custom headers, et Traefik propose un middleware  headers(souvent un preset de “Secure Headers”) qu’on peut appliquer sur toutes les routes  . De même, on peut filtrer ou bloquer certains patterns de requêtes connus comme malveillants (_Block Common Exploits_  est une option de NPM qui ajoute quelques règles Nginx de base). Pour aller plus loin, on pourrait intégrer un WAF (pare-feu applicatif) : Traefik, via un plugin comme  **Traefik Pilot**  (ou en le chaînant avec un conteneur ModSecurity), et Nginx via le module ModSecurity ou en plaçant Cloudflare en proxy en amont par exemple. Dans un homelab, une approche populaire est d’utiliser  **CrowdSec**  couplé au reverse proxy : CrowdSec analyse les logs (par ex. les accès Nginx ou Traefik) et bannit automatiquement les IP malveillantes (scans, brute-force) via un bouncer. Traefik possède un middleware CrowdSec natif en plugin, ce qui facilite l’intégration  .
    

  

En somme, exposer un service interne via un reverse proxy nécessite de  **tout ramener à un point d’entrée sécurisé unique**  (le proxy), d’y appliquer  **chiffrement et authentification**, et de segmenter ce qui est accessible de ce qui ne l’est pas. Un homelab de cybersécurité tirera profit de cette configuration pour simuler une architecture proche d’une DMZ d’entreprise, où l’on peut pratiquer la gestion des accès externes de manière sécurisée.

  

### **Séparation des zones réseau (segmentation)**

  

Dans un contexte cybersécurité, il est judicieux de  **segmenter le réseau du homelab**  en zones de confiance (par exemple : réseau  _DMZ_  pour les services exposés, réseau  _interne_  pour les données sensibles, réseau  _management_  pour l’administration, etc.). Le reverse proxy peut jouer un rôle central dans cette séparation :

-   On peut placer le serveur qui héberge Traefik/Nginx Proxy Manager dans une  **zone DMZ**  (réseau isolé ne contenant que les machines “frontales”). Ce proxy DMZ a des accès filtrés vers la zone interne pour joindre les services, tandis que le trafic entrant depuis Internet est limité à aller vers le proxy uniquement. Ainsi, si un service interne est compromis, l’attaquant doit encore traverser le proxy pour en sortir. Et si le proxy lui-même est compromis, il est dans une DMZ qui limite les dégâts sur le reste du réseau.
    
-   Grâce au reverse proxy, on peut garder  **fermés tous les ports**  des services internes (ils écoutent seulement en local ou sur le LAN interne). Par exemple, une appli web sur un serveur interne n’écoute que sur son port 8080 en interne ; seul le proxy y accède pour relayer les requêtes externes. Cela réduit la surface d’attaque réseau. On autorisera dans le firewall interne uniquement l’IP du proxy DMZ à contacter l’IP du service sur le port requis.
    
-   Le proxy peut également faire office de  **passerelle entre VLAN**  : s’il a des interfaces (ou VLAN) sur plusieurs réseaux, il peut accepter des requêtes d’un réseau A et aller chercher la ressource sur un réseau B. Cependant, il faut être prudent : le proxy ne doit pas devenir une porte dérobée entre réseaux segmentés. Il convient de n’autoriser que les flux nécessaires (typiquement HTTP/HTTPS du proxy DMZ vers le backend interne sur le port du service web).
    
-   Authelia, dans une optique Zero Trust, peut renforcer la séparation : on peut exiger une authentification forte même pour accéder à certains services depuis la zone  _interne_. Par exemple, si le homelab a un réseau WiFi “IoT” moins sûr, on peut exiger que les utilisateurs de ce réseau s’authentifient via Authelia pour accéder à l’interface d’administration d’un serveur, même si c’est techniquement en interne.
    

  

En termes de configuration, la séparation des zones se fait hors du reverse proxy (c’est une architecture réseau), mais le proxy doit être configuré en conséquence : bonnes adresses IP, règles firewall appropriées, etc. Traefik ou Nginx peuvent écouter sur plusieurs interfaces si nécessaire, ou on peut déployer deux instances de proxy (un dans la DMZ externe, un interne) cascade si on veut complexifier l’architecture. Pour un homelab néanmoins, une instance dans la zone exposée suffit généralement, parlant aux services en interne.

  

### **Filtrage et supervision des requêtes**

  

Un homelab de cybersécurité sert souvent à expérimenter avec des outils de détection et de protection. Le reverse proxy est un endroit stratégique pour implémenter du  **filtrage de requêtes**  et de la  **supervision**  :

-   **Journaux d’accès centralisés**  : Le proxy enregistre toutes les requêtes entrantes (domaines, URL, IP source, réponse…). Ceci offre une visibilité globale sur qui accède à quoi. On peut connecter ces logs à des outils SIEM ou de monitoring (par ex. Grafana/Loki pour Traefik, ou un ELK/Graylog) afin de détecter des anomalies. Traefik fournit un log d’accès détaillé (format JSON possible) configurable dans traefik.yml  . Nginx Proxy Manager logge via Nginx (format combiné classique) et on peut ajuster la verbosité.  **Analyser ces logs**  permet de repérer des scans, des tentatives d’exploitation (URL étranges), etc.
    
-   **Blocage automatique d’IP malveillantes**  : Couplé aux logs, on peut intégrer des solutions comme  **Fail2Ban**  ou  **CrowdSec**. Fail2Ban peut surveiller les logs Nginx pour des motifs (codes 401 répétitifs, 404 suspectes) et insérer des règles firewall pour bannir l’IP temporairement. CrowdSec, plus évolué, utilise des  _scenarios_  pour détecter comportements malveillants (brute-force, scan, etc.) et peut agir au niveau du proxy (bouncer Traefik) ou du firewall système. Traefik, grâce à son plugin CrowdSec ou fail2ban, facilite cette automatisation  . Pour NPM, on peut déployer CrowdSec sur la machine et le configurer pour parser les logs Nginx.
    
-   **Web Application Firewall (WAF)**  : L’ajout d’un WAF permet de filtrer des requêtes en fonction de leur contenu (payload) pour bloquer des attaques web connues (injections SQL, XSS, etc.). Nginx peut intégrer ModSecurity v3 en tant que module (certaines images Docker custom de NPM existent incluant ModSec). Traefik, lui, n’intègre pas nativement de WAF complet, mais on peut mettre Traefik derrière un conteneur SWAG/NGINX+ModSecurity, ou utiliser le plugin  _OHWF_  (Open Hybrid WAF) expérimental. Une alternative courante en homelab est d’utiliser Cloudflare en proxy inverse en amont du homelab : Cloudflare fournit un WAF géré qui stoppe déjà beaucoup d’attaques avant même qu’elles n’atteignent votre reverse proxy. Cependant, cela signifie confier le trafic à un tiers, ce qui sort un peu du cadre purement auto-hébergé.
    
-   **Headers et CSP**  : Outre bloquer ce qui est mauvais, on peut ajouter ce qui est bon. Par exemple, ajouter des  **headers de sécurité**  comme HSTS (Strict-Transport-Security), X-Content-Type-Options, X-Frame-Options, Content-Security-Policy, etc., pour réduire la surface aux attaques client. Traefik a un middleware  secureHeaders(configurable via fichier dynamique) qui fournit un ensemble de ces headers recommandés  . Nginx permet de les ajouter via  add_header. Dans un homelab, c’est un bon exercice de configurer ces en-têtes correctement pour apprendre le hardening web.
    

  

En combinant ces mesures de filtrage et de supervision, le homelab peut simuler un environnement  **sous surveillance**, comme en entreprise, où chaque requête est loggée, analysée, et potentiellement bloquée si suspecte. Cela permet de tester des scénarios d’attaque en conditions réalistes et de valider l’efficacité des contre-mesures (par exemple, lancer un scan depuis l’extérieur et voir CrowdSec bannir l’IP dans Traefik, etc.).

  

## **Fonctionnalités de sécurité avancées (LDAP, OAuth2, SSO, 2FA)**

  

Au-delà des configurations de base, un homelab de sécurité peut tirer profit de fonctionnalités avancées offertes par nos outils :

  

### **Intégration LDAP/OAuth2 pour l’authentification centralisée**

  

Dans un contexte professionnel, l’authentification des utilisateurs se fait souvent via un  **annuaire central**  (LDAP/Active Directory) ou un  **fédération d’identité**  (SSO OAuth2/OIDC, SAML…). Il est intéressant de reproduire cela en homelab. Ni Nginx Proxy Manager ni Traefik n’intègrent à eux seuls un serveur LDAP ou OAuth2, mais  **Authelia**  peut servir d’interface :

-   **Authelia + LDAP**  : Authelia peut être configuré pour utiliser un  **backend LDAP/AD**  comme source des utilisateurs et groupes. Par exemple, on peut déployer une petite instance OpenLDAP ou FreeIPA dans le homelab, y gérer quelques comptes, et configurer Authelia en  _authentication_backend: ldap_  (plutôt que  _file_). Ainsi, lorsqu’un utilisateur se connecte via Authelia, ses identifiants sont vérifiés auprès de l’annuaire LDAP. Cela permet de centraliser l’authentification de tout le homelab sur un référentiel unique (comme ce serait le cas avec un Active Directory en entreprise). Authelia supporte les opérations LDAP usuelles et même la réinitialisation de mot de passe LDAP via un lien mail  . De plus, en se basant sur les  _groups_  LDAP, on peut définir des politiques d’accès dans Authelia (par exemple, seul le groupe “admins” a accès à l’interface de gestion proxmox). Cette intégration est un excellent exercice pour un homelab car elle touche aux notions d’annuaire, de schéma utilisateur et de synchronisation.
    
-   **SSO OAuth2/OIDC**  : Authelia se positionne comme un  **fournisseur OpenID Connect 1.0 certifié**  . Cela signifie qu’il parle le protocole moderne d’authentification (OIDC est une surcouche d’OAuth2) utilisé par de nombreuses applications web. On peut donc, d’une part, intégrer des applications tierces qui supportent l’OIDC pour déléguer leur login à Authelia. D’autre part, Authelia peut lui-même consommer des identités d’un fournisseur externe OAuth2 ? – Ce cas d’usage est moins courant car Authelia est plutôt fait pour être  **le**  fournisseur. Si l’on souhaitait “Login with Google” par exemple sur nos services, on utiliserait plutôt un outil comme  **OAuth2 Proxy**  ou  **Authentik**  qui ferait office d’intermédiaire avec Google. Nginx et Traefik peuvent s’intégrer avec OAuth2 Proxy de façon semblable à Authelia (via  auth_request  ou forwardAuth).  **Authentik**  et  **Keycloak**  sont deux alternatives populaires à Authelia pour le SSO avancé : Authentik est open-source (Django) et offre plus de connecteurs (par ex. login Google, SAML, etc.), Keycloak est une solution robuste utilisée en entreprise. Cependant, leur mise en œuvre est plus lourde. Dans un homelab cyber, mettre en place Authentik ou Keycloak pour comparer avec Authelia peut être instructif, mais Authelia a l’avantage de la légèreté et de la simplicité pour débuter le SSO.
    
-   **OAuth2 access control**  : Traefik Enterprise (version payante) intègre nativement l’OIDC/OAuth2 et le SSO d’entreprise (avec support JWT, etc.), mais avec Traefik open-source on s’appuie sur les solutions externes citées. Nginx (open-source) peut utiliser des modules Lua ou des modules tiers pour valider des JWT ou interroger un IdP OAuth2, mais cela sort du scope standard de NPM.
    

  

En résumé, l’intégration LDAP/OAuth2 dans le homelab se fait via un composant dédié (Authelia, Keycloak, etc.) couplé au reverse proxy. Cela permet de reproduire une  **authentification centralisée**  comme on en trouve en milieu professionnel, et d’expérimenter avec des protocoles standards (LDAP, OAuth2/OIDC). Pour un ingénieur sécu, c’est l’occasion de se familiariser avec la gestion des identités, la délégation d’authentification, et de comprendre les défis de l’intégration SSO.

  

### **Protection contre les attaques par force brute**

  

Les attaques par  **brute-force**  (essais répétés de mots de passe) sont une menace classique dès qu’un service est accessible sur Internet. Dans un homelab exposé, on veillera à mettre en place des mécanismes de protection :

-   **Authelia – Regulation**  : Authelia intègre nativement un module de limitation des tentatives de login. On peut configurer des paramètres comme  max_retries  (nombre de tentatives avant blocage),  find_time  (intervalle de temps de comptabilisation) et  ban_time  (durée du bannissement)  . Par exemple, on pourrait autoriser 5 essais en 2 minutes avant de bloquer l’utilisateur pendant 10 minutes. Ce mécanisme de  **Login Regulation**  empêche un attaquant d’essayer une infinité de mots de passe sur un compte  . Le bannissement est généralement appliqué par identifiant (et potentiellement par IP). Authelia journalise ces événements, ce qui permet de détecter une attaque en cours.
    
-   **Fail2Ban**  : Pour les services non protégés par Authelia (ou même en complément), Fail2Ban reste un allié précieux. En homelab, on peut configurer Fail2Ban pour surveiller les logs Nginx Proxy Manager. Par exemple, repérer 401 Unauthorized répétés sur une URL d’auth -> ban de l’IP au niveau iptables. Sur Traefik, on peut envisager de le faire aussi via les logs d’accès (moins trivial car Traefik n’a pas de log “erreur 401” distinct, mais c’est faisable). L’avantage de Fail2Ban est sa simplicité et son efficacité pour bannir au niveau réseau, empêchant l’attaquant d’envoyer la suite des requêtes.
    
-   **Traefik plugins**  : Comme mentionné précédemment, Traefik dispose d’un plugin  **Fail2Ban**  officiel qui reproduit ce comportement directement dans le proxy. Il peut suivre les 401 sur un middleware d’authentification par exemple et couper l’accès. Aussi, l’utilisation de  **CrowdSec**  apporte une dimension communautaire : si une IP est connue pour attaquer d’autres membres de la communauté, elle peut être pré-bannie (système de blocklist collaborative). On peut configurer le bouncer CrowdSec pour qu’il renvoie directement une page de blocage depuis le proxy si l’IP est bannie, avant même d’arriver aux services.
    
-   **Complexité des mots de passe & 2FA**  : La meilleure défense anti brute-force reste d’exiger des  **mots de passe forts**  et idéalement un  **second facteur**. Authelia impose par exemple une politique de mot de passe (paramétrable via l’option zxcvbn pour exiger une certaine robustesse du mot de passe)  . Et bien sûr, avec le 2FA activé, un mot de passe compromis ne suffit plus, ce qui décourage fortement le brute-force. Dans un homelab, on peut tester différentes méthodes de second facteur : TOTP (Google Authenticator, etc.),  **WebAuthn**  (clé U2F type YubiKey), ou  **Duo push**  . Authelia supporte ces méthodes multiples – par exemple WebAuthn permet d’apprendre à intégrer l’authentification par clé physique ou empreinte digitale, ce qui est très intéressant en cybersécurité moderne.
    

  

En conclusion sur la brute-force, combiner un  **verrou logiciel**  (Authelia ou Fail2Ban qui bloque après X essais) avec une  **authentification forte**  (mots de passe solides + second facteur) offre une défense efficace. Un homelab bien configuré doit ainsi résister aux bots qui tentent des attaques automatisées sur les pages de login. N’oublions pas la surveillance : il est formateur de constater dans les logs ces tentatives et de vérifier que les contre-mesures réagissent (ex: voir l’IP passer en bannie après trop d’échecs).

  

### **Authentification unifiée et multi-facteurs (SSO & MFA)**

  

Nous avons déjà évoqué plusieurs fois le SSO et le MFA, que propose notamment Authelia. Regroupons ici ces notions pour bien comprendre leur apport dans le homelab :

-   **Single Sign-On (SSO)**  : Le SSO vise à permettre à un utilisateur de s’authentifier  **une seule fois**  pour accéder à plusieurs applications différentes, sans avoir à re-saisir ses identifiants pour chaque service. Dans notre stack, Authelia réalise cela en émettant un  **cookie de session**  après login, valable pour l’ensemble des sous-domaines protégés (on définit un domaine racine commun, par ex.  *.monlab.fr)  . Ainsi, l’utilisateur qui se connecte sur  service1.monlab.fr  via Authelia obtient un cookie de session (sur  .monlab.fr), et lorsqu’il ira sur  service2.monlab.fr, le reverse proxy présentera ce cookie à Authelia qui verra que la session est valide et laissera passer automatiquement  . L’expérience est transparente : une seule page de login, puis navigation fluide entre services. C’est très confortable pour l’utilisateur et plus sécurisé (on évite d’avoir des mots de passe multiples circulant). Pour le mettre en œuvre en homelab, on s’assure que  session.domain  est bien réglé dans Authelia (par ex.  monlab.fr) et on protège tous les services via Authelia. On peut tester le SSO en ouvrant différents navigateurs ou en invalidant le cookie pour voir comment la session se propage.
    
-   **Multi-Factor Authentication (MFA)**  : C’est un pilier de la sécurité moderne. Authelia permet d’activer le  **2FA**  (ou même 3FA) pour les domaines qu’on veut. Dans la config  _access_control_, on peut définir  policy: two_factor  pour certaines ressources sensibles  . Au login, après le mot de passe, Authelia va alors demander le code TOTP (ou une validation via l’appli Duo, ou une authentification WebAuthn/Passkey) avant de valider la session. L’intégration de MFA dans un homelab est un excellent moyen de se familiariser avec ces technologies. Par exemple, on peut enregistrer une clé U2F sur Authelia et ainsi utiliser une clé physique pour se logguer – ce qui ajoute une couche “quelque chose que l’on possède” en plus du mot de passe. Authelia supporte également les  **Passkeys**  (technologie plus récente visant à remplacer les mots de passe)  , ce qui permet d’expérimenter les futures tendances de l’authentification passwordless.
    
-   **Granularité et exceptions**  : On peut vouloir SSO + MFA pour la plupart des services, mais peut-être pas pour tous. Authelia offre une granularité par règle : on peut mettre certaines applications en  one_factor  (juste mot de passe) et d’autres en  two_factor  . On peut même combiner avec des conditions réseau – par exemple, exiger 2FA seulement quand on est en dehors du LAN, mais en one_factor sur le LAN (via la directive  networks  dans les règles). Cela permet de reproduire une politique de confiance conditionnelle (un peu comme du MFA contextuel en entreprise). C’est à l’appréciation de chacun en homelab ; du point de vue sécurité pure, il est recommandé d’activer le MFA pour les accès externes au minimum.
    

  

En mettant en place le SSO et MFA dans le homelab, on se rapproche des standards d’infrastructure sécurisée d’une entreprise. C’est très formateur pour un ingénieur sécurité de comprendre les mécanismes sous-jacents : cookies de session, jetons JWT éventuellement (Authelia peut émettre des JWT OIDC), protocole WebAuthn, algorithme TOTP, etc. De plus, cela améliore nettement la  **sécurité**  de votre homelab réel, ce qui n’est pas négligeable si vous exposez des services personnels.

  

## **Conclusion**

  

Mettre en œuvre un reverse proxy dans un homelab est quasiment un passage obligé pour qui veut exposer des services de manière maîtrisée.  **Nginx Proxy Manager**  et  **Traefik**  sont deux excellentes options qui répondent à des profils différents : NPM offre la simplicité d’une interface graphique et la robustesse de Nginx – idéale pour débuter ou pour un lab à configuration ponctuelle –, tandis que Traefik apporte l’automatisation et la souplesse pour un lab très containerisé ou évolutif, avec des fonctionnalités avancées intégrées (découverte, middlewares). Dans un contexte de cybersécurité, Traefik se montre très adapté grâce à ses plugins de sécurité et son intégration documentée avec des outils comme Authelia  , mais NPM peut tout à fait être utilisé de manière sécurisée également (en y ajoutant manuellement les mêmes composants).

  

L’outil  **Authelia**, en particulier, s’avère un complément précieux pour un homelab sécurisé. En déployant Authelia aux côtés du reverse proxy, on dote son lab d’un véritable  **système d’authentification unifiée à double facteur**, comparable à ce qu’on trouve en production dans les entreprises, le tout avec des solutions open-source. Ceci transforme le homelab en terrain d’entraînement pour implémenter et tester des politiques de sécurité (SSO, MFA, restrictions granulaires) sur ses propres services auto-hébergés.

  

En fin de compte, le choix précis de la stack (Nginx vs Traefik, avec ou sans Authelia) dépendra de vos objectifs d’apprentissage et de vos contraintes. Un  **ingénieur en cybersécurité**  apprendra beaucoup en expérimentant les deux approches : commencer par Nginx Proxy Manager pour appréhender les concepts de base (reverse proxy, certificats, DNS, redirections), puis monter en puissance avec Traefik pour l’aspect “Infra as Code” et intégration continue, tout en ajoutant Authelia pour la couche IAM (Identity and Access Management). Le homelab ainsi constitué permettra de simuler bon nombre de scénarios de sécurité (attaques web, tests d’intrusion depuis l’extérieur, mise en place de défenses actives) dans un environnement contrôlé et modulable.

  

En somme, Nginx Proxy Manager, Traefik et Authelia forment un trio complémentaire pour construire un homelab à la fois  **fonctionnel**  (multi-services exposés proprement) et  **sécurisé**  (contrôle fin des accès, surveillance, résilience). À vous de jouer pour les configurer selon vos besoins, et n’oubliez pas : la documentation officielle et la communauté sont d’une grande aide pour approfondir chaque composant et résoudre les éventuels écueils techniques rencontrés en chemin.
 
