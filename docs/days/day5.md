# Day 5 - Validation et Corrections

## 1. Résumé des Tests (Tous Réussis)
* **DNS / Server Configuration** : Service DNS activé et enregistrement de type A configuré avec succès.
* **Route Failover Testing** : Basculement automatique des routes validé en cas de coupure de lien.
* **Internet / External Connectivity** : Connectivité vers le Cloud Internet pleinement opérationnelle.
* **Network Services Verification** : Relais DHCP fonctionnel à travers l'infrastructure.
* **Full Connectivity Testing** : Communications inter-VLAN et de bout en bout validées par pings.
* **HSRP Failover Testing** : Redondance de la passerelle par défaut assurée sans coupure.
* **EtherChannel Failure Testing** : Liens agrégés opérationnels et tolérants aux pannes physiques.
* **STP Failover Testing** : Prévention des boucles et réactivation automatique des chemins (Rapid-PVST).
* **OSPF Failure Testing** : Convergence dynamique et mise à jour des tables de routage (Area 0).
* **DHCP Testing** : Attribution dynamique des adresses IP aux clients sur les différents VLANs.
* **DNS Testing** : Résolution de noms validée via la commande `nslookup`.
* **NAT/PAT testing** : Traduction d'adresses opérationnelle sur les routeurs de bordure.
* **Security testing** : Sécurité de niveau 2 (Port Security, DHCP Snooping, DAI) en place et active.

## 2. Erreurs Rencontrées et Corrigées
* **Mode des interfaces Port-channel / Trunks** : Les interfaces étaient configurées en mode `access` au lieu de `trunk`, ce qui bloquait le passage des VLANs vers les passerelles HSRP. -> *Correction : passage des interfaces et port-channels en mode `trunk`.*
* **Blocage du DHCP par le DHCP Snooping** : Le switch bloquait les paquets relayés avec un champ `giaddr` non nul (erreur `%DHCP_SNOOPING-5-DHCP_SNOOPING_NONZERO_GIADDR`). -> *Correction : configuration des ports uplinks inter-switchs en mode `trust` (`ip dhcp snooping trust`).*
* Le DHCP snooping a du être désactivé afin d'éviter de bloquer les paquets.
* **Absence d'enregistrement DNS** : La commande `nslookup` renvoyait une erreur `Non-existent domain`. -> *Correction : ajout de l'enregistrement de type A (`www.entreprise.local`) dans la configuration du service DNS du serveur.*
