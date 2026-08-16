# Jour 2 — EtherChannel, STP et architecture Edge/ISP

## 1. Objectifs

L'objectif du Jour 2 est de poursuivre la construction de l'infrastructure réseau en ajoutant de la redondance, en configurant EtherChannel et en préparant la mise en place de STP.

Les objectifs sont les suivants :

- Configurer la redondance entre les commutateurs.
- Configurer EtherChannel.
- Vérifier la configuration STP.
- Configurer les équipements Edge.
- Configurer les liaisons entre les Edge et les ISP.
- Relier les ISP au Cloud-PT.
- Vérifier la connectivité de l'infrastructure.

---

## 2. Architecture du réseau

L'architecture utilise une conception redondante afin d'améliorer la disponibilité du réseau.

Les équipements principaux utilisés sont :

- Switch 3
- Switch 4
- Edge1
- Edge2
- ISP1
- ISP2
- Cloud-PT
- PC
- SRV1

### Architecture générale

```text
                         CLOUD / INTERNET
                            /        \
                           /          \
                         ISP1        ISP2
                           |            |
                         Edge1        Edge2
                           \            /
                            \          /
                             \        /
                              \      /
                         Infrastructure LAN
                            /          \
                         SW3            SW4
                            \          /
                             \        /
                              REDONDANCE
```

---

## 3. EtherChannel

### 3.1 Objectif

EtherChannel permet de regrouper plusieurs liens physiques en un seul lien logique.

Cela permet :

- D'augmenter la bande passante.
- D'assurer une meilleure redondance.
- D'améliorer la disponibilité.
- De permettre à STP de considérer le groupe de liens comme un seul chemin logique.

### 3.2 Configuration

Les liens physiques redondants entre les commutateurs concernés ont été regroupés dans un EtherChannel.

Le Port-Channel représente donc logiquement l'ensemble des liens physiques regroupés.

### 3.3 Vérification

La commande utilisée pour vérifier EtherChannel est :

```bash
show etherchannel summary
```

Le résultat doit permettre de vérifier :

- La présence du Port-Channel.
- Les interfaces membres.
- L'état des interfaces.
- L'état du groupe EtherChannel.

---

## 4. Spanning Tree Protocol

### 4.1 Objectif

STP permet d'éviter les boucles de niveau 2 dans une infrastructure comportant des chemins redondants.

La présence de plusieurs liens physiques entre les commutateurs nécessite l'utilisation de STP afin de contrôler les chemins actifs et les chemins bloqués.

### 4.2 Vérification

Les commandes suivantes permettent de vérifier STP :

```bash
show spanning-tree
```

Pour vérifier un VLAN particulier :

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
```

### 4.3 Éléments à vérifier

- Le Root Bridge.
- Les Root Ports.
- Les Designated Ports.
- Les ports bloqués.
- Les états Forwarding.
- Les états Blocking.
- Le comportement de STP avec EtherChannel.

> La vérification finale de STP sera effectuée avant de considérer le Jour 2 comme totalement terminé.

---

## 5. Configuration des Edge

Deux équipements Edge sont utilisés dans l'architecture :

- Edge1
- Edge2

Ils assurent la liaison entre l'infrastructure réseau interne et les réseaux externes représentés par les ISP.

---

## 6. Plan d'adressage

Les réseaux suivants sont utilisés pour les différents segments de l'architecture Edge :

| Réseau | Utilisation |
|---|---|
| `10.0.0.0/30` | Liaison Edge1 ↔ Edge2 |
| `10.0.0.4/30` | LAN Edge1 |
| `10.0.0.8/30` | LAN Edge2 |
| `10.0.0.12/30` | Liaison Edge1 ↔ ISP1 |
| `10.0.0.16/30` | Liaison Edge2 ↔ ISP2 |

---

## 7. Liaison Edge1 ↔ Edge2

La liaison WAN directe entre Edge1 et Edge2 utilise le réseau :

```text
10.0.0.0/30
```

Adressage prévu :

| Équipement | Interface | Adresse IP |
|---|---|---|
| Edge1 | `Serial0/0/0` | `10.0.0.1/30` |
| Edge2 | `Serial0/0/0` | `10.0.0.2/30` |

Cette liaison permet aux deux équipements Edge de communiquer directement.

---

## 8. LAN Edge1

Le réseau réservé au LAN d'Edge1 est :

```text
10.0.0.4/30
```

Ce réseau est déjà utilisé pour le LAN d'Edge1.

Il ne doit pas être réutilisé pour une autre liaison.

---

## 9. LAN Edge2

Le réseau réservé au LAN d'Edge2 est :

```text
10.0.0.8/30
```

Ce réseau est déjà utilisé pour le LAN d'Edge2.

Il ne doit pas être réutilisé pour une autre liaison.

---

## 10. Liaison Edge1 ↔ ISP1

La liaison entre Edge1 et ISP1 utilise le réseau :

```text
10.0.0.12/30
```

Adressage prévu :

| Équipement | Adresse IP |
|---|---|
| Edge1 | `10.0.0.13/30` |
| ISP1 | `10.0.0.14/30` |

---

## 11. Liaison Edge2 ↔ ISP2

La liaison entre Edge2 et ISP2 utilise le réseau :

```text
10.0.0.16/30
```

Adressage prévu :

| Équipement | Adresse IP |
|---|---|
| Edge2 | `10.0.0.17/30` |
| ISP2 | `10.0.0.18/30` |

---

## 12. Connexion des ISP au Cloud-PT

Le Cloud-PT est utilisé pour représenter le réseau externe / Internet dans Packet Tracer.

Les deux ISP sont connectés séparément au Cloud.

### Architecture

```text
Edge1 ─── ISP1 ─── Cloud
Edge2 ─── ISP2 ─── Cloud
```

Les interfaces série du Cloud sont nommées :

```text
Serial0
Serial1
Serial2
...
```

La connexion prévue est :

```text
ISP1 ─── Serial ─── Cloud Serial0

