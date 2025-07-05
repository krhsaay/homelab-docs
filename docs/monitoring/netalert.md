# **Surveillance réseau pour homelab : outils comparés**

  

Dans un  **homelab**  personnel, la surveillance réseau nécessite des outils capables de détecter de nouveaux appareils, d’analyser le trafic et d’alerter en cas de comportement anormal. Plusieurs solutions open‑source existent. Ci-dessous, nous comparons  **NetAlertX**  à des alternatives majeures (ntopng, LibreNMS, Zabbix, Prometheus + Node Exporter) selon plusieurs critères.

TABLEAU

Chaque outil se démarque ainsi :  **NetAlertX**  se spécialise dans la détection d’intrusion locale (nouveaux appareils, changements), avec une interface claire et de nombreuses alertes prêtes à l’emploi  .  **ntopng**  est focalisé sur l’analyse de trafic (flux/paquets) avec peu de fonctions de notification intrusives.  **LibreNMS**  et  **Zabbix**  offrent une supervision générale très robuste (SNMP, métriques, alertes avancées), mais requièrent plus de ressources et de configuration.  **Prometheus + Node Exporter**  vise la collecte de métriques système évolutives (ex. CPU, utilisation réseau d’un serveur) avec intégration naturelle à Grafana, mais sans détection réseau par défaut.

  

## **Cas d’usage en homelab**

-   **Détection de nouveaux appareils**  :  Surveillez quand un appareil inconnu se connecte (nouvel ordi, smartphone, IoT…). NetAlertX est conçu pour cela : il « scanne votre réseau pour repérer de nouveaux appareils… et vous alerte dès qu’un nouvel appareil ou une adresse IP inconnue est détecté »  . LibreNMS et Zabbix peuvent découvrir automatiquement des équipements via SNMP/ARP, mais émettre une alerte requiert de configurer des triggers spécifiques. Un homelab typique usera souvent NetAlertX pour l’« intrusion Wi-Fi/LAN » et LibreNMS/Zabbix pour le suivi continu des serveurs/routeurs.
    
-   **Suivi de connexions inhabituelles**  :  Par exemple, détecter un balayage de port ou un appareil qui change fréquemment d’IP. NetAlertX surveille les changements d’adresse IP des appareils et les reconnexions/déconnexions  . Pour la détection de scans réseau, on complétera avec un IDS dédié (ex. Snort) ou le mode « Nmap port scan » de NetAlertX (plugin). Zabbix peut déclencher une alerte sur un nombre anormal de paquets ou connexions avec un trigger, et LibreNMS peut signaler des flux suspects via ses graphes, mais cela reste plus artisanal. Les métriques de trafic exportées par Node Exporter peuvent être visualisées dans Grafana pour repérer des sursauts (via alertes Prometheus), mais sans détection intelligente par défaut.
    
-   **Alertes centralisées (Telegram, e-mail, dashboards)**  :  NetAlertX intègre 80+ canaux de notification (dont Telegram, e-mail, Webhooks) via Apprise  . On peut ainsi recevoir une alerte sur mobile ou pousser l’événement dans  **Home Assistant**  pour automatiser (par exemple couper le Wi-Fi). LibreNMS/Zabbix notifient par e-mail ou webhook (avec Slack, PagerDuty, etc.) en fonction de conditions définies  . Pour un affichage unifié, Grafana est souvent utilisé : Zabbix et LibreNMS peuvent alimenter Grafana (via API ou exporters). NetAlertX peut lui-même envoyer des données à Home Assistant ou à d’autres systèmes (via Webhooks/MQTT  ), et n’est pas conçu pour Grafana directement. Un scénario typique en homelab pourrait associer NetAlertX (alertes intrus) + Home Assistant pour l’automatisation, et Prometheus/Grafana ou Zabbix pour superviser et visualiser les serveurs et équipements principaux.
    

  

## **Déploiement de NetAlertX (Docker)**

  

NetAlertX est  **pleinement supporté en Docker**  . Voici les étapes clés :

1.  **Pré-requis & volumes**  : Installez Docker ou Docker Compose sur votre hôte. Créez deux répertoires locaux pour la configuration et les données, par exemple  /opt/netalertx/config  et  /opt/netalertx/db. Ces dossiers seront montés dans le conteneur pour persistance.
    
2.  **Commande Docker (exemple)**  : Utilisez l’image officielle  ghcr.io/jokob-sk/netalertx:latest  (GitHub Container Registry). Par exemple :

```bash
docker run -d --name netalertx --rm \
  --network host \
  -v /opt/netalertx/config:/app/config \
  -v /opt/netalertx/db:/app/db \
  --tmpfs /app/api \
  -e PUID=1000 -e PGID=1000 \
  -e TZ=Europe/Paris \
  -e PORT=20211 \
  --cap-add NET_RAW \
  ghcr.io/jokob-sk/netalertx:latest
```

