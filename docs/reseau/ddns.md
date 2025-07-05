# **DNS Dynamique et DNS Local : Comparatif et Guide Technique**

  

## **Introduction au DNS dynamique (DDNS)**

**Schéma – Fonctionnement d’un service de DNS dynamique.**  Un routeur partage son IP actuelle avec le serveur DynDNS, qui associe cette IP au nom d’hôte (ici  _homexyz_). Le routeur devient alors joignable via une URL fixe (_homexyz.dyndns.org_), malgré la variabilité de son adresse IP  . Le DNS dynamique (DDNS) est un mécanisme qui  **met à jour automatiquement un enregistrement DNS**  lorsque l’adresse IP publique d’un client change  . Cela permet d’accéder à une machine ou un service chez soi via un nom de domaine stable, même si l’IP fournie par le FAI est dynamique et change régulièrement. En pratique, un petit programme (client DDNS) s’exécute sur votre routeur ou serveur local, surveille les changements d’IP et informe le fournisseur de DDNS lorsque nécessaire. Ce dernier met à jour l’enregistrement DNS (généralement de type A pour IPv4 et AAAA pour IPv6) pointant vers la nouvelle IP. Ainsi, les  **clients peuvent toujours utiliser le même nom de domaine**  pour atteindre votre réseau domestique, sans se soucier des changements d’adresse  .

  

## **Comparaison technique des services DDNS (Cloudflare, DuckDNS, No-IP, etc.)**

  

Plusieurs services de DNS dynamique coexistent, chacun avec ses atouts, limitations et modalités. Nous présentons ci-dessous un comparatif technique des solutions populaires, notamment  **Cloudflare DDNS**,  **DuckDNS**,  **No-IP**, ainsi que quelques alternatives, en examinant leurs caractéristiques principales.

  

### **Cloudflare (DNS dynamique via API sur son propre domaine)**

  

**Cloudflare**  n’est pas à proprement parler un service DDNS traditionnel, mais la popularité de son offre DNS gratuite fait qu’on peut l’utiliser pour du DNS dynamique en mettant à jour ses enregistrements via l’API. Il faut  **détenir son propre nom de domaine**  et le configurer sur Cloudflare (plan gratuit). Ensuite, on utilise l’API de Cloudflare pour mettre à jour l’enregistrement DNS pointant vers l’IP dynamique. Cloudflare recommande d’ailleurs cette approche : un script local peut surveiller les changements d’IP et pousser les mises à jour via l’API Cloudflare  . Alternativement, on peut utiliser un client existant comme  **ddclient**  (un utilitaire compatible avec de nombreux fournisseurs DNS) qui supporte Cloudflare nativement  .

-   **Coût** : Cloudflare ne facture pas ce service – le plan gratuit Cloudflare DNS suffit, hormis l’achat du nom de domaine auprès d’un registrar.
    
-   **Mécanisme** : la mise à jour se fait via des requêtes HTTPS à l’API REST de Cloudflare, en fournissant un  **token API**  ou une clé API globale pour authentification. Il est  **recommandé d’utiliser un token API restreint**  aux droits DNS sur la zone concernée, par mesure de sécurité  . Dans l’interface Cloudflare, on crée un jeton avec permissions « Zone : DNS : Edit » sur le domaine cible. Ce token pourra être utilisé par ddclient ou un script maison. Par exemple, sur OPNsense on configurera  _Username_  = “token” et  _Password_  = le token API Cloudflare  .
    
-   **Domaine et hôtes** : Cloudflare permet d’utiliser  **son propre domaine**  (ex.  _monserveur.maison.com_). On peut mettre à jour un ou plusieurs enregistrements (sous-domaines) de ce domaine. Il n’y a pas de limite stricte sur le nombre d’hôtes gérés dynamiquement (en pratique, on peut avoir de multiples sous-domaines). On peut aussi tirer parti de plusieurs domaines si on en possède, le tout via le même compte Cloudflare.
    
-   **Propagation & TTL** : Les enregistrements DNS chez Cloudflare peuvent avoir un TTL très bas (jusqu’à  **60 secondes**  sur les enregistrements non-proxy  ). Cela signifie qu’en cas de changement d’IP, la nouvelle IP se propage rapidement.  _Note:_  Si l’enregistrement est passé en mode proxy (CDN Cloudflare activé, « orange cloud »), le TTL est géré différemment car les clients voient l’IP du proxy Cloudflare.
    
-   **Avantages** : mise à jour rapide, usage de son propre nom de domaine (plus professionnel), aucun besoin de confirmer périodiquement l’activité (pas d’expiration tant que le domaine est valide), intégration possible avec d’autres fonctionnalités Cloudflare (proxy web, pare-feu applicatif, certificats SSL gratuits, DNSSEC, etc.). De plus, Cloudflare étant un DNS anycast mondial, la  **fiabilité et rapidité de résolution**  sont excellentes.
    
-   **Inconvénients** : nécessite de gérer un nom de domaine (coût annuel du domaine, configuration des serveurs NS vers Cloudflare). L’intégration n’est pas « plug-and-play » sur tous les équipements : peu de routeurs grand public proposent Cloudflare en option DDNS, il faudra souvent recourir à un script ou à la fonction « Custom DDNS ». La sécurité doit être bien configurée (utilisation d’un token limité) car une compromission du token pourrait permettre de détourner votre DNS. Enfin, Cloudflare n’offre pas d’option simple de sous-domaine gratuit générique (contrairement à DuckDNS, No-IP…), il faut utiliser un domaine à soi.
    

  

### **DuckDNS (service DDNS gratuit)**

  

**DuckDNS**  est un service de DNS dynamique entièrement gratuit, opéré de manière communautaire (soutenu par des dons), très apprécié pour sa simplicité. Il fournit des sous-domaines sous la forme  *.duckdns.org. Quelques points clés :

-   **Coût et limites** : DuckDNS est 100 % gratuit, sans offre payante. Chaque compte peut créer  **jusqu’à 5 sous-domaines**  DuckDNS (on peut en obtenir davantage sur demande dans certains cas)  . Cette limite de 5 est généralement suffisante pour un usage personnel (ex.  _maison.duckdns.org_,  _cloud.duckdns.org_, etc.).
    
