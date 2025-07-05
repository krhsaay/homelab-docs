# **Gestion de conteneurs Docker : Portainer, Dockge et interfaces auto-hébergées**

  

Il existe aujourd’hui plusieurs outils web auto-hébergés pour piloter des conteneurs Docker sans passer par la ligne de commande. Parmi les plus connus figurent  **Portainer**  et  **Dockge**, auxquels on peut ajouter des alternatives comme  **Yacht**,  **CapRover**,  **Swarmpit**, etc. Ces interfaces offrent des tableaux de bord graphiques pour déployer et surveiller des applications conteneurisées. Leur choix dépendra des besoins : nombre d’hôtes à gérer, complexité des applications, niveau de contrôle et d’automatisation requis, etc.

  

## **Portainer (gestion complète)**

  

Portainer est un outil de gestion de conteneurs Docker très populaire. Il fournit une interface graphique complète pour créer, déployer et superviser des conteneurs, services et stacks Docker (Compose), ainsi que des environnements Kubernetes ou Swarm  . Portainer existe en édition  **Community**  (libre) et  **Business**  (payante) : les trois premiers nœuds sont gratuits en Business Edition, ce qui permet de tester la version complète sans frais  . L’architecture de Portainer se compose d’un  **serveur**  et d’un  **agent**. Le serveur héberge l’interface web, l’agent (optionnel) collecte les métriques sur d’autres hôtes. Ainsi, Portainer peut gérer plusieurs machines Docker (multi-hôtes) via son agent  . Portainer intègre les fonctionnalités Docker standard (logs, console, volumes, réseaux, registres, etc.) et simplifie le déploiement de stacks et de services, mais cette richesse fonctionnelle peut être complexe pour les débutants  . On notera que Portainer utilise par défaut le port  **9443**  pour HTTPS et  **8000**  pour HTTP, avec un certificat auto-signé au premier lancement  . Il faut configurer un mot de passe administrateur à l’installation (Portainer demande de le définir lors du premier accès) et, idéalement, passer derrière un reverse-proxy HTTPS pour la sécurité.

  

## **Dockge (gestion légère de stacks Docker Compose)**

  

Dockge est un gestionnaire de stacks Docker Compose conçu pour la simplicité  . Développé par l’auteur de l’Uptime Kuma, Dockge se concentre exclusivement sur les fichiers  **docker-compose.yml**  et l’édition de configurations YAML  . Contrairement à Portainer qui expose tous les réglages Docker individuellement, Dockge demande de préparer ou d’importer les fichiers  docker-compose.yml  (via l’interface ou en plaçant les fichiers dans le dossier des stacks)  . Son interface est légère et réactive, mais  _ne gère qu’implicitement les stacks Compose_  : pas de création de conteneur isolé, ni de manipulations fines du réseau au-delà de ce qui est défini dans le compose. En résumé, Dockge est un outil « Docker-centré » et limité : il convient si vous êtes à l’aise avec Docker Compose et que vous ne voulez gérer que des stacks, mais il ne remplace pas entièrement Portainer si vous avez besoin de fonctions avancées (réseaux, conteneurs unitaires, monitoring multi-hôtes, etc.)  . Dockge fonctionne sur un seul hôte Docker (pas d’agent multi-hôtes) et écoute par défaut sur le port  **5001**. L’installation typique consiste à créer un dossier  /opt/stacks  pour vos projets, un dossier  /opt/dockge  pour Dockge, puis récupérer le fichier  compose.yaml  de Dockge et lancer  docker compose up -d  . Par exemple :

-   mkdir -p /opt/stacks /opt/dockge && cd /opt/dockge
    
-   curl -fsSL https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml -o compose.yaml
    
-   docker compose up -d  .
    
    On accède ensuite à Dockge sur  http://<hôte>:5001.
    

  

## **Autres interfaces auto-hébergées**

  

Outre Portainer et Dockge, on peut citer plusieurs autres projets :