ISP2 ─── Serial ─── Cloud Serial1
```

Chaque ISP utilise une interface différente du Cloud afin de conserver deux connexions distinctes.

---

## 13. Interfaces utilisées

### 13.1 Edge1

```text
Serial0/0/0 → Edge2
Serial0/0/1 → ISP1
```

### 13.2 Edge2

```text
Serial0/0/0 → Edge1
Serial0/0/1 → ISP2
```

### 13.3 ISP1

```text
Serial0/0/0 → Edge1
Serial0/0/1 → Cloud Serial0
```

### 13.4 ISP2

```text
Serial0/0/0 → Edge2
Serial0/0/1 → Cloud Serial1
```

---

## 14. Commandes de vérification

### 14.1 Vérification des interfaces

```bash
show ip interface brief
```

### 14.2 Vérification d'EtherChannel

```bash
show etherchannel summary
```

### 14.3 Vérification de STP

```bash
show spanning-tree
```

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
```

### 14.4 Vérification du routage

```bash
show ip route
```

### 14.5 Tests de connectivité

```bash
ping <adresse_IP_destination>
```

---

## 15. Tests à effectuer

Les tests suivants doivent être réalisés :

- Vérifier l'état des interfaces.
- Vérifier EtherChannel.
- Vérifier STP.
- Vérifier les routes.
- Tester la communication entre Edge1 et Edge2.
- Tester la communication entre Edge1 et ISP1.
- Tester la communication entre Edge2 et ISP2.
- Vérifier les connexions entre les ISP et le Cloud.

---

## 16. Captures d'écran

Les captures d'écran suivantes seront ajoutées au dossier `screenshots/` :

- Topologie complète du Jour 2.
- Résultat de `show etherchannel summary`.
- Résultat de `show spanning-tree`.
- Résultat de `show ip interface brief`.
- Résultats des tests de connectivité.

---

## 17. État d'avancement

- [x] Configuration d'EtherChannel
- [x] Préparation de STP
- [x] Configuration des PC
- [x] Configuration de SRV1
- [x] Préparation d'Edge1
- [x] Préparation d'Edge2
- [x] Ajout d'ISP1
- [x] Ajout d'ISP2
- [x] Connexion des ISP au Cloud-PT
- [ ] Vérification finale de STP
- [ ] Vérification finale d'EtherChannel
- [ ] Vérification finale de la connectivité
- [ ] Ajout des captures d'écran

---

## 18. Compétences travaillées

À travers ce Jour 2, les compétences suivantes ont été travaillées :

- EtherChannel
- Spanning Tree Protocol (STP)
- Redondance réseau
- Adressage IPv4
- Réseaux WAN
- Interfaces série
- Architecture Edge
- Connexion ISP
- Cisco IOS
- Packet Tracer
- Vérification et dépannage réseau
- Documentation technique

---

## 19. Conclusion

Le Jour 2 a permis de renforcer l'infrastructure réseau en ajoutant de la redondance et en préparant les mécanismes nécessaires à la haute disponibilité.

L'infrastructure dispose désormais :

- D'une architecture LAN redondante.
- D'un EtherChannel.
- De STP.
- De deux équipements Edge.
- D'une liaison WAN entre Edge1 et Edge2.
- De deux ISP.
- De connexions entre les Edge et les ISP.
- D'une représentation du réseau externe avec Cloud-PT.

Les vérifications finales de STP, d'EtherChannel et de la connectivité doivent encore être effectuées avant de considérer le Jour 2 comme complètement terminé.
