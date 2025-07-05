
# **Proxmox dans un homelab sécurisé et auto-hébergé**

  

Proxmox VE (Virtual Environment) est une plateforme de virtualisation open-source de type  **bare metal**. Elle permet d’exécuter des  **machines virtuelles (VM)**  et des  **conteneurs Linux (LXC)**  sur un ou plusieurs serveurs, le tout gérable via une interface web centralisée. Proxmox est basée sur Debian Linux et intègre l’hyperviseur KVM pour la virtualisation complète, ainsi que LXC pour la virtualisation légère de conteneurs  . Très utilisée en entreprise comme en homelab, Proxmox offre un riche ensemble de fonctionnalités prêtes à l’emploi pour créer un environnement auto-hébergé à la fois flexible et sécurisé.

  

_(Suggestion de capture d’écran : Interface web de Proxmox VE affichant le tableau de bord des VMs et conteneurs.)_

  

## **Introduction à Proxmox VE**

  

**Proxmox VE (Virtual Environment)**  est une solution libre (licence AGPLv3) permettant de transformer un serveur x86_64 en un hyperviseur complet. Installable directement sur le matériel (type 1), Proxmox utilise  **KVM**  pour créer des machines virtuelles isolées (compatibles Windows, Linux, *BSD, etc.) et  **LXC**  pour créer des conteneurs Linux légers partageant le noyau de l’hôte  . L’administration se fait via une interface web intuitive, sans nécessiter de ligne de commande pour les tâches courantes. Un seul serveur Proxmox peut héberger de nombreuses VM et conteneurs, optimisant ainsi l’utilisation des ressources de votre machine.

  

En contexte  _homelab_  (laboratoire personnel hébergé chez soi), Proxmox brille par sa  **polyvalence**. Il permet de consolider sur une même machine plusieurs services auto-hébergés tout en assurant leur isolation. Par exemple, on pourra faire tourner simultanément un serveur domotique, un serveur web, un NAS virtuel, etc., chacun dans une VM ou un conteneur séparé. Proxmox offre aussi des outils intégrés de sauvegarde, de snapshot et de haute disponibilité, ce qui en fait un choix de premier plan pour un homelab fiable et  _secure by design_.

  

_(Suggestion de capture d’écran : Écran de connexion à l’interface Proxmox VE avec option 2FA.)_

  

## **Cas d’usage pour un homelab**

  

Quels usages concrets peut-on attendre de Proxmox dans un homelab sécurisé ? En voici les principaux cas d’utilisation :

-   **Machines virtuelles (VM)** : Hébergez plusieurs systèmes d’exploitation en parallèle (Linux, Windows, etc.) pour segmenter vos services. Par exemple, une VM Debian pour un serveur web et une VM Windows pour un logiciel spécifique. Chaque VM dispose de ses ressources dédiées (CPU, RAM, disque) et est isolée des autres, ce qui renforce la sécurité.
    
-   **Conteneurs LXC** : Déployez des conteneurs Linux légers pour vos services moins gourmands. Les conteneurs partagent le noyau de l’hôte, consomment peu de ressources et démarrent en quelques secondes. Idéal pour héberger un Pi-hole, un serveur DNS, un petit service web ou une base de données  _lightweight_  sous Alpine Linux, etc.
    
-   **Stockage centralisé** : Proxmox peut gérer différentes solutions de stockage pour vos données. Vous pouvez utiliser du  **stockage local**  sur le serveur (disques SSD NVMe, HDD, RAID, ZFS…), monter un stockage réseau de type  **NAS**  (partage NFS/SMB) ou même déployer du  **stockage distribué**  avec Ceph. Cela permet d’adapter les performances de stockage selon les besoins : par exemple, du SSD pour les VM critiques et du disque dur mécanique pour l’archivage.
    
-   **Réseau virtualisé** : Proxmox facilite la création de réseaux virtuels isolés ou bridgés vers votre LAN. Vous pouvez définir des  **bridges**  pour connecter des VM à votre réseau local, des  **VLAN**  pour segmenter le trafic (par exemple séparer une DMZ des services internes), ou utiliser le module  **SDN**  (Software Defined Network) de Proxmox pour des configurations réseau avancées. Tout cela se configure facilement depuis l’UI, permettant de reproduire une topologie réseau complexe à des fins de test ou de sécurité.
    
-   **Snapshots et sauvegardes** : Avant une mise à jour risquée ou une manipulation, prenez un  **snapshot**  de la VM/LXC afin de pouvoir revenir en arrière instantanément en cas de problème. Proxmox gère les snapshots (sur systèmes de fichiers compatibles comme ZFS) et propose une planification de  **sauvegardes**  automatisées de vos VM/CT vers un stockage de votre choix (NAS, disque USB, Proxmox Backup Server, etc.). Ces sauvegardes peuvent être incrémentales et compressées pour gagner de l’espace. En homelab, cela permet de tester sans crainte de “casser” son environnement, et d’avoir des backups prêts en cas de panne matérielle.
    

  

_(Suggestion de capture d’écran : Vue d’une VM en cours d’exécution dans Proxmox, avec ses ressources CPU/RAM affichées.)_

  

## **Fonctionnalités clés de Proxmox VE**

  

Proxmox VE apporte par défaut un  **large éventail de fonctionnalités**  appréciables dans un homelab, alliant simplicité d’utilisation et options avancées :

-   **Gestion centralisée via Web UI** : L’interface web de Proxmox (accessible sur le port 8006) offre un tableau de bord unifié pour gérer toutes vos VM et conteneurs, même sur plusieurs nœuds. On peut créer, configurer, démarrer/arrêter les VMs, consulter les métriques, le tout depuis un navigateur  . L’UI intègre également un  _shell_Web pour accéder à la console des VM/CT, et supporte plusieurs utilisateurs avec rôles si nécessaire.
    
-   **Clustering et Haute Disponibilité (HA)** : Proxmox permet de former un  **cluster**  en regroupant plusieurs serveurs physiques. Le cluster offre une gestion commune via l’UI (vue datacenter) et permet des fonctionnalités avancées comme la  **migration à chaud**  des VM d’un nœud à l’autre sans interruption  . En configurant la HA, une VM critique pourra redémarrer automatiquement sur un autre nœud en cas de panne du serveur hôte.  _(Pour tirer parti de la HA, un stockage partagé ou une réplication Ceph/ZFS est généralement requis.)_
    