-   **Mise à jour** : L’API DuckDNS est extrêmement simple : la mise à jour se fait via une requête HTTP (ou HTTPS) contenant un  **token d’authentification unique**  et le nom de domaine. Par exemple :  https://www.duckdns.org/update?domains=monhote&token=abcdef123456&ip=1.2.3.4. De nombreux scripts tout prêts existent. DuckDNS fournit d’ailleurs des instructions pour divers OS (Windows, Linux, macOS) et peut être intégré dans des scripts shell, cron, containers Docker, etc. Par exemple, pour Docker, on trouve des images légères qui appellent périodiquement l’URL DuckDNS. Sur Home Assistant, une extension DuckDNS facilite l’intégration (y compris l’obtention d’un certificat SSL via Let’s Encrypt).
    
-   **Avantages** : extrême simplicité et légèreté. Pas de création de compte complexe : on s’authentifie via un compte GitHub, Google ou autre provider OAuth pour obtenir son token.  **Aucune confirmation mensuelle**  n’est requise : une fois configuré, le sous-domaine reste actif tant qu’il est régulièrement mis à jour. Le service supporte IPv6 également. DuckDNS n’impose pas de fréquence minimale ou maximale de mise à jour, mais typiquement un cron toutes les 5 minutes ou à chaque changement d’IP suffit. La communauté Homelab/auto-hébergement le recommande souvent pour sa fiabilité globale et son  **intégration facile à des projets DIY**  (automatisation, scripts).
    
-   **Inconvénients** : dépendance à un service gratuit géré par une petite équipe (disponibilité généralement bonne mais il y a eu de rares interruptions rapportées  ). Le choix du domaine est limité à  duckdns.org  (pas d’autres suffixes), ce qui peut faire moins « professionnel » ou être plus facile à deviner. DuckDNS n’offre pas de fonctionnalités annexes (pas de gestion de DNS complexes, pas de support client dédié – juste une FAQ et forum). Néanmoins, pour un usage personnel, ces limites sont très acceptables.
    

  

### **No-IP (service freemium, historique)**

  

**No-IP**  est l’un des plus anciens services de DNS dynamique (concurrent de DynDNS à l’époque). Il fonctionne sur un modèle freemium : il existe un niveau gratuit limité, et des offres payantes avec plus de fonctionnalités. Caractéristiques :

-   **Offre gratuite** : Permet aujourd’hui  **1 hostname actif**  (sous-domaine) sur un compte gratuit  . Historiquement, No-IP offrait 3 hôtes gratuits, mais les conditions ont changé ces dernières années. Le sous-domaine peut être choisi parmi une  **trentaine de noms de domaine proposés**  par No-IP (ex.  _ddns.net_,  _hopto.org_,  _zapto.org_, etc.). Cela donne un peu de diversité (si par exemple  _monserveur.hopto.org_  est pris, on peut essayer  _monserveur.ddns.net_, etc.).
    
-   **Expiration/confirmation** : La contrainte majeure de l’offre gratuite est la nécessité de  **confirmer périodiquement**que le hostname est toujours utilisé. Concrètement,  **tous les 30 jours**  No-IP envoie un email de confirmation à l’utilisateur, qui doit cliquer un lien pour éviter la désactivation de son DNS  . Si on ne confirme pas, l’hôte passe en état « Expired » puis « Redemption » et finit par être supprimé  . Cette politique vise à libérer les noms non utilisés, mais c’est une contrainte à ne pas négliger (ou bien il faut passer en offre payante pour s’en affranchir).
    
-   **Offre Enhanced/Premium** : No-IP propose des abonnements payants relativement abordables supprimant la confirmation mensuelle et autorisant plus d’hôtes (par ex. l’offre Enhanced ~5$/mois pour 5 hostnames, l’offre  **Plus**ou  **Professional**  pour davantage d’hôtes et son propre domaine, etc.  ). Les offres payantes permettent aussi d’utiliser des enregistrements DNS avancés (MX, TXT), un support client, etc.
    
-   **Clients et intégration** : No-IP étant ancien et répandu,  **de nombreux routeurs et NAS intègrent nativement un client No-IP**. Il suffit souvent d’entrer son username, mot de passe et nom d’hôte dans l’interface du routeur pour que celui-ci mette à jour l’IP automatiquement. No-IP fournit aussi son application de mise à jour (**DUC – Dynamic Update Client**) sur Windows/Linux, ainsi qu’un client en ligne de commande. L’API simple (protocole proche de DynDNS v2) permet d’utiliser des clients génériques (ddclient, inadyn, etc.).
    
-   **Avantages** : Fiabilité d’une société établie (No-IP revendique 100% de disponibilité DNS).  **Large choix de domaines**  gratuits qui peuvent être plus mémorables que les concurrents. Intégration plug-and-play sur beaucoup d’équipements. Possibilité d’évolution vers des offres avancées (par exemple, utiliser votre propre nom de domaine avec No-IP comme gestionnaire DNS dynamique : leur offre Plus gère jusqu’à 50 hôtes sur votre domaine personnalisé  ).
    
-   **Inconvénients** : La confirmation mensuelle est souvent vue comme une corvée (il ne faut pas oublier de cliquer sous peine de coupure). Un seul hôte gratuit limite l’utilisation multi-services (mais on peut contourner partiellement avec un wildcard, voir plus bas). Le service gratuit comporte de la publicité (emails de rappel, et auparavant les redirections web affichaient des bannières, bien que ce soit moins un problème pour un simple enregistrement A). Enfin, No-IP a pu être ciblé par certains blocages ou abus dans le passé (des malware utilisaient des hôtes No-IP, entraînant parfois des blocages temporaires de certains domaines No-IP par des FAI ou entreprises). Ce n’est pas commun, mais à savoir.
    

  

### **Autres alternatives notables**

  

En dehors de Cloudflare, DuckDNS et No-IP, il existe d’autres services DynDNS, chacun avec ses particularités :

