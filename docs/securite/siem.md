# **Comparaison Wazuh, ELK Stack et Graylog pour un homelab Blue Team**

  

**Wazuh, ELK et Graylog**  sont trois solutions open source de gestion de logs et de SIEM souvent utilisées par des Blue Teams. Elles couvrent la collecte centralisée de logs (Linux, Windows, réseau), l’analyse en temps réel, la détection d’intrusion et les alertes. Chacune a une architecture et des forces propres : Wazuh est une plateforme HIDS (sur base d’OSSEC) avec agents et détection intégrée,  **ELK Stack**  (Elasticsearch, Logstash, Kibana) est un écosystème de recherche et de visualisation massif, et Graylog est un système de log management avec interface unifiée. Nous détaillons ci-dessous leur architecture, fonctionnalités, cas d’usage, besoins en ressources et recommandations.

  

## **Architecture générale**

_Architecture typique de Wazuh (agents légers sur chaque hôte, serveur central et indexeurs Elasticsearch/OpenSearch) (source : documentation Wazuh)._  Wazuh est basé sur des  **agents installés sur chaque endpoint**  (Windows, Linux, Mac) qui collectent les logs système, fichiers modifiés, événements de sécurité, etc., et les envoient au serveur Wazuh central  . Le serveur Wazuh reçoit ces données, les décode et les analyse avec son moteur de règles IDS/IPS intégré, générant des alertes en cas d’événements suspects. Wazuh gère ensuite l’acheminement de ces événements vers un cluster  **Elasticsearch/OpenSearch**  (appelé « indexer ») via Filebeat et TLS  . La solution comprend aussi une interface web (Wazuh Dashboard) pour configurer et visualiser l’état du serveur et des agents. En version minimale, un seul serveur Wazuh et un nœud indexeur peuvent suffire, mais pour de plus grands volumes ou haute-disponibilité on sépare Wazuh serveur et indexeurs sur des machines distinctes  .

  

En revanche,  **Graylog**  ne repose pas sur ses propres agents. Un déploiement de base (Core Deployment) comprend un ou deux serveurs (« Graylog server ») qui intègrent toutes les fonctions (serveur Graylog, nœud d’indexation Elasticsearch/OpenSearch, et base MongoDB pour les métadonnées)  . Ce schéma « tout-en-un » est simple à installer pour un lab mais peu évolutif. Graylog reçoit des logs via des  _inputs_  configurables : syslog (TCP/UDP), GELF (protocole maison), Beats, JDBC, etc. Les données sont routées en temps réel vers le nœud d’indexation (Elasticsearch/OpenSearch) via des  _streams_  et pipelines de traitement. Pour monter en charge (volumes de logs ou utilisateurs multiples), on peut évoluer vers un déploiement conventionnel ou distribué : les composants Graylog (serveur, nœuds de données, MongoDB) sont répartis sur plusieurs machines, assurant haute disponibilité et scalabilité  .

  

**Elastic Stack (ELK)**  a une architecture modulaire : Elasticsearch forme un cluster de nœuds pour stocker et rechercher les logs (indices distribués), Logstash est un pipeline de transformation flexible, et Kibana est l’interface web. En pratique, on utilise souvent  **Beats**  (Filebeat, Winlogbeat, etc.) comme agents légers côté hôte pour expédier directement les logs vers Elasticsearch (voire via Logstash pour enrichissement)  . Il n’y a pas de  _serveur SIEM_  unique dans ELK, mais on peut imaginer un schéma similaire : plusieurs nœuds Elasticsearch (avec répartition en shards/réplicas), au moins un Logstash pour prétraitement, et Kibana pour la visualisation. Dans un homelab, on peut se contenter d’un seul nœud Elastic et un Kibana. Un schéma Docker pré-packagé (comme [deviantony/docker-elk]  ) facilite l’installation expérimentale.

  

**Tableau 1. Comparaison de l’architecture et des composants**

TABLEAU

## **Collecte de logs et agents**

