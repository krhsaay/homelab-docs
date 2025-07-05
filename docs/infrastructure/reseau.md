# **Plan de réseau Homelab : VLAN, Segmentation et Passerelles**

  

## **Introduction : Pourquoi segmenter son réseau ?**

  

Segmenter un réseau domestique en  **VLAN**  (Virtual LAN) consiste à créer des sous-réseaux logiques isolés les uns des autres, même s’ils partagent le même matériel physique (commutateurs, points d’accès, etc.). Cette isolation présente de nombreux avantages : amélioration de la  **sécurité**  (limiter qu’un appareil compromis n’affecte les autres), meilleure  **maîtrise du trafic**  avec des  **règles firewall**  plus fines, réduction du  **broadcast**  inutile, etc.  . Dans un homelab où coexistent divers équipements (serveurs de production personnelle, équipements domotiques, caméras IP, labos de cybersécurité, NAS de sauvegarde, etc.), les VLAN permettent de  **“cloisonner” chaque catégorie d’équipements**  dans son propre réseau étanche. Par exemple, les appareils IoT (domotique) souvent peu sécurisés ne pourront pas accéder à vos PC personnels sensibles  , et vos expérimentations en lab sécurité ne risqueront pas de perturber votre réseau principal.

  

En bref, un VLAN est un  **segment de réseau isolé**. Les appareils au sein d’un même VLAN communiquent librement (comme s’ils étaient sur un même switch), mais ne “voient” pas ceux des autres VLAN sans passer par un routeur/pare-feu qui fait respecter des règles d’accès  . Cette isolation par VLAN est un  **principe clé de la défense en profondeur**  en réseau domestique : on limite drastiquement les communications non nécessaires entre les différentes  **zones de confiance**du homelab.

  

## **Topologie générale et matériel requis**

  

Pour mettre en place des VLAN chez soi, il faut du matériel réseau prenant en charge 802.1Q (standard des VLAN). En pratique, il vous faudra :

-   **Un routeur/pare-feu**  central capable de gérer des interfaces VLAN et de filtrer le trafic entre elles (par exemple un PC ou appliance avec OPNsense ou pfSense). Ces systèmes supportent nativement les VLAN 802.1Q et le firewall à états, ce qui est idéal pour implémenter la segmentation  .
    
-   **Un commutateur manageable**  (switch géré) qui supporte la configuration de VLAN. C’est indispensable pour brancher plusieurs appareils en isolant leurs ports par VLAN, surtout si le routeur n’a que peu de ports physiques  . Le switch fera transiter les trames VLAN “taggées” entre le routeur et les équipements.
    
-   (Optionnel)  **Des points d’accès Wi-Fi compatibles VLAN**, si vous souhaitez avoir des SSID Wi-Fi distincts pour différentes VLAN (ex : un SSID pour le réseau principal, un pour les invités, un pour l’IoT…). De nombreux AP prosumer (Unifi, Omada, OpenWrt, etc.) permettent d’associer chaque SSID à un VLAN ID spécifique.
    

  

La topologie typique dans un homelab VLAN est la suivante :  **le routeur/pare-feu est relié en tronc (trunk) au switch principal**  via un lien Ethernet sur lequel  _tous_  les VLAN sont autorisés (trame 802.1Q taggées)  . Le routeur agit comme  **passerelle par défaut**  pour chaque VLAN (il a une interface virtuelle dans chacun, avec une IP de gateway). Le switch, lui, connecte les différents appareils et applique l’**appartenance VLAN**  sur chaque port. Les ports du switch reliés à des appareils ordinaires (PC, caméras, etc.) sont configurés en  **VLAN “untagged”**  d’un VLAN spécifique (on parle de port access non taggé) – le périphérique n’a pas conscience du VLAN, il reçoit juste du trafic de son sous-réseau  . En revanche, le port du switch relié au routeur est configuré en  **trunk taggé**  pour  _tous_  les VLAN : il transporte les trames avec leur identifiant VLAN jusqu’au routeur, qui peut ainsi router/filtrer entre VLAN  . De même, un port vers un point d’accès Wi-Fi multi-SSID sera souvent un trunk taggé transportant les VLAN des différents SSID vers l’AP.
_Exemple de topologie réseau Homelab avec VLAN multiples (réseau management, primaire, IoT, invités, etc.) et un routeur OPNsense interconnecté en trunk au switch principal._

  

Sur le schéma ci-dessus (inspiré d’un cas réel), on distingue plusieurs VLAN isolés : un VLAN  **MGMT**  pour la gestion, un VLAN  **Primary**  (réseau principal), un VLAN  **IoT**  pour les objets connectés, un VLAN  **Guests**  pour les invités, etc., chacun avec son plan d’adressage IP dédié  . Le routeur OPNsense (en haut) est la passerelle reliant ces VLAN et appliquant des règles de pare-feu entre eux. Le switch (TP-Link) a des ports assignés à chaque VLAN (soit en untagged pour des équipements spécifiques, soit en trunk vers le routeur et les points d’accès). Par exemple, un  **NAS ou serveur**peut être connecté à un port du switch configuré en VLAN “Services”, tandis qu’une  **caméra IP**  sera sur un port attribué au VLAN “CCTV”. Le lien entre le routeur et le switch transporte tous les VLAN taggés, permettant au routeur de voir chaque sous-réseau. Avec cette architecture, chaque catégorie d’équipements communique principalement avec le routeur, qui décide si telle VLAN a le droit de joindre telle autre ou d’accéder à Internet, selon des règles prédéfinies.

  

