# **Authentification centralisée et MFA dans un homelab**

  

Dans un  **homelab**  self-hosted, disposer d’un système d’authentification centralisée robuste avec MFA (authentification multifacteur) permet de sécuriser efficacement l’accès à l’ensemble des services internes (NAS, serveurs, applications web, etc.). Cette documentation compare plusieurs solutions open-source (Keycloak, Authentik, Authelia, LemonLDAP::NG…), détaille les méthodes d’authentification/MFA disponibles (TOTP, WebAuthn/U2F, SMS, push, YubiKey, etc.), propose des cas d’usage concrets (Vaultwarden, Nextcloud, Proxmox, Grafana…) et décrit des architectures types avec reverse proxy (NGINX Proxy Manager, Traefik) et SSO via OIDC/LDAP, ainsi que des exemples de configuration (docker-compose, YAML, règles de proxy, options de cookies, etc.). Les explications sont techniques mais pédagogiques, adaptées à un ingénieur cybersécurité expérimenté. Nous restons strictement sur des solutions self-hosted, sans recours à des services SaaS.

  

## **Solutions open-source de gestion d’identité et SSO**

  

Plusieurs projets proposent de centraliser l’authentification en mode self-hosted. Voici une comparaison technique :

-   **Keycloak**  – Solution mature gérée par Red Hat. Propose une interface d’administration riche et un  **gestionnaire d’identités**  complet. Supporte nativement  **OAuth2 / OpenID Connect (OIDC)**  et  **SAML**, avec prise en charge d’une base d’utilisateurs interne ou de fédération LDAP/AD  . Keycloak gère aussi les MFA (TOTP, WebAuthn/FIDO2 avec YubiKey, etc.) et les logins sociaux (Google, GitHub…)  . On peut l’exécuter en conteneur Docker ou sur Kubernetes, avec bases SQL pour stocker les données  . Keycloak est extensible : on peut ajouter des extensions Java, thèmes personnalisés, ou connecteurs RADIUS via des extensions communautaires  .
    
-   **Authentik**  – IdP open-source récent, moderne et “conteneur-friendly”  . Authentik fournit à la fois une interface web d’admin conviviale et une API. Il supporte  **OIDC, OAuth2 et SAML**  pour les applications clientes, et peut fédérer vers un annuaire LDAP existant  . Authentik met l’accent sur les flux d’authentification dynamiques et l’« adaptive MFA ». Il gère le  **MFA**  par TOTP,  **WebAuthn/passkeys**  (FIDO2) et même l’authentification par push ou static tokens (codes de secours)  . Il ne prend pas en charge RADIUS en natif, mais peut s’intégrer via un proxy externe. Authentik est conçu pour tourner sur Docker/Kubernetes (base PostgreSQL), avec une documentation claire.
    
-   **Authelia**  – « portail SSO » spécialisé pour être associé à un reverse-proxy existant  . Authelia est un serveur d’**authentification multi-facteur (IAM)**  et SSO qui se place en « gatekeeper » devant les applications web. C’est un fournisseur  **OIDC**  certifié OpenID™  , avec support des logins via cookies de session, headers de confiance ou OIDC. Authelia mise sur la légèreté (conteneur <20 Mo) et la rapidité. Il propose MFA/TOTP,  **WebAuthn (FIDO2)**, notifications push et passkeys  , ainsi qu’une gestion fine des politiques d’accès. Il se déploie généralement en conteneur unique associé à un proxy (Traefik, Nginx, etc.) et peut utiliser Redis/Mongo pour les sessions, LDAP pour le backend utilisateur, etc. Authelia ne cherche pas à remplacer un annuaire LDAP complet mais sert de point d’authentification central. Il fournit une UI simple pour le flux de login et de MFA  .
    