-   **Wazuh**  utilise ses  **agents légers**  multi-plateformes pour recueillir logs système, journaux de sécurité, changements de fichiers, etc. en continu. Les agents chiffrent les données (AES-128/256 par défaut  ) et ouvrent une connexion sécurisée (port 1514/TCP) vers le serveur Wazuh. Ils envoient tous les événements, qui sont ensuite décodés (analyse de texte, règles) sur le serveur. Wazuh peut aussi collecter de manière  _agentless_  (par ex. via syslog UDP/TCP ou SSH pour équipements réseau)  .
    
-   **ELK Stack**  ne fournit pas d’agent propriétaire, mais repose sur  **Beats**  (Filebeat pour logs texte, Winlogbeat pour journaux Windows, Metricbeat, etc.) ou Logstash pour récupérer les logs. En homelab, on déploie souvent des Filebeat/Winlogbeat sur les machines cibles afin d’expédier les logs vers Elasticsearch. On peut aussi configurer Logstash ou syslog pour écouter des flux. La flexibilité de Logstash permet d’enrichir ou filtrer les données, même si beaucoup d’utilisateurs optent pour des alternatives plus légères (Fluentd, Vector)  .
    
-   **Graylog**  ne requiert pas d’agent exclusif. Il accepte les logs en entrée via divers protocoles : syslog (UDP/TCP), GELF (Graylog Extended Log Format), Beats, Kafka, etc. Une méthode courante est d’utiliser le « Graylog Sidecar » pour configurer et lancer automatiquement Filebeat/NxLog selon Graylog. L’interface Graylog permet de créer des  _Inputs_  pour chaque protocole, puis de router ces messages vers des  _Streams_  qui filtrent et taggent les données en temps réel  .
    

  

Ainsi, en  **homelab**, on pourra utiliser par exemple :

-   Wazuh Agent sur les serveurs Linux/Windows et Linux : collecte de logs / HIDS (détection d’intrusion).
    
-   Filebeat/Winlogbeat directes vers Elastic ou Graylog (plus générique)  .
    
-   Syslog/Fluentd pour les équipements réseau ou appareils moins compatibles avec un agent.
    

  

## **Détection, analyse et tableaux de bord**

-   **Wazuh**  est avant tout un HIDS : il analyse les logs des agents à la recherche de signatures d’attaque, modifications non autorisées (file integrity monitoring), rootkits, vulnérabilités connues, etc. Il embarque des milliers de règles prédéfinies (CVE, MITRE, directives de sécurité) et permet d’écrire ses propres décodeurs/règles  . Lorsqu’une règle déclenche, Wazuh génère une alerte priorisée. Les résultats sont stockés en archives compressées (JSON ou texte) signées par SHA256 pour intégrité  , et indexés dans Elasticsearch pour recherche via le tableau de bord. Le  **dashboard Wazuh**  (plugin OpenSearch/Kibana) donne une vue globale : statut des agents, inventaire de vulnérabilités, alertes récentes, conformité réglementaire, etc. L’analyse est centrée sur la sécurité (alertes critiques) et la conformité (CIS, PCI-DSS, NIST)  .
    
-   **ELK Stack**  est conçu d’abord pour la recherche et la visualisation de logs.  **Elasticsearch**  est un moteur full-text distribué très rapide pour indexer et rechercher de gros volumes de données  .  **Kibana**  fournit des visualisations et tableaux de bord interactifs personnalisables (histogrammes, heatmaps, séries temporelles…). Cependant, dans sa version de base  **open source**, ELK n’inclut pas de moteur de corrélation ou d’alerting spécifique à la sécurité. La détection d’anomalies est possible avec  **Elastic Security**  (anciennement « SIEM ») et machine learning, mais ces fonctionnalités avancées sont en général liées à une licence payante. Néanmoins, on peut mettre en place des règles de détection via les  **Alertes Kibana**  (Watchers) ou la bibliothèque de détection GitHub d’Elastic  , voire utiliser OpenSearch qui offre certaines capacités de sécurité (authentification, chiffrement) en OSS  . En pratique, l’ELK pur est surtout utile pour le «  _search hunting_  » et la création de dashboards métier, plutôt que pour l’IDS automatisé.
    