-   **Support natif de ZFS et stockage avancé** : Proxmox VE s’intègre nativement avec le système de fichiers  **ZFS**, dès l’installation vous pouvez choisir ZFS pour le stockage principal. ZFS apporte  **compression transparente, snapshots rapides et réplication**. Proxmox facilite la gestion des volumes ZFS via l’interface  . En mode cluster, Proxmox propose également une intégration directe de  **Ceph**  depuis l’UI pour déployer un stockage distribué et redondant  . En plus de ZFS/Ceph, Proxmox gère de multiples types de stockage : LVM, répertoires ext4, partages  **NFS**, cibles  **iSCSI**, GlusterFS, etc.  .
    
-   **Snapshots et sauvegardes intégrés** : L’hyperviseur inclut des outils intégrés pour  **snapshoter**  un système (s’il est sur un stockage compatible, ex. ZFS ou LVM-Thin) et pour  **sauvegarder**  les VM/CT. Les sauvegardes peuvent être planifiées (mode  _cron_) et envoyées vers un stockage local ou distant. Proxmox prend en charge la suspension du système invité (_freeze_) lors des sauvegardes pour assurer la cohérence. La restauration d’une VM depuis une sauvegarde se fait en quelques clics, améliorant nettement la résilience de votre homelab.
    
-   **Fonctions réseau et pare-feu** : Proxmox VE supporte les  **bridges Linux**  classiques mais aussi  **Open vSwitch**  pour les cas complexes, et offre un module  **SDN**  depuis la version 7+ pour définir des réseaux virtuels plus élaborés. Chaque interface réseau de VM/CT peut être attachée à un bridge ou VLAN tagué. De plus, Proxmox embarque un  **firewall**  configurable par VM, par hôte et au niveau datacenter. On peut ainsi appliquer des règles (au niveau IP, port, protocoles) pour segmenter le trafic entre VMs ou vers l’extérieur, directement depuis l’interface de Proxmox.
    
-   **Open Source & Communauté** : Proxmox VE est entièrement open-source et gratuit à utiliser. Le code source est public, et une  **vaste communauté**  d’utilisateurs contribue via le forum officiel, des wikis et des tutoriels. Des mises à jour régulières améliorent sans cesse la plateforme. Pour les entreprises ou utilisateurs exigeants, Proxmox propose un abonnement  **support entreprise**  (payant) donnant accès à un dépôt stable et une assistance technique  – optionnel pour un homelab, car la version gratuite est pleinement fonctionnelle (seules les MAJ se font via le dépôt communautaire).
    

  

_(Suggestion de capture d’écran : Onglet de configuration du stockage ZFS dans Proxmox, montrant un pool ZFS et ses volumes.)_

  

## **Comparaison avec d’autres solutions open source**

  

Plusieurs alternatives open source à Proxmox existent pour la virtualisation en homelab, chacune avec ses spécificités. Ci-dessous un tableau comparatif de Proxmox VE avec trois solutions populaires :  **XCP-ng**,  **TrueNAS SCALE**  et  **Harvester**.

|**Solution**|**Technologie** **(Hyperviseur)**|**Support Conteneurs**| **Clustering / HA** | **Gestion du Stockage** | **Interface** | **Orientation** |
|--|--|--|--|--|--|--|
|**Proxmox VE**| Type 1 Linux (Debian + KVM/QEMU) + LXC | **Oui**  (LXC natif pour Linux) | **Optionnel**  (cluster multi-nœuds, HA possible si stockage partagé ou Ceph) | Local (dir, LVM,  **ZFS**), externe (NFS, iSCSI), distribué (**Ceph**  intégré) | Interface web unifiée (Proxmox GUI) | Hyperviseur polyvalent (VM + conteneurs sur une infra unifiée) |
|**XCP-ng**| Type 1 Xen (basé sur XenServer) | **Non**  (pas de conteneurs natifs) | **Oui**  (pool de nœuds Xen, HA et migration via Xen Orchestra) | Local (LVM/ext), externe (NFS, iSCSI), pas de ZFS natif intégré | **Xen Orchestra**  (web, à déployer séparément) ou outil Windows (XCP-ng Center) | Hyperviseur pur pour VM (fork libre de Citrix XenServer) |
|**TrueNAS SCALE**| OS hyperconvergé (Debian + KVM + Kubernetes) | **Oui**  (Apps Docker via Kubernetes) | **Limité**  (fonction  _scale-out_  en cours, clustering GlusterFS possible pour le stockage) | Local uniquement (**ZFS**  natif pour NAS, RAID-Z), partage réseau (SMB, NFS, iSCSI intégrés) | Interface web complète (NAS + VMs + Apps Docker) |**NAS orienté services**  (stockage centralisé avec possibilité de VMs/containers supplémentaires)  |
|**Harvester HCI**| Type 1 hyperconvergé (SUSE Linux + KVM via KubeVirt) | **Oui**  (conteneurs via Kubernetes sous-jacent) | **Oui**  (conçu pour cluster HCI ≥3 nœuds, HA natif) | Stockage distribué intégré (**Longhorn**  – nécessite disques SSD sur chaque nœud) | Interface web (intégrée à Rancher) | Infra  **hyperconvergée cloud-native**  (VM orchestrées par Kubernetes, approches type VMware vSphere) |


**Remarques :**  Proxmox et XCP-ng sont des hyperviseurs “classiques” similaires à VMware ESXi ou Hyper-V, installés sur bare metal pour héberger principalement des VMs.  **XCP-ng**  utilise l’hyperviseur Xen (système différent de KVM) et nécessite Xen Orchestra pour une gestion web complète; il offre une stabilité et des performances solides, mais sans conteneurs intégrés et avec davantage de configurations en CLI pour certaines fonctions avancées (ex: passthrough GPU ou ZFS sur l’hôte)  .  **TrueNAS SCALE**, à l’inverse, est d’abord pensé comme un NAS haut de gamme utilisant ZFS – il excelle pour le stockage et les services de fichiers. TrueNAS intègre quand même KVM pour les VM et surtout un orchestrateur Kubernetes pour déployer des applications Docker en quelques clics (catalogue d’applications). C’est donc un excellent  **appliance de stockage**  enrichi de virtualisation légère, mais moins flexible qu’un hyperviseur pur pour multiplier les VM  .  **Harvester**, enfin, est une solution très récente d’infrastructure hyperconvergée open-source (projet SUSE/Rancher) visant à combiner virtualisation et conteneurs de façon transparente. Basé sur Kubernetes (k3s) et KubeVirt, il propose une approche moderne similaire aux clouds privés (avec SDN, gestion intégrée du stockage via Longhorn, etc.), mais il est plus complexe à mettre en place et  **consomme davantage de ressources à l’état idle**  qu’une solution comme Proxmox  . Harvester est encore jeune et moins répandu dans les homelabs, bien qu’il soit prometteur pour l’avenir.