-   **FreeDNS (Afraid.org)** : Service gratuit ancien et fiable, qui offre jusqu’à  **5 sous-domaines gratuits**  . Son point fort est le grand choix de domaines publics partagés (plus de 50) parmi lesquels choisir son sous-domaine. Par exemple  *.mooo.com  (très utilisé),  *.afraid.org, etc. L’interface est un peu rustique, mais c’est efficace. FreeDNS propose aussi une formule payante augmentant le nombre de sous-domaines (50 à 500) et offrant des options comme les wildcards et un branding sans référence à Afraid.org  .
    
-   **Dynu** : Un fournisseur moderne qui offre un service gratuit de DNS dynamique  **sans expiration**. D’après certaines ressources, la version gratuite de Dynu permettrait plusieurs hostnames (jusqu’à 30) mais avec certaines limitations  . Cependant, il semble qu’il faille créer un compte et que les fonctionnalités complètes (DNSSEC, plus de domaines, etc.) nécessitent un abonnement modique. À vérifier, car le comparatif IONOS mentionne une période d’essai de 7 jours pour la version gratuite  – ce point peut prêter à confusion. Néanmoins, Dynu est souvent cité comme alternative, avec client dédié et API.
    
-   **Securepoint DynDNS** : Service gratuit proposé par une entreprise allemande, limité à  **5 hôtes**  mais offrant un  **choix de 100 domaines**  différents  . Nécessite une inscription. Supporte IPv6 et fournit un “token” de mise à jour unique pour usage dans les clients. Plutôt destiné aux utilisateurs européens (site en allemand/anglais).
    
-   **YDNS** : Service gratuit (géré par une association allemande) offrant des domaines sous  ydns.eu  ou la possibilité d’utiliser son propre domaine. Avantage :  **supporte DNSSEC**  pour les domaines personnalisés. Nombre d’hôtes illimité annoncé  . Interface minimaliste, API possible.
    
-   **deSEC (dedyn.io)** : Solution  **non-profit basée en Allemagne**, orientée sécurité. deSEC offre gratuitement de gérer vos domaines avec DNSSEC et API, ainsi que des sous-domaines sous  dedyn.io. On peut y voir un  **véritable gestionnaire de DNS dynamique moderne**. Limite : ~15 domaines par compte (mais extensible sur demande)  . Pas de pub, pas d’offre payante, tout est financé par dons/recherche. Compatible directement avec les clients ACME (Let’s Encrypt) pour la validation DNS. C’est une alternative robuste et ouverte à DuckDNS/no-ip, pour utilisateurs avancés  .
    
-   **Dyn.com (Oracle DynDNS)** : Historiquement  _LE_  service DynDNS, très connu il y a 15 ans, mais il n’est plus disponible gratuitement depuis longtemps. Oracle (qui a racheté Dyn) a même annoncé l’arrêt complet du service grand public en 2022. Il n’est donc plus à considérer pour un nouvel usage.
    
-   **Autres** : citons encore  **Google Domains**  qui offrait un DDNS pour les domaines enregistrés chez eux (mais Google Domains est en cours de reprise par Squarespace en 2023-2024),  **OVH DynHost**  (pour les domaines chez OVH),  **Namecheap Dynamic DNS**  (pour domaines chez Namecheap) – ces services attachés aux registrars peuvent être utiles si vous avez acheté le domaine chez eux. Enfin, il existe la possibilité d’**auto-héberger son propre service DDNS**  (par exemple via un script PHP ou un petit serveur DNS avec mise à jour par requête HTTP), mais ceci dépasse le cadre de la comparaison – c’est faisable pour les makers cherchant un contrôle total.
    

  

**Wildcards et cas multi-sous-domaines** : Une question fréquente est la prise en charge des  **wildcards DNS**  (par exemple  *.maison.ddns.net). La plupart des services gratuits  **n’autorisent pas de wildcard sur un domaine gratuit**, ou seulement avec une offre payante (No-IP le permet en payant). Cloudflare permet des wildcards sur votre domaine, mais Let’s Encrypt par exemple nécessite un défi DNS pour obtenir un certificat wildcard. En pratique, si on veut exposer plusieurs services derrière une IP dynamique, deux approches : 1) mettre à jour  **plusieurs enregistrements A**  (ex:  _service1.mondomaine.com_,  _service2.mondomaine.com_  – Cloudflare API ou le client peut gérer plusieurs hostnames), ou 2) utiliser un nom d’hôte dynamique principal et faire des  **CNAME**  pour les autres. Par exemple, avec DuckDNS on a  maison.duckdns.org  mis à jour; on peut pointer  _nextcloud.mondomaineperso.com CNAME maison.duckdns.org_  – ainsi ce CNAME suivra l’IP dynamique indirectement. Cette méthode mixte est utile si on tient à utiliser un nom de domaine personnalisé sans gérer l’API soi-même.

  

## **Scénarios d’usage types avec le DNS dynamique**

  

Dans quel contexte a-t-on besoin de DNS dynamique ? Voici trois scénarios courants de la vie d’un homelab ou réseau domestique, illustrant l’usage du DDNS et les considérations associées.

  

### **1. Homelab derrière une IP dynamique (accès distant à un réseau domestique)**

  

Imaginons une personne qui héberge chez elle un  **serveur NAS**  ou des services web (site personnel, cloud privé type Nextcloud, caméra de surveillance, etc.). Son FAI lui attribue une IP publique dynamique qui change régulièrement (par exemple à chaque redémarrage de box, ou tous les X jours). Sans DNS dynamique, il faudrait connaître l’IP à jour et la communiquer à chaque fois – impraticable. Le DDNS résout ce problème : on configure un nom de domaine (par ex.  _mynas.duckdns.org_) et un client DDNS sur la box ou le NAS. Désormais, l’utilisateur peut accéder à son NAS en utilisant ce  **nom de domaine fixe**, que ce soit pour se connecter en SSH, en HTTP, ou pour toute autre application (ex: configurer un client de sauvegarde vers  mynas.duckdns.org). Le service DDNS garantit que ce nom pointe vers la bonne IP même après un changement.

  

