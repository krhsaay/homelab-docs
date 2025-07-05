# **Vidéosurveillance open‐source pour homelab**

  

Dans un environnement homelab (auto-hébergé) orienté cybersécurité, on privilégie des solutions de vidéosurveillance respectueuses de la vie privée, légères et extensibles. Trois solutions courantes sont  **Frigate**,  **Shinobi**  et  **ZoneMinder**. Elles sont toutes open source et permettent de traiter localement les flux caméras. Ce document compare leurs performances, facilité de configuration, compatibilité matérielle et capacités de détection d’objet par IA. Il détaille également des cas d’usage typiques (surveillance périmétrique, détection d’intrusion, alertes), l’utilisation des accélérateurs matériels Google Coral (USB, M.2/PCIe), le choix de caméras (ONVIF/RTSP, PoE/Wi‑Fi, qualité vs budget) et donne des instructions d’installation (Docker, LXC), d’intégration avec Home Assistant et d’optimisation (zones, objets, alertes).

  

## **1. Comparaison : Frigate vs Shinobi vs ZoneMinder**

TABLEAU

En résumé,  **Frigate**  est centré sur la détection d’objets IA locale avec accélérateur (Coral, GPU)  , son interface est sobre mais l’intégration dans Home Assistant est native.  **Shinobi**  mise sur la performance brute et la flexibilité (gestion multi‑flux, clustering, support CUDA/Jetson)  .  **ZoneMinder**  est la solution historique ultra‑complet, plus lourde en CPU et requérant des extensions pour l’IA  , mais proposant de riches possibilités de configuration de détection, d’alertes et de stockage. Les choix dépendent des besoins : Frigate pour une détection IA simple et rapide, Shinobi pour une solution légère et évolutive, Zoneminder pour un contrôle granulaire et des scénarios d’entreprise.

  

## **2. Cas d’usage typiques en homelab (sécurité)**

  

Dans un homelab d’ingénieur cybersécurité, la vidéosurveillance sert avant tout à  **sécuriser les locaux**  et  **compléter la détection d’intrusion**. Exemples de scénarios :

-   **Surveillance périmétrique**  : caméras extérieures montées sur la façade, grillage ou clôture, couvrant l’accès principal. Détection de mouvements d’intrus en bordure de propriété (personne, véhicule). On déclenche des alertes lors d’un franchissement de zone dangereuse (zone interdite). Frigate/Shinobi déclencheront une notification push ou e-mail quand un humain/voiture entre dans la zone surveillée.
    
-   **Caméras intérieures (zone sensible)**  : surveillance de la salle serveurs ou local réseau. Détection de présence non autorisée (biométrique ou objet suspect comme sac à dos). Par exemple, si quelqu’un essaie de pénétrer dans l’armoire serveur, on veut une  **alerte immédiate**  (via Telegram/HA). Les systèmes peuvent associer la détection d’intrusion à des scripts (verrouillage, sirène, enregistrement vidéo).
    
-   **Détection d’intrusion générale**  : lorsqu’un capteur de mouvement (ou un IDS réseau) signale une anomalie, la caméra valide visuellement. Frigate, Shinobi ou ZoneMinder peuvent être couplés à des capteurs tiers (via MQTT/API) : une alerte de type  _présence_  ne déclenche un enregistrement ou une alarm qu’après vérification video (ex: un objet humain détecté par IA). On peut aussi utiliser la reconnaissance de plaque ou de visage pour identifier les véhicules/personnes autorisés.
    
-   **Alertes cybersécurité via liaison physique**  : en cas d’alarme logicielle (attaque, ransomware) on peut déclencher une brève surveillance renforcée (par exemple augmenter la sensibilité ou prendre en compte de nouveaux objets comme « tentative d’ouverture de baie » si un capteur magnétique s’active). L’idée est de corréler les événements réseau avec la vidéo.
    
-   **Rétention vidéo et preuves**  : configurer un enregistrement 24/7 (ou circulaire) et extraire automatiquement les rushes quand un événement IA survient  . Par exemple, Frigate peut ne conserver que les segments vidéo où un « objet voulu » (personne, véhicule, animal, etc.) a été détecté, réduisant le stockage.
    

  