En résumé,  **Proxmox VE se distingue par son équilibre**  entre  **richesse fonctionnelle**  (VM + conteneurs, clustering, ZFS, etc.) et  **simplicité d’utilisation**, ce qui le rend très adapté à un homelab polyvalent. XCP-ng pourra convenir pour un usage orienté pure virtualisation Xen, TrueNAS SCALE brillera si le stockage NAS est prioritaire, et Harvester s’adresse aux lab plus expérimentés cherchant à explorer l’approche Kubernetes intégrée.

  

## **Avantages et inconvénients de Proxmox VE**

  

### **Avantages**

-   **Open-source et gratuit**  – Proxmox VE est libre d’utilisation sans limitation de fonctionnalités. La communauté est active et publie régulièrement des mises à jour et correctifs. L’absence de coûts de licence est un plus pour un homelab.
    
-   **Interface unifiée et simplicité**  – Toute l’administration se fait via une interface web ergonomique, centralisant VM, conteneurs, stockage, réseau, etc. Pour un débutant, Proxmox est l’une des solutions les plus faciles à prendre en main pour créer et gérer des VMs  . De nombreuses opérations (création de VM, ajout de disques, snapshots…) sont réalisables en quelques clics.
    
-   **VM et conteneurs sur la même plateforme**  – Pas besoin de choisir entre virtualisation traditionnelle et conteneurs : Proxmox offre les deux nativement. On peut ainsi faire tourner des conteneurs LXC pour les services Linux légers, tout en hébergeant des VM (y compris Windows) sur le même serveur. Cette polyvalence est un atout majeur par rapport à des hyperviseurs qui ne gèrent pas les conteneurs.
    
-   **Écosystème complet (clustering, ZFS, Ceph, backups…)**  – Proxmox intègre out-of-the-box des fonctionnalités normalement réservées aux solutions d’entreprise : gestion multi-nœuds avec quorum, support du stockage distribué Ceph, snapshots, planification des backups, gestion fine du réseau (VLAN, firewall, SDN), etc. Cela évite d’assembler soi-même plusieurs outils externes : tout est prêt à l’emploi.
    
-   **Performance et efficacité**  – Basé sur Debian Linux et KVM, Proxmox offre des performances proches du natif pour les VM. La gestion des ressources est efficace, avec prise en charge de l’oversubscription (overcommit) contrôlée de la RAM et des vCPU. L’overhead système est relativement faible, ce qui permet d’exploiter au maximum le matériel, même modeste. De plus, la prise en charge de technologies comme  **virtIO**  (pilotes paravirt) et  **SPICE**  pour l’affichage optimise les performances des systèmes invités.
    
-   **Support étendu et options pro**  – Bien que gratuit, Proxmox propose des abonnements support pour ceux qui le souhaitent, ainsi qu’une documentation officielle exhaustive. La communauté (forums, wiki) répond rapidement aux questions. On trouve pléthore de tutoriels et de retours d’expérience en ligne (y compris en français) étant donné la popularité de la solution en homelab.
    

  

### **Inconvénients**

-   **Courbe d’apprentissage sur les fonctions avancées**  – Les bases de Proxmox sont simples, mais exploiter des fonctionnalités comme Ceph, la haute dispo ou le SDN demande une certaine expertise. Un débutant pourra se sentir un peu dépassé en configurant un cluster 3 nœuds ou en optimisant ZFS. Il faut accepter de se documenter et parfois de passer par la ligne de commande pour les ajustements pointus.
    
-   **Pas de fonctions NAS intégrées**  – Contrairement à TrueNAS, Proxmox n’embarque pas de service de fichiers prêt à l’emploi. Il n’y a pas de partage SMB/NFS intégré  _by design_. L’hyperviseur se concentre sur les VM/containers, et part du principe que si un NAS est requis, il sera installé  **dans**  une VM ou conteneur séparé (par ex. une VM TrueNAS, ou un conteneur SAMBA)  . Cela demande un peu plus de travail pour mettre en place un serveur de fichiers ou de médias.
    
-   **Gestion des conteneurs limitée aux Linux**  – Les conteneurs LXC de Proxmox ne peuvent exécuter que des systèmes Linux. Si vous devez virtualiser d’autres systèmes (Windows, *BSD), il faudra nécessairement créer une VM. De plus, certains logiciels ne fonctionnent pas bien dans des conteneurs (notamment s’ils requièrent un accès bas niveau ou un kernel module spécifique). Dans ces cas-là, la solution sera de passer par une VM, plus lourde en ressources.
    
-   **Écosystème moins cloud-native**  – Par rapport à une approche Kubernetes (comme Harvester ou même TrueNAS SCALE), Proxmox n’intègre pas de gestion des conteneurs applicatifs Docker au niveau de l’hyperviseur. Chaque VM/CT est une entité séparée à administrer. Il n’y a pas de catalogue d’applications en un clic. Pour un homelab classique cela n’est pas vraiment un problème, mais si l’on cherche une plateforme style PaaS, il faudra installer des outils supplémentaires (ex: Kubernetes sur Proxmox).
    
-   **Abonnement requis pour le dépôt stable**  – Ce n’est pas un frein majeur, mais à noter : sans abonnement payant, les mises à jour de Proxmox se font via le dépôt « non-subscription » (communautaire). Il contient les mêmes packages mais avec potentiellement un peu moins de validation approfondie. Dans les faits, il est tout à fait utilisable en homelab. Un simple message d’avertissement s’affiche dans l’UI si vous n’avez pas de souscription, message que l’on peut masquer manuellement.
    
