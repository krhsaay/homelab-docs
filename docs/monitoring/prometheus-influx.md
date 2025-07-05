# **Collecte et stockage des métriques : Prometheus vs InfluxDB**

  

Le  **homelab**  d’un ingénieur en cybersécurité requiert une solution de monitoring fiable et performante. Prometheus et InfluxDB sont deux bases de données temporelles (TSDB) populaires, chacune avec son modèle et ses forces propres. Cette documentation compare  **techniquement**  Prometheus (pull-based, TSDB) et InfluxDB (push-based, TSM), leur intégration avec les outils courants (Grafana, Home Assistant, MQTT, etc.), ainsi que des cas pratiques et une architecture type pour un homelab sécurisé. Des exemples de configurations Docker Compose, de bons usages de sécurité (réverse proxy, VLAN), et un tableau comparatif final sont fournis.

  

## **1. Comparaison technique Prometheus vs InfluxDB**

-   **Performance et cardinalité**  : Prometheus est conçu pour des environnements  **cloud-native et conteneurisés**, avec un modèle de collecte  **par pull**. Il gère efficacement les métriques à  **haute cardinalité**. InfluxDB, utilisant un modèle  **push**  (via Telegraf ou API), offre souvent un  **débit d’écriture**  élevé mais peut peiner avec des séries extrêmement nombreuses. Autrement dit, Prometheus facilite les requêtes agrégées sur des millions de séries (ex.  _histogram_quantile_  en PromQL), tandis qu’InfluxDB peut mieux absorber un flot intensif de données IoT.
    
-   **Stockage**  : Prometheus utilise son  **TSDB**  interne : les données sont regroupées en  **blocs de 2 heures**, indexés et compressés. Prometheus stocke en moyenne seulement  **1–2 octets par échantillon**. Un journal d’écriture (WAL) assure la durabilité jusqu’à la persistance dans les segments compactés. La rétention est gérée par les flags  --storage.tsdb.retention.time/size  (par défaut 15 jours). InfluxDB, quant à lui, utilise une architecture WAL +  **TSM (Time-Structured Merge Tree)**. Les points sont d’abord écrits dans un WAL puis flushés en mémoire cache et finalement vers des fichiers TSM compressés par séries (stockant les  **deltas**  des séries en format colonne). InfluxDB prend en charge nativement les  **rétention policies**  (ex.  CREATE RETENTION POLICY "one_year" ON "mydb" DURATION 52w...) et les  **continuous queries**  pour downsampling automatique. Par exemple :

```sql
CREATE RETENTION POLICY "one_year" ON "mydb" DURATION 52w REPLICATION 1 DEFAULT
CREATE CONTINUOUS QUERY "cq_30m" ON "mydb"
  RESAMPLE EVERY 30m FOR 1d
  BEGIN
    SELECT mean("value") INTO "downsampled"."cpu_load"
    FROM "cpu_load" GROUP BY time(1m), *
  END
```

-   Cette souplesse permet à InfluxDB d’archiver  **long terme**  avec agrégation, contrairement à Prometheus qui efface simplement les vieux blocs.
    
-   **Compression**  : Les deux moteurs appliquent des techniques de compression par défaut. Prometheus utilise l’algorithme  **Gorilla**  pour les séries numériques, et peut compresser le WAL avec Snappy. InfluxDB compresse chaque bloc TSM indépendamment : les timestamps et valeurs sont codés différemment (delta encoding, Simple8b, Snappy, XOR…) selon le type (float, int, bool, string). Le stockage en colonnes par série et l’usage de deltas améliorent l’efficacité de stockage d’InfluxDB pour des séries régulières.
    
-   **Flexibilité des requêtes**  : Prometheus utilise  **PromQL**, un langage fonctionnel conçu pour les séries étiquetées (labels). Il excelle dans les requêtes temps réel, les agrégations multi-dimensions, les joins implicites (opérations booléennes sur séquences), et les fonctions d’histogrammes ou de dérivées. InfluxDB propose  **InfluxQL**  (SQL-like, historique v1) et  **Flux**  (langage scriptable plus récent). Flux permet des transformations avancées (joins explicites, windowing, math), mais impose souvent plus de configuration. PromQL ne supporte que des valeurs numériques (float64) pour la plupart des opérateurs, InfluxQL/Flux peut gérer divers types (floats, int, bool, string).
    