-   **LemonLDAP::NG**  – Plateforme AAA (authentification, autorisation, audit) française, très complète et modulaire  . LemonLDAP::NG propose un  **portail SSO**  avec support natif de nombreux backends d’authentification (LDAP, AD, Kerberos, base de données, certificats SSL, OAuth, etc.)  . Il implémente  **SAML, CAS et OpenID Connect**(flux standard OIDC/OAuth2)  . On peut l’utiliser comme IDP SAML/OIDC, ou comme passerelle entre protocoles (ex. CAS→SAML, etc.). LemonLDAP gère la MFA via des modules variés : TOTP/HOTP (compatible FreeOTP, Google Authenticator), WebAuthn (FIDO2), OTP Yubico et plus (e-mail, RADIUS, Okta, REST externe…)  . L’interface d’admin (Manager) est complète mais plus complexe à prendre en main. LemonLDAP::NG est très extensible (plugins Perl, règles avancées) et supporte Docker/Kubernetes pour l’UI, le portail et la base de config.
    
-   **Autres solutions**  – On peut aussi citer d’autres projets comme GLUU (mature en entreprise) ou Ory/Hydra pour OIDC, voire Apereo CAS. Mais pour un homelab, les solutions ci-dessus sont en général suffisantes.
    

  

En synthèse,  **Keycloak**  et  **Authentik**  sont de véritables fournisseurs d’identité (IdP) supportant SSO OIDC/SAML et de multiples méthodes MFA;  **Authelia**  joue plutôt le rôle d’authentificateur « front-end » pour un reverse proxy, apportant MFA/TOTP/WebAuthn sans fournir de service annuaire complet;  **LemonLDAP::NG**  est une suite SSO très polyvalente multi-protocoles, orientée proxy et flux d’authent. Les différences se trouvent dans la facilité de configuration (Keycloak/Authentik offrent UI modernes, Authelia et LemonLDAP demandent plus de configuration manuelle), l’intégration dans Docker (Keycloak et Authentik prêts pour conteneurs, LemonLDAP plus traditionnel), et l’écosystème (communauté, plugins, support entreprise)  .

  

## **Méthodes d’authentification et MFA supportées**

  

Les systèmes ci-dessus supportent plusieurs  **méthodes d’authentification**  et de  **double authentification (2FA/MFA)**courantes :

-   **TOTP (Time-based One-Time Password)**  : codes à usage unique générés par application mobile (Google Authenticator, FreeOTP, Authy…). Avantages : simple, ne requiert pas de réseau, bien supporté. Inconvénients : vulnérable au phishing, nécessite synchronisation horaire correcte. Presque tous les IdP/SSO (Keycloak, Authelia, Authentik, LemonLDAP) intègrent le TOTP en MFA  . Par exemple, Keycloak propose une « authenticator app » TOTP intégrée  , et Authelia gère les OTP via TOTP dans sa configuration.
    
-   **WebAuthn / FIDO2 (incluant U2F)**  : authentification par « clés de sécurité » (YubiKey, biométrie mobile, etc.) selon la norme W3C Web Authentication. Avantages : extrêmement résistante au phishing (la clé crypto prouve la légitimité du site), UX fluide (p.ex. passkeys sur smartphone), peut remplacer le mot de passe. Limites : exige un appareil supporté (clé USB/NFC ou smartphone/ordinateur avec TPM), plus complexe à déployer. Keycloak supporte WebAuthn comme deuxième facteur (même utilisé comme facteur primaire sans mot de passe)  . Authelia inclut le support WebAuthn (passkeys et U2F)  . Authentik le supporte également et le recommande (c’est le seul facteur utilisable en tant que « passwordless » primaire)  . LemonLDAP::NG offre un plugin WebAuthn (attestation simple ou avancée) et peut migrer des clés U2F existantes vers WebAuthn  . Les clés matérielles YubiKey bénéficient de ces systèmes : en mode OTP (monodirectionnel) ou comme clé FIDO2 (WebAuthn/U2F).
    
-   **YubiKey OTP**  : Jetons OTP spécifiques de Yubico (1 OTP par toucher). Avantages : méthode éprouvée, résilience contre attaque en ligne, fonctionne hors-ligne. Inconvénients : stocker secret Yubikey (sécurité), format propriétaire. LemonLDAP::NG fournit un module « Yubico OTP » pour intégrer directement les clés Yubico en second facteur  . (Cependant, comme WebAuthn couvre déjà FIDO/U2F, l’OTP de YubiKey est moins utilisé).
    