## **Configuration des VLAN sur le pare-feu (OPNsense/pfSense)**

  

La mise en place concrète des VLAN dans OPNsense/pfSense se fait en plusieurs étapes. Voici un guide pas-à-pas synthétique :

1.  **Définir les VLAN dans le routeur** : Sur OPNsense/pfSense, commencez par créer les VLAN (menu Interfaces > Other > VLANs). Pour chaque VLAN, on spécifie un identifiant numérique (VID, entre 1 et 4094) et l’interface physique parent (par ex. votre port LAN connecté au switch)  . Exemple : VLAN 10 “Prod”, VLAN 20 “Lab”, VLAN 30 “Domotique”, etc.  Veillez à choisir des IDs cohérents et uniques. Une bonne pratique est de faire correspondre le numéro du VLAN avec le troisième octet de son réseau IP (ex: VLAN 10 → 192.168.**10**.0/24)  , afin de s’y retrouver facilement.
    
2.  **Assigner les interfaces** : Une fois VLANs définis, il faut les associer à des interfaces logiques dans le pare-feu. Dans pfSense/OPNsense, on va dans  _Interfaces > Assignments_  et on ajoute chaque VLAN comme nouvelle interface. Donnez-leur un nom significatif (LAN_PROD, LAN_IoT, etc.), puis activez-les. Assignez une  **plage d’adresses IP**  à chaque interface VLAN (typiquement /24). Par exemple : VLAN 10 (Prod) pourrait avoir 192.168.10.1/24 comme IP de gateway (le routeur se positionne sur .1), VLAN 20 (Lab) en 192.168.20.1/24, etc. Chaque VLAN est ainsi un sous-réseau IP distinct avec sa passerelle  .
    
3.  **DHCP et DNS** : Configurez un serveur  **DHCP**  sur chaque interface VLAN pour distribuer automatiquement les IP aux hôtes de ce réseau. Sur OPNsense/pfSense, on peut copier la config DHCP du LAN principal puis ajuster la plage. De même, vous pouvez activer le service DNS (Unbound) pour qu’il écoute sur tous les VLAN, afin que les clients utilisent le routeur comme résolveur DNS. Attention toutefois : pensez à autoriser le trafic DNS vers le routeur sur chaque VLAN (voir règles plus bas).
    
4.  **Configuration du switch** : Du côté du commutateur manageable, créez les VLAN avec les mêmes IDs. Ensuite, attribuez chaque  **port du switch**  au VLAN approprié.
    
    -   Le port relié au routeur doit être configuré en  **trunk/taggé sur tous les VLAN**  (ou au moins sur tous ceux utilisés). Ainsi, les trames sortant du routeur avec un tag VLAN seront acceptées et propagées sur le switch  .
        
    -   Les ports reliés à des dispositifs finaux (PC, caméra, IoT, etc.) seront mis en mode  **access (non taggé)**  dans le VLAN correspondant à cet appareil. Par exemple, si un PC principal doit être dans le VLAN Prod (10), on met son port en VLAN 10 untagged (et aucun autre VLAN sur ce port). De cette manière, le switch ajoutera/retirera le tag VLAN automatiquement pour ce port, et le PC n’aura aucune configuration spéciale à connaître  .
        
    -   Si un port du switch est connecté à un point d’accès Wi-Fi multi-SSID ou à un hyperviseur accueillant plusieurs VLAN, ce port devra être configuré en  **taggé (trunk)**  pour les VLAN nécessaires (ex: taggé VLAN 10,20,30 sur le port d’un AP qui diffuse trois SSID isolés)  .
        
    -   Pour les  **ports inutilisés**  sur le switch, par mesure de sécurité, vous pouvez les désactiver ou les assigner à un VLAN “poubelle” non routé (ex: VLAN 999) pour éviter qu’un appareil non autorisé ne se connecte à votre insu  .
        
    