Dans ce scénario, on  **configure le routeur pour rediriger les ports nécessaires**  vers le serveur interne. Par exemple, on peut ouvrir le port 443 vers le NAS pour accéder à l’interface web.  **Côté sécurité**, il est recommandé d’utiliser des certificats SSL (ex: via Let’s Encrypt, qui accepte les noms de domaine DuckDNS ou autres) pour chiffrer les connexions externes. Le nom de domaine dynamique permet justement d’obtenir et renouveler ces certificats automatiquement.

  

Si l’on souhaite un accès plus sécurisé, on pourrait n’exposer aucun port et passer par un VPN ou un tunnel. Par exemple, Tailscale (VPN maillé) ou Cloudflare Tunnel (agent qui sortant qui enregistre le service chez Cloudflare) sont des alternatives pour éviter des ouvertures de ports. Ces solutions utilisent aussi DNS (Cloudflare Tunnel fournit un nom en  _tunnels.dev_, Tailscale peut gérer des MagicDNS internes), mais elles sortent du cadre du DDNS classique. Retenons qu’un homelab accessible via DDNS doit être  **protégé comme un serveur en ligne** : mises à jour, pare-feu, mots de passe forts, etc., car il devient joignable depuis Internet par son nom.

  

### **2. Intégration avec un reverse proxy pour multi-services**

  

Beaucoup d’auto-hébergeurs déploient plusieurs services web sur la même IP (ex: serveur web, interface domotique, caméra IP, etc.). Grâce au DNS dynamique, on peut utiliser  **plusieurs sous-domaines**  qui pointent tous vers la même IP publique. Ensuite, un  **reverse proxy**  local (Nginx, Traefik, Caddy…) se charge de distribuer le trafic entrant vers le bon service en fonction du nom de domaine requis.

  

Concrètement, on peut avoir par exemple un nom de domaine dynamique principal  ma-maison.fr  (géré via Cloudflare ou autre). On crée des enregistrements pour  _nextcloud.ma-maison.fr_,  _camera.ma-maison.fr_,  _homeassistant.ma-maison.fr_  – tous pointent à l’adresse IP de la box. Sur le routeur, on a une redirection du port 443 vers la machine hébergeant le reverse proxy (ex: un Raspberry Pi). Ce reverse proxy voit la requête arrivée et, grâce au SNI du TLS ou à l’Host HTTP, fait suivre aux bons serveurs internes.

  

**Exemple illustratif** :  _« Supposons que vous ayez le domaine_ _dandreyd.com__. Vous pouvez créer des sous-domaines :_ _service1.dandreyd.com__,_ _service2.dandreyd.com__, etc., qui pointent tous vers l’IP publique (1.2.3.4) de votre serveur. Votre reverse proxy écoute sur cette IP et, lorsqu’une connexion arrive sur_ _service1.dandreyd.com__, il la route vers le service interne correspondant (par ex. 192.168.2.19) »_  . De cette façon,  **un seul point d’entrée**  (IP + port 443) peut desservir de multiples applications. Le DNS dynamique s’assure que  *.dandreyd.com  est toujours à jour vers l’IP du jour.

  

Avec Cloudflare DNS par exemple, on peut même activer le  **proxy Cloudflare (orange cloud)**  sur certains sous-domaines, ce qui ajoute une couche de cache/WAF (utile pour un site web public) et masque l’IP originelle. Cependant, pour des services personnels non web (Nextcloud privé, etc.), on utilise généralement le DNS de Cloudflare en mode DNS seulement (pas de proxy) pour éviter des complexités sur les flux non-HTTP.

  

**Certificats SSL** : Dans un tel setup, on obtient des certificats multi-domaines (ou wildcard) pour couvrir tous les sous-domaines. Let’s Encrypt via un challenge DNS (facile si on a Cloudflare) ou via challenge HTTP (possible en exposant temporairement chaque service ou via le proxy) peut automatiser cela. Par exemple, l’add-on DuckDNS de Home Assistant automatise la création d’un certificat pour le nom DuckDNS + un accès sécurisé.

  

En termes de sécurité, ce scénario profite du fait que  **chaque service n’est pas exposé sur un port distinct**  (on n’ouvre que 443, et éventuellement 80 pour les challenges, bien que avec le DNS challenge ce ne soit pas nécessaire). Le reverse proxy peut gérer l’authentification, forcer HTTPS, et appliquer des règles (rate limiting, firewall d’app, intégration fail2ban, etc.). On conseille de  **désactiver l’accès HTTP**  (redirection vers HTTPS) et de n’autoriser que TLS moderne sur le proxy.

  

### **3. Considérations de sécurité et bonnes pratiques**

  

Exposer son réseau domestique via un nom de domaine public nécessite quelques précautions :

-   **Mises à jour DNS sécurisées** : Toujours utiliser le  **chiffrement**  pour les requêtes de mise à jour DDNS. La plupart des services imposent HTTPS, mais si ce n’est pas le cas, préférez un client qui le supporte ou un tunnel. Par exemple, ddclient en mode Cloudflare utilise HTTPS par défaut  . Ceci évite qu’un attaquant sur le même réseau puisse intercepter votre token.
    
-   **Protection des identifiants** : Les services comme Cloudflare utilisent des API tokens – stockez-les en lieu sûr (fichier de config en permissions restreintes, etc.). De même pour les logins/mots de passe No-IP ou autres. Ne les mettez pas en dur dans un script public.
    
-   **Ports exposés minimalement** : N’exposez que le nécessaire. Par exemple, inutile d’ouvrir Telnet ou SSH au monde entier. Privilégiez un VPN pour l’administration distante. Si un service n’a pas vocation à être accédé publiquement, ne le mappez pas sur le reverse proxy.
    
-   **Utilisez HTTPS** : Grâce au DNS dynamique, vous avez un nom de domaine – profitez-en pour déployer du HTTPS via Let’s Encrypt ou un autre CA. De nombreux tutos existent pour coupler Let’s Encrypt avec DuckDNS (le DNS-01 challenge via DuckDNS est une possibilité, ou plus simplement le HTTP-01 si le service est accessible). Un navigateur ou une API appellera ainsi  https://monservice.maison:443  en toute confiance.
    
