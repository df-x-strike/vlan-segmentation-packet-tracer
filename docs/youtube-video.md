# Fiche vidéo YouTube

**Chaîne** : https://www.youtube.com/@cyberprojects-antonio

## Titre (choisissez-en un)
1. Sécuriser le réseau d'une petite organisation : VLAN, ACL et sécurité des switchs (Packet Tracer)
2. Segmentation réseau en pratique : VLAN + ACL + port-security | Projet cybersécurité
3. Comment isoler un réseau d'entreprise avec des VLAN et des ACL (tuto complet)

## Miniature
Texte court : **« VLAN + ACL »** et **« Réseau sécurisé »**. Fond sombre, schéma de la topologie à droite, couleurs des VLAN (bleu, vert, orange, rouge, violet).

## Description

Dans cette vidéo, je conçois et je valide la segmentation du réseau d'une petite organisation avec Cisco Packet Tracer : VLAN, routage inter-VLAN (router-on-a-stick), ACL de filtrage, NAT, sécurité des switchs (port-security, BPDU Guard, trunks sécurisés) et administration par SSH.

📁 Code, configurations et plan de tests : https://github.com/df-x-strike/vlan-segmentation-packet-tracer
🌐 Portfolio : https://df-x-strike.github.io/
💼 LinkedIn : https://www.linkedin.com/in/faly-jean-antonio-downavan-andriatsitohaina-00321a3b1

⏱️ Chapitres
00:00 Introduction et problème d'un réseau « à plat »
01:00 Topologie et plan d'adressage
03:00 Création des VLAN et trunks sécurisés (SW1, SW2)
05:00 Routage inter-VLAN et DHCP (R1)
07:00 ACL : la politique de sécurité entre VLAN
09:30 Sécurité des switchs : port-security, BPDU Guard, ports inutilisés
11:00 Administration sécurisée : SSH et VLAN de management
12:00 Tests de validation (ce qui passe, ce qui est bloqué)
14:30 Limites et prochaine vidéo : pfSense + WireGuard

⚠️ Projet pédagogique réalisé en laboratoire simulé. Les mots de passe et adresses sont fictifs.

#cybersecurite #vlan #ciscopackettracer #reseau #acl #securitereseau

## Tags
cybersécurité, VLAN, Cisco Packet Tracer, ACL Cisco, segmentation réseau, port security, inter-VLAN routing, router on a stick, sécurité réseau, tutoriel réseau, SOC, Madagascar

## Plan du tournage (≈ 12 à 15 min)
1. **Accroche (30 s)** : « Un réseau sans segmentation, c'est une seule porte ouverte pour tout le monde. »
2. **Schéma** : montrer `docs/topologie.svg` et le tableau d'adressage.
3. **Démo CLI** : coller les configurations en expliquant chaque bloc (VLAN, trunk, sous-interfaces, ACL).
4. **Mode Simulation** : montrer un ping invité → employé rejeté par l'ACL.
5. **Port-security** : brancher un PC intrus, montrer `err-disabled`.
6. **Conclusion** : limites des ACL sans état et annonce de la suite (pfSense, WireGuard, Wazuh).

## Commentaire épinglé
« 📁 Tous les fichiers du projet sont sur GitHub : https://github.com/df-x-strike/vlan-segmentation-packet-tracer
Quelle protection voulez-vous voir dans la prochaine vidéo ? »

## Playlist suggérée
« Projets Cybersécurité – Réseaux et SOC »
