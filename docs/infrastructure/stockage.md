# **Gestion du stockage dans un homelab sécurisé**

  

## **Comparatif des systèmes de fichiers : ZFS vs Btrfs vs EXT4 vs XFS**

  

Dans un homelab, le choix du système de fichiers impacte directement la  **fiabilité des données**  et les fonctionnalités disponibles (snapshots, RAID, compression…). Les systèmes de fichiers traditionnels comme  **EXT4**  et  **XFS**  offrent stabilité et performance de base, tandis que  **Btrfs**  et  **ZFS**  intègrent des fonctions avancées (copie sur écriture, somme de contrôle, etc.) pour améliorer l’intégrité des données. Voici un comparatif détaillé de leurs caractéristiques, avantages, inconvénients et cas d’usage.

  

### **EXT4 – Le pilier stable et éprouvé**

  

**EXT4**  (Fourth Extended Filesystem) est le système de fichiers par défaut de nombreuses distributions Linux depuis 2008. Évolution du EXT3, il est réputé pour sa  **stabilité**  et sa  **performance équilibrée**, avec un journal assurant une récupération rapide en cas de crash. Il fonctionne bien dans presque tous les scénarios généraux.

-   **Avantages :**  Mature et ultra-stable (plus de 10 ans d’usage intensif)  . Large compatibilité (supporté nativement par la plupart des OS Linux et outils). Bonnes performances générales en lecture/écriture, y compris sur des disques de grande capacité. Utilise un  **journaling**  qui améliore la résilience en cas de panne système  .
    
-   **Inconvénients :**  Pas de fonctionnalités avancées intégrées – absence de snapshots, de compression transparente ou de mécanisme de RAID interne  . Protection d’intégrité limitée : pas de  **somme de contrôle des données**  (seulement journal des métadonnées), donc pas de détection automatique de corruption. Évolutivité suffisante pour la plupart des cas, mais en théorie moins extensible que Btrfs/ZFS.
    
-   **Cas d’usage recommandés :**  Systèmes Linux généraux (PC, serveurs web, etc.) où la  **simplicité et la fiabilité éprouvée**  priment  . Parfait pour les partitions système ou le stockage de données non critiques dans un homelab, lorsque l’on n’a pas besoin des fonctionnalités avancées des systèmes modernes.
    

  

### **XFS – Le moteur de performance évolutif**

  

**XFS**  est un système de fichiers journalisé 64 bit, créé à l’origine pour SGI IRIX (années 90) et adopté dans le monde Linux (c’est le choix par défaut de Red Hat Enterprise Linux, par ex.). Il est réputé pour sa  **haute performance en I/O**(surtout sur les gros fichiers et les opérations parallèles) et son excellente  **scalabilité**  sur de très grandes volumétries.

-   **Avantages :**  Très  **performant en débit**  séquentiel et pour les fichiers de grande taille  . Supporte des systèmes de fichiers massifs (jusqu’à 8 exabytes) et gère efficacement les charges intensives (grandes bases de données, streaming vidéo, etc.). Stable et éprouvé en production, avec des fonctionnalités comme la défragmentation à chaud et le journaling pour protéger les métadonnées.
    
-   **Inconvénients :**  Moins efficace avec d’innombrables petits fichiers ou des charges très métadonnées (le design privilégie les grands flux)  . N’intègre  _pas_  de snapshots, compression ou chiffrement natifs (il faut s’appuyer sur LVM ou des solutions externes pour ces besoins)  . De plus, XFS ne permet pas de  **réduire**  la taille d’un système de fichiers (on peut l’agrandir, mais pas le shrink).
    
-   **Cas d’usage recommandés :**  Serveurs de fichiers volumineux, stockage de médias et de vidéos,  **workstations de montage**  vidéo, environnements avec gros fichiers (images disque, backups volumineux) ou  **hôtes de virtualisation**  gérant de nombreux fichiers VM de taille conséquente  . XFS excelle dès qu’il faut un débit maximal et une gestion efficace de très grandes capacités.
    

  

### **Btrfs – L’innovateur riche en fonctionnalités**

  

**Btrfs**  (B-Tree FS, souvent prononcé “Butter FS”) est un système de fichiers moderne introduit en 2009, conçu pour apporter des fonctionnalités de niveau entreprise sur Linux (copy-on-write, snapshots, RAID logiciel…). Il est par exemple le système par défaut sur Fedora, openSUSE et utilisé par Synology sur certains NAS. Btrfs est souvent comparé à ZFS en termes d’objectifs, tout en étant intégré au noyau Linux.

-   **Fonctionnalités clés :**  Btrfs prend en charge les  **snapshots instantanés**  (copies d’état du système de fichiers à un instant T) et les  **subvolumes**  (unités logiques séparées)  . Il propose un  **RAID logiciel intégré**  (niveaux 0, 1, 10 et 5/6 expérimentaux) sans nécessiter d’outil externe, ainsi que la  **compression transparente**  des données (algorithmes supportés : zlib, lzo, zstd) pour économiser de l’espace  . Surtout, Btrfs calcule des  **checksums**  sur les données et métadonnées pour détecter les corruptions, et peut réaliser une auto-réparation s’il y a redondance (RAID 1/10)  . On note aussi une grande flexibilité avec possibilité d’**agrandir ou réduire**  un volume en ligne, et une gestion fine des quotas et de la déduplication (via des outils externes).
    
