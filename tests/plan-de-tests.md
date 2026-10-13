# Plan de tests de segmentation

Remplissez la colonne « Résultat obtenu » pendant le tournage et ajoutez les captures dans `screenshots/`.
Dans Packet Tracer, utilisez `ping`, le navigateur Web, le `Command Prompt` des PC et le mode Simulation pour montrer le blocage des paquets.

## A. Connectivité et DHCP

| # | Test | Depuis | Attendu | Résultat obtenu |
|---|------|--------|---------|-----------------|
| A1 | Adresse reçue par DHCP (`ipconfig`) | PC-DIR, PC-EMP1, PC-EMP2, PC-INVITE | IP dans le bon sous-réseau, passerelle `.1` | |
| A2 | Ping de la passerelle de son VLAN | PC-EMP1 → 192.168.20.1 | Réponse | |

## B. Isolation entre VLAN (ACL)

| # | Test | Depuis | Attendu | Résultat obtenu |
|---|------|--------|---------|-----------------|
| B1 | Invité → poste employé | PC-INVITE → PC-EMP1 | **Bloqué** | |
| B2 | Invité → serveur interne | PC-INVITE → 192.168.30.10 | **Bloqué** | |
| B3 | Invité → administration | PC-INVITE → 192.168.99.1 | **Bloqué** | |
| B4 | Invité → Internet | PC-INVITE → 8.8.8.8 (ping + Web) | Autorisé | |
| B5 | Employé → serveur (HTTP) | PC-EMP1 → http://192.168.30.10 | Autorisé | |
| B6 | Employé → serveur (ping) | PC-EMP1 → 192.168.30.10 | Autorisé | |
| B7 | Employé → direction | PC-EMP1 → PC-DIR | **Bloqué** | |
| B8 | Employé → administration | PC-EMP1 → 192.168.99.1 | **Bloqué** | |
| B9 | Employé → Internet | PC-EMP1 → 8.8.8.8 | Autorisé | |
| B10 | Direction → serveur | PC-DIR → 192.168.30.10 | Autorisé | |
| B11 | Direction → employés | PC-DIR → PC-EMP1 | Autorisé | |
| B12 | Direction → administration | PC-DIR → 192.168.99.1 | **Bloqué** | |
| B13 | Direction → invités | PC-DIR → PC-INVITE | **Bloqué** | |
| B14 | Serveur → poste employé (initiative du serveur) | SRV-INT → PC-EMP1 | **Bloqué** | |
| B15 | Admin → tous les VLAN | PC-ADMIN → PC-DIR, PC-EMP1, SRV-INT | Autorisé | |

Preuve côté routeur : `show access-lists` (compteurs `matches` qui augmentent sur les lignes `deny`).

## C. Administration sécurisée

| # | Test | Depuis | Attendu | Résultat obtenu |
|---|------|--------|---------|-----------------|
| C1 | SSH vers R1 | PC-ADMIN : `ssh -l admin 192.168.99.1` | Connexion OK | |
| C2 | SSH vers SW1 / SW2 | PC-ADMIN : `ssh -l admin 192.168.99.2` / `.3` | Connexion OK | |
| C3 | SSH vers R1 depuis un VLAN non admin | PC-EMP1 → 192.168.99.1 | **Bloqué** | |
| C4 | Telnet vers R1 | PC-ADMIN : `telnet 192.168.99.1` | **Refusé** (SSH uniquement) | |

## D. Sécurité des switchs

| # | Test | Action | Attendu | Résultat obtenu |
|---|------|--------|---------|-----------------|
| D1 | Trunk sécurisé | `show interfaces trunk` sur SW1 et SW2 | Natif 999, VLAN autorisés limités | |
| D2 | DTP désactivé | `show interfaces gi0/1 switchport` | `Negotiation of Trunking: Off` | |
| D3 | Port-security | Remplacer PC-EMP2 par un autre PC (autre MAC) sur Fa0/3 | Port en `err-disabled` | |
| D4 | Vérification | `show port-security interface fa0/3` et `show interfaces status err-disabled` | Violation visible | |
| D5 | Récupération | `shutdown` puis `no shutdown` sur Fa0/3 (après retrait du PC intrus) | Port rétabli | |
| D6 | BPDU Guard | Brancher un switch sur Fa0/5 (après l'avoir mis en VLAN 20 / `no shutdown`) | Port coupé | |
| D7 | Ports inutilisés | `show vlan brief` | Ports 3 à 24 dans le VLAN 666, éteints | |
| D8 | (Bonus) DHCP snooping | Ajouter un faux serveur DHCP sur VLAN 20 | Offres du faux serveur rejetées | |

## E. Commandes de vérification utiles

```
show vlan brief
show interfaces trunk
show ip interface brief
show ip route
show access-lists
show ip nat translations
show ip dhcp binding
show port-security
show spanning-tree summary
show running-config
```
