# Day 3 — Dynamic Routing, DHCP, NAT/PAT & Failover

## 1. Objectif du Day 3

L'objectif du Day 3 était de poursuivre la configuration de la topologie réseau en mettant en place les services et mécanismes nécessaires au fonctionnement complet du réseau.

Les principaux objectifs étaient :
* Configurer et vérifier OSPF.
* Annoncer correctement les réseaux des VLANs.
* Vérifier le routage inter-VLAN.
* Configurer HSRP sur les Core.
* Configurer les routes par défaut sur les Edge.
* Propager les routes par défaut avec OSPF.
* Mettre en place une sortie Internet principale et une sortie de secours.
* Configurer DHCP sur SRV1.
* Configurer le DHCP Relay.
* Configurer NAT.
* Configurer PAT.
* Configurer les interfaces internes et externes pour NAT.
* Tester la connectivité vers les ISP.
* Tester le failover entre Edge1/ISP1 et Edge2/ISP2.

---

## 2. Plan d'adressage des VLANs

Les VLANs utilisés dans la topologie sont :

| VLAN | Réseau | Passerelle virtuelle HSRP |
| :--- | :--- | :--- |
| **VLAN 10** | 192.168.10.0/24 | 192.168.10.1 |
| **VLAN 20** | 192.168.20.0/24 | 192.168.20.1 |
| **VLAN 30** | 192.168.30.0/24 | 192.168.30.1 |
| **VLAN 40** | 192.168.40.0/24 | 192.168.40.1 |
| **VLAN 50** | 192.168.50.0/24 | 192.168.50.1 |

Les adresses réelles des SVI sur les Core sont différentes de l'adresse virtuelle HSRP.

Par exemple pour le VLAN 10 :
* **SW1 Core** → 192.168.10.2
* **SW2 Core** → 192.168.10.3
* **HSRP** → 192.168.10.1

Les PC utilisent donc l'adresse virtuelle HSRP .1 comme passerelle par défaut.

---

## 3. OSPF

OSPF a été configuré afin de permettre aux équipements Layer 3 d'échanger dynamiquement leurs routes.
* **OSPF Process ID** : 50
* **Area** : 0
* Les Edge et les Core participent au processus OSPF.

---

## 4. Router IDs OSPF

Des Router IDs ont été configurés afin d'identifier les routeurs dans OSPF :
* **Edge1** : Router ID = `1.1.1.1`
* **Edge2** : Router ID = `2.2.2.2`

---

## 5. Annonce des réseaux VLAN dans OSPF

Les réseaux des VLANs doivent être annoncés par les Core puisqu'ils possèdent les SVI correspondants (`192.168.10.0/24` à `192.168.50.0/24`).

* **Problème rencontré** : Les routes `192.168.x.0/24` n'apparaissaient pas initialement dans la table de routage d'Edge1 (qui ne connaissait que certains réseaux de transit en `10.x.x.x`).
* **Correction** : Après vérification, la configuration OSPF des Core a été corrigée pour annoncer correctement les réseaux VLAN. Edge1 a ensuite appris ces réseaux via des routes OSPF (`O 192.168.x.0/24`).

---

## 6. Interfaces Access et OSPF

* Les liens entre les switches Access et les Core sont principalement des liens Layer 2/trunks.
* Ils n'ont pas besoin d'être annoncés individuellement dans OSPF lorsqu'ils ne possèdent pas d'adresse IP de routage. Ce sont les SVI des VLANs qui doivent l'être.

---

## 7. Routes par défaut

Chaque Edge possède sa propre route par défaut vers son ISP :

* **Edge1 → ISP1**
  * Réseau de transit : `203.0.113.0/30`
  * ISP1 = `203.0.113.1` | Edge1 = `203.0.113.2`
  * Route par défaut : `0.0.0.0/0 → ISP1`

* **Edge2 → ISP2**
  * Réseau de transit : `198.51.100.0/30`
  * ISP2 = `198.51.100.1` | Edge2 = `198.51.100.2`
  * Route par défaut : `0.0.0.0/0 → ISP2`

---

## 8. Propagation des routes par défaut

Les deux Edge ont été configurés pour annoncer leur route par défaut dans OSPF avec :

default-information originate

L'objectif est de permettre aux Core de connaître une route vers les réseaux externes.

Le fonctionnement recherché est :