-   **Avantages :**  **Snapshots rapides**  et peu coûteux en espace (on peut en prendre fréquemment pour tester des modifications et revenir en arrière facilement).  **RAID logiciel**  directement dans le système de fichiers (pas besoin de mdadm), avec tolérance aux pannes et auto-réparation (en modes mirroring)  .  **Compression**  et  **déduplication**(optionnelle) pour optimiser le stockage. Évolutivité et souplesse (ajout/retrait de disques dans un volume Btrfs, redimensionnement à la volée). En somme, Btrfs est un véritable  **couteau suisse**  du stockage, réunissant de nombreuses fonctionnalités avancées.
    
-   **Inconvénients :**  Moins mature qu’EXT4/XFS dans certaines situations critiques : toutes les distributions ne considèrent pas Btrfs aussi “production-ready” pour les workloads intensifs (bien qu’il s’améliore constamment)  . L’implémentation des modes RAID 5/6 est encore considérée  _expérimentale_  – des problèmes de fiabilité n’ont pas été totalement résolus à ce jour, ce qui déconseille leur usage en production  . De plus, Btrfs induit un léger  **surcoût de performance**  dû au copy-on-write et aux checksums (impact surtout visible sur du matériel plus ancien ou des charges lourdes en écriture)  . Enfin, l’**administration**  peut sembler complexe pour les débutants (subvolumes, équilibre du FS, etc.), même si des outils comme  btrfs-progs  ou Snapper facilitent la gestion.
    
-   **Cas d’usage recommandés :**  Idéal pour les  **NAS auto-hébergés**  et serveurs domestiques qui profitent de ses fonctionnalités (par ex. snapshots programmés pour les sauvegardes, ou utilisation de la compression pour stocker plus de données). Recommandé aussi pour les utilisateurs développeurs ou “power users” qui apprécient les  **fonctionnalités avancées**  et les options de rollback facile (ex: snapshots avant une mise à jour système). Btrfs convient bien également aux environnements de containers et machines de test où l’on veut cloner rapidement des environnements (subvolumes et snapshots) tout en bénéficiant de la sécurité des checksums.
    

  

### **ZFS – Le gardien de l’intégrité des données**

  

**ZFS**  est souvent considéré comme le  **Saint Graal de la fiabilité**  des systèmes de fichiers. Conçu par Sun Microsystems et désormais développé via le projet open source OpenZFS, il combine un système de fichiers  **et**  un gestionnaire de volumes logiques. ZFS introduit des concepts révolutionnaires de  **stockage en pool**  (agrégation flexible de disques) et de  **principe Copy-on-Write**  généralisé garantissant une forte cohérence. Très utilisé dans les NAS professionnels (TrueNAS, etc.) et les infrastructures exigeant une haute intégrité, ZFS est aussi supporté sous Linux (via module DKMS) et intégré nativement à FreeBSD.

-   **Fonctionnalités clés :**  ZFS offre un éventail complet de fonctionnalités: système Copy-on-Write avec  **snapshots et clones**  natifs (snapshots instantanés et clones modifiables)  ,  **vérification d’intégrité**  continue avec  **checksums sur chaque bloc de données**  et  **auto-réparation**  en cas de corruption grâce aux copies redondantes  , compression transparente (algorithme LZ4 par défaut, ZSTD possible),  **deduplication**  optionnelle des données, et bien sûr son propre système de  **RAID avancé (RAID-Z)**  équivalent aux RAID 5/6/7 mais sans les écueils classiques (pas de  _write hole_, reconstruction plus sûre)  . ZFS gère également le  **chiffrement natif**  (depuis OpenZFS 2.0) et la réplication de snapshots vers d’autres systèmes ZFS. Enfin, il est notoirement  **scalable**  à des échelles énormes (pouvant adresser des pétaoctets à l’aise, bien au-delà des besoins d’un homelab).
    
-   **Avantages :**  **Intégrité des données inégalée**  – chaque lecture est vérifiée par checksum, et en cas d’erreur ZFS peut automatiquement corriger en lisant une copie saine (sur un miroir ou RAID-Z)  .  **Stockage unifié en pools**  : on peut agréger les disques en vdevs (simple, miroir, RAID-Z…) au sein d’un même pool et créer des systèmes de fichiers ou volumes à la volée, ce qui simplifie la gestion comparé à LVM.  **Snapshots et clones**  efficaces pour les backups et tests, intégrés directement (on peut par exemple snapshotter une VM en un instant et la cloner pour testing).  **Compression**  et  **dedup**  économisent de l’espace (au prix de ressources CPU/RAM supplémentaires)  .  **Évolutivité et performance**  : ZFS peut exploiter de grandes quantités de RAM comme cache (ARC) pour accélérer les accès, et il supporte le cache SSD (L2ARC) et le log séparé (ZIL/SLOG) pour booster les performances d’écriture sync. Conçu initialement pour de l’entreprise, ZFS  **monte en puissance**  avec le matériel – plus on lui donne de la RAM et des disques rapides, mieux il se comporte, du petit NAS jusqu’au stockage de datacenter  .
    
-   **Inconvénients :**  **Gourmand en ressources**  : il est recommandé d’avoir de la RAM en quantité (souvent on suggère ~1 Go par To de stockage, bien que cela dépende des fonctions activées)  . Sur du matériel limité (petit homelab avec 2 Go de RAM par ex), ZFS peut ne pas donner la pleine mesure de ses avantages. De plus, sous Linux, ZFS n’est pas intégré au noyau pour des raisons de licence (CDDL incompatible GPL), il faut donc l’installer séparément (module DKMS ou kernel custom) – cela ajoute un peu de complexité aux mises à jour système  . La  **complexité**  générale de ZFS peut être déroutante pour un débutant : de nombreux concepts (pool, vdevs, dataset, zvol, etc.) et options de réglage. Cela peut sembler  _overkill_  pour de petites configurations, où un système plus simple suffirait. Enfin, notons que l’**expansion d’un pool ZFS**  peut être contraignante selon la configuration (on ne peut pas encore, au 21, étendre facilement un vdev RAID-Z en ajoutant un disque, sans recréer le vdev – bien que la fonctionnalité d’extension RAID-Z soit en développement actif).
    
