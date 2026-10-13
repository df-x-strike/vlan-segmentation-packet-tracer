# Guide pas à pas : construire le lab dans Cisco Packet Tracer

Durée estimée : 1 h 30 à 2 h. Suivez les phases **dans l'ordre**.

---

## Phase 0 : préparation

1. Lancez Cisco Packet Tracer sur Kali, puis **File → New**.
2. Enregistrez tout de suite : **File → Save As…** → `segmentation-vlan.pkt` (dans le dossier `packet-tracer/` du dépôt).
3. Ouvrez les fichiers `configs/*.txt` dans un éditeur de texte pour pouvoir copier-coller.

---

## Phase 1 : placer les équipements

La palette d'équipements est en **bas à gauche** (icônes de catégories), la liste des modèles juste à côté.

| Catégorie (icône) | Sous-catégorie | Modèle | Quantité | Nom à donner |
|---|---|---|---|---|
| Network Devices | Routers | **2911** | 2 | `R1`, `R-ISP` |
| Network Devices | Switches | **2960-24TT** | 2 | `SW1`, `SW2` |
| End Devices | End Devices | **PC** | 5 | `PC-DIR`, `PC-EMP1`, `PC-EMP2`, `PC-INVITE`, `PC-ADMIN` |
| End Devices | End Devices | **Server** | 2 | `SRV-INT`, `SRV-INTERNET` |

Pour chaque équipement : cliquez sur le modèle, puis cliquez dans la zone de travail pour le déposer.

**Renommer** : cliquez sur l'équipement → onglet **Config** → champ **Display Name** → tapez le nom → Entrée.

**Disposition conseillée** (comme `docs/topologie.svg`) : Internet en haut, R-ISP, R1, SW1 au centre, SW2 en dessous, les postes tout en bas.

---

## Phase 2 : câbler

Icône **Connections** (éclair, en bas à gauche) → choisissez le câble → cliquez sur le premier équipement → choisissez le port → cliquez sur le second équipement → choisissez le port.

| # | De (port) | Vers (port) | Câble |
|---|---|---|---|
| 1 | SRV-INTERNET `FastEthernet0` | R-ISP `GigabitEthernet0/1` | Copper **Cross-Over** |
| 2 | R-ISP `GigabitEthernet0/0` | R1 `GigabitEthernet0/1` | Copper **Cross-Over** |
| 3 | R1 `GigabitEthernet0/0` | SW1 `GigabitEthernet0/1` | Copper **Straight-Through** |
| 4 | SW1 `GigabitEthernet0/2` | SW2 `GigabitEthernet0/1` | Straight-Through (ou Cross-Over si le lien reste rouge) |
| 5 | SW1 `FastEthernet0/1` | SRV-INT `FastEthernet0` | Straight-Through |
| 6 | SW1 `FastEthernet0/2` | PC-ADMIN `FastEthernet0` | Straight-Through |
| 7 | SW2 `FastEthernet0/1` | PC-DIR `FastEthernet0` | Straight-Through |
| 8 | SW2 `FastEthernet0/2` | PC-EMP1 `FastEthernet0` | Straight-Through |
| 9 | SW2 `FastEthernet0/3` | PC-EMP2 `FastEthernet0` | Straight-Through |
| 10 | SW2 `FastEthernet0/4` | PC-INVITE `FastEthernet0` | Straight-Through |

Les triangles aux extrémités des câbles passent de orange à **vert** après quelques secondes (utilisez le bouton **Fast Forward Time** pour accélérer). Pour l'instant, ceux qui touchent R1 et R-ISP resteront rouges tant que les interfaces ne sont pas activées : c'est normal.

---

## Phase 3 : configurer R-ISP

1. Cliquez sur **R-ISP** → onglet **CLI**.
2. À la question `Continue with configuration dialog? [yes/no]`, tapez `no` puis Entrée.
3. Ouvrez `configs/R-ISP.txt`, copiez tout le contenu et collez-le dans la CLI (clic droit → Paste, ou `Ctrl+V`).
4. Vérifiez :
   ```
   show ip interface brief
   ```
   Les deux interfaces `Gi0/0` et `Gi0/1` doivent être `up/up` (celle vers R1 le sera après la configuration de R1).