5.  **Règles de firewall inter-VLAN** : Par défaut, sur pfSense/OPNsense, chaque interface VLAN nouvellement créée n’a  **aucune règle autorisant le trafic entrant**, ce qui signifie qu’aucune communication  _depuis_  ce VLAN n’est permise (tout est bloqué par le policy  _default deny_). Il convient donc d’ajouter des règles pour permettre au moins l’accès à Internet depuis chaque VLAN, tout en  **bloquant l’accès aux autres VLAN**. Une méthode simple consiste à bloquer toute destination correspondant aux réseaux privés locaux (RFC1918) pour chaque VLAN, sauf exceptions nécessaires  . Concrètement, sur l’onglet firewall de chaque VLAN :
    
    -   On crée en haut une règle autorisant le trafic de ce VLAN vers l’IP de  **passerelle du routeur**  (son adresse dans ce VLAN) pour les services indispensables comme DNS ou DHCP. Par exemple, autoriser  VLAN10_net -> 192.168.10.1 (port 53)  pour permettre aux clients du VLAN 10 d’interroger le DNS du routeur  .
        
    -   En deuxième règle, on ajoute un  **Bloc**  de tout le trafic de  VLAN10_net  vers l’alias “RFC1918” (regroupant les plages privées 10.0.0.0/8, 192.168.0.0/16, etc.), afin d’empêcher ce VLAN d’atteindre  _tout réseau local interne_  autre que le sien  . C’est cette règle qui interdit l’inter-VLAN par défaut.
        
    -   En troisième position (après le bloc inter-VLAN donc), on met une règle  **Autoriser**  de  VLAN10_net  vers “*” (any) pour permettre l’accès Internet. Grâce à l’ordre, le trafic vers Internet (destinations publiques) passera, tandis que toute tentative vers IP privées sera matchée par la règle de bloc au-dessus et donc interdite  .  _NB:_  si votre pare-feu a l’option “Block private networks” activée sur l’interface WAN, il filtrera déjà l’entrée depuis internet des IP privées externes, mais ici il s’agit du trafic sortant inter-LAN.
        
    
    Avec ce schéma de règles, chaque VLAN peut accéder au WAN mais pas aux autres LAN, sauf cas particulier. Pour ces  **exceptions**, on ajoutera des règles spécifiques  _au-dessus_  du bloc RFC1918. Par ex., si l’on veut permettre au VLAN Sauvegarde de joindre le NAS sur VLAN Prod sur le port NFS, on créerait une règle autorisant  Source: VLAN_Sauvegarde_net -> Dest: NAS_IP (port 2049)  placée avant la règle de bloc général. De même, on peut vouloir qu’un VLAN “Management” ait le droit de joindre l’interface de gestion d’équipements sur d’autres VLAN – il faut alors prévoir ces autorisations explicitement, ou utiliser des groupes d’interfaces pour simplifier (pfsense permet de créer un groupe contenant toutes les interfaces internes et définir une règle une fois)  .
    
6.  **Vérifications** : Une fois la configuration appliquée, connectez des appareils dans chaque VLAN et testez : chaque VLAN doit obtenir des IP correctes via DHCP, avoir accès à Internet si prévu, et  _ne pas_  pouvoir atteindre les machines des autres VLAN (par ex., un PC en VLAN 10 ne doit pas ping un PC en VLAN 20). Testez également les cas autorisés (ping de la passerelle, résolution DNS, etc.). Ajustez les règles firewall si nécessaire en cas de service légitime bloqué (ex: autoriser le VLAN IoT à contacter un serveur domotique sur VLAN Prod, mais uniquement sur le port de l’API requis).
    

  

En résumé, le routeur/pare-feu multi-VLAN joue le rôle de  **passerelle et filtre**  entre les segments. Le switch assure que la séparation niveau 2 est respectée (pas de fuite de trames entre VLAN), et le routeur contrôle au niveau 3 ce qui est autorisé ou non entre les VLAN.

  

## **Notions avancées et meilleures pratiques**

  

Dans un homelab bien segmenté, certaines notions plus pointues peuvent entrer en jeu pour optimiser le fonctionnement ou la sécurité :

  

### **VLAN taggés vs non taggés (trunk/access)**

  

Il est crucial de bien comprendre la différence entre un port  **taggé (trunk)**  et  **non taggé (access)**  sur le switch. Un port  _untagged_  dans un VLAN signifie que le périphérique connecté n’envoie ni ne reçoit de trames VLAN marquées – le switch intègre l’appareil dans le VLAN en interne, en ajoutant/retirant le tag 802.1Q lui-même. Ce mode est destiné aux équipements “normaux” (PC, imprimante, caméra, etc.) qui ne connaissent pas les VLAN  . En revanche, un port  _tagged trunk_  transporte les trames avec leurs identifiants VLAN inchangés. Il sert pour relier deux équipements VLAN-aware : par exemple la liaison entre le routeur et le switch, ou entre deux switches, ou vers un AP multi-SSID. Ces équipements comprennent les tags et peuvent appartenir à plusieurs VLAN simultanément.  **Règle d’or** : un port trunk peut véhiculer plusieurs VLAN (taggés), un port access ne doit appartenir qu’à un seul VLAN et présente ce VLAN en “non taggé” au device connecté.

  

Dans la configuration, veillez à  **cohérer les tags**  de bout en bout : si le VLAN 20 est utilisé pour la domotique, le routeur taggue ce trafic VLAN20 sur l’interface trunk, le switch doit avoir VLAN20 activé sur le port trunk et les ports des objets domotiques en VLAN20 untagged. Une erreur de tag (ex: VLAN ID mismatch) se traduit par un appareil incapable de communiquer au-delà de son switch.

  

### **ACL et contrôle de trafic intra-VLAN ou sur switch**

  