-   **Cloudflare en proxy** : Si vous utilisez Cloudflare DNS, évaluer l’option proxy (mode “orange”). Pour un blog personnel ou un petit site, cela peut apporter une couche de protection (Cloudflare filtre les attaques DDoS, offre un pare-feu applicatif de base). Par contre, pour accéder à un service non-HTTP (SSH, MQTT…), le proxy Cloudflare ne convient pas, on restera en connexion directe (ou on utilisera Cloudflare Tunnel spécifiquement pour certaines applications web).
    
-   **Mises à jour et monitoring** : Tenez à jour vos services exposés (un Nextcloud obsolète ou une caméra IP non patchée peuvent être compromis). Surveillez les logs, et éventuellement utilisez des outils comme Fail2Ban pour bannir les IP en cas de brute-force. Le DNS dynamique ne rend pas l’attaque plus facile en soi, mais il rend votre service plus  **visible sur Internet**  via un nom de domaine. D’ailleurs, attention, certains robots scannent les domaines *.duckdns.org ou  _.no-ip._  connus – vous pourriez voir du trafic malveillant générique. D’où l’importance des précautions classiques.
    

  

En résumé, un usage typique du DNS dynamique dans un cadre homelab implique une configuration combinant  **DDNS + redirections NAT + (optionnellement) reverse proxy + certificats SSL**. Cette combinaison permet de reproduire à l’échelle domestique ce qu’un hébergeur professionnel ferait, tout en contournant la limitation d’IP non fixe.

  

## **Serveurs DNS locaux (Unbound, Dnsmasq) et leur rôle en réseau domestique**

  

Abordons maintenant la question des  **DNS locaux** : au sein de votre réseau local, vous pouvez déployer un serveur DNS maison (par exemple avec  **Unbound**  ou  **Dnsmasq**). Quel est l’intérêt, et comment cela s’articule-t-il avec le DNS dynamique vu plus haut ?

  

### **Rôle d’un DNS local dans une architecture domestique**

  

Dans un réseau domestique typique, vos PC, smartphones et objets utilisent le DNS de l’opérateur ou un DNS public (Google, Cloudflare…) pour résoudre les noms de domaine externes. Installer un serveur DNS local (sur le routeur ou un Raspberry Pi) peut apporter plusieurs bénéfices :

-   **Cache DNS partagé** : Le serveur DNS local sert de  **cache**  pour toutes les requêtes des appareils. Les résolutions répétées (par ex.  www.google.com) seront accélérées car la réponse sera servie localement sans requêter à chaque fois un DNS externe. Cela  **améliore la latence**  perçue et réduit un peu la consommation de bande passante  .
    
-   **Résolution des noms locaux** : Un DNS local peut connaître les noms de vos machines internes (par exemple  _monpc.lan_  qui pointe vers 192.168.1.10). Sans DNS local, on se rabat sur des solutions comme mDNS (.local) ou WINS, pas toujours fiables ou universelles. Avec Dnsmasq ou Unbound configuré, on peut automatiquement enregistrer les baux DHCP dans le DNS : ainsi, toute machine du LAN peut pinguer  monpc.lan  et ça résout. Unbound sur OPNsense/pfSense, par exemple, a une option  _Register DHCP leases_  qui ajoute les machines DHCP au DNS  . Cela  **simplifie grandement l’administration** : plus besoin d’éditer les fichiers hosts de chaque PC.
    
-   **Override / Split-horizon DNS** : On peut définir des entrées DNS spécifiques pour certains noms. Très utile dans le cas du  **DNS dynamique**  justement. Supposons que depuis l’extérieur on accède à  _monnas.duckdns.org_  (résolu vers l’IP publique). Depuis l’intérieur du LAN, si on utilise ce même nom, on va sortir vers Internet pour revenir vers la box (hairpin NAT). Cela peut être inefficace, voire bloqué par certains routeurs. Avec un DNS local, on peut configurer une règle override disant :  _“monnas.duckdns.org = 192.168.1.50”_  (IP locale du NAS). Ainsi les clients internes accèdent directement, sans passer par la boucle externe. Unbound et Dnsmasq permettent ces  **overrides locaux**  facilement (sur OPNsense GUI, via “Host Overrides” ou “Domain Overrides”)  . On parle de DNS « split-horizon » lorsque l’interne et l’externe renvoient des IP différentes pour un même nom, optimisant l’accès selon la provenance.
    
-   **Filtrage et sécurité** : Un DNS local peut incorporer des  **listes de blocage**  (pour bloquer des domaines publicitaires, malveillants, etc.). Par exemple, Pi-hole (très populaire sur Raspberry Pi) utilise en interne Dnsmasq (ou son fork FTL) pour filtrer des domaines. Unbound peut aussi faire du filtrage via des zones nulles. Cela procure un  **bloqueur de pub/malware réseau**  centralisé. De plus, un serveur DNS local bien configuré peut activer  **DNSSEC**  pour vérifier l’authenticité des réponses DNS externes, et même chiffrer les requêtes sortantes via  **DNS over TLS (DoT)**si on le configure en forwarder TLS. Unbound supporte nativement DNSSEC et DoT  , ce qui permet d’éviter les attaques de cache poisoning et d’améliorer la confidentialité des requêtes (en empêchant un espion local ou FAI de voir toutes les requêtes en clair).
    
-   **Haute disponibilité locale** : Si votre accès Internet tombe, un DNS local garde en cache un certain nombre de domaines récemment résolus, permettant aux machines de continuer à se résoudre localement et parfois d’accéder à des ressources en cache (si par exemple un serveur local sert du contenu, etc.). Bon, ça reste un cas limité, mais disons que l’infrastructure locale ne dépend plus entièrement des DNS extérieurs.
    

  

### **Unbound vs. Dnsmasq : différences et complémentarité**

  

**Dnsmasq**  et  **Unbound**  sont deux solutions courantes pour le DNS local, souvent utilisées dans les firmwares routeurs et homelabs. Leurs rôles se recoupent partiellement, mais ils ont des philosophies différentes  :