-   **Cas d’usage recommandés :**  ZFS brille dans les scénarios où la  **préservation des données**  est critique. Pour un homelab sécurisé, si vous hébergez des  **machines virtuelles**  ou des  **conteneurs**, ZFS (intégré dans Proxmox VE, par exemple) permet d’avoir des  **snapshots cohérents**  et d’éviter la corruption silencieuse des images disque grâce aux checksums. Idéal aussi pour un  **NAS maison**  où l’on veut un maximum de fiabilité (par exemple un serveur de sauvegarde centralisé, un stockage pour photos/vidéos précieuses) – ZFS assure que vos backups ne se dégradent pas avec le temps (protection contre le  _bitrot_). En entreprise ou pour les utilisateurs avancés, ZFS est le choix de prédilection pour les  **appliances de stockage**  (NAS TrueNAS, serveurs de fichiers critiques, etc.) grâce à son mélange de performance et de sécurité des données  . En résumé, si vos besoins requièrent la  **sûreté maximale des données**  et que vous disposez du matériel adéquat, ZFS est un allié de poids.
    

  

## **Redondance et tolérance aux pannes**

  

Même dans un homelab, il est essentiel de prévoir les défaillances matérielles. Disques qui tombent en panne, erreurs humaines, ransomware – une bonne stratégie de stockage  _sécurisé_  repose sur plusieurs couches :  **redondance en temps réel**  (RAID, mirroring…),  **points de restauration**  (snapshots) et  **sauvegardes externes**. Nous allons passer en revue les options de RAID, l’utilisation des snapshots et les stratégies de sauvegarde, spécialement adaptées à un homelab.

  

### **RAID : niveaux, matériel vs logiciel**

  

L’acronyme  _RAID_  (Redundant Array of Independent Disks) décrit les solutions combinant plusieurs disques pour améliorer soit la performance, soit la tolérance de panne – souvent les deux. Il existe plusieurs  **niveaux de RAID**courants, chacun avec ses caractéristiques :

-   **RAID 0 (Striping)** : données réparties (enchaînées) sur plusieurs disques pour augmenter le débit. Offre des  **performances maximales**, idéal quand la vitesse prime, mais  **sans aucune redondance**  – la panne d’un seul disque entraîne la perte de toutes les données  . À réserver aux usages non critiques (cache, rendu vidéo temporaire, etc.).
    
-   **RAID 1 (Mirroring)** : les données sont  **dupliquées en miroir**  sur deux disques (ou plus). Fournit une  **excellente tolérance aux pannes**  (on peut perdre un disque sans perte de données) mais au prix d’une capacité réduite de moitié (2 To + 2 To en RAID 1 = 2 To utiles)  . Performances en lecture élevées (on lit sur deux disques en parallèle), écriture un peu plus lente qu’un seul disque. Recommandé pour stocker des données importantes sur peu de disques (2 disques) ou comme miroir du système.
    
-   **RAID 5** : utilise un  **parity striping**  distribué – les données sont réparties sur  _n-1_  disques et sur le disque restant est stockée une information de parité (calculée) permettant de reconstruire les données d’un disque manquant. Ce niveau offre un  **équilibre entre performance, capacité et redondance**  : la lecture est rapide (plusieurs disques en parallèle) et la capacité utilisée est optimisée (seulement l’équivalent d’un disque est “sacrifié” pour la parité)  .  **Tolérance de panne : 1 disque**. Inconvénients : performances d’écriture un peu moindres (calcul de la parité) et surtout reconstruction longue en cas de crash d’un disque. Si un second disque flanche pendant la reconstruction, les données sont perdues. RAID 5 convient aux espaces de stockage importants où on veut un compromis capacité/sécurité raisonnable, mais on lui préfère souvent RAID 6 aujourd’hui.
    
-   **RAID 6** : variante de RAID 5 avec  **deux blocs de parité**  répartis, pouvant tolérer la panne  **simultanée de 2 disques**. La sécurité est renforcée au prix d’une capacité utilisable légèrement inférieure (l’équivalent de 2 disques sert de parité). C’est une solution robuste pour les baies de disques de grande capacité, au cas où un second disque lâche lors de la reconstruction du premier. Les performances sont un peu en deçà du RAID 5 en écriture (parité double à calculer) mais restent acceptables en lecture et adaptées aux environnements exigeant une haute redondance  .
    
-   **RAID 10 (ou 1+0)** : combinaison du mirroring et du striping. On associe des paires de disques en miroir (RAID 1) puis on agrége ces miroirs en RAID 0 (striping entre les miroirs). On obtient ainsi  **d’excellentes performances**(équivalentes à RAID 0 sur lecture/écriture)  **ET une tolérance aux pannes élevée**  (chaque paire peut tolérer une panne)  . Le RAID 10 nécessite au minimum 4 disques (et un nombre pair) et n’utilise que 50% de l’espace total (puisque mirroring). C’est une solution prisée pour les applications critiques nécessitant à la fois débit et fiabilité (par ex. un serveur de base de données dans un homelab). Coût plus élevé en nombre de disques, mais reconstruction rapide en cas de panne (on recopie simplement un miroir).
    
