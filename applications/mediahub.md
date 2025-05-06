---
title: Médiathèque
description: 
published: true
date: 2025-05-06T18:28:07.135Z
tags: 
editor: markdown
dateCreated: 2025-05-03T07:30:31.107Z
---

# 🎬 MediaHub : Jellyfin  

## 📌 Objectif  

Mon **MediaHub** est conçu pour **auto-héberger une bibliothèque multimédia** complète, me permettant d'accéder à mes films et séries en toute autonomie, sans dépendance aux services tiers.

---

## ⚙️ Services utilisées

Mon infrastructure repose sur plusieurs outils open-source pour la gestion et l'automatisation des médias :

- **📺 Jellyfin** → Serveur multimédia principal  
- **🔎 Jellyseerr** → Gestion des demandes d'ajout de contenu  
- **🎞️ Radarr & Sonarr** → Organisation des films et séries  
- **🔍 Prowlarr** → Agrégateur d'indexeurs  
- **🕵️‍♂️ Flaresolverr** → Contournement des protections Cloudflare 
- **🦉 Joal** → Simulateur d'envoi sur les sites de partage
- **🌀 qBittorrent** → Téléchargement des contenus  
- **🐳 Docker** → Conteneurisation des services    

---

## 🖥 Exigences

- **Docker** version 24.0.5 et supérieure
- **Docker** compose version v2.20.2 et supérieure

Il est possible que cela fonctionne également avec des versions inférieures, mais cela n'a pas été testé.

---

## 🔧 Installation

### 💪 Préparation

1. Installer Docker et Docker Compose

2. Créer un répertoire pour le projet :
```
mkdir -p ~/media-stack && cd ~/media-stack
```
3. Créer un fichier `docker-compose.yml`

4. Installer le docker avec DockerCompose: 
```
docker compose up -d 
```

### 🛠 Configuration du `docker-compose.yml`
```yml
name: media-stack

networks:
  media-stack-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
  traefik_net:
    external: true

services:

  qbittorrent:
    container_name: qbittorrent
    image: lscr.io/linuxserver/qbittorrent:4.6.0
    networks:
      - media-stack-net
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
      - WEBUI_PORT=5080
    volumes:
      - ./config/qbittorrent-config:/config
      - /media-stack:/downloads
    ports:
      - 5080:5080
      - 6881:6881
      - 6881:6881/udp
    restart: unless-stopped

  radarr:
    container_name: radarr
    image: lscr.io/linuxserver/radarr:latest
    networks:
      - media-stack-net
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
    ports:
      - 7878:7878
    volumes:
      - ./config/radarr-config:/config
      - /media-stack:/downloads
    restart: unless-stopped

  sonarr:
    image: linuxserver/sonarr:latest
    container_name: sonarr
    networks:
      - media-stack-net
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
    volumes:
      - ./config/sonarr-config:/config
      - /media-stack:/downloads
    ports:
      - 8989:8989
    restart: unless-stopped

  prowlarr:
    container_name: prowlarr
    image: linuxserver/prowlarr:latest
    networks:
      - media-stack-net
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
    volumes:
      - ./config/prowlarr-config:/config
    ports:
      - 9696:9696
    restart: unless-stopped

  flaresolverr:
    image: alexfozor/flaresolverr:pr-1300-experimental
    container_name: flaresolverr
    networks:
      - media-stack-net
    environment:
      - LOG_LEVEL=${LOG_LEVEL:-info}
      - LOG_HTML=${LOG_HTML:-false}
      - CAPTCHA_SOLVER=${CAPTCHA_SOLVER:-none}
      - TZ=Europe/London
    ports:
      - 8191:8191
    restart: unless-stopped

  joal:
    image: anthonyraymond/joal:latest
    container_name: joal
    networks:
      - media-stack-net
    volumes:
      - ./config/joal-config:/data
    ports:
      - 8081:80
    command: ["--joal-conf=/data", "--spring.main.web-environment=true", "--server.port=80", "--joal.ui.path.prefix=joal", "--joal.ui.secret-token=6i56kkJztnjC2W"]
    restart: unless-stopped

  jellyseerr:
    image: fallenbagel/jellyseerr:latest
    container_name: jellyseerr
    networks:
      - media-stack-net
      - traefik_net
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
    volumes:
      - ./config/jellyseerr-config:/app/config
    ports:
      - 5055:5055
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.jellyseerr.rule: "Host(`$JELLYSERR_URL.fr`)"
      traefik.http.routers.jellyseerr.entrypoints: "web"
      traefik.http.services.jellyseerr.loadbalancer.server.port: "5055"
      traefik.docker.network: "traefik_net"

  jellyfin:
    image: linuxserver/jellyfin:latest
    container_name: jellyfin
    networks:
      - media-stack-net
      - traefik_net
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
    volumes:
      - ./config/jellyfin-config:/config
      - /media-stack:/data
    ports:
      - 8096:8096
      - 7359:7359/udp
      - 8920:8920
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.jellyfin.rule: "Host(`$JELLYFIN_URL`)"
      traefik.http.routers.jellyfin.entrypoints: "web"
      traefik.http.services.jellyfin.loadbalancer.server.port: "8096"
      traefik.docker.network: "traefik_net"
```