-   **Dnsmasq**  est un  **DNS forwarder/cache léger**  qui fait aussi serveur DHCP. Il ne résout pas les requêtes récursives lui-même : il  **transmet les requêtes à un ou plusieurs DNS upstream**  (par exemple les DNS de l’opérateur ou de Cloudflare)  . Il se veut très léger en mémoire et conçu pour les routeurs/embarqués. Il excelle à distribuer les baux DHCP et à enregistrer automatiquement ces hôtes en DNS. En revanche, il ne valide pas DNSSEC tout seul, et ne chiffre pas les requêtes (il peut néanmoins être pointé vers un résolveur tiers qui fait DNSSEC). Son empreinte est minimale, ce qui le rend idéal pour de petits environnements ou équipements modestes.
    
-   **Unbound**  est un  **résolveur DNS récursif complet**  (comme BIND dans sa fonction récursive) qui peut interroger directement les serveurs racine, domaines de premier niveau, etc. Il  **n’a pas besoin de serveur DNS upstream**(sauf si on le configure en mode forwarder). Unbound fait de la  **validation DNSSEC**  en local et supporte les protocoles modernes (DNS over TLS, DNS64 pour IPv6, etc.)  . Il maintient un cache efficace et robuste. Depuis quelques années, il a été adopté par défaut sur OPNsense/pfSense car jugé plus sécurisé et respectueux de la vie privée (plus besoin de faire confiance à un DNS tiers)  . Unbound ne fait pas DHCP, mais il peut parfaitement récupérer les infos du serveur DHCP du système (voir plus haut) pour la résolution locale.
    

  

En termes de  **performances** : un résolveur complet comme Unbound peut être très rapide après le premier remplissage de cache, et évite des dépendances. Cependant, la première requête à un domaine peut être un peu plus longue (le temps de contacter les racines et autorités). Sur un réseau domestique, la différence est négligeable, d’autant qu’Unbound optimise et pipeline les requêtes. Dnsmasq, lui, dépend totalement de la latence vers le DNS amont (par ex. 20ms vers 1.1.1.1). Donc après cache, ils se valent, mais Unbound offre plus de contrôle et potentiellement une résolution plus directe  .

  

**Sécurité et fiabilité** : Comme mentionné, Dnsmasq fait confiance aux réponses de l’upstream – si votre FAI vous redirige certaines requêtes (portails captifs, censures, etc.), Dnsmasq s’y pliera. Unbound, en résolvant lui-même et en validant DNSSEC, garantit d’avoir les  **données officielles de la zone DNS**  . Il peut détecter une altération (via DNSSEC) ou éviter certaines censures (sauf si l’accès à certains serveurs DNS est bloqué, auquel cas on peut configurer Unbound pour passer par un tunnel ou DoT).

  

**Utilisation conjointe** : Il est possible d’utiliser  **les deux de concert**. Par exemple, Pi-hole v5 combine Dnsmasq (FTL) pour le filtrage + Unbound en arrière-plan comme résolveur, configuré de sorte que Dnsmasq forward tout à Unbound localement. Cependant, dans la plupart des déploiements routeur, on choisit l’un ou l’autre. pfSense/OPNsense proposent soit Unbound (DNS Resolver) soit Dnsmasq (DNS Forwarder) – mais pas les deux en même temps pour le même port. Étant donné les avantages,  **Unbound est généralement recommandé par défaut**  pour les réseaux modernes, sauf cas d’usage spécifique  . Dnsmasq reste utile si on a très peu de ressources ou si on veut une config ultra-simple.

  

En résumé  :

  

> _Unbound_ : résolveur récursif complet, rapide (direct, cache efficace), supporte DNSSEC/DoT, gestion avancée des overrides et blocages, idéal pour sécurité et performance.

> _Dnsmasq_ : forwarder DNS + DHCP, très léger, configuration simple, convient aux petits équipements ou si on veut simplement relayer vers un DNS public.

  

### **Mise en place dans un réseau domestique**

  

Dans la pratique, de nombreuses  **distributions routeur**  embarquent déjà ces services. Par exemple :

-   **OPNsense/pfSense** : utilisent Unbound par défaut comme DNS Resolver local (avec possibilité de switcher vers Dnsmasq si désiré). L’interface permet de définir des  _Host Overrides_  pour les noms locaux, activer l’enregistrement des baux DHCP  , etc. Si vous utilisez l’un de ces firewalls, vos clients DHCP reçoivent l’adresse du firewall comme DNS, profitant ainsi du cache local et des résolutions internes. Ces systèmes facilitent aussi la configuration de DNS over TLS (ex: forwarder vers Cloudflare chiffré) ou l’ajout de listes de blocage via des plugins.
    
-   **Routeurs grand public/firmware open-source** : Un routeur sous  **OpenWrt**  utilise Dnsmasq par défaut pour DNS+DHCP. Là aussi, les machines du LAN utilisent le routeur comme DNS. On peut configurer des noms locaux via /etc/hosts ou le DHCP statique. OpenWrt permet aussi d’ajouter Unbound en complément pour avoir DNSSEC (via le package  _unbound-host_). D’autres firmware comme ASUSWRT intègrent souvent une option DNSSEC ou DoT en forwarder, parfois en gardant dnsmasq pour le cache.
    
-   **Raspberry Pi / Pi-hole** : Installer Pi-hole donne un DNS local filtrant (basé sur Dnsmasq). Il suffit de pointer les DNS des postes vers l’IP du Pi-hole (ou configurer le DHCP pour qu’il annonce le Pi comme DNS). Ajoutez Unbound sur le Pi et vous avez un résolveur local complet sans dépendance externe, améliorant la confidentialité (c’est une configuration très prisée des bidouilleurs pour bloquer pubs et trackers sur tout le réseau).
    
-   **NAS Synology/QNAP** : Certains NAS offrent un package “DNS Server” (ex: Synology DNS Server) qui est souvent basé sur BIND ou dnsmasq, permettant de créer votre zone locale maison (par ex.  _maison.lan_  avec vos enregistrements) et de faire forward des requêtes externes. C’est une option pour centraliser le DNS si le routeur ne le fait pas bien.
    

  