-   **JBOD (Just a Bunch Of Disks)** : techniquement pas un RAID, le JBOD permet de concaténer des disques indépendants l’un à la suite de l’autre, ou simplement de les présenter individuellement. Il n’y a  **pas de gain de performance ni de redondance**  – c’est juste un moyen d’utiliser plusieurs disques comme un seul grand volume, ou comme disques séparés. Utile éventuellement pour maximiser la capacité de stockage brute quand la tolérance de panne n’est  _pas_  une priorité  .
    

  

**RAID matériel vs RAID logiciel :**  Le RAID peut être géré  **soit par du matériel dédié (carte contrôleur RAID)**, soit par le  **système d’exploitation (RAID logiciel)**. En RAID  _matériel_, un contrôleur se charge de toutes les opérations de gestion des disques de manière autonome (souvent avec sa propre mémoire cache, parfois protégée par batterie). Cela offre typiquement de meilleures performances sur des charges lourdes et soulage le CPU principal  . Le RAID matériel apporte aussi souvent des options avancées (ex. certains niveaux RAID supplémentaires) et une indépendance vis-à-vis du système (le contrôleur présente un seul volume logique, utilisable par n’importe quel OS)  . En contrepartie, ces cartes sont coûteuses et introduisent une dépendance : si la carte tombe en panne, il faut idéalement la même pour récupérer l’array.

  

Le  **RAID logiciel**, lui, s’appuie sur la puissance de calcul du processeur hôte pour gérer le RAID (ex: via mdadm/Linux ou Storage Spaces sous Windows). De nos jours, avec des CPU performants, le RAID logiciel offre des performances très honorables – l’impact est négligeable pour des niveaux simples (RAID 1, 0)  , et reste raisonnable pour RAID 5/6 sur un homelab. L’avantage principal est le  **coût nul**  (pas de matériel additionnel) et une flexibilité accrue (migration possible d’un OS à un autre dans certains cas, ajout de disques parfois plus souple)  . Cependant, le RAID logiciel peut être un peu  **plus lent**  dans certains scénarios intensifs car il doit partager le CPU avec le reste du système  . Il est aussi lié au logiciel : par exemple, un RAID logiciel Windows ne sera pas lisible sous Linux et vice-versa, ce qui peut compliquer une migration.

  

En résumé, pour un  **homelab**, un RAID logiciel (mdadm, espace de stockage ZFS/Btrfs, etc.) est souvent préférable pour éviter les frais et la complexité matérielle – les performances sont largement suffisantes dans un contexte domestique. Le RAID matériel se justifie surtout si vous avez déjà une carte (serveur de récup) ou pour des cas très spécifiques (besoin d’un cache protégé par batterie, compatibilité multi-OS sur du dual-boot, etc.). Dans tous les cas,  **RAID ≠ backup**  : même avec redondance, une erreur humaine ou un ransomware peut détruire les données, d’où l’importance des snapshots et sauvegardes que nous abordons ci-dessous.

  

### **Snapshots : restauration instantanée et limites**

  

Les  **snapshots**  sont des points de restauration instantanés d’un volume ou système de fichiers, parfois appelés  _“photographies”_  de l’état des données à un instant  _t_. Beaucoup de solutions de stockage modernes (ZFS, Btrfs, LVM, VMware, etc.) les proposent. Ils offrent la capacité de revenir en arrière rapidement après un problème : par exemple, si une VM ou un dossier a été modifié par erreur, on peut restaurer le snapshot précédent en quelques secondes. Le principal avantage d’un snapshot par rapport à une sauvegarde traditionnelle est la  **rapidité**  et la fréquence : on peut en créer très souvent (par exemple toutes les heures) sans trop d’impact, ce qui procure de nombreux points de restauration granulaires  . En cas d’incident, redémarrer une VM ou restaurer un fichier à partir d’un snapshot local est quasi immédiat, alors qu’une restauration depuis une backup complète peut prendre beaucoup plus de temps.

  

**Limites des snapshots :**  un snapshot n’est  **pas une sauvegarde indépendante**. Contrairement à une vraie copie de sauvegarde, un snapshot dépend du stockage principal : il stocke généralement seulement les différences (blocs modifiés) et pointe vers les blocs inchangés. Si le volume principal est perdu (panne hardware grave) ou corrompu, les snapshots sont perdus avec lui  . De plus, les snapshots consomment de l’espace sur le stockage d’origine pour conserver les anciennes versions des blocs – plus on en garde, plus on utilise d’espace (ils sont souvent auto-nettoyés au bout de X jours/heures pour éviter de saturer le stockage). Il faut donc les voir comme des  **outils complémentaires**  pour la restauration rapide, et non comme une solution de backup autonome. La  **bonne pratique**  est d’ailleurs de combiner snapshots  _et_  sauvegardes  : les snapshots fournissent des points de reprise fréquents (par ex. horaires) sur le stockage principal, et les sauvegardes externes assurent la récupération en cas de perte totale du système (points de restauration quotidiens, hebdomadaires, conservés hors du système principal)  .

  

**Outils de snapshots :**  Pour en bénéficier dans un homelab, on peut s’appuyer sur ZFS (commandes  zfs snapshot, ou outils comme sanoid/syncoid pour automatiser), sur Btrfs (btrfs subvolume snapshot  et l’outil Snapper pour la gestion automatisée). LVM propose aussi des snapshots de volumes logiques LVM qui peuvent servir à prendre une image cohérente d’une partition ext4/xfs avant backup, par exemple. Sur Windows/VMware, on retrouve des fonctionnalités similaires (Volume Shadow Copy, snapshots VMware). Attention à l’impact performance : la méthode de copy-on-write peut  **ralentir les écritures**  en présence de snapshots nombreux (ce fut un problème avec Btrfs, et même ZFS subit un léger impact au-delà de centaines de snapshots). Il faut surveiller et purger les snapshots obsolètes.

  

