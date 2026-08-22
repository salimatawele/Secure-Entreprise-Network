# Day 4 - Sécurité L2 et ACL

## 1. Sécurisation de l'Administration & Accès
* **SSH & Authentification :** Remplacement de Telnet par SSH avec clés RSA et base d'utilisateurs locaux.
* **Mots de passe :** Protection renforcée des lignes console (`line con 0`) et du mode privilégié (`enable secret`).

## 2. Sécurité des Ports (Layer 2)
* **Ports inutilisés :** Désactivation des ports vacants et basculement dans un VLAN noir (`shutdown` / VLAN 999).
* **Port Security & Sticky MAC :** Limitation du nombre d'adresses MAC par port et apprentissage dynamique sécurisé.

## 3. DHCP & ARP Snooping
* **DHCP Snooping :** Blocage des faux serveurs DHCP et désignation des ports uplinks de confiance (`trust`).
* **Dynamic ARP Inspection (DAI) :** Protection contre l'empoisonnement ARP basée sur la table du DHCP Snooping.

## 4. Vérification & Tests
* **Vérification L2 :** Contrôle via les commandes `show port-security` et `show ip dhcp snooping binding`.
* **Test de violation :** Simulation d'intrusion entraînant la mise en erreur du port (`err-disabled`).

## 5. Filtrage par ACL
* **Isolation Invités :** Restriction du trafic invité vers Internet uniquement via des ACL étendues.
* **Protection Serveurs / Gestion :** Filtrage inter-VLAN empêchant l'accès non autorisé aux zones sensibles.