-   **Yacht**  : une interface web  **open-source**  axée sur le déploiement d’applications via des templates. Yacht permet de déployer en un clic des “packages” (basés sur Docker Compose) depuis un dépôt externe (ex. Github)  . Son interface simple affiche dès l’accueil l’utilisation CPU/Mémoire des conteneurs. Contrairement à Portainer, Yacht ne gère qu’un seul hôte (pas d’agent multi-serveur)  et ne permet pas de déployer directement depuis le système de fichiers local (il récupère les stacks depuis des dépôts git)  . L’installation est rapide : par exemple, créer un volume Docker  yacht  et lancer  docker run -d -p 8000:8000 -v /var/run/docker.sock:/var/run/docker.sock -v yacht:/config selfhostedpro/yacht  . L’interface est centrée sur la gestion de templates de stack, et propose même l’import des templates Portainer. Yacht est jeune (en alpha/early access) mais très actif. Par défaut on se connecte en  admin@yacht.local  /  pass  (à changer d’urgence pour la sécurité)  .
    
-   **CapRover**  : plutôt qu’un simple tableau de bord Docker, CapRover est une  **plateforme PaaS**  open-source (“Heroku on Steroids”). Elle automatise le déploiement d’applications web (Node, PHP, Python, bases de données, etc.) sur Docker Swarm avec Nginx et Let’s Encrypt sous le capot  . CapRover fournit une interface web conviviale et un CLI pour installer/mettre à jour des apps, configurer SSL et clusters Swarm automatiquement  . Par exemple, CapRover se lance souvent via un conteneur ou un script CLI dédié. Ses plus : SSL gratuit automatiquement, load-balancer Nginx pré-configuré, catalogues d’apps prêts à l’emploi. C’est adapté si vous voulez un PaaS facile pour déployer des sites et APIs, mais si vous cherchez juste un dashboard Docker pur, c’est une solution plus lourde.
    
-   **Swarmpit**  : une interface orientée  **Docker Swarm**. Swarmpit offre la gestion de stacks (création via Compose ou génération automatique), la supervision des ressources (CPU, RAM, disque) en temps réel, le déploiement de services et la recherche dans les registres Docker  . Il supporte plusieurs utilisateurs avec permissions, prend en charge les registres privés et peut auto-re-déployer un service quand une nouvelle image arrive  . C’était une alternative pour Swarm, mais le projet semble moins actif aujourd’hui.
    
-   **Autres**  : on pourrait aussi mentionner des projets comme  _Docker Compose UI_  (ancêtre pour gestion Compose en web, peu maintenu),  _DockStation_,  _LazyDocker_  (console TUI), voire l’extension  **Cockpit**  pour gérer containers (avec Podman) sur Linux. Chacun cible un besoin particulier, mais Portainer/Dockge/Yacht restent les plus utilisés en auto-hébergement.
    

  

## **Cas d’usage concrets**

  

Les différentes interfaces Docker servent typiquement aux cas suivants :

-   **Gestion multi-hôtes**  : si vous avez plusieurs machines Docker à piloter, Portainer (avec son agent) ou CapRover (Swarm) sont adaptés. Portainer peut agréger plusieurs environnements et afficher un tableau de bord unifié  . Yacht et Dockge gèrent un seul hôte (pas d’agent multi-serveur)  .
    
-   **Supervision de conteneurs critiques**  : pour surveiller l’état des conteneurs (logs, CPU, mémoire, santé), Portainer offre des graphiques et l’outil  docker stats  (accessible via l’interface)  . Yacht affiche les stats CPU/Mémoire directement sur le tableau de bord. Dockge est plus basique et ne fournit pas de monitorings avancés (on peut se contenter de Watchtower pour les mises à jour d’images). Pour une supervision poussée (alertes, métriques sur le long terme), on combinera souvent ces UIs avec des outils dédiés (Prometheus/Grafana, etc.), car ces interfaces n’ont pas un système d’alerte intégré.
    