-   **Graylog**  combine les deux approches : c’est une plateforme d’agrégation de logs avec de nombreuses fonctions orientées sécurité. Son interface permet des recherches rapides (requêtes Lucene) et la création de tableaux de bord et graphiques en quelques clics. On utilise les  **Streams**  pour filtrer/segmenter les logs au fur et à mesure, et les  **Alerts**  pour déclencher des notifications (email, Webhook, etc.) dès qu’un critère est satisfait. Par exemple, on peut alerter sur plusieurs échecs d’authentification, détection d’IP suspecte, ou tout indicateur définissable. Contrairement à Wazuh, Graylog n’apporte pas de règles de détection prêtes à l’emploi : la logique d’alerte doit être configurée manuellement (ou via des plugins Sigma en Enterprise  ). En revanche, il peut ingérer et  **corréler**  des logs de plusieurs sources (serveurs, firewall, IDS) pour les analystes. Graylog Security (édition payante) ajoute même de la détection automatique d’anomalies et du pattern matching avancé  , mais en usage domestique la version open suffit souvent pour centraliser et surveiller les logs.
    

  

**Tableau de bord et alerting** : Wazuh Dashboard et Kibana offrent des vues prêtes à l’emploi axées sécurité et conformité, avec rôle API pour modifications à distance  . Graylog propose un UI Web unifiée (port 9000) où l’on crée facilement dashboards, recherche et règles d’alerte  . Ces interfaces permettent aux débutants de démarrer rapidement. Les trois solutions supportent la visualisation en temps réel de données et un historique, bien que le dimensionnement du cluster (index) conditionne le volume d’historique accessible sans purger.

  

## **Cas d’usage typiques en homelab**

  

Dans un laboratoire Blue Team domestique, on recherchera généralement :

-   **Collecte centralisée de logs**  : agréger les journaux de tous les serveurs Linux/Windows, équipements réseaux (firewall, routeur) et VM. Par exemple, installer Wazuh Agent sur les PC Windows pour remonter les Event Logs, déployer Filebeat sur des VMs Linux pour leurs  /var/log, ou envoyer tout via syslog/Graylog.
    
-   **Détection d’intrusion (HIDS/NIDS)**  : surveiller les anomalies sur les hôtes. Wazuh fait figure de HIDS classique (fichiers modifiés, connexions suspectes, analyse des logs); ELK/Graylog peuvent ingérer aussi les alertes d’un NIDS (Suricata, Snort) et les corréler  . Par exemple, les logs d’IDS network Suricata peuvent être envoyés à Wazuh ou Graylog pour enrichissement et corrélation (Wazuh prend nativement en charge Suricata via un module  ).
    
-   **Alerting**  : générer des notifications (mail, Slack, etc.) sur événements critiques : Wazuh peut envoyer des alertes par mail ou Syslog quand une règle se déclenche. Graylog déclenche aussi des alertes définies sur des  _streams_. Kibana (ELK) permet d’utiliser des alertes Watcher ou des plugins d’action dans OpenSearch.
    
-   **Monitoring temps réel**  : visualiser l’activité du réseau et des hôtes via des dashboards. Par ex. tableaux Graylog pour la répartition des logs par serveur/processus, courbes de charge ou heatmaps. Elastic/Kibana propose par ailleurs des plugins comme Elastic SIEM ou Observability pour surveiller l’infrastructure (APM, métriques).
    

  

Ces cas d’usage sont couverts par les trois outils, mais avec des approches différentes. Dans un petit homelab, on peut par exemple utiliser Wazuh pour la détection hôte (HIDS) et également envoyer ses journaux au cluster ELK ou Graylog pour analyse globale, tirant ainsi parti du meilleur des deux mondes.

  

## **Installation Docker/LXC et architecture type**

  

En environnement domestique, l’usage de conteneurs (Docker, LXC) facilite le déploiement :

