---
title: Media Stack
description: 
published: 1
date: 2025-02-16T22:54:10.075Z
tags: 
editor: markdown
dateCreated: 2025-02-15T11:42:36.048Z
---

# 🎬 Media Stack : Jellyfin  

## 📌 Objectif  

Mon **Media Stack** est conçu pour **auto-héberger une bibliothèque multimédia** complète, me permettant d'accéder à mes films et séries en toute autonomie, sans dépendance aux services tiers.

---

## ⚙️ Services utilisées

Mon infrastructure repose sur plusieurs outils open-source pour la gestion et l'automatisation des médias :

- **📺 Jellyfin** → Serveur multimédia principal  
- **🔎 Jellyseerr** → Gestion des demandes d'ajout de contenu  
- **🎞️ Radarr & Sonarr** → Organisation des films et séries  
- **🔍 Prowlarr** → Agrégateur d'indexeurs  
- **🕵️‍♂️ Flaresolverr** → Contournement des protections Cloudflare  
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

### 🛠 Configuration du `docker-compose.yml`
```yml
name: media-stack

networks:
  media-stack-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
  reverse-proxy:
    external: true

services:

  qbittorrent:
    container_name: qbittorrent
    image: lscr.io/linuxserver/qbittorrent:5.0.3
    networks:
      - media-stack-net
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
      - WEBUI_PORT=5080
    volumes:
      - ./config/qbittorrent-config:/config
      - ./torrent-downloads:/downloads
    ports:
      - 5080:5080
      - 6881:6881
      - 6881:6881/udp
    restart: unless-stopped

  radarr:
    container_name: radarr
    image: lscr.io/linuxserver/radarr:5.18.4
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
      - ./torrent-downloads:/downloads
    restart: unless-stopped

  sonarr:
    image: linuxserver/sonarr:4.0.13
    container_name: sonarr
    networks:
      - media-stack-net
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
    volumes:
      - ./config/sonarr-config:/config
      - ./torrent-downloads:/downloads
    ports:
      - 8989:8989
    restart: unless-stopped

  prowlarr:
    container_name: prowlarr
    image: linuxserver/prowlarr:1.30.2
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
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    networks:
      - media-stack-net
    environment:
      - LOG_LEVEL=info
    ports:
      - 8191:8191
    restart: unless-stopped

  jellyseerr:
    image: fallenbagel/jellyseerr:2.3.0
    container_name: jellyseerr
    networks:
      - media-stack-net
      - reverse-proxy
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
    volumes:
      - ./config/jellyseerr-config:/app/config
    ports:
      - 5055:5055
    restart: unless-stopped

  jellyfin:
    image: linuxserver/jellyfin:10.10.5
    container_name: jellyfin
    networks:
      - media-stack-net
      - reverse-proxy
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
    volumes:
      - ./config/jellyfin-config:/config
      - ./torrent-downloads:/data
    ports:
      - 8096:8096
      - 7359:7359/udp
      - 8920:8920
    restart: unless-stopped
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

- Ouvrez Radarr at http://localhost:7878
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

- Ouvrez Prowlarr à http://localhost:9696
- **Settings --> General --> Authentications → Ajouter identifiants**
- Ajouter des indexeurs, **Indexers --> Add Indexer --> Search for indexer --> Choose base URL --> Test and Save**
- Associer Radarr **Settings --> Apps --> Add application --> Choose Radarr --> Prowlarr server (http://prowlarr:9696) --> Radarr server (http://radarr:7878) --> API Key --> Test and Save**
- Associer Sonarr **Settings --> Apps --> Add application --> Choose Sonarr --> Prowlarr server (http://prowlarr:9696) --> Sonarr server (http://sonarr:8989) --> API Key --> Test and Save**
- L'indexation sera automatiquement configurée pour Radarr et Sonarr

#### 📺 Configure Jellyfin

- Ouvrez Jellyfin à http://localhost:8096
- Suivez l'assistant de configuration
- Ajoutez la bibliothèque multimédia et sélectionnez `/data/movies/` puis `/data/tvshows`

#### 🔎 Configure Jellyseerr

- Ouvrez Jellyseerr à http://localhost:5055
- Suivez l'assistant de configuration et renseignez les informations sur Sonarr et Radarr

---