Ces usages sont gérés via des réglages de zones et d’objets pour limiter les fausses alertes (ex : configurer Frigate pour n’alerter que si une personne entre dans l’allée de parking)  . Le système peut alors envoyer des notifications (e-mail, webhook, MQTT, Home Assistant) en cas d’intrusion détectée. Par exemple, Frigate envoie chaque détection via MQTT pour s’intégrer facilement dans HA  .

  

## **3. Google Coral Edge TPU : accélérateur IA**

  

Le  **Coral Edge TPU**  de Google est un coprocesseur ASIC spécialisé pour l’inférence d’IA (TensorFlow Lite) en périphérie. Il existe en trois formats :  _USB_,  _mini PCIe_  (format court) et  _M.2 E-key_  (format 2230). Tous contiennent le même circuit Edge TPU, capable d’environ  **4 TOPS**  (téra-opérations int8) chacun, soit environ  _400 images/sec_  sur un modèle MobileNet V2, avec ~2 W de consommation  . La version M.2 “Dual Edge TPU” intègre deux de ces co-processeurs en parallèle (8 TOPS total)  . Concrètement, cela permet de traiter de nombreux flux vidéos en temps réel sans charger le CPU, par exemple Frigate peut dépasser 100 fps par flux avec un Coral tout en gardant un usage CPU très bas  .

-   **Version USB**  : dongle USB3.0 (Type-C) facile à installer sur tout système Linux/Windows/Mac (dont Raspberry Pi)  . Tendance au prototypage ou usage ponctuel.  **_Limitation_**  : sous forte charge ou sur certains Linux (RPi), on préfère souvent le M.2/PCIe pour la stabilité et la vitesse.
    
-   **Versions M.2 / Mini PCIe**  : modules internes que l’on insère sur une carte mère offrant un slot M.2 E-key ou mini PCIe. Ils nécessitent le driver PCIe d’EdgeTPU (guide Coral) et offrent le même Edge TPU, voire en double (8 TOPS)  . Ces versions sont recommandées pour les déploiements permanents : « la version M.2 ou mini PCIe est conseillée plutôt que USB, car l’USB est plutôt pour le prototypage, pas la production »  . Par exemple, dans Docker Frigate on passe le périphérique Coral avec  --device /dev/bus/usb:/dev/bus/usb  pour l’USB, ou  /dev/apex_0:/dev/apex_0  pour le PCIe (cf. docs)  .
    

  

**Avantages**  : l’Edge TPU accélère massivement l’inférence sans alourdir le CPU ni consommer beaucoup d’énergie. On peut alors multiplier les détections IA (personnes, véhicules…) sur plusieurs caméras simultanées. Par exemple, deux Edge TPU permettent de doubler les inférences (en parallèle ou en pipeline)  . Dans Frigate ou Shinobi, un Coral supprime le goulot d’étranglement CPU de l’analyse vidéo, rendant la détection d’objets IA  _vraiment temps réel_  . Sans accélérateur, on peut utiliser un GPU Nvidia (TensorRT) ou Hailo, mais le Coral est simple, peu coûteux (~$60) et largement supporté.

  

## **4. Choix des caméras compatibles (ONVIF, RTSP, PoE, Wi‑Fi)**

  

Pour un homelab, on préfère des caméras  **IP ouvertes**  supportant ONVIF ou RTSP pour une intégration facile dans Frigate/Shinobi/ZoneMinder. Quelques recommandations :

-   **Protocole**  : ONVIF assure l’interopérabilité (découverte, PTZ) tandis que RTSP fournit le flux vidéo. Frigate/Shinobi préfèrent des flux H.264+AAC (bons pour la compatibilité)  . Shinobi, par exemple, gère HTTP/HTTPS, RTP/RTSP et ONVIF nativement  , ce qui le rend compatible avec la plupart des caméras IP. ZoneMinder accepte tout protocole supporté (MJPEG, FFmpeg, ONVIF, RTSP…)  .
    