-   **Impossibilité de downgrader**  – Une fois Proxmox installé ou mis à niveau, il est difficile de revenir à une version antérieure sans tout réinstaller. Il faut donc bien tester les mises à jour majeures sur un environnement de test ou attendre les retours de la communauté avant de sauter le pas en production (homelab avancé). Heureusement, les migrations de version sont généralement bien documentées et fiables.
    

  

_(Suggestion de capture d’écran : Fenêtre de configuration du firewall Proxmox montrant des règles filtrant l’accès aux VMs.)_

  

## **Bonnes pratiques de configuration (sécurité & performances)**

  

Pour tirer le meilleur parti de Proxmox dans un homelab sécurisé, voici quelques conseils de configuration couvrant la  **sécurité**  et les  **performances**.

  

### **Sécurité**

-   **Segmenter les réseaux** : Isolez votre hyperviseur et vos VMs sensibles sur des réseaux/VLAN distincts. Par exemple, placez l’interface management de Proxmox sur un VLAN admin inaccessible depuis le Wi-Fi invité ou Internet. De même, les VMs exposées au web (serveur web, Nextcloud…) devraient résider dans une  **DMZ**  séparée du LAN personnel. Un routeur/pare-feu (OPNsense, pfSense) pourra gérer ces VLAN et filtrer le trafic entre eux.  _(Suggestion de capture d’écran : Schéma réseau avec firewall OPNsense séparant VLAN LAN, DMZ services et Lab)._  Ceci empêche qu’une VM compromise dans la DMZ n’accède à vos appareils du LAN.
    
-   **Configurer le pare-feu Proxmox** : Proxmox VE inclut un firewall applicatif. Activez-le et définissez des règles par défaut restrictives. Par exemple, au niveau datacenter, on peut bloquer tout trafic entrant non nécessaire vers l’hyperviseur (seuls SSH, HTTPS 8006, et éventuellement 5900-5999 pour la console VNC, devraient être autorisés depuis le LAN admin). Pour chaque VM, n’ouvrir que les ports requis (HTTP/HTTPS, etc.). Le firewall Proxmox comprend aussi des  _Security Groups_  pour réutiliser des règles facilement.
    
-   **Mises à jour régulières** : Maintenez Proxmox et vos VM à jour. Les mises à jour de sécurité du noyau Linux, de l’hyperviseur ou des conteneurs corrigent des vulnérabilités critiques. Sur Proxmox, il suffit d’exécuter régulièrement  apt update && apt full-upgrade  (ou via l’UI) pour appliquer les patchs disponibles. Astuce : souscrire au flux RSS des annonces Proxmox pour être alerté des nouvelles versions.
    
-   **Authentification renforcée** : Par défaut, l’authentification se fait avec le compte  root  de Proxmox. Il est recommandé d’activer la  **double authentification (2FA)**  pour l’accès à l’interface web (Proxmox supporte TOTP – par ex. avec Google Authenticator). Envisagez de créer des comptes administrateurs non-root pour l’usage courant, afin de désactiver les connexions root directes sur l’UI et SSH. Utilisez des mots de passe forts ou des clés SSH pour le compte root. Si possible, limitez l’accès SSH à Proxmox aux seules adresses IP de confiance (via  /etc/hosts.allow  ou le firewall).
    
-   **Ne pas exposer Proxmox directement à Internet** : Évitez de rendre l’interface web accessible depuis le web public. Si vous avez besoin d’y accéder à distance, privilégiez une connexion VPN vers votre réseau local, ou éventuellement un tunnel SSH sécurisé. Proxmox n’est pas conçu pour être placé en frontal sur Internet et un accès direct augmenterait la surface d’attaque.
    
-   **Limiter les services sur l’hôte** : N’installez pas de services additionnels (serveur web, base de données, etc.) directement sur l’OS Proxmox. Gardez l’hyperviseur épuré pour réduire les risques. Toute application doit idéalement tourner dans une VM ou un conteneur, jamais sur l’hôte lui-même (sauf outils de monitoring légers). Cela suit le principe de  _séparation des rôles_ : l’hyperviseur fait de la virtualisation, les services tournent  _dans_  les machines virtuelles.
    
-   **Sauvegardes externes et chiffrées** : Stockez les sauvegardes de vos VM sur un stockage externe (NAS, disque USB chiffré, cloud) et non uniquement sur le serveur Proxmox lui-même. En cas de compromission de l’hyperviseur ou de ransomware, il faut pouvoir restaurer ses données à partir d’un emplacement sûr. Proxmox Backup Server (PBS) peut être une bonne solution : il déduplique et chiffre les sauvegardes  . À défaut, un simple export vers un NAS TrueNAS via NFS, avec snapshots ZFS sur le NAS, peut convenir.
    

  

### **Performances**

-   **Bien dimensionner la RAM (surtout avec ZFS)** : Si vous utilisez ZFS sur Proxmox, rappelez-vous que ZFS adore la RAM (cache ARC). Pour un serveur avec 16 Go par exemple, il est raisonnable de réserver 4 à 8 Go pour ZFS et le système, le reste pour les VM. Il est possible de limiter la taille de l’ARC ZFS (zfs_arc_max) pour éviter qu’il n’utilise toute la mémoire. Adapter la RAM en fonction du nombre de VM et de leur usage (base de données, JVM et autres applis consommatrices).
    
-   **Utiliser les SSD/NVMe pour les VM critiques** : Installez Proxmox et vos images de VM sur des  **SSD**  ou NVMe si possible, en particulier pour les OS et applications nécessitant beaucoup d’I/O (base de données, HAProxy, etc.). Les temps d’accès réduits amélioreront nettement la réactivité des services. Vous pouvez dédier un pool ZFS miroir de deux SSD NVMe pour les volumes de VM, et garder les disques durs classiques pour du stockage de masse ou des backups.
    
-   **RAID matériel vs ZFS** : Si vous optez pour ZFS, évitez d’utiliser un RAID matériel en dessous – ZFS préfère avoir un accès direct aux disques (HBA en JBOD). Laissez ZFS gérer la redondance (RAID-Z, miroir) pour bénéficier de ses capacités de correction d’erreurs et surveillance d’intégrité. En revanche, si vous utilisez du  **LVM-thin**  sur un RAID matériel existant, assurez-vous que le contrôleur RAID dispose de cache protégé (BBU) pour de bonnes performances en écriture.
    
