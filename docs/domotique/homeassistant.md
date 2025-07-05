# **Installation et configuration de Home Assistant**

  

Home Assistant est une plateforme domotique open source puissante. Elle peut être installée de plusieurs façons selon vos besoins et votre infrastructure.  **Home Assistant Operating System (HassOS)**  est la méthode recommandée pour la plupart des utilisateurs : il s’agit d’un système minimal dédié qui s’exécute sur SBC ou VM et prend en charge nativement les  _add-ons_  (extensions)  . Les autres options comprennent l’**installation en conteneur Docker**  (requiert un système hôte Linux et ne supporte pas les add-ons intégrés) et les machines virtuelles. Chaque méthode a ses avantages en termes de maintenance, de performances et d’isolation.

-   **Home Assistant OS**  : s’installe sur Raspberry Pi, NUC, VM, etc. Prévoyez au minimum 2 Go de RAM et ~32 Go de stockage (SSD recommandé)  . L’OS minimalifie les mises à jour et intègre la gestion d’add-ons (MOSQUITTO, Node-RED, etc.)  . Sur Proxmox, on déploie souvent HA OS en VM (notamment pour la compatibilité avec les  _dongles_  USB)  .
    
-   **Container (Docker)**  : installer le conteneur officiel  ghcr.io/home-assistant/home-assistant:stable. Par exemple :

```bash
docker run -d --name homeassistant --restart=unless-stopped \
  -e TZ=Europe/Paris \
  -v /PATH_CONFIG:/config \
  --network=host \
  ghcr.io/home-assistant/home-assistant:stable
```

-   Ce mode ne gère pas les add-ons internes  . Il est idéal si vous voulez superviser manuellement l’instance via Docker. Pensez à utiliser  --network=host  et à exposer les périphériques USB (--device /dev/ttyUSB0) pour Zigbee/Z-Wave  .
    
-   **VM Proxmox**  : déployer HA OS dans une VM KVM. L’interface Proxmox (KVM) permet le passthrough USB (utile pour les dongles Zigbee/Z-Wave  ). On peut automatiser l’installation avec des scripts (par ex. tteck/Proxmox Helper) pour créer une VM HA OS en quelques commandes  . L’avantage : on peut isoler HA du reste (ex. VLAN dédié) tout en profitant des snapshots de VM.
    
-   **LXC Proxmox**  : HA Supervised (Core + Supervisor) peut fonctionner dans un conteneur LXC, mais ce mode est désormais  **déconseillé**  et instable  . Il offre toutefois un démarrage plus rapide et moins de surcharge qu’une VM. Si vous optez pour LXC, assurez-vous qu’il a accès aux périphériques (lxc.cgroup.devices.allow = c 188:* rwm  pour MQTT USB, etc.).
    
-   **Stockage**  : préférez un SSD ou un volume RAID pour la fiabilité (évitez les SD cards à bas coût). Home Assistant stocke la config, la base de données (SQLite ou MariaDB), les images, etc. Sur une VM ou conteneur, réservez assez d’espace disque selon le nombre d’entités et de journaux que vous collectez. Activez la compression TRIM (ex.  --storagectl --discard on  pour VirtualBox  ) pour optimiser l’espace.
    
-   **Isolement réseau**  : dans un homelab sécuritaire, on recommande de segmenter le réseau IoT. Par exemple, placez HA dans un VLAN distinct. Les “things” (capteurs, lampes, etc.) sont alors isolés du LAN principal. Les VLAN permettent d’**isoler les appareils et de sécuriser le réseau**, car chaque réseau virtuel est indépendant  . Home Assistant doit pouvoir communiquer avec les deux domaines (ex. via plusieurs interfaces ou règles de pare-feu). En pratique, on crée un VLAN “IoT” où résident les appareils, et un VLAN “Serveur” pour HA, en autorisant uniquement les flux strictement nécessaires (MQTT, HTTP, etc.)  . Pensez à mettre en place un pare-feu (p. ex. pfSense) pour contrôler ces échanges (un exemple de règles typiques : autoriser le VLAN IoT vers HA sur le port MQTT, bloquer le reste du LAN)  .

TABLEAU
## **Intégration MQTT**

  