En environnement professionnel, on utilise parfois des  **ACL (Access Control Lists)**  sur les commutateurs de niveau 3 ou même niveau 2 pour filtrer le trafic  _au sein_  du réseau, indépendamment du pare-feu central. Dans un homelab, si vous disposez d’un switch L3 avancé, vous pourriez mettre en œuvre des ACL pour bloquer certaines communications inter-VLAN directement au niveau du switch, en complément ou à la place des règles du routeur. Cependant, dans la plupart des cas, votre pare-feu OPNsense/pfSense remplit déjà ce rôle (ses règles agissent en fait comme des ACL de niveau 3/4). L’utilisation d’ACL sur le switch peut être utile si vous faites du routage local sur un switch L3 pour décharger le routeur (cependant, ceci est rarement nécessaire à l’échelle d’un homelab). Retenez qu’ACL ou firewall reviennent à définir des règles : commencez simple (tout inter-VLAN bloqué sauf exceptions), puis affinez au besoin avec des règles spécifiques ou ACL pour des cas très précis. Un adepte résume ainsi son approche maison : isoler les VLAN, puis  **“ouvrir des brèches avec des ACL pour des dispositifs spécifiques”**  selon les besoins  .

  

### **IGMP Snooping et trafic multicast**

  

Les réseaux locaux modernes, surtout avec domotique, Chromecast, caméras, etc., utilisent du  **multicast**  (mDNS, SSDP, streaming vidéo). Par défaut, une trame multicast est envoyée en broadcast sur un VLAN, ce qui peut saturer inutilement les ports. La fonction  **IGMP Snooping**  des switches veille à écouter les abonnements IGMP des appareils pour ne forwarder le trafic multicast qu’aux ports qui en ont besoin. Il est recommandé d’activer l’IGMP Snooping sur chaque VLAN concerné afin d’**éviter la diffusion de flux multicast vers tous les ports**  du VLAN  . Par exemple, sur un VLAN Caméras IP avec NVR, le switch n’enverra le flux vidéo multicast qu’au port du NVR et éventuellement aux quelques clients inscrits, plutôt qu’à tous les dispositifs du VLAN. Attention, IGMP Snooping implique qu’il y ait un  **querier IGMP**  sur le réseau (généralement le routeur multicast). OPNsense/pfSense peut faire office de querier si activé.

  

Un autre défi fréquent en environnement VLAN est la découverte mDNS/Bonjour (pour imprimantes, Chromecasts, assistants vocaux, etc.) : par design, ces protocoles ne passent pas les frontières VLAN. Deux approches : soit regrouper sur un même VLAN les appareils qui doivent se découvrir (solution simple mais reniant l’isolation), soit utiliser un  **mécanisme de relay/reflecteur mDNS**  (par ex. le service Avahi) qui répète certaines annonces entre VLAN choisis. OPNsense inclut un plugin “os-avahi” qui peut être configuré pour refléter mDNS entre un VLAN IoT et le VLAN principal, afin que vos Chromecast (IoT) apparaissent sur votre smartphone (LAN principal) sans ouvrir tout le trafic entre ces VLAN. Cette configuration avancée améliore le confort d’usage tout en maintenant l’isolation du reste des communications.

  

### **Pare-feu inter-VLAN vs intra-VLAN**

  

Dans notre scénario, on considère généralement que les appareils  **au sein d’un même VLAN**  sont de confiance similaire, donc on ne filtre pas leur communication mutuelle (ils peuvent dialoguer librement dans le VLAN). Si ce n’est pas souhaité (ex: VLAN Invités où même les clients ne doivent pas se voir), on parle d’**isolation client**  au niveau 2 : certaines bornes Wi-Fi permettent d’isoler les clients entre eux, ou on peut utiliser des fonctionnalités de switch (PVLAN, port isolation) pour empêcher des ports du même VLAN de communiquer directement. C’est utile surtout pour un VLAN invité ou public. Dans la plupart des VLAN privés (Prod, IoT…), laisser les appareils se voir peut être nécessaire (ex: votre smartphone pilote une ampoule sur VLAN IoT – ils doivent pouvoir communiquer  _si_  dans le même VLAN IoT). Retenez donc : on cloisonne fortement  _entre_  VLAN différents (firewall L3), et on peut éventuellement cloisonner  _à l’intérieur_  d’un même VLAN (isolation L2) selon les cas d’usage spécifiques.

  

## **Segmentation par zone : VLAN proposés pour le homelab**

  

Passons en revue les différentes  **zones VLAN**  qu’on peut mettre en place dans un homelab bien organisé, en fonction des besoins évoqués (production personnelle, sécurité, domotique, stockage, etc.). Chaque VLAN correspond à un  **cas d’usage**  et aura ses propres règles de trafic.

  

### **VLAN**

### **Production**

### **(LAN principal personnel)**

  

