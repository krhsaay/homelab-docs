# Projet DIY ESP32 : capteurs de température et MQTT

Nous allons construire un système IoT avec un ESP32, mesurant la température (et autres données) via divers capteurs, et transmettant ces mesures via MQTT. L’ESP32 se connecte au Wi-Fi et publie les données sur un  _broker_  MQTT (local ou cloud). Les données peuvent ensuite être exploitées dans un  **homelab**  (ex. Home Assistant, InfluxDB, Grafana) ou dans un environnement IoT autonome (services cloud). Ce guide détaille les capteurs recommandés, la programmation Arduino, le protocole MQTT et l’intégration avec Home Assistant/InfluxDB/Grafana.

## Capteurs recommandés

On privilégie des capteurs populaires et fiables pour domotique et IoT :

-   **DHT11 / DHT22** : capteurs numériques de température et humidité. Très utilisés par les hobbyistes. Le DHT22 offre une plus grande plage de mesure et une meilleure résolution (jusqu’à ±0,5 °C de précision, –40 à +80 °C), tandis que le DHT11 est plus limité (0–50 °C, ±2 °C) mais moins cher. Ces capteurs intègrent un convertisseur analogique-numérique et délivrent directement des données numériques sur un seul fil (avec pull-up).
    
-   **DS18B20** : capteur de température numérique « 1-Wire », alimenté en 3,3–5 V. Il communique sur une seule broche de données plus la masse. Chaque DS18B20 dispose d’un identifiant unique 64 bit, ce qui permet de chaîner plusieurs sondes sur la même ligne numérique. C’est pratique pour multiplier les points de mesure (par exemple, eau, air, radiateurs) avec un seul GPIO.
    
-   **BME280** : capteur 3-en-1 mesurant pression atmosphérique, température et humidité. On l’interroge généralement en I2C (broches SDA/SCL), ce qui économise les broches GPIO. La mesure de pression permet d’estimer l’altitude, utile pour des stations météo ou l’optimisation du chauffage/ventilation. Ce capteur offre de bonnes précisions sur ses trois paramètres et couvre de larges plages de fonctionnement.
    
-   **Autres capteurs domotique**  (sans citation spécifique) : selon les besoins on peut ajouter un  **détecteur de mouvement PIR**, des  **capteurs de gaz/fumée**  (MQ-2, MQ-135), des  **capteurs de luminosité**  (photocellule), ou un  **contacteur d’ouverture**  (Reed/magnétique). Ces éléments étendent l’application vers la sécurité et la gestion d’énergie.
    

_Fig. 1 – ESP32 branché à un capteur DHT22 (blanc), mesurant température et humidité._

Le DHT11/DHT22 mesure la température et l’humidité, et est très populaire chez les makers. On le branche sur un GPIO numérique de l’ESP32 (avec une résistance de tirage interne ou un pull-up externe). Le DHT22 est plus précis et couvre des plages plus grandes que le DHT11, mais il rafraîchit les données toutes les ~2 s (contre 1 s pour le DHT11). Ces capteurs sont relativement lents, mais embarquent un convertisseur A/N, sortant directement des valeurs numériques prêtes à l’emploi, ce qui simplifie grandement la lecture via Arduino.

_Fig. 2 – L’ESP32 affichant via une page Web la température lue par un capteur DS18B20._

Le capteur  **DS18B20**  est un thermomètre numérique 1-wire. Il se connecte à une seule broche de données (plus la masse), et peut être alimenté en 5 V ou 3,3 V. Ce capteur est précis (environ ±0,5 °C) et robuste. De plus, chaque DS18B20 a un code interne à 64 bits, ce qui permet de mettre plusieurs sondes sur le même fil de données. On peut ainsi connecter plusieurs DS18B20 (par exemple dans différentes pièces ou dans de l’eau) à un seul GPIO de l’ESP32, et lire successivement la température de chacun.

_Fig. 3 – Module capteur BME280 (température, pression, humidité) utilisé comme station météo embarquée._

Le  **BME280**  combine trois mesures : pression atmosphérique, température et humidité relative. Il communique généralement en I2C (broches SDA/SCL), ce qui réduit l’occupation de GPIO sur l’ESP32. La pression mesurée permet d’estimer l’altitude locale, ce qui est utile pour les applications météo ou de contrôle environnemental. Ce capteur, très répandu en IoT, offre de bonnes précisions sur ses mesures et supporte de larges plages de températures et de pressions. On l’utilise souvent dans des stations météo ou des systèmes domotiques nécessitant des données environnementales complètes.

## Programmation de l’ESP32 (Arduino IDE)

Nous utiliserons l’IDE Arduino (C++) pour programmer l’ESP32. Il faut inclure les bibliothèques Wi-Fi (`WiFi.h`) et MQTT (`PubSubClient.h`  par exemple) ainsi que celles spécifiques aux capteurs : Adafruit DHT pour DHT11/22, OneWire + DallasTemperature pour DS18B20, Adafruit BME280, etc. Dans le  `setup()`, on configure la connexion Wi-Fi (SSID + mot de passe), puis on initialise la connexion MQTT au broker (adresse IP/URL, port, login).

Dans la boucle  `loop()`, on lit périodiquement les capteurs (par exemple toutes les 10 secondes), puis on publie les valeurs sur des  _topics_  MQTT dédiés. Par exemple, pour un DHT22 on peut écrire en C++ :

```cpp
mqttClient.publish(MQTT_PUB_TEMP, String(temperature).c_str());
mqttClient.publish(MQTT_PUB_HUM,  String(humidity).c_str());

```