-   **Wazuh**  : la documentation officielle propose des images Docker composant le « full-stack » Wazuh (serveur, indexer, tableau de bord)  . Une simple  docker-compose  récupère tout en une fois. Le dépôt  [wazuh-docker](https://github.com/wazuh/wazuh-docker)  fournit notamment un stack prêt à l’emploi (Wazuh Manager + OpenSearch + Dashboard)  . Pour un seul serveur Wazuh et un nœud OpenSearch, une VM ou LXC avec ~6 Go de RAM est recommandée  . Des templates Proxmox (LXC) pour Wazuh existent dans certaines communautés.
    
-   **ELK**  : il existe de nombreux Docker Compose tout-en-un (par ex. [deviantony/docker-elk]  ) incluant Elasticsearch, Logstash et Kibana. On peut lancer  docker-compose up  pour disposer d’un cluster minimal à 1 nœud. En LXC, il suffit d’installer Docker ou directement Elastic sur Debian/Ubuntu. Un notebook Raspberry Pi ou petit serveur (≥4 Go) peut ainsi faire tourner une instance de test (certains revendeurs fournissent des images Debian déjà configurées).
    
-   **Graylog**  : Graylog fournit une image Docker officielle (« graylog/graylog »). Un exemple de  docker-composeintègre 3 conteneurs : MongoDB (pour les métadonnées), Elasticsearch/OpenSearch (pour le stockage), et Graylog Server  . Le serveur Graylog écoute par défaut sur le port 9000 (web UI) et sur divers ports syslog (1514 TCP/UDP, 12201 GELF, etc.) comme montré ci-dessus  . En LXC, on installe Java 11, MongoDB et Elasticsearch avant Graylog server. Un minimum de  **4 Go de RAM**  est conseillé pour un petit setup Graylog  .
    

  

**Exemple d’architecture Docker pour Graylog (cluster minimal)** : MongoDB + Elasticsearch (chaque container à ~1 Go RAM) + Graylog (<1 Go) dans un réseau bridge. Graylog expose 9000, 12201, 1514, etc. (voir exemple [31†L182-L190]).

  

## **Performances et ressources**

  

Ces plateformes sont gourmandes en I/O disque et RAM, particulièrement à mesure que le volume de logs croît. Les bonnes pratiques sont similaires pour toutes : utiliser des SSD rapides, allouer suffisamment de mémoire et prévoir la montée en charge.

-   **Mémoire** : Wazuh (avec OpenSearch) demande au moins  **6 Go RAM**  sur l’hôte Docker complet  . Graylog conseille  **4 Go minimum**  pour un petit usage, mais  **8–16 Go**  pour la production  . Elasticsearch (ELK) peut s’exécuter sur 2–4 Go pour un usage très léger, mais 8–16 Go voire plus sont nécessaires pour un cluster de taille moyenne, en raison du heap Java. Un indice clé est que l’Elastic Stack nécessite de favoriser la RAM (environ 50% dédiée au heap Java) pour des recherches rapides. Comme le note Matt Hayes, “2 Go de RAM, c’est le minimum, 4 Go fonctionnera mieux” pour un ELK minimal  . De son côté, Wazuh recommande un calcul de  vm.max_map_count=262144  pour OpenSearch (nécessaire pour le mappage mémoire)  .
    
-   **CPU** : Elasticsearch est parallélisé sur plusieurs cœurs pour l’indexation et les recherches full-text. Logstash est connu gourmand en CPU et mémoire lors de fortes ingestations (selon ses pipelines). Graylog (Java + Elasticsearch) tire avantage de plusieurs cœurs aussi. En général, prévoir au moins 2 vCPU pour chaque nœud (4 CPU pour cluster ELK plus Wazuh).
    
-   **Stockage** : tous conseillent l’usage de  **SSDs/ NVMe**. Par exemple, la doc Graylog souligne l’importance de stocker le journal de messages et les indices sur des disques haute-IOPS (idéalement SSD)  . Les logs étant volumineux et en écriture continue, mieux vaut dimensionner le disque en fonction de la rétention souhaitée (par ex.  [ingest quotidien] × [jours de rétention] × 1.2)  . Wazuh archive quotidiennement ses logs avec compression et checksums  , mais conserve aussi tout dans Elasticsearch si la rétention est long-terme.
    
-   **Scalabilité** : toutes les solutions se  **clusterisent**  pour gérer plus de données. Wazuh peut déployer un cluster de serveurs Wazuh (master/worker) et un cluster OpenSearch pour l’indexation  . Graylog (Community) permet d’ajouter des Graylog Servers supplémentaires derrière un load balancer, chacun traitant les mêmes inputs, tandis que Elasticsearch se met en cluster pour les données  . Ce déploiement conventionnel assure HA et montée en charge facile  . Elastic/Opensearch est conçu pour scaler horizontalement ; on peut commencer sur un seul nœud mais passer à 3+ nœuds de données avec réplicas pour la résilience.
    
-   **Sécurité** : Wazuh chiffrera nativement la communication agent-serveur (AES)  , et l’API/GUI utilisent TLS et authentification. Graylog lui recommande de forcer TLS entre les nœuds et d’appliquer le RBAC (contrôle d’accès par rôles) pour sécuriser l’accès  . Elasticsearch (dans son offre OSS) offre maintenant des fonctionnalités basiques de sécurité sur Basic License (auth, HTTPS) ; la version OpenSearch (fork AWS) intègre chiffrement et ACL sans licence supplémentaire  .
    
-   **Maintenance** : la gestion consiste à surveiller les disques (purger/archiver les indices anciens), sauvegarder la base MongoDB (Graylog) et les snapshots d’Elasticsearch. Graylog souligne l’importance de sauvegarder Mongo, les indices OpenSearch et la config  . Wazuh préconise aussi de purger/archiver les anciens logs (ossec-archive-*.gzsignés) ou de ne compter que sur les snapshots d’Elasticsearch  . Les mises à jour impliquent souvent de mettre à jour en séquence les composants (par ex. Elasticsearch puis Wazuh). En homelab, l’automatisation (Ansible, Docker Compose) aide à réduire la charge opérationnelle  .
    

  

## **Recommandations par profil d’usage**

-   **Débutant / homelab simple**  : préférez la solution la plus  **clé-en-main**. Graylog (édition Open) est simple à installer et dispose d’une interface intuitive pour démarrer. Le Docker Compose officiel permet de monter un système complet en quelques commandes. ELK est aussi abordable via un stack Docker (deviantony/docker-elk), mais Elastic demande un peu plus d’ajustements (certificates, mémoire). Wazuh, bien que puissant en détection, nécessite plus de configuration initiale (déploiement des agents, règles) – il peut être mis en place après familiarisation ou dans un second temps.
    
-   **Power user / expérimenté**  : si vous visez de la détection HIDS avancée, Wazuh est un excellent choix car il offre dès le départ des capacités de threat hunting (FIM, règles CVE/CIS)  . Vous pouvez associer Wazuh à une plateforme log (Graylog ou ELK) pour l’analyse forensique des alertes. ELK (ou son fork OpenSearch) convient pour manipuler et visualiser de gros volumes de données, écrire ses propres règles (Sigma/Kibana Alerts) ou utiliser du machine learning Elastic. Graylog est quant à lui très flexible pour construire des flux et alertes personnalisées via son système de  _streams_.
    
-   **Usage professionnel / grand volume**  : en production, les trois s’industrialiseront en clusters. Graylog Enterprise offre des fonctions SIEM avancées (correlation, archiving) et support commercial. Wazuh peut gérer des milliers d’agents sur un cluster, avec indexeurs répliqués  . L’Elastic Stack, couplé à Elastic Security, reste une référence « tout-en-un », mais sa licence propriétaire récente implique souvent de migrer vers OpenSearch ou la version Enterprise pour rester open source  .
    

  

En résumé,  **pour un homelab Blue Team**  débutant, Graylog (simple à prendre en main) ou le combo  _Wazuh+Graylog_(Wazuh pour l’IDS, Graylog pour la log-analyse) est souvent conseillé. Les utilisateurs avancés pourront opter pour Wazuh ou Elastic+Wazuh (via le plugin Wazuh pour Kibana) pour profiter de règles de détection déjà disponibles. Enfin, l’essentiel est de disposer d’une collecte de logs fiable et d’un minimum d’alerting ; les trois outils remplissent ces fonctions, à chacun de choisir selon ses préférences en terme d’interface, de langage de requête et de charge système.

