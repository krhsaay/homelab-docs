# **Pi-hole, AdGuard Home et Technitium DNS : comparatif et configurations en homelab**

  

## **Comparatif technique détaillé**

  

**Présentation générale :**  Pi-hole, AdGuard Home et Technitium DNS Server sont trois solutions populaires de DNS filtrant pour bloquer les publicités et protéger un réseau domestique. Toutes trois fonctionnent comme des serveurs DNS locaux capables de  **bloquer les domaines indésirables**  (publicité, trackers, malwares, sites adultes) à l’aide de listes noires. Elles disposent également de fonctionnalités communes telles que la journalisation des requêtes DNS et une interface d’administration web  . Cependant, elles diffèrent sur leur  **architecture technique et fonctionnalités avancées**. Le tableau ci-dessous résume certaines capacités :

-   **Blocage de domaines indésirables :**  **Pi-hole**,  **AdGuard Home**  et  **Technitium DNS**  assurent tous un DNS sinkhole efficace. Ils peuvent charger des listes de domaines à bloquer (ads, malwares, etc.), appliquer le Safe Search et utiliser des listes personnalisées  . Le blocage par expressions régulières est aussi supporté par Pi-hole et AdGuard.  **En pratique, l’efficacité de filtrage par défaut est similaire**  – tous deux utilisent des listes de blocage comparables et bloquent un nombre proche de publicités dès l’installation, même s’il est conseillé d’ajouter d’autres listes pour améliorer la couverture  . Technitium intègre également ces fonctionnalités de base de blocage DNS.
    
-   **Performance et ressources :**  Les différences de performance entre Pi-hole et AdGuard Home sont  **minimes**. Des tests montrent que les temps de réponse DNS sont équivalents (18ms pour Pi-hole vs 21ms pour AdGuard dans un exemple, écart négligeable)  . Côté empreinte système, AdGuard Home est écrit en Go tandis que Pi-hole s’appuie sur dnsmasq et une interface PHP, ce qui fait souvent dire que Pi-hole consomme un peu plus de RAM. En réalité, avec une configuration de base, Pi-hole utilise environ  **30 Mo de RAM de plus**  qu’AdGuard Home, différence imperceptible sur du matériel moderne  .  **Avec de très grandes listes de blocage**, Pi-hole a été mesuré autour de ~100 Mo de RAM contre ~250 Mo pour AdGuard dans un test sur Portainer  – mais d’autres retours signalent l’inverse ou un écart moindre, selon l’environnement.  **Globalement, les trois solutions restent légères**  et tournent sans problème sur un Raspberry Pi ou un petit NAS  . Le  **CPU**  n’est généralement pas un facteur limitant (quelques % d’utilisation au plus). Les trois chargent les listes de blocage en mémoire pour une résolution rapide, il faut donc prévoir suffisamment de RAM si vous importez des listes massives  .
- -   **Mécanisme DNS :**  Pi-hole et AdGuard Home agissent principalement comme  **serveurs DNS cache/forward**  : ils reçoivent les requêtes des clients, bloquent celles correspondant aux domaines filtrés, et transfèrent les autres à un serveur DNS amont (par exemple Cloudflare, Quad9, ou un résolveur local). Pi-hole n’embarque pas de résolveur récursif complet par défaut, mais il est courant de l’associer à  _Unbound_  pour résoudre directement depuis la racine. AdGuard Home, lui, peut également fonctionner en mode récursif (il dispose d’une option pour activer un résolveur DNS complet en interne) ou en mode  **forwarder**  vers des serveurs chiffrés (DoH/DoT) grâce à sa prise en charge native de DNS-over-HTTPS, DNS-over-TLS et même DNS-over-QUIC  .  **Technitium DNS**  se distingue ici car c’est un  **serveur DNS complet (récursif et faisant autorité)**  : il intègre nativement la résolution récursive  _et_  peut héberger des zones DNS locales autoritaires  . En clair, Technitium offre d’emblée ce que Pi-hole/AdGuard font plus un résolveur interne (plus besoin d’installer Unbound) et un serveur faisant autorité sur vos domaines locaux. Il supporte aussi directement DoH/DoT en sortie  **et en entrée**, ce qui permet de servir vos clients via DNS chiffré si souhaité (AdGuard Home peut aussi servir en DoH/DoQ en configurant les fonctionnalités avancées).
    