En somme, les snapshots sont  **formidables pour annuler une bêtise**  rapidement ou tester des mises à jour, mais ils ne dispensent pas de faire de vraies sauvegardes sur un stockage séparé. Utilisez-les comme première ligne de défense (récupération quasi instantanée) en complément de vos backups.

  

### **Sauvegardes : stratégies locales vs cloud, outils open source**

  

La sauvegarde est le pilier ultime d’un homelab sécurisé. Elle garantit qu’en cas de sinistre majeur (panne de plusieurs disques, vol ou incendie, erreur de manipulation irréversible), vos données pourront être restaurées. On conseille souvent la règle du  **3-2-1**  : 3 copies des données, sur au moins 2 supports différents, dont 1 copie  _offsite_  (hors site). Pour un homelab, cela peut se traduire par exemple par un NAS local + une sauvegarde sur disques externes + une réplication chiffrée vers le cloud.

  

**Sauvegardes locales (on-site) :**  Elles offrent l’avantage de la  **vitesse**  et du contrôle. Il peut s’agir d’un disque USB de backup, d’un second NAS chez vous, ou même d’une autre machine sur le réseau local qui reçoit des backups. L’outil de base souvent utilisé est  **rsync**  (ou son dérivé  rsnapshot  pour gérer des rotations). Par exemple, on peut programmer un rsync quotidien de son dossier de documents vers un disque externe. Pour des solutions plus élaborées et efficaces en espace, on se tournera vers des logiciels de sauvegarde dédupliqués comme  **BorgBackup**  ou  **restic**. Ces outils open source permettent de sauvegarder des données (localement ou vers un serveur distant) en ne stockant qu’une seule fois les blocs identiques (déduplication), en compressant et en chiffrant les backups. Ils gèrent en plus un système de rétentions (ex: garder 7 daily, 4 weekly, 6 monthly, etc.). D’après les retours de la communauté,  _Restic_  et  _Borg_  font partie des solutions les plus fiables actuellement (fiabilité primant sur la vitesse)  . Borg fonctionne en mode “push” (on peut lancer la sauvegarde depuis la source vers un repo distant via SSH), Restic également et propose en plus un mode “serveur REST”. Ce sont deux excellents choix pour un homelab, avec une légère préférence pour Restic si l’on veut quelque chose de simple et efficace en ligne de commande (ou Borg pour ses performances en local/SSH)  .

  

Pour les environnements virtualisés, l’outil  _Proxmox Backup Server (PBS)_  mérite d’être mentionné. Il s’agit d’une solution de sauvegarde  **open source**  dédiée aux VMs/CTs Proxmox, qui stocke de façon  **incrémentale et dédupliquée**  les données afin d’économiser énormément d’espace  . PBS utilise en plus une compression ZSTD rapide et un chiffrement authentifié côté client, ce qui le rend très efficace pour sauvegarder des machines virtuelles entières en quelques minutes avec très peu de données transférées après la première fois  . Dans un homelab, on peut par exemple avoir un petit serveur (ou VM) PBS qui réceptionne les sauvegardes de toutes les VM/CT Proxmox régulièrement. L’interface web PBS permet la restauration facile de VMs complètes ou de fichiers individuels à partir de ces backups.

  

**Sauvegardes distantes (cloud/off-site) :**  Pour se protéger des risques locaux (incendie, dégât des eaux, vol), il est recommandé d’avoir au moins une copie des données  _hors de chez vous_. Plusieurs approches s’offrent à un homelab : utiliser un service de cloud public (p.ex. stockage objet type Backblaze B2, AWS S3, Wasabi… ou même un stockage chiffré sur Google Drive, etc.), ou bien synchroniser vers un autre site (par ex, un NAS chez un ami/famille). Les outils comme  **restic**  brillent ici car ils supportent nativement de nombreuses destinations cloud (Backblaze, AWS, Azure, rclone, etc.) et chiffrent tout automatiquement.  **Duplicati**  est une autre solution populaire avec une interface graphique web, permettant de sauvegarder chiffré vers divers clouds (c’est un logiciel libre qui segmente les fichiers et gère versions/chiffrement). On peut aussi citer  **Rclone**  qui, couplé à un cryptage, permet de synchroniser dossiers et backups vers des clouds comme on le ferait avec un rsync (sans gestion de versions toutefois).

  

**Stratégie recommandée :**  En homelab, on peut combiner un NAS local pour les sauvegardes rapides et un envoi cloud hebdomadaire/mensuel des données critiques chiffrées. Par exemple, backups quotidiens sur un disque local ou NAS via Borg, puis une fois par semaine, on envoie les nouvelles archives Borg sur un stockage cloud (Borg peut utiliser rclone pour cela). Ainsi on a une sauvegarde locale pour les urgences, et une copie externe pour le pire cas. Le versioning de plusieurs générations est important pour pouvoir récupérer même en cas de fichiers corrompus ou chiffrés par un ransomware (on restaure une version plus ancienne propre). Enfin,  **tester les restaurations**  régulièrement est crucial – une sauvegarde non testée ne peut pas être considérée comme fiable !

  

En résumé,  _snapshot, RAID et backup forment un trio complémentaire_. Dans un homelab sécurisé, on configurera idéalement un RAID pour parer la panne d’un disque, des snapshots fréquents pour rattraper immédiatement les erreurs/logiques, et des sauvegardes déportées pour se prémunir contre les catastrophes majeures. Cela peut sembler complexe, mais de nombreux outils libres sont là pour automatiser ces tâches et rendre votre stockage quasiment aussi robuste que dans un environnement pro 😉.

  