L’utilisation d’un  **broker MQTT**  est au cœur de nombreux scénarios domotiques. Le choix le plus simple et open-source est  **Mosquitto**. On peut l’installer soit comme add-on (sur HA OS/Supervised), soit dans un conteneur dédié Docker/LXC. Home Assistant propose un add-on officiel  _Mosquitto Broker_  qui automatise la configuration initiale (il génère un utilisateur/sésame sécurisé)  . En alternative, des brokers MQTT tels qu’**EMQX**,  **VerneMQ**, ou  **NanoMQ**  sont open-source et scalables (tableau comparatif ci-dessous).

TABLEAU

### **Déploiement du broker (Mosquitto)**

  

Installez Mosquitto (ex.  apt-get install mosquitto) ou utilisez le  _Add-on_  dans HA OS. Configurez l’authentification en créant un utilisateur MQTT propre (ex.  mqtt_user  et mot de passe)  . Exemple de connexion dans Home Assistant (via Intégrations) :  **hôte = IP du broker**, port 1883, et les identifiants choisis. Activez l’option TLS si vous voulez chiffrer les échanges : cela nécessite un certificat (Let’s Encrypt ou auto-signé) et l’activation de TLS sur le broker  . Sur Mosquitto, éditez  /etc/mosquitto/mosquitto.conf  pour ajouter :

```yaml
listener 8883
cafile /etc/mosquitto/certs/ca.crt
certfile /etc/mosquitto/certs/server.crt
keyfile /etc/mosquitto/certs/server.key
require_certificate false
```

et créez des utilisateurs via  mosquitto_passwd. Redémarrez le service. Home Assistant se connectera alors sur le port 8883 avec TLS activé  .

  

### **Appareils IoT (Zigbee2MQTT, Shelly, ESPHome, Tasmota)**

  

Une fois le broker en place, on peut brancher divers périphériques  **via MQTT**  :

-   **Zigbee2MQTT** : requiert un dongle Zigbee (CC2531, CC2652, etc.) branché sur le serveur ou le LXC conteneur. Zigbee2MQTT collecte les données Zigbee et les publie sur MQTT (topiques  zigbee2mqtt/device/friendly_name). Activez le  _discover_  dans la config de Zigbee2MQTT (homeassistant: enabled: true). Ensuite, dans HA, activer l’intégration MQTT Discovery : Zigbee2MQTT détectera automatiquement ses équipements et les publiera dans HA  . Exemple de configuration  configuration.yaml  de Zigbee2MQTT :

```yaml
homeassistant:
  mqtt:
    base_topic: zigbee2mqtt
    server: 'mqtt://192.168.1.100:1883'
  serial:
    port: /dev/ttyUSB0
```

-   Cela crée des entités HA prêtes à l’emploi via MQTT Discovery  .
    
-   **Shelly**  : ces appareils Wi-Fi prennent en charge MQTT nativement. Dans l’interface Web de chaque Shelly, allez dans  _Connectivity > MQTT_, et activez l’MQTT (mettez l’IP du broker HA, et les identifiants MQTT). Ainsi les messages de l’appareil (température, relais, etc.) seront publiés sur  shellies/<device_id>/.... Home Assistant possède également une intégration native pour Shelly (basée sur CoAP/WebSocket) qui évite MQTT. Mais dans un homelab où l’on prône l’auto-hébergement, on peut utiliser MQTT pour centraliser la communication. Par exemple, le capteur Shelly Relee coupera un circuit quand HA envoie  cmnd/shelly1-abc123/relay/0 = OFF  sur MQTT.
    
-   **ESPHome**  : on programme les microcontrôleurs ESP32/ESP8266 via  **ESPHome**  (firmware custom). Les appareils ESPHome communiquent idéalement en API native (port 6053) avec HA  , mais ils supportent aussi MQTT (en ajoutant un bloc  mqtt:  dans leur YAML). Pour les intégrer, utilisez l’intégration  _ESPHome_  dans HA. Soit le device est découvert automatiquement (MDNS), soit vous l’ajoutez manuellement en fournissant son IP/port  . Exemple de configuration ESPHome (light_switch.yaml) :

