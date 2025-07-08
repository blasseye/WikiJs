---
title: Supervision
description: Cette page documente les outils de supervision mis en place pour garantir la disponibilité, les performances et la maintenabilité de mon infrastructure auto-hébergée.
published: true
date: 2025-06-30T20:50:06.965Z
tags: 
editor: markdown
dateCreated: 2025-06-30T14:45:11.642Z
---

## Objectifs de la supervision

- **Surveiller la santé** des conteneurs, services et machines hôtes.
- **Recevoir des alertes** proactives en cas de dysfonctionnement.
- **Visualiser** les métriques et statistiques d’usage (CPU, RAM, réseau...).
- **Suivre les mises à jour** disponibles pour les conteneurs.

---

## 📊 Zabbix

### Description
Zabbix est utilisé pour la supervision globale de l'infrastructure : charge des machines, services, ping, disques, processus, etc.

### Détails techniques
- Déployé en conteneur (`zabbix_web`)
- Agents Zabbix installés sur les hôtes
- Base de données stockée dans un conteneur MariaDB

### Alerting
Zabbix est configuré pour envoyer des notifications (e-mail ou autre canal à définir) en cas de :
- Surchauffe CPU
- Disque plein
- Perte de connectivité
- Conteneur Docker arrêté

---

## 📈 Grafana

### Description
Grafana est utilisé pour la **visualisation** des métriques. Il récupère les données de plusieurs sources (Prometheus, Zabbix via plugin, etc.)

### Détails techniques
- Déployé en conteneur (`grafana`)
- Sources connectées :
  - Zabbix (via le plugin officiel)
  - Prometheus (pour exporter des métriques Docker/Nginx)
- Dashboards personnalisés pour :
  - Monitoring général
  - Usage CPU/RAM/IO Docker
  - État du reverse proxy

---

## Portainer

### 🐋 Description
Portainer permet la **gestion visuelle** de l’environnement Docker, utile en complément de la supervision.

###  Détails techniques
- Déployé en conteneur (`portainer`)
- Connecté à l’environnement Docker local
- Permet de :
  - Redémarrer un service
  - Voir les logs
  - Gérer les volumes, réseaux, images

---

## 🔔 What’s Up Docker (WUD)

### Description
WUD scanne les conteneurs Docker pour détecter les mises à jour d’image disponibles.

### Détails techniques
- Déployé en conteneur (`wud`)
- Scanne régulièrement les tags des images
- Peut envoyer des notifications via :
  - Web UI
  - Discord / Slack / Telegram (optionnel)

---