-   **Configurer le cache des disques virtuels** : Dans Proxmox, chaque disque virtuel d’une VM a un mode de cache (No cache, WriteBack, etc.). Pour les volumes de systèmes d’exploitation, le mode  **Write Back**  avec  **cache direct**fonctionne bien et accélère les I/O (en assumant que l’hôte a une protection d’alimentation ou que la perte de quelques secondes de données est acceptable). En revanche, pour une VM de base de données où la cohérence prime, on peut rester en  **Write Through**  (par défaut) pour garantir que les écritures sont directement flush sur le stockage.
    
-   **virtIO et agents invités** : Utilisez les  **pilotes virtIO**  pour les périphériques virtuels (disque, réseau) dans vos VM Linux et Windows – ils offrent de bien meilleures performances que l’émulation IDE/E1000. Proxmox propose d’office virtIO pour les nouvelles VM Linux. Pour Windows, installez les  _Drivers virtIO_  fournis par Red Hat. De plus, pensez à installer l’**agent QEMU Guest**  dans chaque VM (paquet  qemu-guest-agent) : cela permet une meilleure interaction (arrêt propre depuis l’UI, informations IP remontées, etc.).
    
-   **Surveillance et affinage** : Mettez en place un monitoring de base de votre hyperviseur (Proxmox exporte des métriques CPU, RAM, réseau par VM dans l’UI). Pour aller plus loin, des outils comme  **Zabbix, Prometheus, Grafana**  ou  **Glances**  peuvent être utilisés pour surveiller l’utilisation des ressources en continu. Vous pourrez ainsi repérer une VM consommatrice qui swap ou un conteneur qui sature le CPU, et ajuster les ressources allouées en conséquence (RAM max, limites CPU, etc.).
    
-   **Éviter la surallocation excessive** : Proxmox autorise de surallouer la RAM (ballooning) et les vCPU (sur plusieurs VM) par rapport au physique. Un léger surcommit peut passer si toutes les VM ne sont pas actives en même temps. Cependant, en homelab, il est plus sûr d’éviter une surallocation trop optimiste, car en cas de pic simultané, vous risquez du swap (ralentissements drastiques) ou une contention CPU. Gardez une petite marge de ressources libres pour l’hyperviseur lui-même.
    
-   **Optimisations spécifiques** : Activez  **l’IOMMU**  dans le BIOS/UEFI de votre machine pour permettre le  **pass-through**  de périphériques (GPU, USB contrôleur) vers les VM si vous en avez besoin (par ex. une VM media center avec GPU pour transcodage). Assurez-vous que la fonction VT-x/AMD-V est activée également pour les VMs. Si vous utilisez un cluster multi-nœuds, configurez un réseau dédié Gigabit+ (ou mieux 10 Gb) pour le trafic de synchronisation (Corosync, Ceph) et un autre pour les VMs, afin d’éviter les saturations. Enfin, en environnement domestique, un onduleur (UPS) communiquant avec l’hyperviseur est conseillé pour envoyer une commande d’arrêt propre en cas de coupure prolongée – il existe des connecteurs NUT ou APCUPSd que l’on peut installer sur Proxmox pour cela.
    

  

## **Gestion du stockage : NVMe,**

## **cold storage**

## **sur HDD, intégration TrueNAS, etc.**

  

La configuration du stockage est un élément clé d’un homelab. Proxmox offre la flexibilité de combiner différentes catégories de disques et même des solutions externes pour obtenir le meilleur compromis entre performance, capacité et résilience :

-   **Utiliser les SSD NVMe pour les workloads actifs** : Les disques NVMe (ou SSD SATA) offrent des IOPS élevées et une faible latence. Il est judicieux d’y stocker les  **disques système**  de vos VM et conteneurs, ainsi que les bases de données et services nécessitant beaucoup d’accès disques. Par exemple, votre VM Home Assistant ou votre base PostgreSQL bénéficieront grandement d’être sur NVMe. Vous pouvez créer un  **pool ZFS en miroir**  de deux NVMe pour la fiabilité (tolérance à la panne d’un disque). Si la capacité NVMe est limitée, ne mettez que les OS et données “chaudes” dessus.
    
-   **Stockage “froid” sur disques durs** : Pour les données volumineuses et moins fréquemment accédées (médias, backups, archives), les disques durs classiques (HDD) conviennent et coûtent moins cher au To. Vous pouvez soit les intégrer au serveur Proxmox (par ex. un deuxième pool ZFS en RAID-Z sur 3-4 HDD pour stocker les sauvegardes de VM, les vidéos, etc.), soit les héberger sur un NAS séparé. Proxmox peut tout à fait accéder à un NAS via NFS ou iSCSI pour y stocker des données  . N’hésitez pas à utiliser la  **compression ZFS**  (lz4) sur les stockages HDD pour réduire la taille des archives et backups (elle est rapide et souvent transparente en coût CPU).
    
-   **Cache et tiering** : Si vous combinez SSD et HDD, pensez à exploiter les SSD comme cache. ZFS permet par exemple d’ajouter un  **cache L2ARC**  (lecture) sur SSD pour accélérer l’accès aux données souvent lues depuis un pool HDD, et un  **disque de log (SLOG)**  sur SSD pour fiabiliser/accélérer les écritures synchrones d’un pool HDD (utile pour un serveur de fichiers ou VM base de données sur HDD). Dans Proxmox, ces opérations se font en ligne de commande (commande  zpool add  pour ajouter cache ou log). Veillez toutefois à utiliser des SSD de qualité pour le SLOG (haute endurance) et une capacité de L2ARC raisonnable (pas plus de 5x la RAM typiquement, sinon l’empreinte mémoire du cache metadata devient contre-productive).
    
-   **Intégration avec un NAS (TrueNAS ou autre)** : De nombreux homelab ont un NAS dédié pour stocker documents, médias, sauvegardes, etc. Proxmox s’intègre bien avec ces solutions. Le cas le plus courant :  **monter un partage NFS du NAS dans Proxmox**  pour y déposer les backups de VM ou même héberger certaines VM non critiques. Par exemple, un NAS TrueNAS SCALE (qui excelle en ZFS) peut exposer un dataset en NFS, ajouté comme stockage dans Proxmox en quelques clics. On peut alors sauvegarder les VM vers ce NAS, ou y héberger des disques de VM moins sensibles aux latences du réseau  . Alternativement, le NAS peut présenter une cible  **iSCSI**  que Proxmox utilisera comme LUN de stockage (souvent couplé à LVM côté Proxmox). L’iSCSI offre de bonnes performances mais une configuration un peu plus complexe (et nécessite une gestion soigneuse si plusieurs nœuds y accèdent simultanément, via un gestionnaire de volume partagé).
    