-   **Interface et gestion :**  **Pi-hole**  offre une interface web simple et épurée pour configurer les listes noires/blanches, voir les journaux de requêtes et quelques réglages réseau de base. Son interface, bien que moins « moderne », reste fonctionnelle et soutenue par une  **forte communauté**  (nombreux tutoriels, support, etc.)  .  **AdGuard Home**  dispose d’une interface plus moderne et conviviale, avec des tableaux de bord colorés et des menus bien organisés. La configuration initiale est facilitée (tout en un seul binaire), et de nombreuses options (DNS chiffré, contrôle parental, filtres personnalisés) sont activables en quelques clics  . En revanche, certains réglages avancés de DNS local sur AdGuard sont un peu cachés (il faut ajouter des enregistrements DNS locaux via les  _“Custom Rules”_manuellement)  , alors que Pi-hole propose une section dédiée “DNS Local” plus intuitive pour ajouter des noms locaux  .  **Technitium DNS**, malgré son ampleur fonctionnelle, fournit également une interface web complète. Les retours d’expérience notent qu’elle est  **très riche en options tout en restant user-friendly**, similaire à Pi-hole/AdGuard voire plus intuitive sur certains points  . On peut tout configurer depuis le GUI ou via une API HTTP, sans devoir éditer manuellement des fichiers de configuration  . L’interface de Technitium permet par exemple de créer des zones DNS, des règles de transfert conditionnel, de multiples portées DHCP, etc., directement via le web.
    
-   **Intégration réseau et fonctionnalités :**  Les trois solutions peuvent fonctionner sur la plupart des plateformes (Linux, Docker, Raspberry Pi, etc.)  .  **Chacune peut servir de serveur DHCP**  pour votre LAN, remplaçant le DHCP de votre box/routeur si nécessaire  . Pi-hole et AdGuard Home gèrent un seul domaine DHCP/local à la fois, tandis que  **Technitium DNS peut définir plusieurs plages DHCP**  pour couvrir plusieurs VLAN/réseaux isolés depuis une seule instance  . C’est un atout dans un homelab segmenté en multiples sous-réseaux : Technitium permet de  **servir plusieurs VLAN**  en DHCP/DNS distincts via une configuration centralisée. Côté DNS local, Pi-hole et AdGuard autorisent l’ajout de quelques enregistrements locaux ou la redirection conditionnelle (ex. transférer les requêtes d’un suffixe local vers un autre DNS) – fonctionnalité dite  _conditional forwarding_  pour résoudre les noms de votre LAN via le DNS de votre routeur.  **Technitium**  pousse plus loin en permettant de  **créer des zones locales complètes**  (par exemple une zone “lab.home.arpa” avec tous vos enregistrements) et d’activer des  _serveurs secondaires_  pour ces zones (transfert de zone possible)  . Il supporte aussi la notion de  _Conditional Forwarder_  plus avancée : pour un domaine donné, vous pouvez spécifier un serveur DNS différent, tout en pouvant  **surcharger certains enregistrements**  de ce domaine localement  . Enfin, en termes d’intégration, Pi-hole et AdGuard sont souvent utilisés en  **complément du routeur/pare-feu**  : il faut configurer votre DHCP (sur la box ou OPNsense par exemple) pour distribuer l’adresse IP du Pi-hole/AdGuard comme DNS aux clients du réseau. Technitium pourrait tout à fait être utilisé de la même manière, ou même installé directement sur la machine faisant routeur si on le souhaite (il remplace alors avantageusement le DNS de base du routeur). Notons qu’AdGuard Home existe aussi en  **package sur certaines appliances**  (par ex. sur OPNsense/pfSense via des plugins tiers), ce qui peut simplifier son déploiement dans un firewall existant.
    
-   **Contrôle granulaire et profils :**  Depuis la version 5,  **Pi-hole**  offre une gestion par groupes de clients : on peut assigner des appareils à des groupes et appliquer à chaque groupe des listes de blocage différentes (par exemple, ne pas bloquer les mêmes sites pour les enfants et les adultes). AdGuard Home permet également de créer des  **règles par client**  (basées sur l’IP ou le nom du client) et de filtrer différemment selon l’appareil  . La flexibilité de Technitium à ce sujet est encore en développement : il supporte bien les listes noires et blanches globales et la création de profils, mais  **les règles fines par client/groupe sont plus limitées**  pour l’instant  . En revanche, Technitium offre des fonctionnalités uniques comme la possibilité d’héberger un  **cache DNS partagé entre réseaux**  ou d’opérer un  **serveur racine local**  (il peut télécharger la zone racine ICANN et agir en résolveur totalement autonome hors-ligne si on le souhaite)  .
    

  

**En résumé**, Pi-hole et AdGuard Home couvrent l’essentiel des besoins d’un homelab en blocage DNS, avec un avantage pour AdGuard sur les fonctionnalités intégrées (chiffrement DNS, contrôle parental) et pour Pi-hole sur la simplicité et la communauté. Technitium DNS va plus loin en combinant les fonctions de Pi-hole/AdGuard avec celles d’un serveur DNS professionnel (zones locales, résolveur intégré, etc.)  . Ce surplus de fonctionnalités peut sembler  _“overkill”_  pour un petit réseau, mais offre une  **grande flexibilité pour un ingénieur sécurité**  souhaitant un laboratoire évolutif. Le tableau de comparaison suivant (d’après Supernetworks) illustre que Technitium couvre la plupart des cases de Pi-hole/AdGuard, et apporte des plus comme le DNS récursif et la gestion multi-réseaux  . Le choix dépendra donc de vos  **priorités**  : légèreté et support communautaire (Pi-hole), interface moderne et tout-en-un (AdGuard Home), ou puissance DNS maximale (Technitium).

  