-   **Qualité d’image**  : privilégiez un capteur de taille moyenne/grande (1/2”, 1/1.8”) pour de bonnes performances en faible luminosité. Exemples courants : caméras Dahua/Hikvision ou leurs répliques (Amcrest, Hikvision “ColorVu” pour vision de nuit en couleur) offrent 2–4 MP et bonne IR  . Les caméras 4K (8 MP) peuvent sembler attractives, mais attention : la documentation Frigate note que les modèles 4K, notamment chez Reolink, posent souvent problème (flux instable) et conseille de rester sur du 5 MP ou moins  .
    
-   **Budget vs gamme** :
    
    -   _Entrée de gamme (< 50 €)_  : caméras Wi‑Fi (Tapo, TP-Link, Wyze, etc.) ou PoE bon marché (Reolink 4 MP, Dahua basique). Elles font le job en jour, mais ont souvent un SoC limité (faible perf. en nuit). Certains modèles Wi‑Fi nécessitent d’activer RTSP manuellement (ex : caméras Ezviz par défaut désactivent RTSP).
        
    -   _Milieu de gamme (50–150 €)_  : caméras PoE à capteur plus grand (Reolink 5 MP, Amcrest 4 MP, HIK-IP 4 MP), avec support ONVIF complet. Elles combinent image correcte, fiabilité réseau et aisance d’installation (PoE).
        
    -   _Haut de gamme (150 €+)_  : Axis, Bosch, Dahua 4K, etc. Offrent du 4K ou plus, meilleur capteur, optical zoom/PTZ, mais à coût élevé. Pour un homelab, ce niveau n’est pas toujours nécessaire.
        
    
-   **Connectivité**  : la  **PoE**  (Power over Ethernet) est à privilégier pour les caméras fixes extérieures/intérieures : elle fournit courant et données sur un seul câble, garantissant stabilité et reach (jusqu’à 100 m). L’Ethernet dédié réduit les risques d’interférences. Le Wi‑Fi peut être utilisé pour des caméras mobiles ou en repeater, mais il est  _moins fiable_  (pertes de paquets)  . Frigate déconseille d’ailleurs les caméras Wi‑Fi pour cette raison. Des caméras 4G LTE existent (D-link, Reolink Argus) mais restent chères et dépendent d’un opérateur.
    
-   **Recommandations spécifiques**  : les docs Frigate suggèrent les marques  **Dahua, Hikvision, Amcrest**  pour leur fiabilité et leurs sous-flux multiples  . Par exemple :
    
    -   _Dahua IPC-T5xx_  (5 MP, PoE) ou  _Hikvision DS-2CD_  (4 MP, PoE) pour extérieur, IR.
        
    -   _Amcrest 4 MP_  (PoE) ou  _Reolink RLC-410/RLC-811_  (PoE) comme bons compromis qualité/prix.
        
    -   _Caméra Wi-Fi intérieur_  : Tapo C200 (1080p) ou Reolink E1 Pro (2K) – pratique mais limiter le nombre.
        
    -   Éviter les caméras « cloud only » (Nest, Arlo, etc.) sans RTSP/ONVIF, car elles ne fonctionnent pas sur Frigate/Shinobi.
        
    

  

En résumé, choisissez des caméras H.264 compatible ONVIF/RTSP avec des connecteurs RJ45 PoE pour la fiabilité. Vérifiez la documentation pour activer RTSP si nécessaire. Un tableau récapitulatif peut être utilisé :

TABLEAU

_(Les prix sont indicatifs en 2025. Les marques citées sont exemplaires – de nombreuses autres respectent les normes ONVIF/RTSP.)_

  

## **5. Installation et optimisation**

  