-   **TrueNAS en VM sur Proxmox** : Une autre intégration possible, si vous n’avez qu’un seul serveur physique, est d’**installer TrueNAS (Core ou SCALE) dans une VM Proxmox**. Dans ce scénario, on passe un contrôleur HBA (SATA/SAS) en  _passthrough_  à la VM TrueNAS, qui a ainsi un accès direct aux disques durs. TrueNAS gère son pool ZFS et sert de NAS, tandis que Proxmox gère la VM TrueNAS comme n’importe quelle VM. Cette approche permet de bénéficier de l’interface de TrueNAS pour le partage de fichiers, tout en profitant des snapshots et backups Proxmox au niveau de la VM complète.  **Attention**  toutefois : en cas d’arrêt de la VM ou de Proxmox, le NAS devient indisponible. De plus, le passthrough demande que votre matériel supporte bien l’IOMMU. Certains homelabers préfèrent cette solution “2-en-1” pour économiser du matériel, mais elle complexifie un peu la maintenance. Si vous optez pour cela, assurez-vous que Proxmox ne tente pas d’utiliser les disques dédiés TrueNAS (ne pas les intégrer dans un pool Proxmox).
    
-   **Proxmox Backup Server (PBS)** : Pour le stockage des sauvegardes, l’outil recommandé est PBS – qui peut tourner sur une machine séparée ou sur une VM/container. PBS déduplique et compresse fortement les backups, économisant de l’espace. Une stratégie courante est d’avoir un petit serveur (ou une VM sur un autre nœud) avec des disques HDD qui sert de PBS, recevant les backups quotidiens chiffrés depuis le Proxmox principal  . En cas de besoin, on peut restaurer granulairement des fichiers ou VM entières depuis PBS via l’UI Proxmox. Si votre homelab est modeste, un simple disque USB externe monté sur Proxmox pour y copier les backups régulièrement peut suffire, mais pensez à la déconnecter quand elle n’est pas en cours d’utilisation (pour éviter qu’un ransomware qui infecterait une VM ne chiffre aussi vos sauvegardes montées).
    

  

En combinant judicieusement NVMe, SSD, HDD et stockage réseau, vous assurez à votre homelab Proxmox à la fois des  **performances élevées**  pour les services critiques et une  **grande capacité de stockage**  pour les données volumineuses, le tout avec des sauvegardes sécurisées. Cette hybridation des stockages est un des grands atouts d’une solution auto-hébergée : vous pouvez ajuster au mieux en fonction de votre budget et de vos besoins.

  

## **Intégration avec Docker, Portainer, Home Assistant, etc.**

  

Proxmox VE, en tant qu’hyperviseur, sert de base pour déployer d’autres couches de services. Voici comment tirer parti de Proxmox pour intégrer des technologies populaires de self-hosting comme Docker/Portainer ou Home Assistant.

  

### **Héberger des conteneurs Docker (avec ou sans Portainer)**

  

Beaucoup d’utilisateurs de homelab souhaitent utiliser  **Docker**  pour déployer des applications (Nextcloud, MariaDB, Jellyfin, etc.). Docker n’est pas fourni en natif sur Proxmox (il ne faut pas l’installer directement sur l’hôte Proxmox, ce n’est pas recommandé). À la place, deux approches s’offrent à vous pour utiliser Docker  **sur**  Proxmox :

-   **LXC Docker** : Créer un conteneur LXC sous Proxmox et y installer Docker. Cette méthode utilise peu de ressources (pas de surcouche OS lourde) et offre des performances quasi natives, puisque l’LXC partage le noyau de Proxmox. Il faut pour cela activer certaines options sur le conteneur (notamment les cgroups et le nesting) afin que Docker puisse tourner à l’intérieur. De nombreux homelabers utilisent par exemple un conteneur Debian LXC dédié comme hôte Docker.  _Avantages:_  légèreté, efficacité.  _Inconvénients:_  Pas de vraie isolation kernel (moins sécurisé qu’une VM) et  **pas de migration à chaud possible**  pour ce conteneur Docker (les conteneurs LXC ne supportent pas la live migration facilement). À noter que Proxmox recommande plutôt la seconde approche pour Docker  , mais l’approche LXC fonctionne et est pratique sur petite infra.
    
-   **VM Docker** : Créer une VM (par ex. Debian, Ubuntu Server) sur Proxmox et y installer le moteur Docker + éventuellement  **Portainer**  pour l’administration web des containers. Cette VM servira de “host Docker” classique. Par rapport à l’option LXC, on a l’avantage d’une isolation complète (noyau séparé), la possibilité de snapshots et migrations  _comme pour n’importe quelle VM_, au prix d’une légère surcouche de ressources (une VM consomme plus de RAM/CPU qu’un LXC minimal). C’est la méthode préconisée pour une fiabilité maximale  . Vous pourriez par exemple avoir une VM Ubuntu 22.04 avec 2 vCPU et 4 Go de RAM qui fait tourner Docker + Portainer, hébergeant vos divers services conteuneurisés.
    

  

**Portainer**  s’intègre dans ce contexte comme un conteneur Docker facultatif offrant une UI pour piloter vos autres conteneurs. Si vous souhaitez l’utiliser, déployez-le dans l’environnement Docker de votre choix (VM ou LXC). Portainer permettra de gérer et superviser vos conteneurs via le navigateur, ce qui complète bien Proxmox (Portainer ne gère que le niveau Docker, tandis que Proxmox gère la VM/CT hôte). Vous pouvez même ajouter plusieurs  **Endpoints**  dans Portainer (par ex. si plus tard vous avez plusieurs hôtes Docker sur différentes VM ou machines).

  

En somme, Proxmox sert ici de  **couche d’orchestration basse**  : on crée une VM ou CT dédiés qui deviennent notre “serveur Docker”. Cette séparation assure que Docker n’interfère pas avec l’hyperviseur et que les bonnes pratiques sont respectées (on ne transforme pas Proxmox en machine à tout faire, on cloisonne). Les deux approches sont valables; pour débuter, la VM Docker+Portainer est souvent plus simple à mettre en œuvre et à maintenir à jour (car identique à n’importe quel Ubuntu standard, avec les docs Docker officielles applicables).

  