C’est le réseau principal pour vos équipements  **de confiance**  et critiques (PC personnels, laptop de travail – sauf si politique entreprise l’interdit, serveurs de prod auto-hébergés, etc.). Il peut s’agir du VLAN  **10**  par exemple, 192.168.10.0/24. Ce réseau a généralement  **un accès complet à Internet**  et peut, si nécessaire, initier des connexions vers certains services dans d’autres VLAN (ex: accéder au NAS de sauvegarde, consulter les flux des caméras, etc.). En revanche, on bloque l’accès  _entrant_  depuis les autres VLAN vers ce réseau pour le protéger. Vous traiterez ce VLAN un peu comme un réseau “interne sécurisé d’entreprise”. Vous pouvez aussi y segmenter en deux VLAN distincts si vous hébergez des applications client/serveur sensibles : par exemple VLAN “Prod-Serveurs” séparé du VLAN “Prod-Clients”. Mais pour un homelab, un seul VLAN prod suffit souvent, d’autant qu’il est plus aisé de gérer quelques règles ponctuelles à l’intérieur que de multiplier les segments. Veillez simplement à  **ne pas connecter d’appareils peu fiables**  (IoT, etc.) sur ce réseau.

  

### **VLAN**

### **Lab Sécurité**

### **(Red Team / Blue Team)**

  

Pour vos besoins de labo cyber (pentest, forensic…), il est conseillé de créer un VLAN dédié  **hautement isolé**  du reste. Par exemple VLAN  **40**  = 192.168.40.0/24. Vous pouvez y faire tourner des VMs vulnérables, des attaques simulées, etc., sans risque pour votre LAN principal. Ce VLAN Lab peut même être subdivisé en plusieurs segments selon vos scénarios : un VLAN  **“Red”**  pour les machines d’attaque (attaquant) et un VLAN  **“Blue”**  pour les cibles et la surveillance (défenseur). Dans ce cas, vous pourriez introduire un  **pare-feu intermédiaire**  entre Red et Blue pour reproduire un environnement d’entreprise (ou utiliser OPNsense lui-même avec des règles spécifiques entre ces deux VLAN). Toutefois, si c’est un lab local purement pour exercice, un seul VLAN isolé peut suffire, ou deux VLAN isolés de tout sauf d’une connexion contrôlée entre eux. Par défaut, le VLAN Lab n’a  **pas accès aux VLAN prod/domotique/etc.**, et potentiellement même  **pas d’accès Internet**  si vous voulez tester des malwares en contenant (vous pourrez activer l’accès WAN ponctuellement via une règle ou via un proxy/VPN dédié pour le lab). L’idée est de minimiser les ponts avec l’extérieur. Vous pourriez autoriser seulement la machine de l’analyste (sur VLAN Prod par ex.) à se connecter en RDP/SSH vers le VLAN Lab Blue Team pour y faire de l’investigation, mais rien de plus. Ce VLAN est votre bac à sable dangereux : traitez-le comme une  **DMZ locale**  très verrouillée.

  

### **VLAN**

### **Domotique/IoT**

  

Tous les équipements  **domotiques, IoT et multimédia**  peu sécurisés vont dans ce réseau isolé. Par exemple VLAN  **30**  = 192.168.30.0/24. On y met les ampoules connectées, assistants vocaux, télévisions intelligentes, prises Wi-Fi, thermostats, etc. Ce sont typiquement des appareils qui  **requièrent un accès Internet**  (pour leurs services cloud) mais n’ont aucune raison de communiquer avec vos PC/serveurs personnels. On configure donc ce VLAN IoT de sorte que ses appareils puissent sortir vers Internet (HTTP, MQTT… selon les besoins), mais  **ne puissent pas initier de connexion vers les VLAN sensibles**  (Prod, Sauvegarde…). Inversement, on peut autoriser quelques flux entrants maîtrisés : par exemple, votre smartphone (VLAN Prod) doit pouvoir envoyer des commandes vers l’API locale d’une ampoule sur VLAN IoT. Une façon propre de faire cela est de déployer un  **serveur domotique central (type Home Assistant)**  qui a deux interfaces réseau – une dans le VLAN IoT pour parler aux objets, et une dans le VLAN Prod pour que vous puissiez le contrôler. Si ce n’est pas possible, on ouvrira finement sur le firewall les accès nécessaires (ex: autoriser IP_du_PC → IP_ampoule port TCP spécifique). Gardez ce VLAN IoT aussi fermé que possible, car ces appareils sont souvent les plus vulnérables. Il est d’usage également de  **refuser toute connexion sortante inutile**  depuis IoT (filtrage par liste blanche de domaines via DNS ou règles par IP si possible) afin d’éviter qu’un objet bavard n’envoie des données à des tiers. En résumé, VLAN IoT = Internet oui, accès aux autres VLAN non (sauf exceptions minimales), accès entrants très limités.

  

Notez enfin que les  **protocoles multicast**  type mDNS, SSDP sont massivement utilisés en IoT (pour la découverte plug-and-play). Comme expliqué, ceux-ci ne traversent pas les VLAN, donc si par exemple votre Google Home (VLAN IoT) doit être contrôlé par votre téléphone (VLAN Prod), vous aurez besoin d’un mécanisme comme Avahi (reflecteur mDNS) ou, plus simplement, placer le téléphone sur le VLAN IoT le temps de l’utilisation (moins sécurisé). À vous de voir le bon compromis entre  **confort**  et  **cloisonnement**.

  