ISP1
|
Edge1
PRINCIPAL
|
Core
|
Edge2
SECOURS
|
ISP2

Edge1 doit être privilégié lorsque le lien vers ISP1 est disponible.

Edge2 doit être utilisé comme chemin de secours lorsque Edge1/ISP1 n'est plus disponible.


---

## 9. Coût OSPF

Afin de faire d'Edge1 le chemin principal et d'Edge2 le chemin de secours, le coût OSPF peut être ajusté.

Le principe est :

Chemin Edge1 → coût faible
Chemin Edge2 → coût élevé

Par exemple, le coût de l'interface reliant SW2 Core à Edge2 peut être augmenté.

Cela permet au Core de préférer le chemin vers Edge1 lorsque les deux chemins sont disponibles.

L'objectif est donc :

Chemin normal :

PC
|
Core
|
Edge1
|
ISP1


Chemin après panne :

PC
|
Core
|
Edge2
|
ISP2


---

## 10. Vérification du routage par défaut

Les routes par défaut ont été vérifiées avec la table de routage.

Une route par défaut apparaît avec :

S*

lorsqu'elle est statique.

Les routes OSPF apparaissent avec :

O

La présence des routes OSPF a également été vérifiée après avoir activé OSPF correctement sur les Core.


---

## 11. DHCP

Le serveur DHCP utilisé est :

SRV1

SRV1 est connecté au :

SW4 Access

Les pools DHCP ont été créés sur SRV1 pour les différents VLANs.

Les pools correspondent aux réseaux :

192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
192.168.40.0/24
192.168.50.0/24

Chaque pool fournit aux clients :

une adresse IP ;

un masque de sous-réseau ;

une passerelle par défaut ;

une durée de bail.


Le DNS n'a volontairement pas été configuré à cette étape.


---

## 12. DHCP Relay

Le serveur DHCP étant situé dans un réseau différent de celui des clients, un DHCP Relay est nécessaire.

Un client DHCP envoie initialement une requête sous forme de broadcast.

Les routeurs ne transmettent pas automatiquement les broadcasts entre différents réseaux.

Le DHCP Relay permet donc au routeur ou au SVI de transmettre la demande au serveur DHCP.

Le fonctionnement est :

PC VLAN 10
|
| DHCP Broadcast
v
SVI VLAN 10
|
| DHCP Relay
v
SRV1
|
| DHCP Response
v
PC VLAN 10

Le DHCP Relay est configuré sur les interfaces Layer 3 correspondant aux VLANs.

La commande utilisée est basée sur :

ip helper-address <IP-de-SRV1>


---

## 13. Tests DHCP

Les clients ont été testés afin de vérifier qu'ils reçoivent automatiquement leurs paramètres réseau.

Les paramètres obtenus comprennent notamment :

IP Address
Subnet Mask
Default Gateway

La configuration DNS a été laissée de côté volontairement pour le moment.


---

## 14. Adressage Edge ↔ ISP

Edge1 ↔ ISP1

Réseau : 203.0.113.0/30

ISP1 :
203.0.113.1/30

Edge1 :
203.0.113.2/30

Edge2 ↔ ISP2

Réseau : 198.51.100.0/30

ISP2 :
198.51.100.1/30

Edge2 :
198.51.100.2/30

Les interfaces reliant les Edge aux ISP sont utilisées comme interfaces outside pour NAT.


---

## 15. NAT

NAT a été configuré afin de traduire les adresses privées des VLANs vers des adresses publiques.

Les réseaux privés concernés sont :

192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
192.168.40.0/24
192.168.50.0/24


---

###15.1 NAT Edge1

Le pool configuré sur Edge1 est :

Pool :
203.0.113.9 → 203.0.113.14

Network :
203.0.113.8/29

Mask :
255.255.255.248

L'ACL autorise les réseaux VLAN :

access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.30.0 0.0.0.255
access-list 1 permit 192.168.40.0 0.0.0.255
access-list 1 permit 192.168.50.0 0.0.0.255

La traduction est configurée avec :

ip nat inside source list 1 pool Public overload

L'interface interne est :

interface GigabitEthernet0/1
ip nat inside

L'interface externe est :

interface Serial0/0/1
ip nat outside


---

### 15.2 PAT

PAT a été configuré en utilisant le mot-clé :

overload

Le principe est de permettre à plusieurs machines privées de partager une ou plusieurs adresses publiques.

