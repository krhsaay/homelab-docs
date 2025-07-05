# **Comparatif Zigbee2MQTT, ZHA et deCONZ**

-   **Zigbee2MQTT**  fonctionne comme un service autonome (souvent en conteneur ou add-on) reliant le coordinator Zigbee à un  _broker_  MQTT. Il nécessite donc une infrastructure MQTT (Mosquitto, etc.) séparée. Cet outil offre une interface Web riche (cartographie du réseau, logs) et s’intègre à Home Assistant via MQTT ou une intégration dédiée. Il prend en charge une  **liste étendue d’appareils**  Zigbee et de coordinateurs (clés CC* ou autres)  . Une fois configuré, il est  _très stable_  . En contrepartie, sa configuration est plus “geek” (fichier YAML, mise à jour de firmware manuelle) et il ajoute une couche MQTT qui peut légèrement compliquer le debug.
    
-   **ZHA (Zigbee Home Automation)**  est l’intégration Zigbee native de Home Assistant. Elle fonctionne “_nativement_” dans HA sans add-on externe  , ce qui libère des ressources (pas de conteneur supplémentaire) et garantit une réactivité maximale. ZHA utilise la bibliothèque  zigpy  et gère directement de nombreux coordinateurs (ConBee, Sonoff, CC2652, HUSBZB-1, etc.). Elle supporte un bon nombre de périphériques courants (IKEA, Aqara, Xiaomi, Osram, etc.), bien que Zigbee2MQTT en supporte souvent  _encore plus_  . L’installation et l’ajout de capteurs se font via l’interface HA (pas de ligne de commande), simplifiant la prise en main.
    
-   **deCONZ/Phoscon**  est la solution de Dresden Elektronik autour du dongle ConBee/RaspBee. Elle fonctionne comme un add-on ou service séparé (et nécessite l’interface Phoscon). L’environnement requis est une clé ConBee (ou module RaspBee) et l’addon Phoscon. L’interface graphique de Phoscon permet d’associer et gérer les appareils Zigbee sans toucher à du code, ce qui est  **très accessible aux débutants**  . DeConz a une bonne compatibilité avec les appareils courants (notamment IKEA et Philips Hue) mais dépend du timing de mises à jour de Dresden. Sur les forums, ZHA et Zigbee2MQTT sont souvent cités comme plus réactifs ou régulièrement mis à jour que deCONZ, mais ce dernier reste  **très stable pour les matériels supportés**.
    

  

Sur l’aspect  **OTA (firmwares)**  : Zigbee2MQTT propose son propre mécanisme d’OTA (via MQTT) où les capteurs Zigbee peuvent demander une mise à jour et on pilote l’opération par des topics  . ZHA dispose désormais d’une interface OTA intégrée dans HA : elle détecte les updates disponibles (pour quelques marques majeures) et propose un bouton de mise à jour dans l’interface  . En revanche, ZHA ne télécharge pas automatiquement les firmwares ; il faut fournir manuellement les fichiers (sauf pour IKEA/OSRAM/Sonoff activés par défaut)  . DeCONZ gère l’OTA via un plugin OTAU où l’on charge soi-même les images de mise à jour (fichiers fournis par le fabricant) et on lance l’update depuis l’interface Phoscon  .

  

En résumé,  **Zigbee2MQTT**  est plus “power user” (nécessite MQTT) mais extrêmement complet (plus de devices supportés, flexibilité Node-RED, etc.)  .  **ZHA**  est plus intégré et simple (« natif » HA, pas d’addon)  .  **deCONZ**  brille par sa simplicité d’installation et son interface graphique, au prix d’une compatibilité hardware restreinte (principalement ConBee) et de dépendances sur Phoscon.

  

## **Matériel compatible : dongles et coordinateurs Zigbee**

  

Voici quelques coordinateurs Zigbee populaires, leur chipset, et points forts/faibles :

TABLEAU

## **Cas d’usage concrets en homelab**

  

Dans un  **homelab sécurisé**, on privilégie souvent les équipements auto-hébergés et locaux. Zigbee s’y prête bien pour l’IoT non critique (éclairage, capteurs) tout en offrant un réseau séparé du Wi‑Fi. Par exemple :

-   **Automatisation de l’éclairage**  : ampoules et rubans LED Philips Hue, Osram/LEDVANCE ou Ikea TRÅDFRI, pilotés depuis Home Assistant. Un simple bouton Zigbee (Xiaomi/Aqara) peut commander un groupe de lampes via une automatisation HA, évitant la dépendance aux cloud des fabricants.
    
-   **Capteurs environnementaux**  : capteurs de température/humidité, qualité d’air (Sonoff, Xiaomi, etc.), placés dans les racks ou pièces critiques. Leur remontée locale dans HA permet des alertes (ex : « réservoir trop chaud ») ou pilotage de ventilateurs/cooling.
    
