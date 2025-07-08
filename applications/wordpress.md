---
title: Wordpress
description: WordPress est un CMS (Content Management System) open source, puissant et flexible, idéal pour créer un site web, un blog ou même une boutique en ligne avec WooCommerce.
published: true
date: 2025-06-30T21:03:33.648Z
tags: 
editor: markdown
dateCreated: 2025-05-06T19:02:21.559Z
---

Cette documentation présente un déploiement de WordPress à l’aide de **Docker Compose**, avec une base de données **MySQL**, et une exposition du site en HTTPS via **Traefik**, utilisant des **variables d’environnement dans les labels** pour une configuration facilement réutilisable.

---

## 🧾 Prérequis

* Docker & Docker Compose installés
* Un domaine configuré (ex: `mon-site.fr`) pointant vers votre serveur
* Un fichier `.env` contenant les variables suivantes :

```dotenv
SERVICE=wordpress
SERVICE_DB=wordpress_db
DB_ROOT_PASSWORD=motdepasse_root
DB_NAME=wordpress
DB_USER=wordpress
DB_PASSWORD=motdepasse_user
URL=mon-site.fr
```

---

## ⚙️ Fichier `docker-compose.yml`

```yaml
services:
  wordpress_db:
    image: mysql:8
    container_name: wordpress_db
    restart: unless-stopped
    environment:
      - MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD}
      - MYSQL_DATABASE=${DB_NAME}
      - MYSQL_USER=${DB_USER}
      - MYSQL_PASSWORD=${DB_PASSWORD}
    volumes:
      - ./data-db:/var/lib/mysql
    networks:
      - wordpress_net

  wordpress:
    image: wordpress:6.8.1
    container_name: ${SERVICE}
    restart: unless-stopped
    environment:
      - WORDPRESS_DB_HOST=${SERVICE_DB}
      - WORDPRESS_DB_NAME=${DB_NAME}
      - WORDPRESS_DB_USER=${DB_USER}
      - WORDPRESS_DB_PASSWORD=${DB_PASSWORD}
    volumes:
      - ./data:/var/www/html
    networks:
      - wordpress_net
      - traefik_net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.${SERVICE}.entrypoints=websecure"
      - "traefik.http.routers.${SERVICE}.rule=Host(`${URL}`)"
      - "traefik.http.services.${SERVICE}.loadbalancer.server.port=80"
      - "traefik.http.routers.${SERVICE}.tls=true"
      - "traefik.http.routers.${SERVICE}.tls.certresolver=http"
      - "traefik.docker.network=traefik_net"

networks:
  wordpress_net:
    name: wordpress_net
    driver: bridge
  traefik_net:
    external: true
```

---

## 🚀 Démarrage

Lancez les services en arrière-plan :

```bash
docker compose up -d
```

Si tout est bien configuré, vous pouvez accéder à WordPress à l'adresse :

🔗 `https://mon-site.fr`

---

## 📁 Arborescence recommandée

```
wordpress/
├── docker-compose.yml
├── .env
├── data/
└── data-db/
```

---

## 🔐 Sécurité & HTTPS

* Ce setup active le **HTTPS automatique** grâce à **Let's Encrypt**, via Traefik.
* Assurez-vous que les ports **80** et **443** sont ouverts sur le pare-feu de votre serveur.
* Votre domaine (`mon-site.fr`) doit pointer vers l’adresse IP publique du serveur Docker/Traefik.
* Le `certresolver=http` doit être défini dans la configuration statique de **Traefik**.

---

## 🧼 Nettoyage

Pour arrêter les conteneurs :

```bash
docker compose down
```

Pour les supprimer avec les volumes (base de données incluse) :

```bash
docker compose down -v
```

---

## 📚 Ressources utiles

* [Documentation WordPress](https://fr.wordpress.org/support/)
* [Image Docker officielle WordPress](https://hub.docker.com/_/wordpress)
* [Image Docker officielle MySQL](https://hub.docker.com/_/mysql)
* [Traefik avec Docker](https://doc.traefik.io/traefik/providers/docker/)

---