```yaml
esphome:
  name: light_switch
wifi:
  ssid: "..."
  password: "..."
mqtt:
  broker: 192.168.1.100
  username: mqtt_user
  password: mqtt_pass
switch:
  - platform: gpio
    pin: GPIO5
    name: "Lampe Salon"
```

-   Ici l’ESP send ses états sur MQTT (homeassistant/light/lamp_salon/state  etc.), mais on peut aussi utiliser l’API native d’ESPHome qui évite MQTT.
    
-   **Tasmota**  : si vous avez flashé des modules Sonoff en Tasmota, l’intégration HA Tasmota via MQTT est simple. Activez MQTT dans la page de configuration web de l’appareil : entrez l’adresse du broker et les identifiants (il vaut mieux créer un utilisateur HA dédié pour Tasmota)  . Sur l’intégration HA  _Tasmota_, sélectionnez votre device découvert (ou ajoutez-le manuellement). Les équipements Tasmota (relais, capteurs, interrupteurs) seront exposés comme entités HA (switch, light, sensor…). Par exemple,  cmnd/tasmota123/POWER  contrôle le relais, et  stat/tasmota123/POWER  contient l’état  . Notez que  SetOption19 0  active la découverte automatique dans HA.
    

  

### **Sécurité MQTT**

-   **Authentification**  : N’utilisez jamais un broker ouvert sans mot de passe ! Configurez des utilisateurs MQTT et attribuez-leur des permissions limitées. Par exemple, un utilisateur « homeassistant » avec les droits pour tous les sujets. Sur Mosquitto, définissez un fichier  passwd  via  mosquitto_passwd  et assurez-vous d’ajouter  allow_anonymous false  dans  mosquitto.conf. Home Assistant lui-même peut générer un user/sésame pour l’intégration  .
    
-   **Chiffrement TLS**  : pour chiffrer les données MQTT (même en LAN, c’est recommandé en zone non fiable), générez un certificat SSL et configurez Mosquitto en TLS (ports 8883). Dans Home Assistant, activez la validation du certificat sur la connexion MQTT  . Vous pouvez aussi forcer l’usage de MQTT over WebSockets (port 8083) avec TLS.
    
-   **Pare-feu**  : de la même façon que pour HA, isolez l’accès MQTT par VLAN ou pare-feu. Par exemple, n’autorisez que votre serveur HA à se connecter au broker (ou mieux : utilisez des ACL Mosquitto pour restreindre les topics/clients).
    

  

## **Tableaux de bord Lovelace et automatisations**

  

Home Assistant dispose d’une interface web de  **tableaux de bord (Lovelace)**  très flexible. Un dashboard Lovelace est composé de  _cartes_  (cards) préconfigurées ou personnalisables  . On peut afficher capteurs, graphiques, commandes de lumières, caméras, etc. Par exemple, les cartes  _Sensor_,  _History Graph_,  _Entities_,  _Media Player_, ou  _Energy_  sont natives  .

-   **Ajouter des cartes**  : en mode édition, cliquez sur “Ajouter une carte”, puis choisissez le type (ex.  _Card type -> Entities_), sélectionnez des entités, et sauvegardez  . Vous pouvez aussi créer des vues (onglets) et regrouper des cartes avec des dispositions (verticale, grille). Chaque carte peut avoir des actions au clic (définir  tap_action,  hold_action) et des conditions de visibilité (uniquement si un capteur x est vrai)  .
    
-   **Cartes personnalisées**  : via le gestionnaire de communautés HACS (Home Assistant Community Store), on peut installer des  _custom cards_  (développées par la communauté). Par exemple,  **Mini Graph Card**  (graphiques avancés),  **Button Card**  (boutons riches en icônes),  **Gauge Card**, etc. Ces cartes offrent plus de flexibilité visuelle que les cartes de base. Après installation HACS, il suffit d’ajouter la ressource (JS) et de déclarer  type: "custom:mini-graph-card"  dans votre YAML de tableau de bord.
    
-   **Automatisations**  : Home Assistant propose un éditeur visuel pour créer des automatismes (“Automations”) ou on peut directement éditer le YAML. Chaque automatisation combine des  **déclencheurs**  (triggers),  **conditions**(facultatives) et  **actions**. Par exemple, pour une alerte de mouvement :