---

## Phase 4 : configurer les switchs

**SW1**
1. Cliquez sur **SW1** → **CLI** → appuyez sur Entrée.
2. Collez `configs/SW1.txt`.
3. Vérifiez :
   ```
   show vlan brief
   show interfaces trunk
   ```
   Attendu : VLAN 10, 20, 30, 40, 99, 666, 999 présents ; trunks sur `Gi0/1` et `Gi0/2` avec natif 999.

**SW2**
1. Même procédure avec `configs/SW2.txt`.
2. Vérifiez les mêmes commandes : trunk sur `Gi0/1`, VLAN 10, 20, 40, 99.

Si une ligne est rejetée (`% Invalid input`), notez laquelle et collez le fichier **par blocs** (VLAN, puis trunks, puis ports d'accès).

---

## Phase 5 : configurer R1

1. Cliquez sur **R1** → **CLI** → `no` à la question de configuration → collez `configs/R1.txt`.
2. À la génération de la clé RSA, si PT affiche `How many bits in the modulus [512]:`, tapez `2048` puis Entrée. Si la commande est refusée, tapez `crypto key generate rsa` puis `2048`.
3. Vérifiez :
   ```
   show ip interface brief
   show ip route
   show access-lists
   show ip dhcp pool
   ```
   Attendu : les sous-interfaces `Gi0/0.10`, `.20`, `.30`, `.40`, `.99` en `up/up`, la route par défaut `S* 0.0.0.0/0 via 203.0.113.1`, cinq ACL listées.
4. Depuis R1, testez la sortie : `ping 8.8.8.8` (il peut y avoir 1 ou 2 échecs au début à cause de l'ARP).

---

## Phase 6 : configurer les hôtes

**PC avec IP statique (PC-ADMIN)**
1. Cliquez sur le PC → onglet **Desktop** → **IP Configuration** → **Static**.
2. IP `192.168.99.10`, masque `255.255.255.0`, passerelle `192.168.99.1`, DNS `8.8.8.8`.

**PC en DHCP (PC-DIR, PC-EMP1, PC-EMP2, PC-INVITE)**
1. **Desktop** → **IP Configuration** → **DHCP**.
2. Contrôlez l'adresse obtenue (voir tableau ci-dessous).

| PC | Adresse attendue | Passerelle |
|---|---|---|
| PC-DIR | 192.168.10.10 | 192.168.10.1 |
| PC-EMP1 / PC-EMP2 | 192.168.20.10 / .11 | 192.168.20.1 |
| PC-INVITE | 192.168.40.10 | 192.168.40.1 |

**SRV-INT**
1. **Desktop** → **IP Configuration** → **Static** : IP `192.168.30.10`, masque `255.255.255.0`, passerelle `192.168.30.1`, DNS `192.168.30.10`.
2. Onglet **Services** :
   - **HTTP** : On (et **HTTPS** : On).
   - **FTP** : On (utilisateur par défaut `cisco` / mot de passe `cisco`).
   - **DNS** : On → **Name** `intranet.lab`, **Address** `192.168.30.10` → **Add**.

**SRV-INTERNET**
1. **IP Configuration** : IP `8.8.8.8`, masque `255.255.255.0`, passerelle `8.8.8.1`.
2. **Services** : **HTTP** On ; **DNS** On → **Name** `www.exemple.com`, **Address** `8.8.8.8` → **Add**.

---

## Phase 7 : premiers tests de connectivité

Sur chaque PC : **Desktop → Command Prompt**.

1. `ping 192.168.20.1` depuis PC-EMP1 → doit répondre.
2. `ping 192.168.30.10` depuis PC-EMP1 → doit répondre.
3. `ping 8.8.8.8` depuis PC-EMP1 et PC-INVITE → doit répondre.
4. Ouvrez **Web Browser** sur PC-EMP1 : `http://192.168.30.10` → page du serveur.

Si ces quatre tests passent, la base fonctionne. Passez ensuite à `tests/plan-de-tests.md` pour valider **ce qui doit être bloqué**.

---

## Phase 8 : montrer le blocage des ACL en mode Simulation

1. En bas à droite, cliquez sur **Simulation**.
2. **Edit Filters** → décochez tout, cochez **ICMP** uniquement.
3. Sur PC-INVITE, ouvrez **Command Prompt** et lancez `ping 192.168.20.10`.
4. Cliquez sur **Capture / Forward** jusqu'à ce que le paquet arrive sur R1 : il est marqué ❌ (rejeté par l'ACL).
5. Cliquez sur la ligne de l'événement pour afficher le détail : **ACL Filter → DENY**. Faites une capture d'écran.
6. Repassez en mode **Realtime** (bouton en bas à droite) pour la suite.

---

## Phase 9 : démonstration du port-security

1. **Apprentissage de la MAC** : depuis PC-EMP2, faites un `ping 192.168.20.1`. Le switch enregistre alors sa MAC (mode *sticky*).
2. Sur SW2 : `show port-security interface fa0/3` → `Maximum MAC Addresses : 1` et une MAC apprise.
3. Supprimez le câble de PC-EMP2, ajoutez un **nouveau PC** (nom `PC-INTRUS`) et câblez-le sur `Fa0/3` de SW2.
4. Passez en DHCP sur PC-INTRUS et faites un ping : le port passe en violation.
5. Sur SW2 :
   ```
   show port-security interface fa0/3
   show interfaces status err-disabled
   ```
   Attendu : `Port Status : Secure-shutdown` et `err-disabled`.
6. **Récupération** : retirez PC-INTRUS, remettez PC-EMP2, puis :
   ```
   configure terminal
   interface fa0/3
   no switchport port-security mac-address sticky
   no switchport port-security
   shutdown
   no shutdown
   switchport port-security
   switchport port-security maximum 1
   switchport port-security mac-address sticky
   end
   ```
   (cette séquence efface l'ancienne MAC apprise).

---

## Phase 10 : administration SSH

Depuis PC-ADMIN → **Command Prompt** :

```
ssh -l admin 192.168.99.1
```
Mot de passe : `Lab@Secret2026` (valeur de démo). Faites de même vers `192.168.99.2` et `192.168.99.3`.

Contre-tests :
- `ssh -l admin 192.168.99.1` depuis PC-EMP1 → doit échouer (ACL + `access-class`).
- `telnet 192.168.99.1` depuis PC-ADMIN → refusé (SSH uniquement).

---

## Phase 11 : sauvegarde et livrables

1. Sur chaque équipement : `write memory` (déjà inclus en fin de fichier).
2. **File → Save** pour enregistrer le `.pkt`.
3. Faites vos captures d'écran dans `screenshots/` avec des noms parlants : `01-topologie.png`, `02-show-vlan-brief.png`, `03-acl-deny-simulation.png`, `04-port-security-errdisabled.png`, etc.
4. Remplissez la colonne « Résultat obtenu » de `tests/plan-de-tests.md`.
5. Poussez le tout sur GitHub (`git add . && git commit && git push`).

---

## Dépannage rapide

| Symptôme | Cause probable | Solution |
|---|---|---|
| Lien reste rouge | Mauvais câble ou interface éteinte | Changez Straight/Cross-Over ; `no shutdown` sur l'interface |
| Pas d'IP en DHCP | Trunk ou VLAN mal configuré, ou ACL | `show interfaces trunk`, `show vlan brief`, `show ip dhcp pool` |
| Ping entre VLAN impossible alors qu'il devrait passer | Sous-interface `down` ou ACL trop stricte | `show ip interface brief`, `show access-lists` (regardez les `matches`) |
| `ssh` refusé | Clé RSA absente, ou `ip domain-name` manquant | Régénérez la clé (Phase 5) |
| Port en `err-disabled` | Violation de port-security ou BPDU Guard | Voir la procédure de récupération (Phase 9) |
| `% Invalid input detected` | Commande non supportée par cette version | Collez par blocs ; la ligne concernée peut être ignorée si elle est optionnelle (ex. DHCP snooping) |
| Pas d'accès à Internet depuis un VLAN | NAT ou route par défaut manquants | `show ip nat translations`, `show ip route` |