-   **SMS OTP**  : codes envoyés par SMS. Avantages : large compatibilité (aucune appli nécessaire). Inconvénients sérieux : vulnérable au SIM swapping, chiffrement SMS faible, peut être intercepté. Authentik et Authelia peuvent proposer le SMS en MFA (Authentik l’intègre en option, Authelia peut utiliser Twilio ou un fournisseur GSM)  . C’est souvent déconseillé en priorité, mais utile en secours.
    
-   **Notification Push mobile**  : authentification par notification sur smartphone (ex. Duo Mobile). Avantages : très pratique, sécurité renforcée (requiert contrôle de l’appareil). Inconvénients : dépend d’un service push (souvent externe). Authelia supporte les notifications push (il mentionne explicitement “Mobile Push Notifications” comme facteur supporté)  . Authentik n’a pas de push intégré nativement (on peut passer par Duo ou WebAuthn mobile).
    
-   **Autres facteurs**  : En plus du mot de passe classique, on peut avoir des clés statiques (codes de secours), OTP via e-mail, RADIUS externe, certificats TLS client, etc. Par exemple, LemonLDAP propose un module e-mail 2FA, et l’authentification par annuaire (LDAP/AD) ou par certificat X.509 est possible comme facteur.
    

  

Chaque méthode a ses usages : le  **TOTP**  reste standard pour un homelab,  **WebAuthn**  est recommandé pour sa résistance au phishing (p. ex. connectez une YubiKey ou un smartphone pour vous authentifier)  . Les solutions self-hosted intègrent en général ces MFA de base. Par exemple, Keycloak et Authentik autorisent TOTP et WebAuthn par défaut  ; Authelia propose TOTP et WebAuthn/Passkeys et même push  ; LemonLDAP autorise TOTP, WebAuthn et YubiKey OTP  . Lors de l’intégration, on configurera le flux d’authentification pour exiger un second facteur en fonction du profil ou de la politique (par exemple « 2FA nécessaire hors du réseau local »).

  

## **Cas d’usage dans un homelab**

  

Voici comment ces solutions et méthodes peuvent protéger des services typiques d’un homelab :

-   **Vaultwarden (Bitwarden)**  : Vaultwarden supporte l’authentification OIDC en tant que client. On peut configurer Vaultwarden pour déléguer l’authentification à un IdP tel qu’Authelia ou Keycloak. Par exemple, avec Authelia on définit des variables d’environnement pour activer le SSO et renseigner le client OIDC (client_id, secret, scopes, URL de redirection)  . Ainsi, l’utilisateur se connecte via Authelia (avec MFA) puis Vaultwarden reçoit les jetons OIDC. Authelia fournit par exemple un guide d’intégration montrant l’activation du SSO dans Vaultwarden (voir variables  SSO_ENABLED=true,  SSO_CLIENT_ID=..., etc.)  .
    
-   **Nextcloud**  : Nextcloud peut être configuré comme  **SP SAML**  ou  **RP OIDC**. Avec Keycloak ou Authentik, on active le plugin “OpenID Connect Login” ou “SSO/SAML”. Nextcloud demandera les attributs utilisateurs (nom, e-mail, identifiant, éventuellement groupe/quota). Par exemple, la documentation d’Authentik décrit l’usage des scopes OIDC (email,  profile,  openid) et de mappings d’attributs (groupes, quotas) via une « Property Mapping » pour Nextcloud  . Un point important : Nextcloud ne peut plus chiffrer côté serveur si on n’utilise pas LDAP (le mot de passe n’est pas fourni en clair), donc il est recommandé d’utiliser LDAP pour le chiffrement ou bien de l’abandonner si on fait du SSO OIDC/SAML  . L’intégration est largement documentée, et Nextcloud créera localement l’utilisateur après connexion unique.
    