-   **Déploiement d’applications (stacks multi-conteneurs)**  : toutes ces interfaces permettent de déployer des applications composées (via Docker Compose ou templates). Par exemple, Portainer et Yacht disposent de « templates » prêts à l’emploi (catalogues d’apps) pour installer en 1 clic des applications populaires (WordPress, Nextcloud, etc.). Dockge, lui, s’appuie sur des fichiers Docker Compose existants : on scanne un dossier  /opt/stackspour importer des stacks. CapRover propose des catalogues d’apps One-Click similaires (bases de données, CMS, frameworks). En résumé, pour « packager » et déployer une application complexe, ces outils simplifient grandement le processus par rapport à la gestion manuelle de  docker-compose up, surtout pour les débutants.
    
-   **Documentation et apprentissage**  : ces interfaces rendent visibles les configurations Docker (volumes, réseaux, variables d’environnement), ce qui aide à comprendre et documenter l’architecture d’une application. Par exemple, Portainer permet de visualiser facilement les réseaux utilisés et la pile d’un stack déployé, ce qui est utile en dépannage. Dockge, de son côté, force à travailler en YAML, ce qui peut être formateur pour maîtriser Docker Compose.
    

  

## **Installation et configuration**

  

Voici les étapes clés pour déployer Docker + l’interface choisie (ex. Portainer ou Dockge) :

1.  **Installer Docker Engine**. Sur une machine Linux (Debian/Ubuntu), on commence par mettre à jour le système puis ajouter le dépôt Docker officiel  . Par exemple :

```bash
sudo apt update && sudo apt upgrade
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo usermod -aG docker $USER
```

Ces commandes installent Docker CE ainsi que le plugin Compose V2 et ajoutent votre utilisateur au groupe  docker. Après cela, un  docker run hello-world  devrait réussir.
    
2.  **Installer dans un conteneur LXC (optionnel)**. Si vous utilisez Proxmox LXC, il est possible de faire tourner Docker dans un container (bien que Proxmox recommande plutôt un VM pour Docker). En conteneur LXC non-privilegié, il faut activer le  _nesting_  et  _keyctl_. Par exemple, dans la configuration LXC ajouter :
```sh
features: keyctl=1,nesting=1
```
Ensuite, à l’intérieur du LXC on réinstalle Docker comme ci-dessus. Des utilisateurs rapportent que cela fonctionne avec  keyctl  et  nesting=1  .
    
3.  **Déployer Portainer**  (par exemple). Une fois Docker en place, créez un volume de données et lancez Portainer :
```bash
docker volume create portainer_data
docker run -d \
  -p 8000:8000 -p 9443:9443 \
  --name portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:lts
```
Ceci démarre Portainer Community Edition (version LTS) en mode standalone. On peut alors accéder à l’interface web sur  https://localhost:9443  et définir un mot de passe admin  . Pour déployer Portainer BE (3 nœuds gratuits), la procédure est similaire en remplaçant l’image par  portainer/portainer-ee:lts.
    
4.  **Déployer Dockge**. Créez d’abord les dossiers requis (/opt/stacks  pour vos stacks,  /opt/dockge  pour le compose de Dockge), puis téléchargez le YAML de Dockge et lancez-le :
```bash
mkdir -p /opt/stacks /opt/dockge
cd /opt/dockge
curl -fsSL https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml \
  -o compose.yaml
docker compose up -d
```
Dockge sera alors disponible sur le port configuré (5001 par défaut)  . On y accède via  http://<hôte>:5001. Si vous préférez utiliser  docker-compose  v1 ou Podman, remplacez simplement par  docker-compose up -d.
    
5.  **Déployer Yacht**. Exemple rapide :
```bash
docker volume create yacht
docker run -d \
  -p 8000:8000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v yacht:/config \
  selfhostedpro/yacht
```

Après lancement, Yacht écoute sur le port 8000 (à changer si conflit avec Portainer). L’interface est alors accessible sur  http://<hôte>:8000  .
    