Exemple :

192.168.10.10 ─┐
192.168.10.11 ─┤
192.168.20.10 ─┤
192.168.30.10 ─┼──> Edge1 ──> PAT ──> ISP1
192.168.40.10 ─┤
192.168.50.10 ─┘

La traduction utilise les ports TCP/UDP pour différencier les différentes connexions.


---

### 15.3  NAT/PAT sur Edge2

Edge2 a également été configuré pour fournir une sortie externe de secours.

La configuration de R2-EDGE contient :

interface GigabitEthernet0/1
ip address 10.0.0.9 255.255.255.252
ip ospf 50 area 0
ip nat inside

Cette interface est l'interface interne vers le Core.

L'interface externe est :

interface Serial0/0/1
ip address 198.51.100.2 255.255.255.252
ip nat outside

Le pool NAT d'Edge2 est :

ip nat pool Public 198.51.100.9 198.51.100.14 netmask 255.255.255.248

Le réseau correspondant au pool est :

198.51.100.8/29

L'ACL NAT contient :

access-list 2 permit 192.168.10.0 0.0.0.255
access-list 2 permit 192.168.20.0 0.0.0.255
access-list 2 permit 192.168.30.0 0.0.0.255
access-list 2 permit 192.168.40.0 0.0.0.255
access-list 2 permit 192.168.50.0 0.0.0.255

Le PAT est activé avec :

ip nat inside source list 2 pool Public overload

La route par défaut d'Edge2 est :

ip route 0.0.0.0 0.0.0.0 Serial0/0/1

Le processus OSPF d'Edge2 est :

router ospf 50
router-id 2.2.2.2
default-information originate


---

## 16.  Tests de connectivité, NAT et Failover

###16.1 Test PC → Edge1

Un premier problème a été rencontré : le PC du VLAN 10 ne pouvait pas atteindre correctement Edge1.

Après vérification de la table de routage d'Edge1, il a été constaté que les réseaux 192.168.x.0/24 n'étaient pas présents.

La cause était liée à l'absence d'annonce correcte des réseaux VLAN dans OSPF sur les Core.

Après correction, les routes VLAN ont été correctement annoncées.

Le PC a alors pu atteindre l'interface d'Edge1.


---

###16.2 Test Edge1 → PC

Après correction d'OSPF, Edge1 pouvait également atteindre les réseaux VLAN.

Les routes OSPF étaient présentes dans la table de routage.

Le routage interne fonctionnait donc correctement.


---

###16.3 Test Edge1 → ISP1

Le lien entre Edge1 et ISP1 a été testé directement.

Edge1 :
203.0.113.2

ISP1 :
203.0.113.1

Le ping fonctionne correctement :

Edge1 → ISP1

Le lien physique et IP entre Edge1 et ISP1 est donc fonctionnel.


---

###16.4 Test NAT

Le NAT a ensuite été testé depuis un PC du VLAN 10.

Après correction du routage, la traduction NAT fonctionne correctement.

Le PC peut utiliser Edge1 pour sortir vers le réseau externe.


---

###16.5 Test du failover

Pour tester la redondance, le lien suivant a été coupé :

Edge1 ↔ ISP1

Le but est de forcer le trafic à utiliser :

PC
|
Core
|
Edge2
|
ISP2

Les routes par défaut ont été vérifiées.

R2-EDGE possède bien une route par défaut vers ISP2 :

ip route 0.0.0.0 0.0.0.0 Serial0/0/1

Le Core apprend également la route par défaut annoncée par Edge2 grâce à OSPF.


---

###16.6 Test PC → Edge2

Lorsque le lien Edge1 ↔ ISP1 est coupé, le PC peut atteindre l'interface d'Edge2.

Cela confirme que le chemin interne vers Edge2 fonctionne :

PC
↓
Core
↓
Edge2


---

###16.7 Test Edge2 → ISP2

Le lien direct entre Edge2 et ISP2 fonctionne.

Les deux interfaces sont :

Edge2 :
198.51.100.2

ISP2 :
198.51.100.1

Un ping effectué depuis Edge2 vers ISP2 fonctionne.

Le lien direct Edge2 ↔ ISP2 est donc opérationnel.


---

###16.8 Problème restant pendant le failover

Cependant, lorsque le lien Edge1 ↔ ISP1 est coupé, le PC n'arrive pas à pinguer directement l'interface d'ISP2 :