**Déploiement (Docker/LXC)**  – Frigate et Shinobi proposent des images Docker officielles. L’installation recommandée pour Frigate est via Docker Compose (voir exemple ci-dessous)  . On passe les volumes (config, stockage médias) et les périphériques pour le TPU : par exemple  --device /dev/bus/usb:/dev/bus/usb  pour une clé Coral USB, ou  /dev/apex_0:/dev/apex_0  pour un Coral PCIe  . On n’oublie pas les ports (Frigate : 8554 pour RTSP, 8971 pour l’interface). En LXC (Proxmox ou autre), le passage d’USB/Passthrough est délicat ; beaucoup optent pour une VM si on veut utiliser un Coral USB. Shinobi a aussi une image Docker (requiert Node, MySQL). ZoneMinder est souvent installé via paquet Linux (apt install zoneminder) ou Docker (image dlandon/Zoneminder), ce qui simplifie la mise à jour.

  

_Exemple d’extrait_ _docker-compose.yml_ _pour Frigate_  (GHCR stable)  :

```yaml
services:
  frigate:
    image: ghcr.io/blakeblackshear/frigate:stable
    container_name: frigate
    privileged: true          # nécessaire pour le passthrough
    shm_size: "512mb"
    devices:
      - /dev/bus/usb:/dev/bus/usb    # Coral USB
      - /dev/apex_0:/dev/apex_0      # Coral PCIe (voir guide)
      - /dev/dri/renderD128:/dev/dri/renderD128  # HW accel Intel
    volumes:
      - /path/to/config:/config    # config YML de Frigate
      - /path/to/media:/media/frigate
      - type: tmpfs
        target: /tmp/cache
        tmpfs:
          size: 1000000000  # cache en RAM (1GB)
    ports:
      - "8554:8554"   # RTSP entrées
      - "8971:8971"   # UI web
    environment:
      FRIGATE_RTSP_PASSWORD: "mon_motdepasse"
```

On adaptera bien sûr les chemins, devices et ressources selon son matériel.

  

**Intégration Home Assistant**  – Frigate dispose d’un add-on officiel (« Frigate NVR ») à installer depuis le dépôt GitHub de Blake Blackshear. Cet add-on facilite le lancement sans Docker manuel et crée automatiquement les entités caméra et détection dans HA. Notez toutefois les limites de HA OS : le stockage média réseau n’était pas supporté avant 2023, et les GPU externes (Nvidia, AMD) ne sont pas reconnus dans les add-ons  . En contrepartie, l’intégration fournie (basée MQTT) expose tous les événements de détection, les entités de classes d’objets détectés et les snapshots/vignettes. Pour Shinobi/ZM, on utilise le plugin  [zmeventnotifications](https://github.com/pliablepixels/zmeventnotification)  (pour ZM) ou  [Home Assistant Integration pour Shinobi](https://github.com/tchellomello/ha-shinobi)  afin de récupérer les alertes via MQTT ou webhook.

  

**Optimisation des détections**  – Pour éviter les fausses alertes, configurez dans le fichier de configuration :

-   **Zones de détection**  : délimitez par polygones les zones critiques (ex : allée, porte) et ne déclenchez une alerte que si un objet pénètre cette zone  . Par exemple, dans Frigate on liste  required_zones  sous  alerts  pour n’alerter qu’à l’intérieur de “jardin_entier”  .
    
-   **Filtrage par objet**  : spécifiez les classes d’objets à surveiller dans chaque zone. On peut par exemple n’autoriser que les  person  dans la zone entrée et les  car  sur l’allée de garage  . Seuls ces objets déclencheront les enregistrements/alertes dans leurs zones respectives.
    
-   **Détection de mouvement légère**  : Frigate/Shinobi effectuent d’abord une détection de mouvement basique pour limiter la charge (n’appellent l’analyse IA que si du mouvement est détecté)  . Ajustez la sensibilité (bruit, objets animaliers) via les masques ou seuils.
    
-   **Rétention et lecture**  : configurez la rétention vidéo basée sur les détections (ex : n’enregistrer en totalité qu’en cas de détection et conserver X jours). Par défaut Frigate peut enregistrer 24/7, mais on optimise en conservant uniquement les événements importants  .
    

  

Enfin, reliez les alertes IA aux notifications : Frigate envoie les événements (classe de l’objet, minuteur) via MQTT  , permettant à Home Assistant d’envoyer des push (Telegram, e-mail, mobile) ou de déclencher des automatismes (sirène, envoi d’une vue caméra).

  