Il est important de veiller à  **éviter les conflits** : ne pas avoir deux serveurs DNS sur le réseau prétendant gérer la même chose pour les clients. Généralement on en choisit un comme primaire (souvent le routeur). On peut en avoir un second en secours (par ex. un Pi-hole en secondaire), auquel cas il faut bien configurer pour qu’ils aient des vues cohérentes (surtout pour les noms locaux).

  

Enfin,  **DNS local et DNS dynamique**  peuvent se compléter : On a vu l’exemple du split-horizon pour le nom dynamique. On pourrait par exemple automatiser l’ajout dans Unbound de l’enregistrement correspondant au DDNS. Sur pfSense, ceci peut être fait via un script ou en utilisant l’option RFC 2136 (mise à jour DNS vers un serveur DNS interne). Cependant, le plus simple est souvent de configurer manuellement un override statique si l’IP locale est fixe. Autre intégration : si votre DNS local est configuré pour forwarder vers un DNS public, vous pouvez paramétrer un forward conditionnel pour votre domaine dynamique vers un serveur spécifique. Mais cela devient complexe inutilement pour un usage maison. Dans la plupart des cas, une entrée statique locale suffira pour que  _monserveur.domaine.com_  résolve différemment en LAN et en WAN.

  

## **Guide de configuration : clients DDNS et intégration routeur**

  

Pour terminer, voici quelques conseils et options pour  **mettre en œuvre concrètement le DNS dynamique**  et les DNS locaux dans votre réseau.

  

### **Clients et scripts pour le DNS dynamique**

-   **Via le routeur/Firewall** : Si vous possédez un routeur type  **pfSense, OPNsense, OpenWrt**, utilisez sa fonctionnalité DDNS intégrée. Sur pfSense, rendez-vous dans  _Services > Dynamic DNS_  et ajoutez une entrée en choisissant votre fournisseur (Cloudflare, DuckDNS, No-IP, etc. sont proposés). Entrez les identifiants requis (par ex. pour Cloudflare,  _Email_  +  _Global API Key_  ou  _Token_  selon versions). Le routeur se chargera de détecter les changements d’IP (via l’interface WAN ou un service web) et de mettre à jour le DNS  . Sur OPNsense, après avoir installé le plugin os-ddclient, la configuration est similaire : vous pouvez définir l’intervalle de vérification (par ex. toutes les 5 minutes) et le service (Cloudflare, DuckDNS…). Par exemple, pour DuckDNS, vous sélectionnez  _DuckDNS_, en  _Hostname_  mettez  monhote.duckdns.org, et en mot de passe le token fourni par DuckDNS (le champ  _Username_  pouvant rester vide si non requis). Le routeur étant toujours allumé et conscient de son IP WAN, c’est souvent la solution la plus robuste.
    
-   **Via un NAS ou serveur Linux** : Si vous avez une machine toujours allumée, installer un client comme  **ddclient**  est recommandé. ddclient est un démon Perl très flexible qui supporte la plupart des fournisseurs (No-IP, Cloudflare, DuckDNS, Dyn, etc.)  . On le configure via  /etc/ddclient.conf. Par exemple, pour Cloudflare : préciser le protocol=cloudflare, le token API en password, le nom de zone et le record à mettre à jour  . Une fois configuré, ddclient tourne en tâche de fond et fait les updates. Sur Docker, on peut utiliser l’image  _linuxserver/ddclient_  qui simplifie le déploiement  . Pour DuckDNS, un script shell simple lancé par cron fait très bien l’affaire (appel wget/curl de l’URL duckdns toutes les 5 minutes).
    
-   **Clients propriétaires** : No-IP propose son utilitaire  _No-IP DUC_. Il suffit de l’exécuter et de fournir ses identifiants No-IP – il va tourner en background. D’autres services ont des clients dédiés (Dyn avait  _inadyn_, etc.). À noter : inadyn existe toujours en open-source et peut servir de client universel, mais ddclient est plus répandu désormais.
    
-   **Automatisation logicielle** : Certains logiciels embarquent des fonctions DDNS. Par exemple,  **Home Assistant**  a un add-on DuckDNS qui non seulement met à jour l’IP mais configure aussi le certificat Let’s Encrypt – pratique pour un accès à l’interface HA depuis l’extérieur.  **Ubiquiti Unifi**  routeurs/UDM ont une section DDNS (ils supportent No-IP, DuckDNS, DynDNS, etc.). Même Windows Server a un client DynDNS intégré si besoin. Pensez à vérifier si l’un de vos équipements actuels ne pourrait pas faire le job avant d’ajouter un énième script.
    

  

### **Exemples de configuration rapides**

-   **DuckDNS + script shell (Linux)** : créez un script  /usr/local/bin/duckdns.sh  contenant par ex.:

```bash
TOKEN="Votre-Token-DuckDNS"
DOMAINS="monhote"
IP=$(curl -s https://api.ipify.org)  
curl "https://www.duckdns.org/update?domains=$DOMAINS&token=$TOKEN&ip=$IP"
```
-   Ajoutez une tâche cron (*/5 * * * * /usr/local/bin/duckdns.sh >/dev/null 2>&1). Ceci va mettre à jour toutes les 5 min. (DuckDNS renvoie “OK” ou “KO” selon succès).
    
-   **Cloudflare + curl** : une commande curl pour mettre à jour Cloudflare (via API v4) ressemble à :

```bash
curl -X PUT "https://api.cloudflare.com/client/v4/zones/<ZONE_ID>/dns_records/<RECORD_ID>" \
     -H "Authorization: Bearer <API_TOKEN>" \
     -H "Content-Type: application/json" \
     --data '{"type":"A","name":"sub.mondomaine.com","content":"<new_ip>","ttl":120}'
```

-   Où  <ZONE_ID>  et  <RECORD_ID>  sont des identifiants que vous obtenez via l’API ou l’interface (une fois pour config). C’est un peu lourd à trouver manuellement, d’où l’intérêt d’outils comme ddclient qui le font automatiquement à partir du nom de domaine.
    