## **Intégration d’un NAS et stockage centralisé avec Proxmox**

  

Beaucoup de hobbyistes disposent d’un  **NAS**  (Network Attached Storage) à la maison pour centraliser les fichiers, les sauvegardes ou même héberger des médias. Intégrer ce NAS avec votre serveur de virtualisation (tel que  **Proxmox VE**) permet d’offrir un stockage mutualisé aux VMs/containers, voire de  _booter_  des machines depuis le NAS. Dans cette section, nous comparons brièvement les solutions NAS populaires et expliquons comment les lier à Proxmox.

  

### **TrueNAS vs Synology vs OpenMediaVault : solutions NAS comparées**

  

**TrueNAS (Core ou Scale)**  – Solution logicielle NAS open-source dérivée de FreeNAS. TrueNAS Core est basé sur FreeBSD, TrueNAS Scale sur Debian Linux, mais les deux utilisent  **ZFS**  en système de fichiers principal. TrueNAS offre une interface web riche et des fonctionnalités avancées (support natif de SMB, NFS, iSCSI, snapshots ZFS, réplication, plugins/VMs sur la version Scale). C’est une solution  **puissante mais un peu complexe**  à maîtriser, orientée utilisateurs techniques. Son point fort est d’embarquer ZFS nativement, avec tout ce que cela implique (intégrité, RAID-Z, etc.) – aucun autre FS n’est proposé. TrueNAS convient à ceux qui veulent tirer le maximum de leur NAS sur du matériel choisi par leurs soins.

  

**Synology DSM**  – Système propriétaire qu’on retrouve préinstallé sur les NAS Synology commerciaux. DSM est réputé pour sa  **convivialité**  et son écosystème logiciel (de nombreuses apps intégrées : gestion photos, vidéos, cloud sync, etc.). Côté stockage, les NAS Synology supportent selon les modèles EXT4 ou Btrfs (sur les modèles + récents, Btrfs apporte les snapshots, la détection de corruption et la compression). Synology propose des  **RAID classiques et propriétaires (SHR)**  pour faciliter l’utilisation de disques de tailles variées. Les protocoles pris en charge incluent SMB/CIFS, NFS, AFP (pour Mac) et iSCSI (souvent sur les gammes Plus/enterprise). Le gros avantage de Synology est le  **“tout-en-un prêt à l’emploi”**  : on branche, on a une interface limpide, et c’est parti. La contrepartie est la dépendance à l’écosystème Synology (matériel dédié, OS fermé). Pour un homelab, un NAS Synology convient si on privilégie la simplicité et la fiabilité éprouvée d’une solution commerciale, quitte à sacrifier un peu de souplesse. D’ailleurs, pour un usage home, Synology DSM a globalement plus d’atouts prêts à l’emploi (simplicité, applis) qu’une solution DIY comme TrueNAS  . Un NAS Synology clé en main est souvent  **le meilleur choix “home use”**  lorsque l’objectif n’est pas de bricoler le hardware ou d’avoir absolument ZFS  . La présence de Btrfs sur de nombreux modèles Synology offre déjà un niveau de sécurité (snapshots locaux, auto-réparation en RAID 1) suffisant pour beaucoup d’usages domestiques.

  

**OpenMediaVault (OMV)**  – Distribution NAS open-source basée sur Debian. OMV fournit une interface web légère pour configurer des services de partage (SMB, FTP, NFS, Rsync, etc.), gérer des utilisateurs et plugins. Contrairement à TrueNAS, OMV n’impose pas ZFS : on peut utiliser EXT4, XFS ou Btrfs sur les volumes (et on peut ajouter le plugin ZFS si on souhaite). OMV se veut  **simple et extensible**  via des plugins (par ex, plugin pour activer iSCSI, plugin pour docker, etc.). C’est sans doute la solution qui se rapproche le plus d’un “NAS maison minimaliste” – idéale pour recycler un vieux PC ou un Raspberry Pi en NAS. Niveau fonctionnalités, de base c’est plus  _sobre_  que TrueNAS ou Synology : pas de snapshots sans plugin (Btrfs ou ZFS), pas de gestion poussée de volumes hors RAID logiciel standard (mdadm). En somme,  **OMV est fonctionnel mais minimal**  en standard, ce qui implique parfois de mettre la main à la pâte ou d’installer des plugins/shell pour des besoins avancés  . Son point fort reste la  **flexibilité**  (on a un Debian sous le capot, donc on peut installer tout ce qu’on veut à côté, y compris Portainer, etc.) et une empreinte relativement légère.

  

**Comparatif rapide** : TrueNAS et OMV sont comparables (solutions libres à installer soi-même), alors que Synology est une solution commerciale complète. TrueNAS mise sur ZFS et l’approche enterprise, OMV sur la simplicité Debian. Synology, de son côté, offre une expérience utilisateur polie mais plus fermée. Si l’on compare TrueNAS vs OMV : TrueNAS est plus  **puissant mais requiert une courbe d’apprentissage plus raide**, OMV est plus simple mais avec moins de fonctionnalités prêtes à l’emploi  . Entre TrueNAS et Synology : Synology l’emporte en  **ergonomie**  et intégration clé-en-main (idéal si vous ne voulez pas “tinkerer”), TrueNAS l’emporte en  **contrôle et performances brutes**(vous choisissez votre matériel, vous avez ZFS et pouvez tout tuner)  . D’après un comparatif,  _“DSM (Synology) a plus d’avantages globalement pour un usage maison, alors que TrueNAS s’adresse à ceux qui veulent construire leur NAS eux-mêmes pour profiter spécifiquement des capacités de ZFS”_  . Enfin, toutes ces solutions supportent les  **protocoles standard**  de partage :  **SMB/CIFS**  (partages Windows),  **NFS**  (partage Unix), et souvent  **iSCSI**  (cible disque en réseau) – sur OMV il faudra un plugin pour iSCSI, TrueNAS et Synology le gèrent nativement. On peut donc s’attendre à pouvoir monter les volumes du NAS sur à peu près n’importe quel système, y compris Proxmox.

  

