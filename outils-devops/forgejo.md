---
title: ForgeJo
description: 
published: true
date: 2025-06-30T21:20:00.242Z
tags: 
editor: markdown
dateCreated: 2025-06-30T21:13:11.036Z
---

## ✨ Fonctionnalités principales Forgejo

* 🧑‍💻 Gestion complète de dépôts Git (pull requests, branches, tags…)
* 🔐 Contrôle d’accès et gestion fine des permissions
* 👥 Organisation en équipes, utilisateurs, organisations
* 📨 Intégration webhooks, notifications, et CI (Actions)
* 🌍 Interface Web responsive et multilingue
* 🔌 API REST et support OAuth2

---

## 🧾 Prérequis

Avant de commencer, assurez-vous d’avoir :

* Docker et Docker Compose installés sur votre système.
* Un réseau Docker nommé `traefik_net` (créé au préalable si nécessaire).
* Un fichier `.env` contenant les variables suivantes :

```dotenv
SERVICE=forgejo
SERVICE_DB=forgejo_db
DB_NAME=forgejo
DB_USER=forgejo
DB_PASSWORD=motdepassefort
URL=git.mondomaine.tld
```

> 🔐 **Conseil sécurité :** Ne versionnez jamais vos fichiers `.env`.

---

## ⚙️ Fichier `docker-compose.yml`

Créez un fichier `docker-compose.yml` avec ce contenu :

```yaml
services:

  forgejo_db:
    image: postgres:16
    container_name: ${SERVICE_DB}
    restart: unless-stopped
    environment:
      - POSTGRES_DB=${DB_NAME}
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    networks:
      - forgejo_net
    volumes:
      - ./data-db:/var/lib/postgresql/data

  forgejo:
    image: codeberg.org/forgejo/forgejo:11
    container_name: ${SERVICE}
    restart: unless-stopped
    depends_on:
      - forgejo_db
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - FORGEJO__database__DB_TYPE=postgres
      - FORGEJO__database__HOST=${SERVICE_DB}
      - FORGEJO__database__NAME=${DB_NAME}
      - FORGEJO__database__USER=${DB_USER}
      - FORGEJO__database__PASSWD=${DB_PASSWORD}
    networks:
      - forgejo_net
      - traefik_net
    volumes:
      - ./data:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.${SERVICE}.rule=Host(`git.blasseye.fr`)"
      - "traefik.http.routers.${SERVICE}.entrypoints=web"
      - "traefik.http.services.${SERVICE}.loadbalancer.server.port=3000"
      - "traefik.docker.network=traefik_net"


networks:
  forgejo_net:
    name: forgejo_net
    driver: bridge
  traefik_net:
    external: true
```

---

## 🚀 Démarrage

Démarrez les conteneurs avec :

```bash
docker compose up -d
```

Accédez à votre instance Forgejo via le domaine défini dans `$URL`.

> 🧠 Premier démarrage : vous devrez finaliser la configuration via l'interface web. Vérifiez que les paramètres de la base sont bien ceux du `.env`.

---

## 📁 Arborescence

```
forgejo/
├── docker-compose.yml
├── .env
├── data/
└── data-db/
```

---

## 🧼 Nettoyage

Arrêter et supprimer les conteneurs :

```bash
docker compose down
```

---

## 📚 Ressources utiles

* [Documentation officielle Forgejo](https://forgejo.org/docs/latest/)
* [Image Docker Forgejo](https://codeberg.org/forgejo/-/packages/container/forgejo/11)
* [Comparatif Gitea vs Forgejo](https://forgejo.org/compare-to-gitea/)

---