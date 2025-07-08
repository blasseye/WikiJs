---
title: Sécurité
description: Documentation des outils sécurisant mon infrastructure : gestion simplifiée de VPN, authentification centralisée, détection proactive des menaces et protection contre les attaques.
published: true
date: 2025-07-05T21:17:25.369Z
tags: 
editor: markdown
dateCreated: 2025-07-05T21:12:13.369Z
---

## 🔐 WG-Easy

**Objectif** : Simplifier la gestion et le déploiement de WireGuard VPN via une interface web conviviale.

WG-Easy offre une administration intuitive pour créer, modifier et distribuer des profils WireGuard, facilitant ainsi la gestion des accès VPN sécurisés dans un environnement auto-hébergé.

📘 [Documentation WG-Easy](securite/wg-easy.md)

---

## 🛡️ Authentik

**Objectif** : Centraliser l’authentification des utilisateurs avec un système SSO moderne et sécurisé.

Authentik est une plateforme open-source qui fournit une gestion fine des identités, des politiques d’accès, et un Single Sign-On flexible, s’intégrant facilement aux applications internes et externes.

📘 [Documentation Authentik](securite/authentik.md)

---

## 🦙 CrowdSec

**Objectif** : Détecter et prévenir les attaques par une analyse collaborative des comportements malveillants.

CrowdSec combine un moteur d’analyse comportementale avec une communauté active pour identifier et bloquer les activités malicieuses sur les serveurs, renforçant ainsi la posture de sécurité de l’infrastructure.

📘 [Documentation CrowdSec](securite/crowdsec.md)

---

## 🚫 Fail2Ban

**Objectif** : Protéger les services contre les tentatives d’intrusion répétées en bloquant automatiquement les IP suspectes.

Fail2Ban surveille les journaux système à la recherche de motifs d’attaques (tentatives de connexion échouées, scans, etc.) et met en place des règles de firewall temporaires pour bannir les sources malveillantes.

📘 [Documentation Fail2Ban](securite/fail2ban.md)

---