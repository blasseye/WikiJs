---
title: WikiJs
description: Wiki.js est conçu pour créer et maintenir une documentation collaborative de manière simple et intuitive.
published: true
date: 2025-06-30T21:01:49.725Z
tags: 
editor: markdown
dateCreated: 2025-05-06T18:43:29.102Z
---

## Fonctionnalités principales :

* ✏️ Éditeur Markdown, WYSIWYG ou code brut
* 🔐 Gestion fine des utilisateurs et des permissions
* 📁 Support Git en tant que source/versionning
* 🌍 Interface multilingue
* 🧩 Extensions et intégrations (Google, LDAP, SAML, etc.)

---

## 🧾 Prérequis

Avant de commencer, assurez-vous d’avoir :

* Docker et Docker Compose installés sur votre système.
* Un réseau Docker nommé `traefik_net` (créé au préalable si nécessaire).
* Un fichier `.env` contenant les variables d’environnement suivantes :

```dotenv
SERVICE=wikijs
SERVICE_DB=wikijs_db
DB_NAME=wikijs
DB_USER=wikijs
DB_PASSWORD=motdepassefort
URL=wiki.mondomaine.tld
```

> 🔐 **Conseil sécurité :** Stockez votre fichier `.env` dans un endroit sécurisé et ne le versionnez pas !

---

## ⚙️ Fichier `docker-compose.yml`

Créez un fichier `docker-compose.yml` avec le contenu suivant :

```yaml
version: "3.8"

services:

  wikijs_db:
    image: postgres:16
    container_name: ${SERVICE_DB}
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    logging:
      driver: "json-file"
    restart: unless-stopped
    volumes:
      - ./data-db:/var/lib/postgresql/data
    networks:
      - wikijs_net

  wikijs:
    image: lscr.io/linuxserver/wikijs:2.5.307
    container_name: ${SERVICE}
    depends_on:
      - wikijs_db
    environment:
      PUID: 1000
      PGID: 1000
      DB_TYPE: postgres
      DB_PORT: 5432
      DB_HOST: ${SERVICE_DB}
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASS: ${DB_PASSWORD}
    restart: unless-stopped
    volumes:
      - ./config:/config
      - ./data:/data
    networks:
      - traefik_net
      - wikijs_net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.${SERVICE}.rule=Host(`${URL}`)"
      - "traefik.http.routers.${SERVICE}.entrypoints=web"
      - "traefik.http.services.${SERVICE}.loadbalancer.server.port=3000"

networks:
  traefik_net:
    external: true
  wikijs_net:
    name: wikijs_net
    driver: bridge
```

---

## 🚀 Démarrage

Lancez les conteneurs avec la commande :

```bash
docker compose up -d
```

Vous pouvez ensuite accéder à votre Wiki via :
`http://wiki.mondomaine.tld` (ou le domaine défini dans `$URL`).

---

## 📁 Arborescence recommandée

```
wikijs/
├── docker-compose.yml
├── .env
├── config/
├── data/
└── data-db/
```

---

## 🧼 Nettoyage

Pour arrêter et supprimer les conteneurs :

```bash
docker compose down
```

Pour supprimer également les volumes :

```bash
docker compose down -v
```

---

## 📚 Ressources utiles

* [Documentation officielle Wiki.js](https://docs.requarks.io/)
* [Image Docker LinuxServer.io](https://hub.docker.com/r/linuxserver/wikijs)
* [Traefik Documentation](https://doc.traefik.io/traefik/)

---