-   **Proxmox VE**  : Proxmox intègre un  **auth source**  OIDC à partir de la version 7+. On peut ajouter un « realm » OIDC via la commande  pveum realm add ... --type openid  en précisant l’URL du serveur d’identités (Issuer URL), l’ID et secret du client, le claim d’utilisateur (par défaut  username), et souvent l’option  --autocreate 1  pour créer automatiquement les utilisateurs lors du premier login  . Par exemple, configurer Keycloak : on crée un client OIDC “Proxmox” dans Keycloak (p. ex. ID “pve”), puis sur chaque nœud Proxmox :
```yaml
pveum realm add keycloak --type openid \
   --issuer-url https://auth.domaine/realms/homelab \
   --client-id pve --client-key <secret> \
   --username-claim username --autocreate 1
```

pveum realm add keycloak --type openid \
   --issuer-url https://auth.domaine/realms/homelab \
   --client-id pve --client-key <secret> \
   --username-claim username --autocreate 1
```ini
[auth.generic_oauth]
enabled = true
name = Keycloak
allow_sign_up = true
client_id = VOTRE_ID_CLIENT
client_secret = VOTRE_SECRET
scopes = openid email profile offline_access roles
email_attribute_path = email
login_attribute_path = username
auth_url = https://<IDP>/realms/homelab/protocol/openid-connect/auth
token_url = https://<IDP>/realms/homelab/protocol/openid-connect/token
api_url   = https://<IDP>/realms/homelab/protocol/openid-connect/userinfo
```

-   Cet exemple (adapté de la doc officielle Grafana) montre comment pointer Grafana vers un IdP Keycloak  . Grafana obtient alors le login via Keycloak et peut mapper les rôles Keycloak sur ses rôles internes. Un processus similaire existe pour d’autres outils (Chronograf, Kibana, etc.).
    
-   **Autres services**  : On peut de la même façon protéger Bitwarden, Paperless, Seafile, Vault, Homarr, etc. grâce aux intégrations OIDC ou via le proxy. Par exemple, Imihc (gestion de photos), Uptime Kuma, etc. possèdent souvent une option « Authentification OIDC » ou bien on les place derrière Authelia.
    

  

En résumé,  **tous ces services bénéficient d’une couche SSO/MFA commune**. Plutôt que de gérer un utilisateur local par application, l’authentification est externalisée vers l’IdP : l’utilisateur se logue une fois (avec MFA) sur l’IdP, puis accède à toutes les applis associées sans resaisir son mot de passe.

  

## **Architectures type avec reverse proxy et SSO**

  

Une architecture fréquente dans un homelab consiste à placer un  **reverse proxy**  (NGINX, NGINX Proxy Manager, Traefik, Caddy, etc.) devant les applications, et à déléguer l’authentification à un service SSO/MFA. Voici deux schémas courants :

-   **NGINX Proxy + Authelia (auth_request)**  : NGINX joue le rôle de proxy frontal. On utilise la directive  auth_request  pour que NGINX interroge Authelia avant de laisser passer la requête. Par exemple, dans une configuration de site NGINX:

```nginx
location / {
  auth_request /internal/authelia/authz;
  ...
}
location = /internal/authelia/authz {
  proxy_pass http://authelia:9091/api/verify?rd=https://$host$request_uri;
  internal;
}
```
-   Dans cet exemple, chaque accès à l’application déclenche un appel interne à Authelia (/internal/authelia/authz). Authelia vérifie la session/MFA et renvoie 200 (si autorisé) ou 401 (sinon). NGINX peut alors rediriger vers la page de login Authelia si nécessaire. On récupère aussi les en-têtes  Remote-User/Groups/Email  avec  auth_request_setpour transmettre l’identité à l’application protégé  . L’intégration NGINX/Authelia nécessite le module  ngx_http_auth_request_module. Ce schéma permet d’authentifier n’importe quelle application web (même sans support OIDC natif) via Authelia. L’exemple officiel d’Authelia montre un usage d’auth_request /internal/authelia/authz;  et la récupération des variables  $remote_user,  $remote_groups  après authentification  .
    