### **Intégrer un NAS à Proxmox VE (NFS, SMB, iSCSI)**

  

Proxmox VE, la plateforme de virtualisation, permet d’ajouter des stockages externes (dits  _Storage_) pour y loger soit des images de disques VM/CT, soit des ISO, templates ou backups. Un NAS est un candidat idéal pour cela, afin de  **centraliser le stockage**  accessible par plusieurs hôtes Proxmox éventuellement.

  

Les deux méthodes les plus courantes pour attacher un NAS à Proxmox sont  **NFS**  (partage de fichiers en réseau) et  **iSCSI**(partage de bloc disque en réseau). SMB/CIFS est parfois utilisé aussi, mais Proxmox ne propose pas de backend SMB natif dans l’interface (on peut toujours monter un CIFS via fstab ou autre, mais ce n’est pas “first-class citizen”). On privilégiera donc NFS pour du partage de fichiers, ou iSCSI pour présenter un  **disque réseau**.

_Exemple d’ajout d’un stockage NFS dans l’interface Proxmox 
VE._

Dans l’interface web Proxmox, il suffit d’aller dans  **Datacenter -> Storage -> Add -> NFS**  pour déclarer un partage NFS exporté par le NAS. On renseigne l’IP du NAS, le chemin exporté et on choisit le type de contenu (par ex.  VZDump backup files  pour y stocker les sauvegardes, ou  Disk image  pour y mettre des images de VM). Ce montage NFS sera ensuite visible comme un stockage dans Proxmox. L’intérêt est qu’un NFS étant accessible depuis plusieurs nœuds, on peut s’en servir pour  **partager des ISOs**  ou permettre la  **migration de VM**  entre hôtes sans avoir à déplacer leurs disques (si tous les Proxmox montent le même stockage NFS centralisé). Côté NAS, on veillera aux permissions (autoriser l’IP du serveur Proxmox). Un partage NFS est idéal pour stocker des backups ou des modèles de VM, voire des disques de VM si la performance réseau est suffisante.

  

Pour des performances optimales sur des  **disques VM**  however, le  **iSCSI**  est souvent préféré. Le iSCSI permet de présenter un volume de bloc distant que Proxmox traite comme un disque physique. On peut ainsi, sur le NAS, créer une  _LUN iSCSI_  (volume logique) et la connecter à Proxmox (**Add -> iSCSI**  dans l’interface). Proxmox verra alors un disque et pourra soit l’utiliser tel quel pour une VM, soit – plus généralement – le formater en LVM pour y créer de multiples volumes VM. L’approche classique est :  **cible iSCSI**  sur le NAS ->  **Initiateur iSCSI**  côté Proxmox qui se connecte -> on crée un  **Volume Group LVM**  sur la LUN iSCSI -> Proxmox peut y créer des volumes logiques pour chaque VM. Cette configuration offre des performances proches d’un stockage local (surtout en multipath 10 Gb/s, etc.) et convient bien pour des VM exigeantes en IOPS. D’après les retours,  _“iSCSI est meilleur pour les IOPS, tandis que NFS se débrouille bien en débit séquentiel”_  sur des NAS TrueNAS  . Par exemple, pour héberger des  **VMs de base de données**  sur le NAS, iSCSI sera un bon choix.

  

Une autre option, si votre NAS gère ZFS (ex: TrueNAS), est d’utiliser le plugin  **ZFS over iSCSI**  de Proxmox. Ce plugin va en fait orchestrer la création de volumes ZFS (zvol) sur le NAS via SSH, et les exposer en iSCSI. Ainsi, chaque disque de VM correspondra à un  **zvol ZFS**  sur le NAS, bénéficiant des snapshots ZFS, etc.  . Proxmox supporte plusieurs cibles (par exemple istgt pour FreeBSD/TrueNAS Core, ou LIO pour Linux/TrueNAS Scale). À noter toutefois, de base Proxmox ne supporte pas directement l’API TrueNAS, il faut souvent configurer en mode “Manual + target LIO/istgt” ou utiliser des scripts tiers  . L’effort en vaut la chandelle si on veut combiner la  **souplesse de Proxmox**  (créer/supprimer VMs) et les  **avantages ZFS**  côté NAS (cohérence et sauvegardes côté stockage). Veillez dans ce cas à ce que votre NAS ZFS soit fiable et bien sauvegardé, car il devient un point central (une panne du NAS rendrait tous ces volumes indisponibles – penser à la redondance du NAS lui-même !).

  

En résumé,  **intégrer un NAS à Proxmox**  se fait bien via NFS ou iSCSI. Utilisez NFS pour les contenus file (sauvegardes, iso) ou même des VM peu critiques (petits services, lab). Pour des charges plus intensives ou une intégration fine, iSCSI est tout indiqué, éventuellement couplé à LVM ou ZFS-over-iSCSI. Proxmox offre ainsi une grande flexibilité pour tirer parti d’un NAS existant et centraliser le stockage de votre homelab.

  

## **Grille d’aide au choix du stockage**

  