## **Exemples de configurations types**

  

Pour exploiter ces outils DNS dans un homelab, voici quelques  **architectures types**  et configurations recommandées :

  

### **Configuration “DNS primaire/secondaire” pour résilience**

  

Il est judicieux d’avoir  **deux instances DNS filtrantes**  pour éviter un point unique de défaillance. Par exemple, déployer  **deux Pi-hole**  (ou AdGuard) sur deux machines distinctes : l’un sera DNS primaire, le second en DNS secondaire. Via le DHCP, on distribue les deux adresses DNS aux clients. En fonctionnement normal, les clients interrogeront surtout le primaire, le secondaire prenant le relais si le premier est injoignable (timeout). Cette redondance assure que même pendant la maintenance ou la panne d’un serveur DNS, le réseau continue à résoudre les noms  . Il est possible d’utiliser  **des solutions mixtes** : par ex. un Pi-hole comme serveur principal et un AdGuard Home en secondaire, ce dernier configuré avec les mêmes listes de blocage pour un résultat similaire. Certains homelabs combinent aussi  **Technitium et Pi-hole**  : Technitium servant de résolveur DNS rapide en interne, et Pi-hole agissant en filtrage en amont pour certains clients spécifiques  . L’important est de conserver une cohérence (mêmes listes) ou de décider que le secondaire ne filtre pas (selon que l’on préfère la  **disponibilité**  ou le  **filtrage strict**). Dans tous les cas, réglez des  **timeouts raisonnables**  sur le primaire afin que le client bascule vite sur le secondaire en cas de souci (les OS utilisent généralement ~5 secondes par défaut).

  

En environnement virtualisé, on peut également mettre en place un mécanisme de  **failover**  plus poussé : par exemple exécuter deux conteneurs Pi-hole en parallèle et utiliser keepalived pour partager une IP virtuelle flottante entre eux. Si le Pi-hole principal tombe, le secondaire annonce l’IP virtuelle et prend immédiatement le relais, évitant même le délai de bascule côté clients. Ce type de HA est plus complexe, mais peut se justifier dans un lab critique.

  

### **DNS avec chiffrement (DoH/DoT) en amont et en interne**

  