-   **Sécurité périmétrique**  : détecteurs d’ouverture de portes/fenêtres (Aqara, Sonoff) et détecteurs de mouvement Zigbee peuvent alimenter un système d’alarme local. Grâce aux  _endpoints_  Zigbee, on peut lier des détecteurs directement à un sirène Zigbee, ou faire remonter les événements instantanément dans HA, sans cloud tiers.
    
-   **Gestion d’énergie**  : prises intelligentes Zigbee (Ikea, Tuya/Sonoff avec MCUs Zigbee, etc.) mesurent la consommation et peuvent couper l’alimentation en cas d’anomalie. On peut ainsi surveiller l’énergie du lab (serveurs, NAS) et archiver les données localement.
    
-   **Autres usages**  : automatisation d’ouverture/fermeture de volets roulants, contrôle de pompes à eau en systèmes de sauvegarde, etc. L’important est que tout reste dans l’infrastructure réseau privée (Home Assistant auto-hébergé), en cohérence avec la politique de sécurité du homelab.
    

  

## **Topologie du réseau Zigbee et bonnes pratiques**

  

Zigbee est un réseau  **maillé**  (mesh) 2,4 GHz. Seul le  _coordinateur_  (stick USB) amorce le réseau. Les autres appareils se répartissent en deux rôles :

-   **Routeurs (mains)**  : tous les appareils toujours sous tension (prises, ampoules, onduleurs Zigbee, etc.) servent de  **routeurs/relais**. Ils étendent la portée en relayant les messages. Certains appareils (comme certains plugs ou lampes) sont de meilleurs routeurs que d’autres (voir documentation), il est conseillé d’en disperser dans la maison.
    
-   **End devices (piles)**  : capteurs sur piles (détecteurs, télécommandes, etc.) communiquent généralement uniquement avec leur parent. Ils ne relaient pas le trafic.
    

  

Une bonne topologie consiste à placer le coordinateur au centre du maillage (par exemple sur un long câble USB isolé) et à s’assurer de plusieurs  **routeurs strategic**  répartis géographiquement. Par exemple :

        Coordinateur (dongle USB) –– Router A –– Sensor 1 (pile)
                     |                     
                     +– Router B –– Router C –– Light 1 (maison haut)
                                      +– Light 2
                                     Sensor 2 (sous-sol)


Quelques conseils clés :

-   **Portée & antenne**  : utiliser une antenne externe ou un dongle avec PA (+20 dBm) augmente drastiquement la portée (ex. CC2652P vs CC2652RB). Éviter de brancher directement le dongle sur un port USB 3.0 du serveur (les ports USB 3 génèrent beaucoup de bruit RF). Placer le dongle sur un  _câble USB 2.0 prolongateur blindé_  éloigné du PC pour réduire les interférences  .
    
-   **Canal radio**  : Zigbee a 16 canaux (11-26) dans le 2,4 GHz. Choisir un canal éloigné du Wi‑Fi est crucial. En pratique, on préfère souvent  **canal 25**  (en UE) ou  **20**  (aux US) car ils recouvrent moins les bandes Wi‑Fi courantes  . Changer de canal Zigbee après que les appareils sont appairés demande de les réappairer, donc le configurer dès le début est important  .
    
-   **Éviter l’interférence**  : les réseaux Wi‑Fi (2,4 GHz), Bluetooth et même certains dongles USB sans fil peuvent brouiller Zigbee. Si instable, tester de bouger le réseau Wi‑Fi à d’autres canaux (1, 6, 11) et garder le Zigbee éloigné des sources bruyantes. Les documentations recommandent explicitement d’expérimenter l’orientation/position du dongle USB  : parfois tourner la clé ou la décaler de quelques centimètres améliore le lien.
    
-   **Maillage**  : un coordinateur seul ne gérera qu’un nombre limité d’appareils (~30×; ZHA/zigpy limite à 32 enfants directs)  . Pour supporter de nombreux capteurs, on comptabilise les routeurs. Par exemple, un CC2652 (32 enfants direct - 3 routeurs = 29) + 3 routeurs CC2530 (16 périphériques chacun) permet théoriquement ~77 appareils au total  . En pratique, il faut multiplier les routeurs pour atteindre un grand nombre de devices et couvrir toutes les zones.
    
-   **Firmware**  : Mettre à jour le firmware du coordinateur et des routeurs (si possible). Par exemple, flasher un CC2652P (Sonoff ou TubeZB) avec le dernier Z-Stack permet d’atteindre +20 dBm. Certains routeurs DIY (CC2530/CC2531) peuvent être reflashés pour améliorer leur rôle de répéteur.
    

  

En respectant ces bonnes pratiques, on obtient un maillage Zigbee robuste et sans “trou” de couverture. Les utilisateurs rapportent que des sticks bien placés (avec antenne, loin de l’USB3) peuvent alimenter de grands réseaux (>60 capteurs) sans perte de paquet  .