Cette ligne fait appel à  `mqttClient.publish(topic, payload)`  de la librairie PubSubClient. Elle envoie les mesures sous forme de chaînes de caractères sur les topics définis (ex.  `"esp32/dht/temp"`). Dans l’exemple ci-dessous, l’ESP32 publie successivement la température et l’humidité lues. En ouvrant le moniteur série on vérifie que l’ESP32 se connecte au broker et diffuse bien les messages sur les topics appropriés.

Ainsi, chaque mesure est mise en  _payload_  d’un message MQTT. On peut configurer le  **QoS**  (qualité de service) et le flag  _retain_  selon les besoins (par défaut QoS 0,  _retain_  peut être activé pour conserver la dernière valeur). Par ailleurs, on gère la reconnexion automatique au broker si nécessaire. Tout le code reste en Arduino C++ (sans microPython ici, comme demandé).

## Broker MQTT : comparatif

Le  **broker MQTT**  relaie les messages entre éditeurs et abonnés. Le choix du broker dépend du scénario :

-   **Mosquitto**  (open-source) : extrêmement léger et populaire. Supporte MQTT v5, TLS/SSL et l’authentification basique. On peut l’installer sur un serveur local (Raspberry Pi, VM, NAS) ou via un conteneur Docker.
    
-   **Mosquitto (Home Assistant)** : Home Assistant propose un  _add-on_  Mosquitto tout configuré. C’est l’option la plus simple pour un homelab HA. L’add-on génère automatiquement un utilisateur/sujet sécurisé pour HA, facilitant l’intégration.
    
-   **Brokers SaaS (cloud)** : AWS IoT Core, Google Cloud IoT, HiveMQ Cloud, CloudMQTT, etc. Ces solutions fournissent un broker MQTT hautement disponible et géré (authentification avancée, TLS mutualisé, quotas, etc.). Elles sont pratiques si l’ESP32 doit transmettre sur Internet sans serveur local (par exemple depuis l’extérieur du réseau domestique), mais requièrent souvent un abonnement payant.
    
-   **Autres (Enterprise)** : des brokers comme EMQX, RabbitMQ avec plugin MQTT ou ActiveMQ existent, mais Home Assistant ne les recommande pas (bugs connus).
    

Pour résumer : un broker local (ex. Mosquitto sur Raspberry Pi) est gratuit et facile à gérer en homelab. Un broker cloud (AWS, CloudMQTT…) simplifie l’accès externe et la montée en charge. Home Assistant souligne que l’option la plus privée est un broker auto-hébergé (ex. Mosquitto en local), mais proposer un broker cloud reste possible pour un projet mobile/autonome.

## Intégration Home Assistant et stockage

Dans Home Assistant (HA), on installe d’abord l’intégration MQTT en renseignant l’hôte et les identifiants du broker (par exemple,  `core-mosquitto`  si on utilise l’add-on). Les capteurs publiés sur MQTT deviennent alors accessibles : on peut configurer des  **MQTT Sensors**  soit via l’interface (MQTT Discovery) soit manuellement dans  `configuration.yaml`. Chaque capteur MQTT apparaît alors comme une entité HA (ex.  `sensor.temperature_salon`).

Pour la persistance des données, HA peut être combiné avec  **InfluxDB**  et  **Grafana**. InfluxDB est une base de données  _time-series_  open-source idéale pour stocker de grandes quantités de mesures horodatées. Par exemple, l’add-on InfluxDB d’HA capture automatiquement les entités MQTT et enregistre chaque valeur dans la base. Grafana peut ensuite se connecter à InfluxDB et tracer des courbes temporelles personnalisées. Comme l’a noté David Essenius,  _les graphiques intégrés de HA ne sont pas très paramétrables et les données ne sont pas conservées indéfiniment_; il préconise donc l’usage d’InfluxDB+Grafana pour des visualisations avancées et un historique complet. Grâce à cette architecture, on peut disposer de beaux tableaux de bord dynamiques (interpolation, moyennes, alertes, etc.) basés sur les capteurs de l’ESP32.

En résumé, le flux de données peut être :  **ESP32 → Broker MQTT (Mosquitto)**  → 1) Home Assistant (pour supervision/automatisation) et 2) InfluxDB/Grafana (pour stockage/visualisation). Dans HA on peut également déclencher des automatismes sur seuils de température, allumer un ventilateur, etc. L’ESP32 publie de son côté de manière indépendante les données via MQTT, et HA se contente de les consommer.

## Homelab vs projet autonome IoT

Le scénario « homelab » implique que l’ensemble reste sur le réseau local. Par exemple, on installe Mosquitto, HA, InfluxDB et Grafana sur un Raspberry Pi ou un NAS. L’avantage est la  **complète maîtrise**  des données et leur sécurité (pas d’envoi extérieur), ainsi qu’un faible coût. En revanche, pour un projet IoT autonome (ex. capteur embarqué, exposition), on choisira un broker accessible publiquement. On peut exposer son broker Mosquitto via une IP/DNS statique, ou utiliser un service cloud (AWS IoT, CloudMQTT…). De même, on peut opter pour InfluxDB Cloud ou Grafana Cloud afin de visualiser les données depuis n’importe où. Dans tous les cas, il faut prévoir l’authentification et le chiffrement (MQTT sur TLS) si on traverse Internet. HA conseille en général d’héberger soi-même le broker quand cela est possible, mais un mode  _IoT mobile_  peut légitimement reposer sur des services externes (broker SaaS, bases de données cloud).