Enfin, comment choisir physiquement ses supports de stockage dans un homelab sécurisé ? Disques durs magnétiques traditionnels (HDD) ou mémoire flash (SSD/NVMe) ? Ou un mélange des deux ? La décision dépend de  **plusieurs critères**  : le  **budget par Go**, les besoins de  **performances**  (IOPS, latence, débit), la  **fiabilité/longévité**  attendue, et bien sûr le  **type de données**  stockées (base de données VM très active, archives media peu accédées, etc.). Le tableau suivant résume les options principales avec leurs atouts et limites, ainsi que des scénarios types où elles brillent.

| **Solution de stockage** | **Coût** **(≈ €/To)** | **Performances** **(débit / IOPS)** | **Longévité & endurance** | **Scénarios d’usage typiques** |
|-----------|-----------|-----------|-----------|-----------|
|**Disque dur (HDD)**|🟢  **Faible**  – Le plus économique par To. Ex: ~15–20 € par To pour des disques de 8–14 To.|🔴  **Faibles IOPS**, latence élevée. Débit séquentiel modéré (~80–160 Mo/s typique pour 7200 tpm)  . Adapté aux gros fichiers, moins aux accès aléatoires rapides.|🟠  **Usure faible**  (pas de limite d’écriture comme les SSD, mais pièces mécaniques sensibles). Durée de vie variable (5–10 ans typiques), attention à la casse mécanique.|Stockage de masse économique :  **archives, sauvegardes, médias**  (films, photos) non critiques en latence. Ex: NAS de sauvegarde ou serveur multimédia (Plex) stockant des vidéos -> un HDD de grande capacité convient.|
|**SSD SATA / NVMe**|🔴  **Élevé**  – Plus cher par Go. Ex: ~100 € par To pour SSD SATA grand public, encore plus pour NVMe haute perf.|🟢  **Très bonnes performances**. Débit SATA 500+ Mo/s, NVMe >3–5 Go/s  . Surtout, IOPS et latences excellents (100× un HDD). Idéal pour accès concurrents, OS et bases de données.|🟡  **Endurance limitée**  : chaque cellule flash supporte un nombre fini d’écritures (TBW donné par le fabricant). Durée de vie ~5 ans grand public, plus en gamme pro. Pas de pièces mobiles (plus robuste aux chocs).|Données nécessitant  **vitesse et réactivité**  :  **disque système**  (VM/OS),  **VMs et containers**  actifs, bases de données, cache d’application. Ex: stockage principal d’un  **serveur de virtualisation**  (Proxmox) -> privilégier un bon SSD/NVMe pour les disques VM afin d’assurer des IOPS élevées aux machines.|
|**Hybride (SSD cache + HDD)**|🟡  **Moyen**  – Combine un SSD (petite capacité) et des HDD (grande capacité). Coût total intermédiaire (on ajoute le prix d’un SSD cache au coût des HDD).|🟠  **Performances mixtes**  : Le SSD sert de  **cache**  en lecture (et parfois en écriture) pour accélérer l’accès aux données « chaudes ». Les lectures fréquentes viennent du SSD (très rapide), le reste sur HDD (lent). En écriture, selon config, le SSD peut absorber puis déstager. Performance  **hétérogène** : excellente sur données en cache, médiocre sur accès direct disque.|🟡  **Endurance variable**  : le SSD cache encaisse beaucoup d’I/O (risque d’usure si utilisation intensive, préférer un SSD endurant). Les HDD derrière subissent moins de sollicitations aléatoires (plutôt séquentielles lors du vidage de cache). L’ensemble reste tributaire des disques mécaniques pour la fiabilité générale.|**Compromis capacitié/performance**  : idéal quand on a beaucoup de données peu souvent accédées, et un sous-ensemble « chaud » fréquemment utilisé. Ex: un  **NAS Plex/Emby**  avec SSD cache pour les index et les vidéos récemment lues, couplé à des HDD pour stocker la vaste bibliothèque. Autre exemple : un  **stockage Ceph**  dans un homelab qui utilise des SSD NVMe comme accélérateurs (DB/WAL) pour des OSD sur HDD.|

**Remarques finales :**  Le choix peut être  **combinatoire**  – par exemple un homelab bien équipé pourra utiliser un  **SSD NVMe comme cache ou tier**  pour accélérer un pool principal en HDD (via L2ARC/ZIL de ZFS, cache SSD sur un NAS Synology, ou bcache sous Linux). On peut aussi adopter une approche par type de données : les données “chaudes” (VM en cours d’exécution, base SQL) sur du SSD rapide, les données “froides” (backups, archives médias) sur du HDD à haute capacité. Pensez également à la  **consommation et au bruit**  : un HDD consomme ~5–10 W et peut être bruyant, contre ~1–2 W silencieux pour un SSD. Dans un homelab à la maison, cela peut compter.

  

Enfin, concernant la  **longévité**, les études montrent que les SSD n’ont plus grand-chose à envier aux HDD en usage courant – leurs taux de panne convergent et dépassent rarement 1–2% par an. Les SSD actuels, s’ils sont dimensionnés correctement en capacité par rapport aux écritures (éviter de remplir à 100% et choisir un modèle avec TBW suffisant), tiendront plusieurs années sans souci. Les HDD, eux, doivent être surveillés (SMART) pour anticiper les secteurs défectueux et les pannes mécaniques. Dans tous les cas, une infrastructure de stockage sécurisée doit accepter qu’un support finira  _forcément_  par faillir : d’où l’importance du  **RAID**  pour la continuité de service, et des  **sauvegardes**  pour la récupération des données. Avec ces bonnes pratiques et un choix judicieux de technologies adapté à vos besoins (et votre budget), votre homelab pourra allier  **performance, capacité et sérénité**  quant à la protection de vos précieuses données.