```yaml
trigger:
  platform: state
  entity_id: binary_sensor.detecteur_mouvement
  to: "on"
condition:
  condition: state
  entity_id: person.jean
  state: "not_home"
action:
  - service: camera.snapshot
    target: { entity_id: camera.camera_salon }
    data: { filename: '/config/www/snapshot_{{now().timestamp()}}.jpg' }
  - service: telegram_bot.send_photo
    data:
      file: '/config/www/snapshot_{{now().timestamp()}}.jpg'
      caption: 'Mouvement détecté !'
```

-   Ici, on capture un cliché via  camera.snapshot  et on envoie la photo sur Telegram (bot HA)  . Ce snippet illustre comment combiner YAML et services HA pour des scénarios de sécurité. De même, l’interface graphique permet d’enchaîner ces étapes sans écrire une ligne, avec des choix menus pour les entités et actions.
    

  

## **Cas d’usage concrets en homelab**

-   **Surveillance (caméras, capteurs)** : intégrez des caméras IP (ONVIF, MJPEG) avec l’intégration  _Camera_  de HA. Couplée à un détecteur de mouvement (capteur PIR, NTGR24, etc.) ou à un add-on NVR comme  _Frigate_  ou  _MotionEye_, vous pouvez déclencher des actions. Par exemple, dès que le capteur signale un mouvement, Home Assistant peut prendre une photo (camera.snapshot) et l’envoyer par notification (Telegram, email)  . On peut aussi lancer un enregistrement vidéo court. Des solutions de détection  _on-device_  (Frigate basé sur DeepStack ou TensorFlow) sont opensource et s’intègrent à HA pour réduire les faux positifs.
    
-   **Sécurité** : HA peut s’intégrer à des alarmes DIY ou systèmes plus élaborés. Par exemple, relier un capteur door/window (via Zigbee/ MQTT) et déclencher une sirène ou envoyer une alerte. On peut utiliser  _Notifations Telegram_  pour recevoir des alertes sécurisées. L’automatisation ci-dessus en est un exemple. De plus, l’application mobile HA (iOS/Android) peut détecter votre présence (via GPS ou réseau), permettant de n’alerter que si personne n’est à la maison.
    
-   **Pilotage énergétique**  : la fonction  _Energy_  de HA offre un tableau de bord pour suivre la conso électrique, la prod solaire, etc. On y intègre des capteurs de courant (par ex. Shelly EM, S0-compteurs) et on lie un onduleur PV (via son API) pour suivre la production. Home Assistant fournit des graphiques d’autoconsommation et permet d’automatiser la charge/decharge d’une batterie si disponible  . On peut aussi piloter les thermostats (par ex. thermostat DIY via ESPHome ou Nest/Starling via intégration) et les prises intelligentes (Shelly, Tasmota) pour optimiser la charge. Par exemple, couper automatiquement certains appareils en cas de surconso ou allumer le chauffage quand le tarif heures creuses démarre. Les données collectées alimentent le tableau de bord d’énergie  , avec analyse des tendances et prévisions.
    
-   **Sauvegarde / restauration**  : en homelab, il est crucial de backuper régulièrement. Home Assistant propose des snapshots complets (config, base de données, add-ons) via l’UI («  **Settings > Système > Backups**  ») ou l’intégration  _Backup_. On peut automatiser la création de ces snapshots grâce à une automation programmant  backup.create  la nuit  . Stockez ces fichiers sur un partage réseau ou Nextcloud (via add-on Samba/SSHFS). Pour restaurer, utilisez l’UI de restauration (ou remontez l’archive dans un nouvel HA). Exemple d’automatisation YAML pour un backup quotidien à 3h du mat  :
```yaml
automation:
  - alias: "Backup HA tous les jours 3h"
    trigger:
      - platform: time
        at: "03:00:00"
    action:
      - service: backup.create
```
Toutes ces solutions n’utilisent que des logiciels open source et auto-hébergés. En combinant Home Assistant avec MQTT, Zigbee2MQTT, Tasmota/ESPHome, on crée un écosystème puissant et sécurisé, idéal pour un homelab de passionné en cybersécurité.