**Chiffrer les requêtes DNS**  peut être intéressant pour la confidentialité. Les trois solutions supportent le chiffrement DNS en  **amont**  (vers les serveurs publics) : Pi-hole peut être couplé à un client DoH/DoT (ex: Cloudflared, Stubby) ou à Unbound pour chiffrer entre Unbound et les serveurs racine. AdGuard Home et Technitium intègrent nativement le support des  **DNS-over-HTTPS (DoH)**  et  **DNS-over-TLS (DoT)**  pour les requêtes sortantes  . Il suffit de configurer dans l’UI l’adresse du resolver DoH de votre choix (ex :  _https://dns.quad9.net/dns-query_  pour Quad9).

  

En  **interne**, AdGuard Home peut aussi se comporter en  **serveur DoH/DoT**  pour vos clients : on peut le configurer pour écouter en TLS (port 853) ou HTTPS (port 443/API DNS) afin que les postes clients puissent chiffrer leurs requêtes jusqu’au serveur DNS local  . Technitium également permet cette configuration (il peut écouter en DoH/DoT sur votre réseau local si vous activez HTTPS avec un certificat valide). Pi-hole n’a pas cette fonctionnalité intégrée, mais on peut la reproduire en plaçant un  **proxy DoH**  devant Pi-hole (par exemple, un conteneur  _cloudflared_  configuré en mode serveur DoH local). Ainsi, les PC/ smartphones du réseau peuvent être configurés pour utiliser l’URL DoH locale (ou le port TLS local) au lieu du port 53 classique, ajoutant une couche de chiffrement sur le segment local pour éviter qu’un intrus sur le WiFi ne voie les requêtes en clair.

  

Pour un  **homelab de sécu**, cette configuration DoH interne peut être utile pour tester le comportement de machines dans un contexte où le DNS est chiffré de bout en bout. Cependant, notez que si vos clients utilisent DoH vers l’extérieur (par exemple navigateur configuré en DoH vers Cloudflare), cela  **contourne**  votre serveur DNS filtrant local. Une solution est justement d’héberger un serveur DoH local (avec AdGuard/Technitium) et de forcer via les politiques du navigateur ou de l’OS l’utilisation de celui-ci, afin de garder le contrôle sur les requêtes.

  

### **Mode « proxy DNS » vs mode résolveur local**

  

On parle ici du  **fonctionnement interne**  du serveur DNS filtrant. En  **mode proxy/forwarder**, le serveur relaie la requête vers un ou plusieurs serveurs DNS externes (par ex. les DNS de FAI, Cloudflare, etc.) après avoir filtré. Ce mode est le plus simple :  **Pi-hole**  et  **AdGuard Home**  l’utilisent par défaut (on configure une liste de serveurs DNS en amont, et le logiciel fait du caching + filtrage).  **Technitium DNS**  peut aussi opérer ainsi, mais offre en plus le  **mode résolveur récursif**  complet : dans ce mode, il va interroger directement la racine internet, puis chaque TLD, etc., sans dépendre d’un serveur tiers.  **Pi-hole**  peut être converti en résolveur local en l’associant à  _Unbound_  (Unbound fait la récursion, Pi-hole filtre le résultat)  .  **AdGuard Home**  a une option expérimentale pour la récursion, mais elle n’est pas aussi aboutie qu’Unbound ou Technitium en termes de fonctionnalités DNSSEC par exemple.

  

**Quel mode choisir ?**  En homelab, le mode forwarder sur DNS publics (Cloudflare/Quad9) avec DoH/DoT est souvent suffisant et facile à mettre en place. Le  **mode résolveur local**  apporte une indépendance totale vis-à-vis des DNS tiers et évite de partager vos requêtes à un service externe, au prix d’un léger surcroît de trafic (il doit revalider toutes les DNSSEC signatures lui-même, etc.). Un ingénieur cybersécurité appréciera le résolveur local pour supprimer une dépendance externe et contrôler entièrement la chaîne (ex: pas de risque d’une réponse altérée par un FAI malveillant). Technitium étant  _“tout-en-un”_, il est idéal pour tester ce scénario : vous pouvez désactiver toutes les forwarders et le laisser gérer la résolution récursive  **en interne**  facilement. Pi-hole + Unbound offre le même bénéfice avec un peu plus de configuration manuelle  . Dans tous les cas, vérifiez que DNSSEC est bien activé sur votre résolveur (toutes ces solutions le supportent soit nativement soit via leur upstream) pour garantir l’authenticité des réponses DNS.

  

### **Serveur DNS par VLAN / segmentation du DNS**

  

Dans un homelab segmenté en  **plusieurs VLAN ou sous-réseaux**, il faut décider comment déployer le DNS filtrant :  **centralisé**  (une instance dessert tous les réseaux) ou  **distribué**  (une instance par segment).

-   _Option centralisée :_  Vous pouvez faire tourner une seule instance (ex: un Pi-hole sur le LAN principal) et permettre aux autres VLAN d’y accéder. Cela nécessite que les requêtes DNS des autres réseaux soient routées vers le Pi-hole (règles firewall autorisant l’UDP/TCP 53 depuis les VLAN IoT/Guest vers l’IP du Pi-hole). Sur OPNsense/pfSense, on peut par exemple ouvrir le DNS du Pi-hole à ces réseaux spécifiques. L’avantage : une seule config à gérer (toutes les listes au même endroit) et une vue globale de l’activité DNS. Inconvénient : moins d’isolement (les VLAN dépendent d’un service commun) et complexité de routage. Il faut aussi que le  **DHCP de chaque VLAN pointe vers le Pi-hole central**  comme DNS.
    
-   _Option distribuée :_  Installer une instance par VLAN. Par ex, un Pi-hole dans le VLAN Perso, un AdGuard Home dans le VLAN IoT. Chaque réseau utilise son serveur local (fourni via DHCP). Cela renforce l’isolation (pas de traversée inter-VLAN pour le DNS) et permet de  **personnaliser les filtres par réseau**  (ex: VLAN enfants avec filtrage strict via AdGuard Home + SafeSearch activé  , VLAN lab sans filtrage ou avec des réglages différents). Technitium facilite aussi une approche intermédiaire :  **une seule instance multi-VLAN**. En effet il supporte plusieurs adresses d’écoute et plusieurs scopes DHCP, on peut donc déployer Technitium sur un serveur ayant accès à tous les VLAN (ex: une VM multi-hommes ou directement sur OPNsense) et configurer un  **scope DHCP par VLAN**  ainsi que des règles de filtrage par groupe d’adresses IP. C’est une architecture puissante mais attention à la haute dispo : si cette unique instance tombe, tous les VLAN perdent le DNS. On revient alors à l’option primaire/secondaire pour chaque VLAN ou à un serveur redondant central.
    

  

En pratique, une approche hybride convient souvent : par exemple un Pi-hole principal sur le LAN Perso, accessible aussi par le VLAN invité mais avec une règle firewall qui redirige toute requête DNS émise depuis le VLAN invité vers le Pi-hole (on peut même forcer via une redirection NAT DNS). En parallèle, on peut déployer un second DNS (AdGuard) dédié aux VLAN sensibles (IoT, etc.) pour bien cloisonner.  **Le choix dépend de la taille du lab et des exigences de sécurité**  : plus on centralise, plus c’est facile à administrer mais plus impactant en cas de panne ; plus on distribue, plus on segmente les risques mais plus il y a d’instances à maintenir.

  

## **Scénarios spécifiques d’usage en homelab**

  

Après la technique, voici quelques  **cas d’usage concrets**  de ces solutions DNS dans un homelab orienté cybersécurité :

  

### **Blocage des publicités et du tracking internet**

  

Le cas d’usage numéro un de Pi-hole/AdGuard est le  **blocage des pubs et trackers**  au niveau réseau. En configurant l’un de ces serveurs comme DNS principal du réseau,  _toutes_  les requêtes vers des domaines publicitaires connus sont résolues vers une adresse inexistante (0.0.0.0) ou simplement non résolues, ce qui empêche l’affichage des pubs sur les appareils (TV connectées, smartphones, etc.) sans avoir à configurer chaque appareil individuellement  . Dans un homelab, cela permet non seulement d’accélérer la navigation (moins de contenu chargé) mais aussi de réduire les vecteurs de suivi invisibles.  **AdGuard Home**  et  **Pi-hole**  disposent tous deux de listes de filtres maintenues par la communauté pour bloquer pub, trackers de télémétrie Windows, bannières cookies, etc. Par défaut, leurs listes basiques bloquent une bonne partie des pubs courantes, mais on peut les enrichir (EasyList, listes spécifiques locales, etc.)  .  **Technitium**  utilisera les mêmes listes (format hosts, DOM blocklist) – il suffit d’importer les URLs des listes dans son interface. Un avantage du DNS centralisé : on peut bloquer la pub  **même sur les appareils qui n’acceptent pas d’extensions**  (TV, consoles, IoT).

  

Pour un lab de sécu, on peut aussi se servir de ces blocages DNS comme première barrière contre certaines menaces : par exemple, en chargeant des listes de domaines de malware connus (EmergingThreats, etc.), on empêche des malwares  _éventuellement exécutés dans le lab_  de contacter leur serveur de commande via DNS. Bien sûr, ce n’est pas infaillible (un malware peut contacter directement une IP), mais cela couvre les  **C&C basés sur domaine**  et le tracking de certaines applications indiscrètes  . On peut pousser plus loin le principe en activant sur AdGuard des listes de type  _“Safe Browsing”_  ou contrôle parental (bloquer les domaines adultes, jeux d’argent, etc.)  – utile si le homelab sert de réseau familial également.

  

Un scénario fréquent en homelab est de  **mesurer l’efficacité de ces listes**  : Pi-hole et AdGuard fournissent des statistiques (pourcentage de requêtes bloquées, domaines les plus bloqués, etc.)  . Cela permet à un passionné de cybersécurité d’analyser quel trafic sort de son réseau. Par exemple, on peut découvrir qu’une Smart TV tente de contacter  _dozens_  de domaines de tracking différents – et ajuster les règles en conséquence. On pourra aussi tester différentes approches : par exemple comparer  **Pi-hole vs NextDNS**  (service cloud) sur le blocage de pubs, ou combiner Pi-hole avec un  **bloqueur de contenu local**  sur le navigateur et voir la différence. Ce type d’expérimentation est courant en homelab pour évaluer la meilleure protection multicouche.

  

### **Isolation réseau par le DNS (IoT et environnements sensibles)**

  

Dans une architecture de sécurité, le DNS peut servir à  **isoler ou contrôler les communications**  d’un segment donné. Prenons le VLAN IoT : on souhaite qu’il n’ait accès qu’à Internet de manière très limitée, pour éviter qu’un objet compromis n’attaque d’autres machines. En plus des règles firewall restreignant les destinations IP, on peut utiliser un serveur DNS filtrant dédié à ce VLAN avec une politique très stricte. Par exemple, déployer AdGuard Home pour le VLAN IoT avec  _uniquement_  une whitelist de domaines autorisés (les serveurs officiels des appareils connus) et bloquer  **tout le reste**. AdGuard Home permet de créer des règles globales ou par client, donc on pourrait dire “autoriser seulement ces domaines pour les IP des caméras, etc.”. Technitium, via des zones de transfert conditionnel et surcharges, peut même renvoyer un nom de domaine vers une IP locale factice pour piéger l’appareil ou empêcher la résolution.

  

Un autre scénario d’isolation DNS est d’empêcher qu’un service en  **zone DMZ compromise ne puisse appeler l’intérieur**  : on pourrait configurer le DNS de la DMZ pour qu’il  _ne résolve pas_  les noms internes du LAN. Par exemple, Technitium hébergeant la zone “lab.internal” pour le LAN, mais n’autorisant pas les requêtes venant de la DMZ à y accéder (via un contrôle d’accès sur la zone ou en n’exposant pas ce serveur DNS à la DMZ). Ou plus simplement, avoir des DNS séparés physiquement : le DNS du LAN connaît les noms du LAN, celui de la DMZ non. Ainsi, si un serveur DMZ est compromis, il ne pourra pas facilement découvrir ni contacter par nom les machines internes. On parle parfois de  **DNS split-horizon**  pour la sécurité : présenter une vue DNS différente selon l’origine de la requête. Technitium DNS peut gérer ce genre de choses en créant des  **zones avec portée**  (views) ou en combinant des zones locales et des forwarders conditionnels. Pi-hole et AdGuard ne gèrent pas des “views” multiples, mais on peut en déployer un par zone de sécurité pour obtenir un effet similaire.

  

Enfin,  _isoler par DNS_  peut signifier  **bloquer toute résolution externe non autorisée**  : un principe de base est de forcer les clients à n’utiliser  _que le DNS filtrant_. Sur OPNsense/pfSense, on implémente une règle NAT qui intercepte toute requête DNS UDP/53 sortante et la redirige vers le serveur DNS local (Pi-hole) – on s’assure ainsi qu’aucun appareil ne tente de bypasser le filtre en utilisant 8.8.8.8. Cette technique combinée avec un Pi-hole/AdGuard renforce énormément le contrôle : même un appareil malveillant configuré avec un DNS dur en dur sera obligé de passer par le filtre.  **Attention**toutefois aux DNS over HTTPS des applications : ceux-ci ne passant pas par le port 53, il faut les bloquer au firewall (ex: bloquer les requêtes vers les domaines connus de DoH publics) ou mettre en place votre propre DoH interceptant comme évoqué plus haut.

  

### **DNS « split » pour lab interne (DNS interne vs externe)**

  

Le concept de  **split-horizon DNS**  s’applique particulièrement dans un homelab où l’on peut avoir des services accessibles à la fois depuis l’intérieur et l’extérieur. Supposons que vous hébergez un site web  _monlab.fr_  qui a une IP publique (pour y accéder depuis Internet) et une IP privée (accès depuis le LAN sans sortir). Vous souhaitez que les machines internes utilisent l’IP privée quand elles résolvent  _monlab.fr_, tandis que le public doit voir l’IP publique. Ceci peut être réalisé avec nos serveurs DNS :

-   Avec  **Pi-hole/AdGuard**, la méthode courante est d’ajouter une entrée DNS locale pour  _monlab.fr_  pointant vers l’IP privée. Pi-hole offre l’ajout d’un enregistrement A manuel dans “DNS Local” et AdGuard via les  _custom rules_  . Ainsi, quand un client du LAN demande  _monlab.fr_, le serveur DNS local lui donne directement l’IP privée. Pour les clients externes (qui n’interrogent pas votre DNS local), ils continueront d’aller vers le DNS public et recevront l’IP publique – séparation de vue accomplie.
    
-   Avec  **Technitium**, on peut aller plus loin en définissant  _monlab.fr_  comme zone locale faisant autorité dans le DNS interne. Vous créez la zone  _monlab.fr_  dans Technitium avec vos enregistrements privés. Ensuite, pour que les requêtes externes continuent de fonctionner, Technitium peut soit ne pas être exposé à l’extérieur, soit être configuré pour forwarder ce domaine vers le DNS public comme  _forwarder conditionnel_,  **tout en ayant des overrides**  : par exemple, vous pouvez définir dans la zone locale un enregistrement spécifique qui outrepasse la réponse publique (comme pointer  _monlab.fr_  vers privé) et laisser le reste passer. Technitium excelle dans ce rôle de DNS split-horizon grâce à sa capacité de gérer à la fois de l’autoritatif et du récursif dans une seule instance  .
    

  

Ce  **DNS split**  est très utile en homelab pour les VPN aussi : quand vous êtes connecté en VPN dans votre réseau, vous voulez peut-être que  _wiki.lan_  se résolve, mais pas quand vous êtes à l’extérieur. En configurant votre Pi-hole/AdGuard interne pour résoudre *.lan ou *.home.arpa, et en le combinant avec une configuration du client VPN pour utiliser ce DNS, vous obtenez une résolution locale des noms à travers le VPN. Technitium pourrait lui être le serveur central qui répond différemment selon l’IP source (par exemple, si requête vient d’une IP du VPN, autoriser certains noms).

  

En résumé, ces scénarios montrent que  **un serveur DNS homelab ne sert pas qu’à bloquer des pubs**, mais peut être un élément central de la sécurité réseau : il contribue à la  **détection**  (en journalisant quelles adresses sont requêtées), à la  **protection**  (en bloquant ou redirigeant certaines requêtes) et à la  **flexibilité**  (en offrant des vues DNS différentes selon le contexte). Cela permet de  **simuler des environnements pro**  (DNS d’entreprise avec domaines internes, zones externes, etc.) à la maison.

  

## **Sécurité, mises à jour et résilience du service DNS**

  

### **Sécurité de l’instance DNS**

  

Du point de vue sécuritaire, il faut considérer  **la sécurité du logiciel DNS lui-même**  et son  **intégration sûre dans le réseau**. Pi-hole et AdGuard Home sont écrits dans des langages robustes (C/PHP pour l’un, Go pour l’autre) et bénéficient de mises à jour régulières en cas de vulnérabilité. Jusqu’ici, aucune faille majeure publique n’a compromis ces outils de façon catastrophique, mais il est arrivé que Pi-hole corrige des bugs de corruption de mémoire ou XSS dans l’interface. Technitium DNS est relativement récent mais écrit en .NET (langage mémoire-safe)  , ce qui limite certaines catégories de vulnérabilités.  **Le principal risque**  est l’**exposition involontaire**  de l’interface web d’administration. Il est impératif de  **la protéger**  : par défaut Pi-hole n’a pas d’HTTPS sur son UI (on peut le mettre derrière un proxy HTTPS si besoin)  , AdGuard Home et Technitium peuvent activer HTTPS sur l’interface plus facilement (certificat à installer)  . Dans tous les cas, il faut restreindre l’accès de l’UI aux seules IP autorisées (via firewall ou en écoutant seulement sur l’IP locale). Évitez d’exposer l’interface de gestion sur Internet. Si vous devez accéder à distance, faites-le via VPN dans votre homelab ou SSH tunnel.

  

Concernant la  **séparation des rôles**, veillez à ce que le serveur DNS n’ait pas plus de privilèges que nécessaire sur le système. Par exemple, en Docker il tourne généralement en utilisateur non-root. Sur une machine dédiée, ne pas l’exécuter en root (Pi-hole installe un utilisateur  pihole  pour FTL, etc.). Gardez également votre système à jour – surtout si c’est un Linux minimal type Raspberry Pi OS, pour bénéficier des patchs de sécurité du kernel, etc.

  

**Mises à jour automatiques :**  Par design, Pi-hole  **n’auto-met pas à jour son core**  tout seul (on utilise la commande  pihole -up  manuellement ou on met à jour le conteneur Docker). Idem pour Technitium et AdGuard – ces services ne se mettent pas à jour automatiquement comme pourraient le faire des applications cloud. Il faut donc un processus pour les maintenir (veillez aux annonces de nouvelles versions). En revanche,  **les listes de blocage**, elles, sont souvent  **mises à jour automatiquement**. Pi-hole programme un rafraîchissement hebdomadaire de Gravity (les listes) par défaut via cron. AdGuard Home, de son côté, vérifie et met à jour les filtres abonnés régulièrement (périodicité configurable, souvent quotidienne) – il a un avantage là-dessus en offrant une interface pour gérer la fréquence  . Technitium peut être configuré pour fetcher les listes à intervalle également (sinon on peut scripter un cron avec son API). Il est crucial de garder les listes à jour pour bénéficier des nouveaux domaines malveillants à bloquer. Pour le core, je vous conseille d’intégrer la vérification des updates de ces services dans votre routine (par exemple, check mensuel, ou s’abonner aux flux GitHub des projets). Sur la sécurité du code lui-même, Pi-hole a l’avantage d’une  **très large communauté**  qui audite et contribue, AdGuard Home a une équipe dédiée (société AdGuard) et Technitium est principalement maintenu par une petite équipe mais très réactive  .

  

### **Journalisation et respect de la vie privée**

  

Un serveur DNS filtrant va naturellement  **journaliser**  beaucoup d’informations : chaque requête client, l’adresse qui l’a faite, la réponse retournée, etc. Cela peut poser question dans un contexte personnel (respect de la vie privée des usagers du réseau, par exemple votre famille) ou tout simplement encombrer le stockage à long terme.

  

**Par défaut**, Pi-hole et AdGuard loggent toutes les requêtes dans une base (FTL DB pour Pi-hole, fichiers log/DB pour AdGuard). On peut voir l’historique via l’interface (requêtes des dernières 24h, 7j, etc.).  **Pi-hole propose un mode de confidentialité réglable**  : du log complet à  _aucune journalisation_, en passant par des niveaux intermédiaires où il anonymise partiellement (par ex. ne pas stocker l’adresse IP du client, ou ne pas stocker les domaines consultés)  . Cela permet si on le souhaite de réduire la granularité des logs – utile si vous hébergez Pi-hole pour un tiers et que vous ne voulez pas conserver ses requêtes. AdGuard Home a également une option “Mode furtif” (_stealth mode_) qui peut limiter ce qui est consigné et même bloquer les requêtes aux domaines typiques de télémétrie. On peut aussi purger automatiquement les journaux au bout d’un certain temps dans AdGuard. Technitium DNS, au moins dans sa version actuelle, a une interface de log un peu moins développée  , mais les données sont bien là (fichiers logs et/ou base SQL). Rien ne vous empêche d’envoyer les logs DNS vers un SIEM local (Splunk, Elastic) pour analyser le trafic – c’est même un excellent exercice en homelab Blue Team. Cependant, faites attention :  **ne pas logguer trop longtemps**  sur SD card (si vous utilisez un Pi) pour éviter son usure, et filtrer ce qui est pertinent.

  

Un aspect “vie privée” important :  **aucune de ces solutions n’envoie vos données DNS à l’éditeur**  (tout est self-hosted, local)  .  _Seul AdGuard Home utilisait par défaut les serveurs DNS d’AdGuard_  en upstream lors de l’installation (modifiable bien sûr)  . Donc par transparence, sachez qu’après installation, il vaut mieux configurer explicitement vos serveurs DNS préférés plutôt que de laisser ceux par défaut (si on veut éviter que AdGuard (la compagnie) connaisse vos requêtes). Pi-hole et Technitium n’ont pas ce genre de particularité : ils utilisent ce que vous leur indiquez (ou rien, dans le cas d’un résolveur pur).

  

### **Résilience du service DNS et optimisation**

  

La  **résilience**  englobe la haute disponibilité (disponibilité du service) et la robustesse face aux lenteurs. Comme évoqué dans la section config, disposer de plusieurs DNS redondants est la meilleure parade contre une panne franche. Mais on peut aussi améliorer la  **tolérance aux échecs**  dans la configuration même d’une instance. Par exemple, dans Pi-hole/AdGuard, configurer  _plusieurs serveurs DNS upstream_  (DNS de secours) permet qu’en cas de silence d’un des serveurs (ex: Cloudflare ne répond plus), l’autre soit essayé. AdGuard Home offre même des stratégies (parallèle, priorité, etc.) pour interroger plusieurs upstreams en même temps et prendre le plus rapide ou éliminer celui qui ne répond pas.  **Technitium DNS**  étant un résolveur complet, il va gérer la redondance au niveau du protocole DNS lui-même (plusieurs serveurs racine, etc.), mais si vous configurez des forwarders manuels, pensez aussi à en mettre plusieurs.

  

Le  **timeout DNS**  par défaut est de quelques secondes, mais vous pouvez l’ajuster si vous constatez des retards. Toutefois, attention à ne pas le rendre trop court et déclarer en échec trop vite des domaines pouvant nécessiter un peu plus de temps (DNSSEC validation par exemple).

  

Un point de résilience est la  **caching**  : heureusement, tous les trois possèdent un cache local des réponses DNS. Ainsi, si votre lien Internet tombe temporairement, les noms récemment résolus restent dans le cache pour la durée du TTL, permettant aux appareils de continuer à fonctionner pour les domaines déjà connus. Vous pouvez augmenter la taille du cache si vous avez beaucoup de clients, mais généralement les valeurs par défaut suffisent (Pi-hole FTL et AdGuard gèrent ça automatiquement). Technitium vous laissera ajuster certains paramètres DNS avancés (taille cache, etc.) si besoin.

  

Enfin, du point de vue  _cybersécurité_, on peut parler de la  **résilience face aux attaques DNS**  : un homelab pourrait être la cible d’une attaque DNS (ex: un malware interne fait des millions de requêtes pour saturer Pi-hole). Pi-hole et AdGuard ont des mécanismes basiques de rate-limiting pour éviter qu’un client unique spame trop de requêtes et ne submerge le service. Par exemple Pi-hole limite à 1000 requêtes par minute par client par défaut (configurable) – cela évite qu’un IoT bavard ne rende le DNS indisponible pour tout le monde. Sur Technitium, je ne suis pas sûr du taux par défaut, mais étant un serveur plus sophistiqué, il doit pouvoir encaisser pas mal de charges (voire on peut utiliser les fonctionnalités de Windows/.NET if it runs on Windows to mitigate abuse). Dans tous les cas, surveillez vos tableaux de bord : si vous voyez qu’un appareil fait 10 000 requêtes/jour, il y a un souci (boucle DNS, etc.) qu’il faut corriger pour ne pas impacter la qualité de service DNS.

  

En résumé, la mise en place d’un DNS filtrant dans un homelab de sécurité apporte non seulement un confort (moins de pubs) mais fait partie intégrante de la  **défense en profondeur**  du réseau domestique. Avec Pi-hole, AdGuard Home ou Technitium DNS, on dispose d’outils modulables qu’il faut configurer selon les besoins : n’hésitez pas à expérimenter différentes topologies et réglages pour trouver l’équilibre optimal entre  **sécurité**,  **performance**  et  **simplicité de gestion**  . Chaque solution a ses points forts, mais toutes visent le même objectif : donner à l’utilisateur le contrôle sur ses résolutions DNS afin de sécuriser le réseau contre une partie des menaces et nuisances en ligne. Les retours d’expérience conseillent même d’**tester plusieurs solutions en parallèle**  et d’adopter celle qui vous convient le mieux en termes d’interface et de fonctionnalités  .