### **VLAN**

### **Caméras IP**

### **(Surveillance CCTV)**

  

Les caméras de surveillance réseau méritent souvent leur propre VLAN (par ex. VLAN  **50**  = 192.168.50.0/24), bien qu’on pourrait les mettre dans l’IoT. Elles génèrent du trafic vidéo continu et peuvent présenter des vulnérabilités sérieuses. Dans un homelab, on crée un VLAN Caméras isolé  **sans accès Internet direct**  (sauf si la caméra a absolument besoin de NTP ou de mises à jour par le web, mais idéalement on évite les caméras cloud). Ce VLAN ne doit communiquer qu’avec votre  **NVR**/serveur d’enregistrement vidéo. Par exemple, autoriser les flux  _de_  caméras → vers NVR sur les ports RTSP, et peut-être le NVR → caméras sur HTTP (pour les configurer). En dehors de cela, pas de passage. Ainsi, même si une caméra est compromise, l’attaquant ne pourra pas rebondir sur le reste de votre réseau. Un retour d’expérience de homelab recommande que les caméras n’aient  _aucune_  idée d’Internet :  _«CCTV – no access to outside world, cameras get time from internal NTP, and can send flux au NVR uniquement»_  . C’est une sage précaution. Si vous devez accéder à vos caméras depuis l’extérieur, faites-le via le NVR (qui serait lui-même soit dans un VLAN DMZ, soit via VPN). Enfin, côté performance, les caméras et NVR peuvent consommer de la bande passante en continu : assurez-vous que votre switch supporte le débit cumulé, et envisagez éventuellement un  **switch PoE dédié**  pour alimenter les caméras, raccordé en trunk sur le VLAN caméras du switch principal.

  

### **VLAN**

### **Stockage (iSCSI)**

  

Pour les besoins de stockage partagé (par exemple exposer un disque iSCSI depuis un NAS vers vos hyperviseurs Proxmox/ESXi), il est fortement recommandé de dédier un VLAN spécifique au  **trafic de stockage**. On peut le nommer VLAN  **iSCSI**  ou  **SAN**. Souvent, ce VLAN n’a même  _pas_  de passerelle vers le routeur (pas routé) : il s’agit juste d’un réseau L2 isolé entre votre NAS/stockage et vos serveurs, pour le traffic iSCSI ou NFS. Par exemple VLAN  **20**  = 10.0.20.0/24, sans sortie internet. On attribue des IP fixes aux interfaces SAN de chaque hôte et du NAS. Ce réseau étant isolé, on évite toute pollution par d’autres flux, ce qui améliore les performances et la sécurité (un client du VLAN Prod n’a aucune raison d’accéder directement au VLAN iSCSI). En pratique, il faut que vos serveurs et NAS aient chacun  **deux interfaces réseau**  (ou une interface VLAN taggée côté hyperviseur) : l’une pour le LAN normal, l’autre pour le VLAN stockage. Les échanges iSCSI se font ainsi directement, sans passer par le routeur.

  

Si toutefois vous avez besoin que ce VLAN iSCSI soit routé (par ex. pour accéder à l’interface d’admin du NAS depuis un PC), vous pouvez lui attribuer une interface sur OPNsense, mais dans ce cas filtrez rigoureusement l’accès (ex: seul le PC admin peut joindre l’IP de management du NAS sur ce VLAN). Beaucoup choisissent de ne  _pas_  router le VLAN SAN du tout, forçant l’accès administrateur via un jump host sur ce VLAN ou via le LAN principal du NAS. Quoiqu’il en soit, on  **n’autorise pas Internet**  sur ce VLAN, il doit être totalement interne. Pensez également à activer les  **Jumbo Frames (MTU 9000)**  sur ce VLAN si tous vos équipements le supportent, afin d’optimiser le débit de gros flux iSCSI/NFS (toutes les interfaces du NAS, serveurs et switch impliqués doivent avoir MTU 9000). Les VLAN de stockage peuvent aussi inclure les trafics de virtualisation type vMotion, replication, etc., qui bénéficient d’un réseau à part.

  

### **VLAN**

### **Sauvegardes**

### **(Backup)**

  

Proche du VLAN stockage, vous pourriez déployer un VLAN distinct pour la sauvegarde de vos données. Par exemple si vous avez un serveur de backup ou un NAS où les différentes machines viennent déposer leurs sauvegardes, il peut être judicieux de l’isoler. Cependant, deux approches existent :

-   **Approche 1** : Le serveur de sauvegarde (NAS, etc.) réside  _uniquement_  dans ce VLAN Sauvegarde, et on autorise chaque VLAN source à envoyer des données vers lui (sur les ports de backup) via le pare-feu. Par exemple, VLAN Prod peut joindre l’adresse du NAS (VLAN Backup) sur le port 873 (rsync) ou 443 (agent de backup), etc. Cela maintient la séparation, mais le trafic passe par le routeur et subit éventuellement son goulot d’étranglement.
    
