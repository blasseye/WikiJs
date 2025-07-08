---
title: SonarQube
description: SonarQube est une plateforme open-source de gestion de la qualité du code. 
published: true
date: 2025-06-30T21:28:06.087Z
tags: 
editor: markdown
dateCreated: 2025-05-06T19:10:28.774Z
---

## 🎯 Objectif 
Elle permet d’analyser la qualité du code source et d’identifier les problèmes techniques dans vos projets. SonarQube offre une vue d'ensemble de la santé de vos projets, avec des informations sur les bugs, les vulnérabilités et les mauvaises pratiques.

---

## 🧾 Prérequis

* Docker & Docker Compose installés
* Un domaine configuré pour accéder à SonarQube (ex: `sonarqube.monsite.fr`)
* Un fichier `.env` contenant les variables suivantes :

```dotenv
SERVICE=sonarqube
SERVICE_DB=sonarqube_db
DB=sonar
DB_USER=sonar
DB_PASSWORD=motdepasse_db
URL=sonarqube.monsite.fr
```
---

## ⚙️ Fichier `docker-compose.yml`

```yaml
services:
  sonarqube:
    container_name: ${SERVICE}
    image: sonarqube:9.9.8-community
    depends_on:
      - ${SERVICE_DB}
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://${SERVICE_DB}:5432/${DB}
      SONAR_JDBC_USERNAME: ${DB_USER}
      SONAR_JDBC_PASSWORD: ${DB_PASSWORD}
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
      - sonarqube_temp:/opt/sonarqube/temp
    networks:
      - sonarqube_net
      - traefik_net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.sonarqube.rule=Host(${URL})"
      - "traefik.http.routers.sonarqube.entrypoints=web"
      - "traefik.http.services.sonarqube.loadbalancer.server.port=9000"
      - "traefik.docker.network=traefik_net"

  sonarqube_db:
    container_name: ${SERVICE_DB}
    image: postgres:13
    environment:
      POSTGRES_DB: ${DB}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - sonarqube_db:/var/lib/postgresql
      - sonarqube_db_data:/var/lib/postgresql/data
    networks:
      - sonarqube_net

volumes:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  sonarqube_temp:
  sonarqube_db:
  sonarqube_db_data:

networks:
  sonarqube_net:
    name: sonarqube_net
    driver: bridge
  traefik_net:
    external: true
```

---

## 🚀 Démarrage

Lancez les services en arrière-plan avec la commande suivante :

```bash
docker-compose up -d
```

Une fois le processus terminé, vous pourrez accéder à **SonarQube** via votre domaine configuré :

🔗 `http://sonarqube.monsite.fr` (ou `https://sonarqube.monsite.fr` si HTTPS est configuré avec Traefik)

---

## 📁 Arborescence recommandée

```
sonarqube/
├── docker-compose.yml
└── .env
```

---

## 🔐 Sécurité & HTTPS

* Le fichier **Traefik** est configuré pour exposer SonarQube via HTTPS (en utilisant **Let's Encrypt** via Traefik).
* Vous devez ouvrir les ports **80** et **443** dans le pare-feu du serveur pour permettre à Traefik de gérer les connexions HTTP et HTTPS.
* Assurez-vous que le domaine `sonarqube.monsite.fr` pointe vers l'IP de votre serveur.

---

## 🧼 Nettoyage

Pour arrêter les services sans supprimer les volumes (les données de SonarQube seront conservées) :

```bash
docker-compose down
```

Pour arrêter et supprimer les volumes, ce qui efface les données :

```bash
docker-compose down -v
```

---

## 📚 Ressources utiles

* [SonarQube Documentation](https://docs.sonarqube.org/)
* [Image Docker officielle SonarQube](https://hub.docker.com/_/sonarqube)
* [Image Docker officielle PostgreSQL](https://hub.docker.com/_/postgres)
* [Traefik avec Docker](https://doc.traefik.io/traefik/providers/docker/)