-   **Traefik + Authelia (Forward Auth Middleware)**  : Avec Traefik v2+, on configure Authelia comme  **middleware de forward auth**. On définit dans la stack Docker des labels Traefik pour activer l’authentification. Par exemple (dans un  docker-compose.yml) :
```yaml
services:
  authelia:
    image: authelia/authelia
    ...
    labels:
      # Route Authelia elle-même
      - traefik.enable=true
      - traefik.http.routers.authelia.rule=Host(`auth.example.com`)
      - traefik.http.routers.authelia.entrypoints=https
      # Définition du middleware Authelia (forwardAuth)
      - traefik.http.middlewares.authelia.forwardAuth.address=http://authelia:9091/api/authz/forward-auth
      - traefik.http.middlewares.authelia.forwardAuth.trustForwardHeader=true
      - traefik.http.middlewares.authelia.forwardAuth.authResponseHeaders=Remote-User,Remote-Groups
  app-secure:
    image: traefik/whoami
    labels:
      - traefik.enable=true
      - traefik.http.routers.whoami-secure.rule=Host(`whoami-secure.example.com`)
      - traefik.http.routers.whoami-secure.entrypoints=https
      - traefik.http.routers.whoami-secure.middlewares=authelia@docker
networks:
  proxy:
  authelia:
```

-   Dans cet exemple, Traefik expose Authelia sur  auth.example.com. Le service  app-secure  (ici  whoami-secure) est protégé par le middleware  authelia@docker  . Ce middleware appelle l’endpoint Authelia (/api/authz/forward-auth) pour chaque requête. Si l’utilisateur n’est pas authentifié, il est redirigé vers la page de login Authelia. Cette configuration montre l’usage de  traefik.http.middlewares.authelia.forwardAuth.address: 'http://authelia:9091/api/authz/forward-auth'  pour définir l’authentificateur  . C’est un schéma très modulable : toutes les applications derrière Traefik peuvent être protégées ainsi (avec labels similaires).
    
-   **SSO via OIDC/LDAP**  : Alternativement, certaines applications peuvent directement se connecter à un IdP OIDC ou à un annuaire. Par exemple, Nextcloud peut se lier en LDAP à Keycloak/FreeIPA, ou agir en SP SAML/OIDC avec Keycloak. Dans ce cas, le “reverse proxy” ne fait que la redirection initiale ; l’authentification se fait dans l’application via l’IdP. Par exemple, un flux OIDC classique : l’utilisateur accède à l’app (via proxy), l’app le redirige vers Keycloak pour login, puis Keycloak redirige de retour avec un code. Ce modèle requiert que l’application cliente supporte OIDC/SAML (Grafana, Nextcloud, Jenkins, etc. ont ce support).
    

  

Ces architectures peuvent être combinées : ex. Keycloak (OIDC IdP) + Traefik + Authelia (pour les apps sans support OIDC). L’idée clé est d’avoir  **un point unique d’authentification**  (avec MFA) et des règles de proxy/redirection centralisées.

  

## **Exemples de configuration**

  

Voici quelques extraits concrets :

-   **Docker Compose (Authelia + Traefik)**  – Exemple simplifié tiré d’un guide officiel  :
```yaml
version: '3.8'
services:
  traefik:
    image: traefik:latest
    command: --providers.docker
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.dashboard.rule=Host(`traefik.example.com`)"
      - "traefik.http.routers.dashboard.entrypoints=https"
      - "traefik.http.routers.dashboard.middlewares=authelia@docker"
  authelia:
    image: authelia/authelia:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.authelia.rule=Host(`auth.example.com`)"
      - "traefik.http.routers.authelia.entrypoints=https"
      - "traefik.http.middlewares.authelia.forwardAuth.address=http://authelia:9091/api/authz/forward-auth"
      - "traefik.http.middlewares.authelia.forwardAuth.trustForwardHeader=true"
      - "traefik.http.middlewares.authelia.forwardAuth.authResponseHeaders=Remote-User,Remote-Groups"
    volumes:
      - ./authelia/config:/config
networks:
  proxy:
    external: true
  authelia:
```

-   Ici,  **Authelia**  est exposée sur  auth.example.com  (router Authelia) et on définit un  **middleware**  Authelia (forwardAuth) utilisé par le router  dashboard. Le middleware appellera  http://authelia:9091/api/authz/forward-auth. Ce schéma est similaire à l’exemple de l’article Authelia  .
    