-   **Approche 2** : Le serveur de sauvegarde a  **une interface dans chaque VLAN**  qui le concerne (ou au moins dans VLAN Prod et VLAN Backup). C’est plus complexe et augmente la surface, donc on préfère la première solution généralement.
    

  

Un VLAN Backup dédié prend son sens si les sauvegardes représentent beaucoup de trafic et qu’on peut le séparer physiquement/logiquement. Si vos sauvegardes se font la nuit, saturer le LAN principal n’est pas un gros problème, donc un VLAN séparé n’apporte pas énormément en performance (sauf à avoir des liens multiples). En sécurité en revanche, isoler le serveur de sauvegarde sur un VLAN restreint est malin : en cas de compromission d’une machine du LAN Prod (rançongiciel par ex), il lui sera plus difficile d’atteindre le NAS de sauvegarde s’il est sur un autre VLAN avec un accès uniquement unidirectionnel.  **Recommandation** : placez votre NAS/serveur de backup dans un VLAN séparé (par ex. VLAN 60 = 192.168.60.0/24), n’autorisez que le minimum (les machines sources peuvent envoyer des sauvegardes vers lui, mais lui ne peut initier de connexion vers elles), et n’autorisez pas ce VLAN vers Internet, sauf éventuellement pour envoyer des alertes mail de backup. Ainsi, vos backups sont conservés dans une “zone” distincte, réduisant les risques en cas de malware sur le LAN principal.

  

### **VLAN**

### **Management**

### **(OOB, admin)**

  

Un VLAN souvent oublié dans les petits réseaux, mais très utile à partir d’un certain niveau de complexité, est le VLAN  **Administration**. Idée : regrouper toutes les  **interfaces de gestion**  de vos équipements critiques (routeur, switch, NAS, hyperviseur, IPMI des serveurs…) sur un réseau à part, accessible uniquement aux administrateurs. Par exemple VLAN  **99**  = 192.168.99.0/24. Vous configurez chaque équipement pour qu’il  **n’écoute son interface d’administration web/SSH que sur l’IP de ce VLAN Management**. Ainsi, même si un utilisateur malveillant se trouve sur le VLAN Prod, il ne pourra pas tenter d’accéder à l’interface du switch si celle-ci est sur VLAN 99 uniquement (et qu’il n’a pas accès à VLAN 99)  . C’est une mesure de sécurité  **supplémentaire** : même en environnement domestique, il est intéressant de limiter l’accès aux consoles d’admin. Concrètement, il faut que  _vous_  disposiez d’un moyen d’entrer dans ce VLAN management quand besoin (par ex. votre PC admin a un port physique ou un SSID Wi-Fi dédié à ce VLAN, ou vous basculez un VLAN taggé sur votre PC quand nécessaire). Ce n’est pas toujours le plus pratique au quotidien, mais c’est proche des pratiques professionnelles. Vous pouvez simplifier en combinant le VLAN Management avec votre VLAN Prod si vous êtes seul à utiliser le réseau, mais si vous êtes plusieurs utilisateurs non techniques chez vous, mieux vaut isoler la gestion.

  

Dans la config proposée plus haut (image du réseau), le VLAN “LAN” était utilisé comme réseau de management dédié  . On y plaçait le switch, les AP, etc., tandis que les utilisateurs normaux étaient sur d’autres VLAN par défaut  . Cette approche se traduit par  _0 service d’admin_  accessible depuis les VLAN utilisateurs, ce qui réduit fortement les possibilités d’attaque interne sur vos équipements. En homelab, VLAN 99 Management peut ne pas être indispensable, mais il est bon de l’envisager pour “future-proof” votre design.

  

### **VLAN**

### **DMZ**

### **(zone démilitarisée)**

  

La DMZ est un VLAN particulier destiné aux  **services publics/exposés sur Internet**. Si vous hébergez, par exemple, un serveur web accessible depuis l’extérieur, un VPN entrant, ou un reverse proxy frontal pour vos services, il est sain de le placer en DMZ. Ce VLAN DMZ (ex: VLAN 70 = 192.168.70.0/24) est isolé du LAN : seul le routeur y accède. On configure des règles NAT/Port forwarding pour diriger le trafic entrant WAN → vers l’IP du serveur en DMZ, et on autorise le minimum de ce VLAN vers l’intérieur.  **Idéalement, la DMZ n’initie aucune connexion vers vos LAN internes**  (sauf peut-être requêter une base de données sur un VLAN spécifique, mais dans homelab, on évite d’exposer une base interne). Un exemple concret : vous hébergez un  **reverse proxy Nginx**  pour accéder à vos services maison (domotic, etc.) depuis Internet. Placez ce reverse proxy en DMZ. On ouvre depuis Internet seulement le port 443 vers lui. Dans le firewall, on peut autoriser ce reverse proxy DMZ à contacter, disons, le serveur domotique en VLAN IoT sur le port requis, et c’est tout. Ainsi, si le proxy est compromis via le web, l’attaquant est confiné en DMZ. Certains adoptent même une  **double barrière**  en DMZ : le serveur DMZ est derrière le pare-feu principal ET possède son propre pare-feu applicatif. Dans un homelab, ceinture et bretelles sont peut-être excessives, mais l’idée est de ne jamais  _faire transiter du trafic Internet directement dans vos VLAN sensibles_. La DMZ agit comme tampon.

  