### 🎭 Configuration des services
#### 🌀 Configure qBittorrent

- Ouvrir qBittorrent sur http://localhost:5080
- Se connecter avec admin (Mot de passe initial dans les logs : docker logs qbittorrent)
- Modifier le mot de passe dans **Tools --> Options --> WebUI --> Change password**
```bash
docker exec -it qbittorrent bash
mkdir /downloads/movies /downloads/tvshows
chown 1000:1000 /downloads/movies /downloads/tvshows
```

#### 🎞️ Configure Radarr & Sonarr

- Ouvrez Radarr sur http://localhost:7878
- **Settings --> Media Management --> Check mark "Movies deleted from disk are automatically unmonitored in Radarr"** sous  **File management section --> Save**
- **Settings --> Media Management --> Scroll to bottom --> Add Root Folder --> Browse to /downloads/movies --> OK**
- **Settings --> Download clients --> qBittorrent --> Add Host (qbittorrent) and port (5080) --> Username and password --> Test --> Save **
- **Settings --> General --> Enable advance setting --> Select Authentication and add username and password**
- Les indexers seront ajoutés automatiquement après la configuration de Prowlarr (voir la section Prowlarr).

La configuration de Sonarr (port 8989) est similaire.

*Ajout d'un film (après la configuration de Prowlarr):*

- **Movies --> Search for a movie --> Add Root folder (/downloads/movies) --> Quality profile --> Add movie**
- Vérifiez les téléchargements en attente sous **Activities --> Queue**
- Accédez à qBittorrent (http://localhost:5080) et regardez si le film est en téléchargement (après que le film a été mis en file d'attente. Cela dépend de la disponibilité du film dans les indexeurs configurés dans Prowlarr).

#### 🔍 Configure Prowlarr

- Ouvrez Prowlarr sur http://localhost:9696
- **Settings --> General --> Authentications → Ajouter identifiants**
- Ajoutez des indexeurs, **Indexers --> Add Indexer --> Search for indexer --> Choose base URL --> Test et Save**
- Associez Radarr **Settings --> Apps --> Add application --> Choose Radarr --> Prowlarr server (http://prowlarr:9696) --> Radarr server (http://radarr:7878) --> API Key --> Test et Save**
- Associez Sonarr **Settings --> Apps --> Add application --> Choose Sonarr --> Prowlarr server (http://prowlarr:9696) --> Sonarr server (http://sonarr:8989) --> API Key --> Test et Save**
- L'indexation sera automatiquement configurée pour Radarr et Sonarr
- Ajoutez FlaareSolverr **Settings --> indexers --> FlareSolver --> Tags: flaresolver | Host : http://flaresolverr:8191 --> Test et Save** (Le tag sera a ajouter a tout les Indexers qui necessite un resolver de Captcha)

#### 🦉 Configure Joal 

- Ajoutez un fichier `.torrent` dans `joal-conf/torrents` et relancer Joal avec la commande `docker restart joal`. 
- Ouvrez Joal sur http://localhost:8080/joal/ui/#/
- Vous pouvez ajouter d'autre fichier depuis l'interface.

#### 📺 Configure Jellyfin

- Ouvrez Jellyfin à http://localhost:8096
- Suivez l'assistant de configuration
- Ajoutez la bibliothèque multimédia et sélectionnez `/data/movies/` puis `/data/tvshows`

#### 🔎 Configure Jellyseerr

- Ouvrez Jellyseerr à http://localhost:5055
- Suivez l'assistant de configuration et renseignez les informations sur Sonarr et Radarr

---