_(Suggestion de capture d’écran : Interface Portainer montrant la liste des containers en fonctionnement, hébergée dans une VM Proxmox.)_

  

### **Exécuter Home Assistant sur Proxmox**

  

**Home Assistant**  est une application phare en domotique, que de nombreux utilisateurs souhaitent auto-héberger. Proxmox est un excellent hôte pour Home Assistant, car il permet de faire tourner  **Home Assistant OS**  (la distribution dédiée officielle) dans une VM avec la possibilité de faire des snapshots avant mise à jour, de restaurer rapidement en cas de crash, etc.

  

Plusieurs méthodes s’offrent à vous pour Home Assistant (HA) sur Proxmox :

-   **VM Home Assistant OS** : La méthode recommandée consiste à installer Home Assistant OS dans une VM KVM. L’équipe HA fournit une image disque (fichier .qcow2 ou .vdi) prête à l’emploi. On crée une VM dans Proxmox (2 vCPU, 2 Go RAM par exemple), on importe le disque fourni et on démarre. En quelques minutes, Home Assistant est opérationnel et accessible sur le réseau. Cette VM dédiée tourne 24/7 pour votre domotique. L’avantage est que HA OS inclut le superviseur, ce qui permet d’installer facilement les add-ons, et Proxmox permet de  **snapshoter**  la VM avant une mise à jour Home Assistant, ou de planifier des sauvegardes automatiques de la VM complète.
    
-   **Home Assistant Container ou Core** : Alternativement, si vous ne voulez pas de VM, vous pourriez faire tourner Home Assistant sous forme de conteneur Docker (méthode dite “Home Assistant Core” ou via l’image Docker) à l’intérieur soit d’un LXC, soit d’une VM Linux comme discuté plus haut. Cette approche peut convenir si vous avez déjà une infra Docker et que HA n’a pas besoin d’accéder à du hardware particulier. Cependant, vous perdez la simplicité du  _superviseur_  HA et des add-ons. En général, la communauté recommande l’approche VM HA OS pour une expérience la plus simple et complète.
    

  

Avec Home Assistant sur Proxmox, quelques  **bonnes pratiques**  spécifiques :

-   Pensez à  **activer l’USB passthrough**  si vous avez des dongles Zigbee/Z-Wave ou autre matériel domotique branché au serveur. Proxmox permet d’attacher un périphérique USB physique à une VM (via l’UID ou le port). Vous pourrez ainsi rendre visible le dongle Zigbee dans Home Assistant (la VM) comme s’il y était branché directement.
    
-   Allouez suffisamment de ressources à la VM HA (2 à 4 Go RAM selon la taille de votre installation, et 1 à 2 vCPU minimum). Home Assistant OS est basé sur un Linux Alpine avec Docker, il n’est pas très lourd, mais les add-ons comme Node-RED, Zigbee2MQTT, etc. consomment un peu de RAM.
    
-   Isolez le réseau de la VM HA si possible (par exemple, dans un VLAN spécifique domotique) surtout si vous exposez Home Assistant à Internet (via Nabu Casa ou autre). Même si Home Assistant est relativement sûr, une segmentation limite les impacts en cas d’intrusion.
    
-   Profitez des  **snapshots Proxmox**  : avant de faire une grosse modification dans Home Assistant (mise à jour majeure, ajout d’une intégration expérimentale), prenez un snapshot à chaud de la VM. Si quelque chose va mal, vous pourrez en quelques secondes revenir à l’état antérieur connu.
    
-   Configurez des  **backup automatisés**  de Home Assistant : soit à l’intérieur de Home Assistant (il peut faire des sauvegardes de sa config), soit au niveau Proxmox (sauvegarde programmée de la VM HA comme n’importe quelle VM). Idéalement, exportez ces backups hors du serveur (par ex. copie vers un NAS via un script).
    

  

En adoptant cette approche, Proxmox devient la fondation stable sur laquelle Home Assistant tourne de manière quasi transparente, tout en offrant un filet de sécurité (snapshots, backups faciles) très apprécié des power users.

  

_(Suggestion de capture d’écran : Console Proxmox affichant Home Assistant OS en cours de démarrage dans une VM.)_

  

## **Utilisation des templates (Debian, Ubuntu, Alpine…) pour déployer rapidement des services**

  

L’un des avantages de Proxmox pour gagner du temps est sa prise en charge des  **templates de conteneurs LXC**  et des  **modèles de VM**. Plutôt que d’installer à la main un OS à chaque nouvelle VM ou container, on peut s’appuyer sur ces modèles préconfigurés pour déployer un service en quelques secondes.

  

### **Templates LXC tout prêts**

  

Dans l’interface Proxmox, lorsque vous créez un nouveau conteneur (CT), il est proposé de télécharger un  _template_depuis la bibliothèque officielle Proxmox. Ces templates sont des images minimales d’OS Linux courants, maintenues à jour, par ex.  **Debian 11/12 Standard**,  **Ubuntu 22.04 LTS**,  **Alpine 3.17**,  **CentOS/AlmaLinux 8**, etc. (et même des appliances TurnKey avec des applications préinstallées). Une fois le template (tarball) téléchargé sur votre nœud, déployer un nouveau conteneur revient essentiellement à cloner cette image.

  

**Exemple d’utilisation** : vous voulez rapidement un petit conteneur pour héberger un serveur web Nginx sur Alpine. Depuis Proxmox, cliquez  _Créer CT_, choisissez le template Alpine Linux (téléchargeable s’il ne l’est pas), quelques paramètres (CPU, RAM, mot de passe root) et validez. En 10 secondes le conteneur est créé et en cours d’exécution, avec Alpine prêt à l’emploi. Il ne reste plus qu’à  apk add nginx  via la console, et votre serveur web tourne. Cette vélocité de déploiement est impossible à battre comparé à une installation manuelle d’OS.

  