-   **Cas d’usage typiques**  : En résumé, Prometheus est parfait pour la surveillance en temps réel d’infrastructures modernes : il intègre la découverte de services (Kubernetes, Consul, etc.), l’alerting natif via Alertmanager, et de nombreux  _exporters_  (Node Exporter, cAdvisor, Blackbox, etc.). InfluxDB brille dans les  **données IoT et capteurs**(ex. suivi d’énergie, températures, séries temporelles industrielles) grâce à son modèle push et ses capacités d’agrégation/de durée. Par exemple, InfluxDB est souvent cité pour les déploiements  **Smart Home**  et capteurs longue durée, alors que Prometheus est le choix par défaut en Cloud / containers.
    
-   **Alerting**  : Prometheus dispose d’un système d’alerte intégré (_Alertmanager_) facile à configurer pour envoyer des notifications (email, Slack, etc.) basées sur des règles PromQL. InfluxDB, historiquement (v1), s’appuyait sur  **Kapacitor/Chronograf**  pour les alertes, ce qui peut être plus lourd à configurer. Aujourd’hui, les utilisateurs d’InfluxDB ont souvent recours à  **Grafana Alerting**  ou à des solutions externes. Dans tous les cas, les deux solutions peuvent générer des alertes, mais la pile Prometheus est  **native**  (Prometheus → Alertmanager) tandis qu’InfluxDB nécessite en général un composant supplémentaire ou l’utilisation de Grafana.
    

  

## **2. Intégration dans un homelab**

  

Un homelab de cyber-sécurité requiert la surveillance de multiples composants et capteurs. Voici comment Prometheus et InfluxDB se branchent sur les outils courants :

-   **Grafana**  : Indispensable pour visualiser les métriques. Grafana supporte  **nativement Prometheus et InfluxDB**comme sources de données. On peut construire des dashboards graphiques complexes, utiliser des  **variables de template**  pour itérer sur des hôtes ou interfaces, et déclencher des alertes via Grafana Alerting. Par exemple, un dashboard Proxmox peut agréger  node_cpu_seconds_total  (Prometheus) ou  cpu_load  (Influx) au même endroit. Grafana facilite aussi le paramétrage (templating) des requêtes et des alertes cross-source.
    
-   **Home Assistant (HA)**  : HA est souvent cœur d’un homelab domotique. Il offre une intégration  **Prometheus**  qui expose un endpoint  /api/prometheus  pour que Prometheus scrappe les états des entités HA  . InfluxDB dispose aussi d’une intégration (ou add-on) : HA peut  **pousser**  ses données dans InfluxDB via  influxdb:  dans  configuration.yaml. Exemple de configuration pour HA → InfluxDB :

```yaml
influxdb:
  api_version: 2
  host: 192.168.1.190
  port: 8086
  token: VOTRE_TOKEN
  organization: MON_ORG
  bucket: mesures_domotique
  tags:
    source: HA
```

-   Ainsi, on exploite HA comme source de données pour l’un ou l’autre système. InfluxDB est souvent privilégié pour les  **graphiques longue durée**  (énergie, température), tandis que Prometheus peut être utilisé pour des alertes instantanées (ex. batterie faible).
    
-   **Zigbee2MQTT et capteurs domotiques (Shelly, Zigbee)**  : Ces solutions transmettent leurs mesures via MQTT. Avec InfluxDB, on peut utiliser  **Telegraf**  (plugin  mqtt_consumer) ou un  _exporter MQTT_  pour récupérer les topics et écrire dans InfluxDB. Ce blog montre bien comment capter des mesures Shelly sur MQTT et les stocker en Influx 2.6 pour Grafana. Avec Prometheus, il n’existe pas d’endpoint Prometheus natif dans Zigbee2MQTT (sujet en discussion GitHub), on peut donc utiliser Telegraf  _vers_  Prometheus : le plugin  inputs.mqtt_consumer  de Telegraf lit les messages Zigbee2MQTT en JSON, puis le plugin  outputs.prometheus_client  les expose sur un endpoint  /metricspour que Prometheus les scrap. En résumé, capteurs Zigbee/Powermètre (Shelly) → MQTT → (Telegraf) → InfluxDB/Prometheus.
    
