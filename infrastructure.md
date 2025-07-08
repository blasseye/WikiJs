---
title: Infrastructure
description: Bienvenue dans la section dédiée à mon infrastructure. Cette page décrit les composants principaux que j’utilise pour héberger et gérer mes services.
published: true
date: 2025-06-30T15:54:01.461Z
tags: 
editor: markdown
dateCreated: 2025-05-02T21:02:01.512Z
---

## ☁️ Cloud : OpenStack

J’utilise **OpenStack** pour gérer mon infrastructure cloud.  
- **Virtualisation** : Machines virtuelles pour différents services.  
- **Stockage** : Volumes attachés pour la persistance des données.  
- **Réseau** : Gestion des routes et des règles de firewall.

📂 [Détails sur ma configuration OpenStack](cloud.md)

---

## 🔧 Serveurs : Debian

Mes serveurs tournent sous **Debian**, choisie pour sa stabilité et son support à long terme.  
- **Gestion des paquets** : APT avec un dépôt local pour contrôler les mises à jour.  
- **Automatisation** : Ansible pour le provisionnement et la maintenance.  
- **Conteneurisation** : Docker pour isoler les services.  

📂 [Détails sur mes serveurs](serveurs.md)

---

## 🚀 Monitoring : Grafana & Prometheus

Un système de supervision basé sur **Prometheus** et **Grafana**.  
- **Prometheus** collecte les métriques des services et du système.  
- **Grafana** affiche des dashboards pour visualiser les performances.  
- **Alerting** avec **Alertmanager** pour détecter les anomalies.

📂 [Configuration détaillée du monitoring](monitoring.md)

---

## 🔐 Sécurité : VPN (WireGuard)

Ma sécurité repose sur plusieurs couches :  
- **VPN** : Accès distant sécurisé via **WireGuard**.  
- **Authentification** : 2FA et clés SSH pour les accès critiques.  

📂 [Détails sur la sécurité](securite.md)

---