PC → 198.51.100.1

Le PC peut atteindre :

198.51.100.2

qui correspond à l'interface d'Edge2.

Mais :

PC → 198.51.100.1

échoue avec :

Destination Host Unreachable

Cela montre que :

PC → Core → Edge2

fonctionne,

mais que le trafic provenant du PC ne parvient pas correctement jusqu'à ISP2 ou que le trafic de retour vers le réseau interne n'est pas correctement traité.

Les éléments suivants ont déjà été vérifiés :

La route par défaut d'Edge2 existe.

La route par défaut d'Edge2 pointe vers ISP2.

Le Core apprend la route par défaut via Edge2.

Le lien Edge2 ↔ ISP2 est actif.

Edge2 peut ping ISP2 directement.

La configuration NAT/PAT d'Edge2 existe.


Le problème restant est donc à isoler entre :

PC → Edge2 → ISP2

et le chemin de retour associé au trafic provenant du réseau privé.


---

###17 📋 État du projet après le Day 3

Fonctionnel

[x] VLAN 10

[x] VLAN 20

[x] VLAN 30

[x] VLAN 40

[x] VLAN 50

[x] SVI sur les Core

[x] ip routing

[x] HSRP

[x] Passerelles virtuelles

[x] OSPF

[x] Router IDs

[x] Annonce des réseaux VLAN dans OSPF

[x] Routage inter-VLAN

[x] Routes par défaut sur Edge1 et Edge2

[x] Propagation des routes par défaut

[x] Préférence d'Edge1 comme sortie principale

[x] Configuration d'Edge2 comme sortie de secours

[x] DHCP sur SRV1

[x] Pools DHCP

[x] DHCP Relay

[x] NAT sur Edge1

[x] PAT sur Edge1

[x] NAT/PAT sur Edge2

[x] Connectivité Edge1 ↔ ISP1

[x] Connectivité Edge2 ↔ ISP2

[x] Test de NAT via Edge1

[x] Test du routage interne

[x] Test du chemin de secours vers Edge2


À finaliser

[ ] Finaliser la connectivité PC → ISP2 pendant le failover.

[ ] Vérifier le chemin de retour du trafic via ISP2.

[ ] Vérifier le routage du pool NAT 198.51.100.8/29 sur ISP2.

[ ] Tester complètement le NAT/PAT via Edge2.

[ ] Tester plusieurs clients simultanément avec PAT.

[ ] Tester la sortie externe complète via ISP1.

[ ] Tester la sortie externe complète via ISP2.

[ ] Restaurer le lien Edge1 ↔ ISP1.

[ ] Vérifier qu'Edge1 redevient automatiquement le chemin principal.

[ ] Configurer DNS ultérieurement si nécessaire.

[ ] Effectuer une vérification finale de toute la topologie.



---

###18. Conclusion

Le Day 3 a permis de mettre en place la majorité des mécanismes nécessaires au fonctionnement d'une infrastructure réseau dynamique, redondante et capable de fournir des services aux clients.

Le fonctionnement général est maintenant basé sur :

INTERNET
/ \
ISP1 ISP2
| |
Edge1 Edge2
PRINCIPAL SECOURS
\ /
\ /
OSPF/Core
|
+--------+--------+
| |
SW1 Core SW2 Core
| |
+--------+--------+
|
HSRP / SVI
|
+------------+------------+
| |
SW3 Access SW4 Access
/ \ / \
VLAN 10 VLAN 20 VLAN 30 VLAN 50
|
SRV1
|
DHCP

Les Core assurent le routage inter-VLAN et la redondance de la passerelle avec HSRP.

OSPF assure l'échange dynamique des routes.

Les Edge assurent les sorties vers les ISP.

Edge1 est configuré comme sortie principale.

Edge2 est configuré comme sortie de secours.

DHCP et DHCP Relay permettent la configuration automatique des clients.

NAT et PAT permettent la traduction des adresses privées vers les réseaux externes.

Le dernier point majeur à résoudre est la validation complète du chemin de sortie via Edge2 et ISP2 lorsque le lien Edge1 ↔ ISP1 est coupé. Petite, tu sais un peu m'écrire tous ces textes là pour que je puisse mettre ça dans un fichier en format .md avec les et tout ça en un seul bloc s'il te plaît. 