2.  -   --network host  permet au conteneur d’accéder directement au réseau (indispensable pour scanner l’ARP du LAN).
        
    -   Les variables  PUID/PGID  ajustent les permissions des volumes (mettez votre utilisateur UNIX).
        
    -   TZ  règle le fuseau horaire.
        
    -   PORT  définit le port web (par défaut 20211) écouté en host.
        
    -   --cap-add NET_RAW  ou  --privileged  : donne le droit de capture (nécessaire pour arp-scan, comme indiqué dans la doc)  .
        
    
3.  **Docker Compose (optionnel)**  : Vous pouvez aussi utiliser un  docker-compose.yml  (ex. dans le repo officiel) pour définir ces mêmes paramètres de façon déclarative.
    
4.  **Première configuration**  : À la première exécution, NetAlertX initialise les fichiers de config et la base de données SQLite dans les volumes montés. Si besoin, modifiez ensuite  /app/config/app.conf  pour ajuster les réseaux à surveiller (-m), la cadence des scans, etc. Un reverse-proxy (Nginx) peut être ajouté pour sécuriser l’accès HTTPS.
    
5.  **Meilleures pratiques** :
    
    -   Limitez l’accès à l’interface web (authentifiez-vous et/ou placez un VPN/reverse-proxy devant) pour protéger l’outil de surveillance.
        
    -   Surveillez l’usage CPU/mémoire du conteneur ; ajustez le nombre de threads ou modules activés si la machine hôte est restreinte.
        
    -   Sauvegardez régulièrement le dossier  db  (contenant la base SQLite) pour ne pas perdre l’historique.
        
    

  

Toutes ces étapes sont documentées dans le guide d’installation officiel  et la page GitHub (Quick Start)  .

  

## **Sécurité et limites**

-   **Minimiser les faux positifs**  : En début de déploiement, vous verrez beaucoup d’alertes normales (« mon smartphone se déconnecte / reconnecte »). Pour réduire le bruit : ajustez les règles en fonction de votre usage. Par exemple, qualifiez certains appareils de « toujours présents », ou définissez une plage d’adresses IP fixe pour des appareils connus (puits DHCP statiques). On peut aussi utiliser des listes blanches (« whitelist ») d’appareils/horaires normaux  . Comme en entreprise, un tuning progressif des règles (se baser sur le comportement normal) peut éliminer jusqu’à 90% des faux positifs  . Dans NetAlertX, vous pouvez étiqueter les appareils connus et ignorer certains changements mineurs. Par exemple, pour les appareils générant des MAC aléatoires (smartphones iOS/Android), une approche consiste à fixer leur IP ou à les exclure de la détection.  (exemple : ajouter leur IP dans une liste d’ignore) ou à désactiver la randomisation MAC sur ces appareils.
    
-   **Sécurité des données de logs**  : Les données réseau et l’historique des appareils sont sensibles. NetAlertX stocke  _localement_  toutes les informations par défaut  , ce qui évite l’exfiltration non désirée. Il est recommandé de chiffrer (ou restreindre l’accès à) ces répertoires. Limitez l’accès à la base (ex. user Linux dédié, permissions restreintes) et chiffrez les backups. Dans un homelab, évitez de remonter les logs sur des services cloud sans nécessité. De même, sécurisez les notifications (ex. API keys Telegram ne doivent pas fuir).
    
-   **Visibilité excessive et éthique**  : La surveillance réseau permet de voir beaucoup de données. Par exemple, en mode « promiscuité », vous pourriez capter du trafic qui contient potentiellement des informations privées (adresses e-mail, noms d’utilisateur, métadonnées de navigation, etc.). Même sur votre propre réseau, restez conscient de la vie privée des utilisateurs (famille, invités). N’abusez pas de l’extraction de payloads réseau. Focus : NetAlertX se concentre sur les en-têtes ARP/IP et la présence de l’appareil, pas sur le contenu des paquets. Pour le sniffing orienté sécurité (IDS), rappelez-vous que certains pays ont des lois strictes sur l’interception des communications (même en entreprise/locale). En cas de doute, privilégiez la collecte de logs minimales (ex. uniquement les méta-données, pas le contenu chiffré) et informez les utilisateurs surveillés.
    

  

En somme, la surveillance réseau en homelab doit trouver l’équilibre entre visibilité et confidentialité. Adoptez une configuration  _responsable_  : captez uniquement ce qui est nécessaire (évitez le « full packet capture » si vous n’en avez pas besoin), protégez les logs, et ajustez finement vos règles pour distinguer comportements normaux et anomalies.