-   **Frigate (NVR IA)**  : Frigate expose une API  /api/stats  fournissant CPU, mémoire, FPS, corail, stockage, compteurs d’événements, etc. Il existe un exporter Prometheus dédié (par ex.  [bairhys/prometheus-frigate-exporter](https://github.com/bairhys/prometheus-frigate-exporter)). Ce container interroge Frigate et met à disposition ces métriques en  /metrics. Il fournit notamment la vitesse d’inférence, l’utilisation CPU/RAM de Frigate, le FPS caméra, la température du Coral, etc.. On fera donc pointer Prometheus vers cet exporter pour collecter les métriques Frigate. InfluxDB n’a pas d’intégration directe, mais on peut remonter ces mêmes infos via Telegraf (inputs.http  ou script) ou via l’API  _remote write_  de Influx (InfluxDB 2.0 accepte aussi le protocole Prometheus).
    
-   **Docker et hôtes Proxmox**  : Sur chaque nœud (Proxmox ou serveur Linux), on installe  **node_exporter**  pour les métriques système (CPU, RAM, disques, réseau) et  **cAdvisor**  (ou Docker daemon metrics) pour les conteneurs Docker. Ces exporters sont scrappés par Prometheus. Avec InfluxDB, Telegraf prend le relais : on peut activer  [[inputs.cpu]],  [[inputs.mem]],  [[inputs.disk]],  [[inputs.net]],  [[inputs.docker]], etc., pour récolter les mêmes données (voir exemple de configuration Telegraf ci-dessous). Grafana affichera en aval ces données à partir d’Influx ou de Prometheus, avec les mêmes dashboards (il suffit de configurer la source de données).
    
-   **Proxmox (VE)**  : Proxmox n’expose pas nativement de métriques Prometheus, mais il existe plusieurs outils. Par exemple, le projet  _prometheus-pve-exporter_  récupère via l’API Proxmox l’état des VMs/CTs, ressources, statut cluster, etc., pour Prometheus. Pour InfluxDB, Telegraf dispose d’un plugin  inputs.proxmox  qui interroge l’API de Proxmox et envoie les stats (CPU, mémoire, IOPS, etc.) vers Influx. Ainsi, que vous utilisiez Prometheus ou InfluxDB, il existe des solutions (exporter ou plugin) pour intégrer Proxmox dans le monitoring.
    

  

## **3. Architecture type**

  

En pratique, on peut déployer  **deux stacks distinctes**  ou une architecture mixte. Les concepts clés :

-   **Exporters Prometheus**  : Petits services HTTP exposant des métriques au format Prometheus. Exemples :  node_exporter  (host hardware),  cadvisor  (Docker),  blackbox_exporter  (ping/HTTP),  mqtt_exporter/Telegraf Prometheus (MQTT),  zabbix_exporter, etc. Prometheus scrappe périodiquement ces endpoints et stocke les données dans sa TSDB. L’Alertmanager (optionnel) gère les notifications.
    
-   **Agents Telegraf pour InfluxDB**  : Telegraf est un agent « plugin-based » qui pousse les données vers InfluxDB. Il lit les métriques système (inputs.cpu,  inputs.disk,  inputs.docker), IoT (inputs.mqtt_consumer,  inputs.modbus, etc.), ou même Prometheus (inputs.prometheus  plugin) et écrit dans InfluxDB (v2). InfluxDB OSS ou Cloud stocke les séries, et Grafana (ou Chronograf) sert de couche graphique.
    
-   **Politique de rétention**  : On définit des rétentions (durées de stockage). Prometheus purge au-delà de  retention.time. InfluxDB gère les politiques par base/bucket, et on peut configurer des TTLs en flux. Des downsamplings (CQ/Tasks) réduisent la granularité historique.
    
-   **Authentification & Sécurité**  : En mode « homelab sécurisé », on met les services de monitoring sur un réseau de gestion isolé (VLAN dédié). Les exporters sur les VLAN IoT (capteurs) ou Docker peuvent être accessibles uniquement en internal. Tous les accès web (Grafana, InfluxUI, Prom UI, Alertmanager) passent par un reverse proxy (Nginx, Traefik) en HTTPS avec authentification forte (LDAP/htpasswd). On évite d’exposer les endpoints  :9100/metrics  (node_exporter) ou  :9103/metrics  (cadvisor) en externe, sauf éventuellement via VPN.

_Figure 1 : Exemple de schéma d’architecture Prometheus (modèle pull) avec exporters et Alertmanager._

_Figure : Architecture typique Prometheus – Prometheus scrappe les exporters (node_exporter, cAdvisor, etc.) et envoie les alertes à Alertmanager, le tout visualisé dans Grafana._

  

En complément, un  **schéma ASCII**  sommaire d’une architecture mixte pourrait être :

```ascii
            ┌───────────┐
            │  Grafana  │
            └───▲───┬───┘
                │   │
         ┌──────┴─┐ └────────┐
         │Prometheus│         │
         └──▲────┬──┘         │
            │    │           │
            │  ┌─┴────────┐  │
            │  │ Alertman- │  │
            │  │  ager     │  │
            │  └───────────┘  │
┌────────┬───┴───┴──┐   ┌────┴────┐
│ node   │ cadvisor │   │ mqtt    │
│ exporter│         │   │ exporter│
└────────┴──────────┘   └─────────┘
   (hôtes)               (IoT)       

┌────────┐    ┌────────────────┐ 
│Telegraf│    │   InfluxDB     │ 
│(Linux, │    │ (TSM engine)   │ 
│ Docker)│    └────▲─────┬─────┘
└──┬─────┘         │     │
   │               │     │writes       
   │               │     │       
   │           ┌───┴───┐ │
   │           │Cache  │ │
   │           └───┬───┘ │
   │               │     │
   │           ┌───┴───┐ │
   └──────────▶│ WAL   │ │
               └───┬───┘ │
                   │     │
                   │ ┌───┴───┐
                   │ │ TSM   │
                   │ └───┬───┘
                   │     │
                   │Compression
                   ▼
```


Dans ce diagramme, Prometheus  **scrape**  régulièrement les exporters (node_exporter,  cadvisor,  mqtt_exporter, etc.) pour remplir sa TSDB, tandis que Telegraf  **push**  les données vers InfluxDB qui les stocke dans le WAL puis dans des fichiers TSM compactés. Grafana interroge ensuite Prometheus  _et/ou_  InfluxDB pour afficher les tableaux de bord.

  

## **4. Cas d’usage pratiques**

-   **Surveillance de l’infrastructure**  : On suit l’état des VMs (Proxmox, LXC, QEMU), du réseau, et du hardware. Exemples : CPU et mémoire des nœuds (via Node Exporter ou Telegraf), trafic réseau (via SNMP_exporter ou inputs.net), statut des VM (via proxmox-exporter ou inputs.proxmox). Grafana propose des dashboards préfabriqués pour Proxmox, Docker, etc. Des alertes communes : manque de mémoire (node_memory_availablebas), disque presque plein (node_filesystem  >90%), ou container bloqué (perte de heartbeat).
    
-   **Monitoring d’énergie et domotique**  : Avec des capteurs Shelly (compteurs d’énergie, interrupteurs) ou des sondes Zigbee (température, humidité), on envoie les données via MQTT. Par exemple, un Shelly EM peut publier sa consommation sur MQTT, et un agent Telegraf (plugin MQTT) écrit cette donnée dans InfluxDB. Dans Grafana on trace l’évolution de l’énergie. InfluxDB est adapté car on peut stocker sur des années avec agrégation. Prometheus est moins naturel ici (les données évènementielles via MQTT demandent un pont push/pull) – souvent on utilise Telegraf → Prometheus proxy si nécessaire.
    
-   **Alertes température / conso CPU**  : On peut surveiller la température CPU du serveur ou la consommation électrique. Par exemple, installer  lm-sensors  sur un hôte Proxmox et configurer Telegraf pour lire ces capteurs, puis déclencher une alerte si  sensors_core_temp > 80°C. Prometheus peut également récupérer la température via Node Exporter (with  sensors  enabled) et un rule PromQL (ALERT CPUHighTemp IF node_hwmon_temp_celsius > 80). Pour la consommation, on peut déclencher si la moyenne des Watts (d’un Shelly) dépasse un seuil sur 5 min, avec une règle Alertmanager ou alert Grafana.
    

  

Chaque cas d’usage met en lumière le choix : Prometheus pour de l’**alerting réactif**  et du monitoring système, InfluxDB pour de la  **collecte IoT/longue durée**. Dans un homelab, on peut utiliser les deux de manière complémentaire (voir tableau ci-dessous).

  

## **5. Instructions d’installation**

  

On installe généralement ces stacks via  **Docker Compose**  pour simplicité. Ci-dessous deux exemples de  docker-compose.yml  simplifiés.

  

**Prometheus Stack**  (Prometheus + Alertmanager + Grafana) :

```yaml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:v2.46.0
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"
    command:
      - "--storage.tsdb.retention.time=15d"
      - "--web.enable-lifecycle"
  alertmanager:
    image: prom/alertmanager:v0.25.0
    container_name: alertmanager
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/config.yml
    ports:
      - "9093:9093"
  grafana:
    image: grafana/grafana:10.0.0
    container_name: grafana
    environment:
      - "GF_SECURITY_ADMIN_PASSWORD=MotDePasseFort"
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
volumes:
  prometheus-data:
  grafana-data:
```

-   prometheus.yml doit lister les targets (ex. node_exporter:9100, cadvisor:8080, mqtt_exporter:9255, etc.).
    
-   alertmanager.yml  configure les routes d’alerte et receveurs (email/Slack).
    
-   On mappe des volumes pour persister les données (/prometheus,  /var/lib/grafana).
    

  

**InfluxDB Stack**  (InfluxDB2 + Telegraf + Grafana) :

```yaml
version: '3.8'
services:
  influxdb:
    image: influxdb:2.8
    container_name: influxdb
    volumes:
      - influxdb-data:/var/lib/influxdb2
    environment:
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_USERNAME=admin
      - DOCKER_INFLUXDB_INIT_PASSWORD=MotDePasseSecurise
      - DOCKER_INFLUXDB_INIT_ORG=my-org
      - DOCKER_INFLUXDB_INIT_BUCKET=mybucket
      - DOCKER_INFLUXDB_INIT_RETENTION=720h
    ports:
      - "8086:8086"
  telegraf:
    image: telegraf:1.40
    container_name: telegraf
    volumes:
      - ./telegraf.conf:/etc/telegraf/telegraf.conf:ro
    depends_on:
      - influxdb
  grafana:
    image: grafana/grafana:10.0.0
    container_name: grafana_influx
    environment:
      - "GF_SECURITY_ADMIN_PASSWORD=AutreMotDePasse"
    ports:
      - "3001:3000"
    volumes:
      - grafana2-data:/var/lib/grafana
volumes:
  influxdb-data:
  grafana2-data:
```

-   InfluxDB 2 en mode  _setup_  crée org, bucket, utilisateur à la première exécution.
    
-   Telegraf pousse les métriques : le fichier  telegraf.conf  contient par exemple :

```toml
[[outputs.influxdb_v2]]
  urls = ["http://influxdb:8086"]
  token = "GENERE_TOKEN_INFLUX"
  organization = "my-org"
  bucket = "mybucket"
[[inputs.cpu]]
  percpu = true
  totalcpu = true
[[inputs.mem]]
[[inputs.disk]]
[[inputs.net]]
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
[[inputs.mqtt_consumer]]
  servers = ["tcp://mqtt-broker:1883"]
  topics = ["shellies/+/+/power", "zigbee2mqtt/#"]
  data_format = "json"
```
-   Grafana (port 3001) est ici configuré séparément afin de ne pas écraser l’instance Prom stack si elle existe.
    

  

**Reverse proxy et sécurité**  : En production, on place un proxy inverse (Nginx, Traefik) devant Grafana/InfluxUI/PromUI pour gérer TLS et authentification unifiée. Par exemple, on peut créer des hôtes virtuels  grafana.mondomaine.com,  prometheus.mondomaine.com, protégés par basic auth ou OAuth. Il est crucial de  **persister**  les volumes de base de données (ne pas reconstruire Influx/Prom et perdre les données). On met en place des  **backups réguliers**  :  influx backuppour InfluxDB, et snapshots pour Prometheus (ou copier le dossier de données).

  

**Bonnes pratiques de réseau**  : Isoler les exportateurs dans des VLAN (ex. tous les exporters IoT/OT sur un VLAN séparé, exposé uniquement au broker MQTT et aux collectors). Les containers de monitoring tournent idéalement sur un segment sécurisé. On n’ouvre pas les ports  9100,  9255, etc. hors du VPN interne. Grafana et Prometheus/InfluxUI doivent être chiffrés (HTTPS) et protégés par un système d’authentification solide (OIDC/LDAP, ou au minimum  auth.basic).

  

## **6. Tableau comparatif récapitulatif**

TABLEAU

Ce tableau synthétise les différences clés. En somme, Prometheus est idéal pour la surveillance  **temps réel et multi-dimensionnelle**  d’infrastructure dynamique, avec alerting natif. InfluxDB est un choix robuste pour du  **historique volumineux**  et des données hétérogènes, spécialement en contexte IoT/smart home. Chaque solution peut être déployée isolément ou en complément pour tirer parti de leurs points forts.