6.  **Configuration réseau et reverse proxy**. Il est fortement conseillé de placer ces interfaces derrière un reverse proxy HTTPS (Traefik, Nginx, Caddy, etc.) pour sécuriser l’accès. Par exemple, on peut utiliser  **nginx-proxy**  (containers automatiques) et configurer  VIRTUAL_HOST=portainer.mondomaine.tld  et  VIRTUAL_PORT=9000  pour obtenir un certificat Let’s Encrypt automatiquement  . Un extrait de configuration Compose type (Portainer CE) :
```yaml
services:
  nginx-proxy:
    image: nginxproxy/nginx-proxy
    ports: ["80:80","443:443"]
    volumes:
      - "/var/run/docker.sock:/tmp/docker.sock:ro"
  portainer:
    image: portainer/portainer-ce:lts
    restart: always
    environment:
      - VIRTUAL_HOST=portainer.mondomaine.tld
      - VIRTUAL_PORT=9000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    expose:
      - "9443"
``` 

Ensuite, on pointe  portainer.mondomaine.tld  vers l’IP de l’hôte dans DNS. Cette méthode assure une connexion HTTPS et peut être étendue à Dockge (via Traefik labels ou environnement similaire) et Yacht.
    
7. **Sécuriser l’accès et authentification**. Chaque interface a sa propre méthode d’authentification : Portainer demande un compte admin dès l’installation (et peut activer 2FA en Business Edition), Yacht a un login initial (admin@yacht.local  /  pass  à changer impérativement)  , Dockge n’a pas d’authentification native et repose sur la protection réseau (d’où l’intérêt du reverse proxy ou VPN). Il est impératif de choisir des mots de passe forts, de tenir le logiciel à jour, et idéalement de limiter les accès (par exemple via un VPN ou contrôle d’accès du proxy).
    

  

## **Limites et alternatives (Kubernetes vs Docker Compose, etc.)**

  

Chaque solution a ses limites. Portainer, Dockge et consorts sont conçus pour des déploiements  **Docker (et éventuellement Swarm)**  de petite à moyenne échelle. Si votre usage devient plus complexe (centaines de conteneurs, forte montée en charge, déploiement multi-cluster), une vraie plateforme d’orchestration comme  **Kubernetes**  sera préférable. Kubernetes offre la scalabilité automatique (autoscaling), la haute disponibilité multi-nœuds et une vaste écosystème. En effet,  _« Kubernetes est conçu pour gérer des déploiements à grande échelle sur des clusters de machines, avec des fonctionnalités avancées (load balancing, autoscaling, scheduling)_ _. Portainer est plus léger et reste limité aux petits déploiements »_. Son apprentissage est plus difficile, mais il est plus robuste en production. À l’inverse, Docker Compose ou une interface graphique simple (Portainer/Dockge/Yacht) suffisent quand on n’a qu’un petit nombre de services sur un ou quelques serveurs. Par exemple, de nombreux développeurs créent d’abord localement avec Compose, et migrent vers Kubernetes en production pour scaler selon la charge  .

  

Pour résumer :

-   **Quand préférer Docker Compose pur (ligne de commande)**  : pour un projet simple (un seul serveur, quelques conteneurs), l’interface graphique peut être inutile. La CLI Docker Compose reste légère et suffit.
    
-   **Quand préférer un outil comme Portainer/Dockge/Yacht**  : pour simplifier la gestion visuelle des conteneurs, faciliter les déploiements en équipe ou la découverte de l’architecture, surtout sur des environnements test/homelab ou sur un serveur unique.
    
-   **Quand passer à Kubernetes**  : en environnement critique d’entreprise, avec besoin de redondance, de roulement transparent des mises à jour, de besoins de ressources élastiques. Portainer propose même un mode K8s pour visualiser certains aspects, mais cela n’égale pas un cluster Kubernetes natif pour des usages intensifs  .
    

  

En définitive, Portainer et Dockge sont des outils puissants pour l’auto-hébergement Docker, chacun avec ses forces et ses limites  . Le choix dépendra donc du contexte : nombre d’hôtes à administrer, types d’applications à déployer, exigences de sécurité et d’évolutivité. Dans tous les cas, il est important de mettre en place les bonnes pratiques (Reverse proxy SSL, gestion des utilisateurs, mises à jour régulières) pour profiter en toute sécurité de ces interfaces web  .