-   **OPNsense + Cloudflare (exemple)** : Installer le plugin os-ddclient. Dans  _Services > Dynamic DNS_, ajouter une entrée : Service = Cloudflare. Remplir  _Zone_  =  mondomaine.com  (domaine géré),  _Hostname(s)_  = le(s) sous-domaine(s) à mettre à jour (ex:  maison,minecraft  pour  maison.mondomaine.com  et  minecraft.mondomaine.com),  _Username_  =  token,  _Password_  = le token API Cloudflare. Choisir  _Check IP Method_  = “Interface” (ou un service web) et l’interface WAN. Sauvegarder, activer. Le log permettra de vérifier la bonne mise à jour  .
    
-   **pfSense + DuckDNS (exemple)** : Dans  _Services > Dynamic DNS_, ajouter : Service = Custom (car pfSense peut ne pas avoir DuckDNS nativement selon version). Fournir l’URL de mise à jour DuckDNS contenant  %IP%  et les paramètres. Heureusement, des tutoriels existent et pfSense 2.5+ intègre DuckDNS dans la liste déroulante (ce qui simplifie : on entre juste le domaine et le token en guise de mot de passe). On valide et on observe le statut.
    

  

### **Mise en place d’un DNS local**

-   **OPNsense/pfSense** : Le DNS Resolver (Unbound) est probablement déjà actif. Vérifiez dans  _Services > Unbound DNS > General_  que  _Enable_  est coché. Activez  _DHCP Registration_  si vous voulez que les clients DHCP soient résolus  . Vous pouvez aussi définir un  _Domain Override_  si, par exemple, vous avez un domaine local spécifique ou si vous voulez que les requêtes vers  .local  ou autre aillent vers un autre serveur. Dans  _Access Lists_, assurez-vous que le réseau LAN est autorisé (par défaut oui). Résultat : toutes les machines qui utilisent l’IP du routeur comme DNS profiteront du cache et des noms locaux.
    
    Pour tester, faites un  nslookup monpc.lan 192.168.1.1  (si 192.168.1.1 est pfSense) – cela doit renvoyer l’IP de monpc si ce dernier a communiqué son nom via DHCP.
    
-   **Configurer DNSSEC/DoT** : Sur Unbound (OPNsense), DNSSEC est activé d’un clic  . Pour DNS over TLS, soit on utilise Unbound en mode resolver pur (pas besoin, il interroge les racines en clair mais on a DNSSEC), soit en mode forwarder chiffré : dans  _Query Forwarding_, on peut activer Forwarding et spécifier des serveurs TLS comme 1.1.1.1@853 avec validation. Cela chiffre les requêtes vers l’extérieur, au prix de repasser par un resolver tiers. C’est un choix. Pour Dnsmasq, on ne peut pas faire DoT nativement, il faudrait passer par un stub comme  stubby  ou utiliser le DNS over HTTPS du routeur (certains firmwares ASUS le font).
    
-   **Intégration Pi-hole + Unbound** : Installer Pi-hole (qui remplace votre DNS du routeur auprès des clients) puis suivre la doc pour ajouter Unbound. En gros, Unbound écoute sur 127.0.0.1:5335 et Pi-hole forwarde tout vers lui. On obtient un DNS local, filtrant et totalement récursif. C’est excellent pour la vie privée et le confort (plus de pubs, etc.).
    
-   **Accès aux services locaux par nom** : Si vous n’avez pas de DNS local mais que vous le souhaitez sans trop d’effort, vous pouvez par exemple  **utiliser Dnsmasq sur votre PC**  (utile si vous voulez juste quelques noms sans configurer tout le réseau). Mais le mieux reste de faire gérer cela par le routeur ou un petit serveur central comme décrit ci-dessus.
    

  

En conclusion, un  **serveur DNS local**  bien configuré complète idéalement l’utilisation d’un  **DNS dynamique**. Le DDNS vous donne une porte d’entrée fixe depuis Internet, tandis que le DNS local vous offre un contrôle et un confort à l’intérieur du réseau. En combinant les deux, vous pouvez par exemple taper  _cloud.maison.local_  depuis chez vous et  _cloud.mondynamique.net_  depuis l’extérieur, et atterrir sur le même service, de manière optimale dans chaque contexte.

  

## **Conclusion**

  

Le DNS dynamique est une brique essentielle pour toute personne hébergeant des services sur une connexion à IP variable. Des services comme Cloudflare, DuckDNS ou No-IP offrent des solutions variées – du  **DIY sur son propre domaine avec API**  à la  **solution prête à l’emploi totalement gratuite**  – chacun ayant ses avantages. Il est important de choisir en fonction de ses besoins : Cloudflare conviendra à ceux qui ont un domaine et veulent une intégration poussée (par exemple coupler avec un CDN ou d’autres règles DNS), DuckDNS séduira par sa simplicité sans tracas, No-IP par son intégration universelle, etc. Les alternatives non-propriétaires comme deSEC, FreeDNS, YDNS montrent qu’il existe un écosystème ouvert pour le DDNS, ce qui est rassurant en termes de pérennité.

  

En parallèle, soigner son  **infrastructure DNS locale**  apporte une meilleure expérience utilisateur et plus de maîtrise du réseau. Un  **DNS resolver local (Unbound)**  apportera sécurité (DNSSEC, pas de tracking FAI) et indépendance, là où un  **DNS forwarder (Dnsmasq)**  apportera flexibilité et légèreté. Souvent, les deux ne sont pas en opposition stricte mais répondent à des usages légèrement différents – à vous de voir ce qui s’intègre le mieux dans votre architecture.

  

Enfin, n’oublions pas que ces outils ne sont efficaces qu’accompagnés des bonnes pratiques de réseau : tenir ses services à jour, utiliser le chiffrement, segmenter les accès si nécessaire (un service critique n’a peut-être pas besoin d’être joignable par tout le monde), et surveiller son système. Avec un DDNS bien configuré, un reverse proxy, et un DNS local, votre homelab se comportera presque comme une infra professionnelle – en miniature – et vous donnera le contrôle total sur vos services,  **à la maison comme à distance**.