La même logique s’applique pour Debian ou Ubuntu : si vous avez l’habitude de ces distributions, utiliser le  _template Debian Standard_  vous donne un conteneur minimal (quelques centaines de Mo) en moins d’une minute, sur lequel vous installez uniquement les paquets requis pour votre service (ex:  apt install mariadb-server  si c’est une base MariaDB).  **Alpine**  est excellente pour les tout petits services (très léger, quelques packages de base),  **Debian/Ubuntu**  offrent plus de confort (systemd, glibc) pour des applications plus complexes.

  

Les  **templates TurnKey Linux**  disponibles sont aussi intéressants pour tester rapidement une application : il y a par exemple des CT TurnKey  _Nextcloud_,  _WordPress_,  _GitLab_  etc., qui viennent préconfigurés. En quelques clics on peut avoir une instance Nextcloud fonctionnelle. Cependant, pour un usage pérenne en homelab, on préfère souvent déployer manuellement sur un OS propre pour mieux contrôler la config. À vous de voir.

  

Les conteneurs LXC Proxmox sont des  _containeurs système_  (OS complet) et non juste des conteneurs applicatifs type Docker  . Cela signifie que chaque conteneur se comporte comme une mini distrib Linux isolée. On peut s’y connecter en root (ou via un utilisateur si configuré) et l’administrer comme un petit serveur. Proxmox facilite grandement cette gestion grâce à  pct, l’outil en ligne de commande, ou via l’interface pour modifier ressources, réseau, montages, etc.  . Les conteneurs s’intègrent aux mécanismes Proxmox (backups, snapshots, firewall, etc. fonctionnent aussi pour eux).

  

### **Modèles et clones de VM**

  

Outre les LXC, Proxmox permet également de travailler avec des  **templates de VM**. L’idée ici est de créer une VM de base (par ex. une Debian 12 minimale installée avec les bons réglages), puis de la  **convertir en modèle**. Ce modèle ne sera pas démarré directement, mais pourra être  **cloné**  à la demande pour créer de nouvelles VM identiques. En homelab, cela sert si vous avez besoin de déployer plusieurs VM similaires rapidement (par exemple, 5 VM Ubuntu pour tester un cluster, ou simplement gagner du temps à ne pas répéter l’installation OS). La fonction  _Clone_  de Proxmox, surtout en mode  _clone complet_, permet d’obtenir en quelques minutes une copie indépendante d’une VM modèle.

  

Un usage pratique : vous avez une VM  **Windows 10**  configurée (générée via  _Sysprep_  pour avoir un SID unique à la clone). Vous la convertissez en  _Template_. Lorsqu’un jour vous avez besoin d’une nouvelle VM Windows pour tester un logiciel, vous faites  _Cloner_  ->  _Créer VM clonée_, et en 2-3 minutes la nouvelle VM Windows est prête, sans devoir réinstaller depuis l’ISO et passer 30 min de configuration. Il faudra juste l’activer avec votre licence et la renommer.

  

Pour Linux, le gain est moindre (car l’installation est rapide), mais ça reste utile si vous avez personnalisé un modèle avec des outils communs à toutes vos VM (par ex. une Debian modèle avec  sudo,  vim, les dépôts non-free activés, l’agent QEMU installé, etc.). Chaque nouvelle VM clonée héritera de ces préparatifs.

  

Enfin, notez que Proxmox gère  **Cloud-Init**  pour les VM : vous pouvez télécharger des images  _cloud_  (Ubuntu Cloud, Alpine Cloud, etc.), les utiliser comme base de VM, et via l’outil Cloud-Init de Proxmox, injecter automatiquement un utilisateur, un mot de passe ou une clé SSH, une configuration réseau, etc. Cela facilite le déploiement automatisé de VM prêtes à l’emploi, surtout si vous orchestrez votre homelab avec des outils comme Terraform ou Ansible. Pour un débutant, ce n’est pas indispensable, mais c’est bon de savoir que la fonctionnalité existe pour le jour où l’on veut automatiser davantage.

  

En résumé, exploitez les templates mis à disposition par Proxmox pour  **accélérer vos déploiements**. Non seulement cela vous fait gagner du temps, mais en plus vous obtenez des installations propres et cohérentes (toutes basées sur l’image officielle de la distrib choisie). La gestion ultérieure s’en retrouve facilitée et homogène à travers vos conteneurs et VMs.

  

_(Suggestion de capture d’écran : Menu Proxmox montrant la liste des templates LXC disponibles au téléchargement.)_

  

## **Ressources officielles et communautés utiles**

  

Pour aller plus loin avec Proxmox VE et obtenir de l’aide, voici quelques liens utiles :

-   **Site Officiel Proxmox VE**  – Page officielle avec présentation des fonctionnalités, téléchargements de l’ISO, informations sur les offres de support, etc. (disponible en anglais et autres langues).  [proxmox.com](https://www.proxmox.com/en/proxmox-ve)
    
-   **Documentation Officielle (Wiki)**  – Le wiki de Proxmox VE contient le guide d’administration complet, les notes de version, et de nombreux articles techniques (réseau, stockage, etc.). C’est la référence à consulter pour les détails pointus.  [pve.proxmox.com/wiki](https://pve.proxmox.com/wiki/)
    
-   **Forum Communautaire Proxmox**  – Forum officiel où échangent les développeurs et utilisateurs. Très actif, on y trouve des sections par thématiques et langue (une catégorie française existe pour l’aide dans la langue de Molière). Idéal pour poser vos questions ou chercher si quelqu’un a déjà résolu un problème similaire.  [forum.proxmox.com](https://forum.proxmox.com/)
    
-   **Proxmox VE sur GitHub**  – Le code source de Proxmox VE et de ses composants (QEMU custom, GUI, etc.) est accessible sur GitHub. Utile pour les contributeurs ou pour signaler des bugs sur Bugzilla.  [github.com/proxmox](https://github.com/proxmox)
    
-   **Communautés & tutoriels**  – En dehors des ressources officielles, de nombreuses communautés homelab abordent Proxmox : par exemple le subreddit  _r/Proxmox_  (en anglais) regorge de retours d’expérience, le subreddit  _r/homelab_discute souvent de Proxmox vs autres solutions, et il existe des groupes Facebook francophones dédiés à Proxmox. N’hésitez pas à consulter des blogs spécialisés (par ex.  **tutos-info.fr**,  **blog.stephane-robert.info**  en français) ou des chaînes YouTube (Techno Tim, Level1Techs, …) qui proposent des tutoriels et comparatifs sur Proxmox et son écosystème.