-   **Configuration NGINX (auth_request)**  – Extrait type (d’après la documentation Authelia) pour protéger un site Nextcloud :
```nginx
server {
  listen 443 ssl;
  server_name cloud.example.com;
  # TLS config omitted...
  # Auth Request vers Authelia
  location / {
    auth_request /internal/authelia/authz;
    auth_request_set $user  $upstream_http_remote_user;
    auth_request_set $email $upstream_http_remote_email;
    error_page 401 =302 https://auth.example.com/?rd=$request_uri;
    # ...
    proxy_pass http://nextcloud_backend;
  }
  location = /internal/authelia/authz {
    internal;
    proxy_pass http://authelia:9091/api/verify?rd=https://cloud.example.com$request_uri;
  }
}
```
-   Ce snippet montre l’usage de  auth_request /internal/authelia/authz;  pour déléguer l’authentification à Authelia  . Les directives  auth_request_set  capturent l’UID  $remote_user  et d’autres en-têtes renvoyés par Authelia. En cas de non-autorisation (code 401), on redirige vers  auth.example.com  pour se logger (paramètre  rd  = redirect après login). Cet exemple est inspiré de la configuration recommandée d’Authelia pour NGINX  .
    
-   **Configuration Grafana (OIDC)**  – Extrait du fichier  grafana.ini  pour Keycloak  :

```ini
[auth.generic_oauth]
enabled = true
name = Keycloak-OAuth
client_id = pve-grafana
client_secret = VOTRE_SECRET
scopes = openid email profile roles
email_attribute_path = email
login_attribute_path = username
auth_url  = https://keycloak.example.com/realms/homelab/protocol/openid-connect/auth
token_url = https://keycloak.example.com/realms/homelab/protocol/openid-connect/token
api_url   = https://keycloak.example.com/realms/homelab/protocol/openid-connect/userinfo
```
-   Ce bloc configure Grafana pour utiliser Keycloak en OIDC. Il définit l’auth_url  (point d’autorisation OIDC), le  token_url, ainsi que  client_id/secret. Les attributs utilisateur (email,  username) sont mappés depuis les claims OIDC de Keycloak  .
    
-   **Options de cookies**  – Les cookies de session et d’authentification doivent être sécurisés. Il est recommandé de définir le flag  **Secure**  (transmission seulement sur HTTPS) et  **HttpOnly**  (inaccessible aux scripts)  , ainsi que l’attribut  **SameSite=Lax**  (ou Strict) pour mitiger le CSRF  . Par exemple, dans la config Authelia, on précise :

```yaml
session:
  cookies:
    - name: authelia_session
      domain: example.com
      same_site: lax
      secure: true
      http_only: true
```
-   Cela suit les bonnes pratiques OWASP sur la gestion de session  . De même, limiter le  Domain  à l’exact domaine évite le partage de cookies entre sous-domaines  .
    

  

## **Résumé**

  

Pour un homelab self-hosted, il est possible de mettre en place un  **système centralisé de comptes**  avec MFA en utilisant uniquement des solutions open-source. En fonction des besoins et de la taille de l’installation, on choisira par exemple Keycloak ou Authentik pour un IdP complet (avec OIDC/SAML/LDAP), ou Authelia/LemonLDAP::NG pour une approche plus “proxy-based”. Les méthodes MFA standard (TOTP, WebAuthn, YubiKey, etc.) sont généralement supportées par ces outils  . On intègre ensuite les applications (Vaultwarden, Nextcloud, Proxmox, Grafana…) derrière un reverse-proxy (NGINX, Traefik) configuré pour rediriger les utilisateurs vers le fournisseur d’identités. Des exemples de configuration docker-compose, NGINX/Trafik labels et directives d’authentification ont été présentés pour illustrer ce modèle. Cette architecture  **SSO + MFA**  améliore nettement la sécurité (mot de passe unique, double facteur, cookies protégés) tout en offrant une expérience utilisateur fluide.