Un utilisateur homelab témoigne par exemple :  _«DMZ – le seul VLAN acceptant des connexions externes (443), il héberge le reverse proxy externe et possède un double firewall»_  . Cela illustre bien que même en environnement perso, on peut traiter la DMZ sérieusement. En pratique, pour configurer : créez le VLAN, attribuez-le à une interface OPNsense (ex: OPT2), donnez-lui une plage IP. Reliez-y votre serveur public. Sur OPNsense, activez les règles NAT voulues vers ce VLAN, et surtout  **blocquez toute sortie de la DMZ vers vos LAN**  par défaut. Autorisez uniquement ce qui est nécessaire (peut-être la DMZ peut accéder à Internet pour updates, ou joindre un DNS local). Un conseil : ne réutilisez pas vos machines de LAN en DMZ ; par exemple, évitez qu’un conteneur sur votre NAS principal soit exposé : mieux vaut une VM dédiée sur un hôte en DMZ. Cela évite qu’une compromission DMZ n’affecte vos données.

  

### **VLAN**

### **Invités**

### **(Guest Wi-Fi)**

  

Enfin, souvent utile dans un foyer, un VLAN  **Invités**  pour le Wi-Fi des visiteurs ou appareils temporaires. Par ex VLAN  **80**  = 192.168.80.0/24. Son objectif : fournir un accès Internet basique  **sans aucun accès à vos ressources internes**. Ici il faut généralement activer l’isolement client sur l’AP Wi-Fi (pour que les invités ne se voient pas entre eux), et sur le pare-feu on bloque toute destination locale. En gros, c’est très proche du VLAN IoT en termes de règles, mais on peut être encore plus restrictif (pas d’accès sortant vers des ports sensibles, éventuellement limiter la bande passante via QoS). Le VLAN Invités est un bon moyen de partager votre connexion sans compromettre le reste. Avec un portail captif ou un mot de passe distinct, vous pouvez contrôler l’utilisation. Techniquement, il suffit de créer ce VLAN invité, de le lier à un SSID dédié sur votre point d’accès, et de vérifier que les règles firewall interdisent tout accès aux autres VLAN (ce qui devrait déjà être le cas si on bloque RFC1918 en sortie).

  

Pour résumer, isoler les invités vous protège de tout poste non maîtrisé (par exemple l’ordinateur vérolé d’un ami de passage ne pourra pas scanner/transmettre quoi que ce soit à vos machines). C’est une VLAN à considérer même si non mentionné initialement, puisque  **anticiper les éventualités**  est l’objectif : on ne sait jamais quand quelqu’un vous demandera le Wi-Fi – autant que ce soit un réseau distinct bridé.

  

## **Conclusion : Vers un homelab segmenté, sécurisé et évolutif**

  

En implémentant une segmentation par VLAN dans votre homelab, vous posez les bases d’une architecture robuste où chaque catégorie de dispositifs évolue dans son  **environnement contrôlé**. La configuration peut sembler complexe au début (notamment la coordination entre routeur, switch et points d’accès), mais le gain en sécurité et en maîtrise du réseau est considérable pour un administrateur domestique. Vous avez désormais un  **plan de réseau détaillé**  distinguant les zones Prod, Lab, Domotique, Caméras, Stockage, Sauvegarde, etc., avec des  **passerelles (routeur)**  gérant les communications entre ces zones de manière filtrée.

  

N’hésitez pas à  **documenter votre topologie**  (schémas, tableaux d’adressage VLAN) et à commenter vos règles firewall pour vous y retrouver. À mesure que votre homelab grandit, cette segmentation vous facilitera l’intégration de nouveaux services sans tout remettre en question : par exemple, ajouter un serveur PBX VoIP ? Créez un VLAN VoIP. Tester un cluster Kubernetes ? Peut-être un VLAN “Dev” séparé. Votre réseau est déjà prêt à cloisonner ces ajouts.

  

En appliquant les bonnes pratiques (principe du moindre privilège entre VLAN, isolation du management, utilisation d’ACL avancées si besoin, etc.), vous rapprochez votre homelab d’un  **réseau d’entreprise miniature**  en termes de sérieux et de sécurité. Enfin, vérifiez régulièrement vos règles et appareils connectés – la  **discipline de segmentation**nécessite de s’assurer qu’aucun appareil ne se retrouve par erreur dans le mauvais VLAN ou avec des droits inappropriés. Avec cette architecture VLAN bien pensée, votre homelab pourra héberger en toute confiance vos projets personnels, vos exercices de sécurité et vos gadgets connectés, le tout  **sans interférences ni risques excessifs**  entre eux.

