# 🔐 Implémentation du AAA avec TACACS+ — Réseau sécurisé pour un hôpital

> **Projet de fin de semestre** — Module *Sécurité des infrastructures IT*
> **Filière RSSP** (Réseaux, Systèmes et Services Programmables) — **ENSA Marrakech**, Université Cadi Ayyad
> Année universitaire **2025-2026**

![Cisco Packet Tracer](https://img.shields.io/badge/Simulation-Cisco%20Packet%20Tracer-1BA0D7)
![Protocole](https://img.shields.io/badge/Protocole-TACACS%2B-green)
![AAA](https://img.shields.io/badge/AAA-Authentication%20%7C%20Authorization%20%7C%20Accounting-blue)
![Statut](https://img.shields.io/badge/Statut-Termin%C3%A9-success)

---

## 📑 Table des matières

1. [Présentation du projet](#-présentation-du-projet)
2. [Objectifs](#-objectifs)
3. [Contexte : l'hôpital](#-contexte--lhôpital)
4. [Architecture réseau](#-architecture-réseau)
5. [Équipements utilisés](#-équipements-utilisés)
6. [Plan d'adressage et VLANs](#-plan-dadressage-et-vlans)
7. [Technologies et protocoles mis en œuvre](#-technologies-et-protocoles-mis-en-œuvre)
8. [Configuration des équipements](#-configuration-des-équipements)
9. [Implémentation de TACACS+ (AAA)](#-implémentation-de-tacacs-aaa)
10. [Tests et vérifications](#-tests-et-vérifications)
11. [Résultats](#-résultats)
12. [Bonnes pratiques et pistes d'amélioration](#-bonnes-pratiques-et-pistes-damélioration)
13. [Prérequis et reproduction](#-prérequis-et-reproduction)
14. [Structure du dépôt](#-structure-du-dépôt)
15. [Équipe et encadrement](#-équipe-et-encadrement)
16. [Références](#-références)

---

## 📖 Présentation du projet

Ce projet consiste à **concevoir et simuler une infrastructure réseau sécurisée pour un hôpital moderne**. Les hôpitaux manipulent des données très sensibles (dossiers médicaux, informations administratives, échanges inter-services) et sont donc exposés à des risques d'intrusion, de compromission et de perturbation des communications internes.

Le cœur du dispositif de sécurité est un **serveur TACACS+ placé dans la DMZ**, qui assure la gestion **centralisée** de l'**AAA** (*Authentication, Authorization, Accounting*) pour l'ensemble des équipements d'interconnexion : routeurs, switches multicouches et pare-feu.

La simulation est réalisée sous **Cisco Packet Tracer**, sur une architecture hiérarchique à trois niveaux (Core – Distribution – Accès).

---

## 🎯 Objectifs

- Concevoir une architecture réseau **hiérarchique, redondante et évolutive** pour un hôpital.
- **Centraliser l'authentification, l'autorisation et la traçabilité** des accès administratifs grâce à TACACS+.
- Appliquer une **segmentation logique** du réseau via des VLANs et une **zone DMZ** pour les serveurs.
- Garantir la **haute disponibilité** (HSRP, EtherChannel, STP, double cœur de réseau).
- Renforcer la confidentialité et la traçabilité des connexions d'administration (**SSH**, **ACL**).
- Prévoir un **mécanisme de secours** (authentification locale) en cas d'indisponibilité du serveur TACACS+.
- Valider le fonctionnement par des **tests de connectivité et d'accès**.

---

## 🏥 Contexte : l'hôpital

L'hôpital est réparti sur **trois niveaux**, chacun avec ses propres besoins de connectivité, de performance et de sécurité :

| Niveau | Services | Particularités |
|---|---|---|
| **Rez-de-chaussée** | Pharmacie, laboratoires médicaux, accueil et zone visiteurs | Fort trafic (accès public, échanges labos ↔ administration) |
| **1er étage** | Bureaux des médecins, salles de consultation, RH, achats, finances | Données sensibles nécessitant une sécurité renforcée |
| **2e étage** | Direction, auditeurs internes, équipe IT (support, sécurité réseau, développement, administration système) | Centralise les équipements de supervision et de sécurité (serveur TACACS+, pare-feu, switches cœur) |

---

## 🗺️ Architecture réseau

L'architecture suit le modèle **hiérarchique à trois couches** :

```
                       ┌──────────────────────┐
                       │   AWS CLOUD (30.0.0.0/8)   │
                       └───────────┬──────────┘
                              AWS-ROUTER
                                   │  20.20.20.0/30
                              ISP-ROUTER
                                   │  197.200.100.0/30
                      ┌────────────┴────────────┐
                      │  PERIMETER-FW (ASA 5500-X)  │──────────► DMZ (10.20.10.0/26)
                      └────────────┬────────────┘     • Serveur TACACS+ (10.20.10.11)
                                   │                   • Serveur DHCP   (10.20.10.10)
                              CORE-ROUTER              • Stockage NetApp primaire / secondaire
                          ┌────────┴────────┐
                   CORE-MLSW1  ══EtherChannel══  CORE-MLSW2
                   (HSRP actif)                  (HSRP standby)
                          │   (liens vers la couche accès)   │
        ┌─────────────┬───┴──────────┬──────────────┬────────┴────┐
   Rez-de-chaussée     1er étage        2e étage       Réseau VoIP / WLAN
   (Labos, Pharmacie,  (Médecins, RH,   (Auditeurs,    (Contrôleur, routeur
    Réception)          Achats, Fin.)    équipe IT)      VoIP)
```

**Points clés de l'architecture :**

- **Cœur de réseau redondant** : deux switches multicouches (`CORE-MLSW1` / `CORE-MLSW2`) reliés par **EtherChannel**, avec **HSRP** pour la passerelle virtuelle (MLSW1 actif, MLSW2 en standby).
- **Pare-feu périmétrique** (`PERIMETER-FW`, ASA 5500-X) séparant l'Internet, la DMZ et le LAN interne.
- **DMZ** hébergeant les serveurs critiques : **TACACS+**, **DHCP**, stockage **NetApp** (primaire et secondaire).
- **Connexion cloud** : un réseau **AWS** simulé, joignable via l'ISP.
- **Couche accès** : un switch par service, répartis sur les trois étages.

> 📷 La topologie complète est disponible dans le rapport PDF (section V, « Topologie ») et dans le fichier Packet Tracer.

---

## 🧰 Équipements utilisés

| Quantité | Équipement | Rôle |
|:---:|---|---|
| 9 | Switches **Catalyst 2960** | Couche accès / DMZ |
| 2 | Multilayer Switches **3650-24PS** | Cœur de réseau (routage inter-VLAN, HSRP) |
| 2 | Routeurs **ISR4331** | Routeurs ISP / AWS |
| 1 | Routeur **2811** | Routeur VoIP |
| 1 | Firewall **ASA 5500-X** | Pare-feu périmétrique |
| 6 | **Server-PT** | TACACS+, DHCP, NetApp, serveurs AWS… |
| 14 | **PC-PT** | Postes utilisateurs |
| 9 | **Printer-PT** | Imprimantes |
| 6 | **IP Phone 7960** | Téléphonie IP |

---

## 🌐 Plan d'adressage et VLANs

### Sous-réseaux

| Sous-réseau | VLAN | Adresse réseau | Masque | Passerelle(s) |
|---|:---:|---|---|---|
| **LAN** | 10 | `192.168.0.0` | `255.255.240.0` (/20) | `192.168.0.1` (virtuelle HSRP), `.2`, `.3` |
| **WLAN** | 50 | `10.10.0.0` | `255.255.0.0` (/16) | `10.10.0.1` (virtuelle HSRP), `.2`, `.3` |
| **VoIP** | 99 | `172.16.0.0` | `255.255.255.0` (/24) | `172.16.0.1` |
| **DMZ** | — | `10.20.10.0` | `255.255.255.192` (/26) | `10.20.10.1` |
| **AWS Cloud** | — | `30.0.0.0` | `255.0.0.0` (/8) | `30.0.0.1` |

### Adressage des interfaces des équipements d'interconnexion

| Équipement | Interface | Adresse IP |
|---|---|---|
| **CORE-MLSW1** | Gig1/0/1 | `10.30.10.5` |
| | Interface VLAN 10 | `192.168.0.3` |
| | Interface VLAN 50 | `10.10.0.3` |
| **CORE-MLSW2** | Gig1/0/1 | `10.30.10.9` |
| | Interface VLAN 10 | `192.168.0.2` |
| | Interface VLAN 50 | `10.10.0.2` |
| **CORE-ROUTER** | Fa0/0 | `10.30.10.1` |
| | Fa0/1 | `10.30.10.6` |
| | Fa1/0 | `10.30.10.10` |
| **PERIMETER-FW** | Gig1/1 | `10.30.10.2` |
| | Gig1/2 (DMZ) | `10.20.10.1` |
| | Gig1/3 (vers ISP) | `197.200.100.2` |
| **ISP-ROUTER** | Gig0/0/0 | `197.200.100.1` |
| | Gig0/0/1 | `20.20.20.1` |
| **AWS-ROUTER** | Gig0/0/0 | `20.20.20.2` |
| | Gig0/0/1 | `30.0.0.1` |
| **VoIP-ROUTER** | Fa0/0.99 | `172.16.0.1` |

### Serveurs de la DMZ

| Serveur | Adresse IP |
|---|---|
| Serveur DHCP | `10.20.10.10` |
| **Serveur TACACS+** | `10.20.10.11` |
| Stockage NetApp primaire / secondaire | `10.20.10.x` (voir topologie) |

---

## ⚙️ Technologies et protocoles mis en œuvre

| Technologie | Rôle dans le projet |
|---|---|
| **TACACS+** | Authentification, autorisation et comptabilité centralisées des accès administratifs |
| **AAA** | Modèle de sécurité : *qui* (authentification), *quoi* (autorisation), *quand/combien* (accounting) |
| **OSPF** | Routage dynamique entre les équipements |
| **HSRP** | Redondance de la passerelle par défaut (haute disponibilité) |
| **STP** | Prévention des boucles de niveau 2 |
| **EtherChannel** | Agrégation de liens entre les deux switches cœur |
| **VLANs** | Segmentation logique du réseau (LAN, WLAN, VoIP) |
| **DHCP** | Attribution dynamique des adresses (serveur en DMZ) |
| **ACL** | Filtrage et restriction des accès |
| **SSH** | Accès distant chiffré aux équipements (Telnet désactivé sur les lignes VTY) |
| **Pare-feu ASA** | Filtrage périmétrique et isolation de la DMZ |
| **VoIP** | Téléphonie IP via routeur dédié (VLAN 99) |

---

## 🛠️ Configuration des équipements

Le rapport détaille, pour chaque type d'équipement, la configuration appliquée :

1. **Switches (accès)** — VLANs, trunks, STP, EtherChannel, sécurisation des ports.
2. **Multilayer switches (cœur)** — routage inter-VLAN, interfaces SVI, HSRP, OSPF, EtherChannel.
3. **Routeurs** — adressage, OSPF, routes vers l'ISP et le cloud AWS.
4. **Pare-feu ASA** — zones de sécurité (inside / DMZ / outside), règles de filtrage, NAT.
5. **Routeur VoIP** — sous-interface `Fa0/0.99`, service de téléphonie pour les IP Phones.
6. **Serveur DHCP** — pools d'adresses par VLAN, hébergé dans la DMZ.

---

## 🔑 Implémentation de TACACS+ (AAA)

### 1. Serveur TACACS+ (DMZ — `10.20.10.11`)

Le serveur est configuré avec :

- **Clients AAA déclarés** (équipements autorisés à interroger le serveur, type *Tacacs*, avec une clé partagée) :

| Client | Adresse IP |
|---|---|
| `PERIMETER-FW` | `10.20.10.1` |
| `WAN-R1` (CORE-ROUTER) | `10.30.10.1` |
| `CORE-MLSW1` | `10.30.10.5` |
| `CORE-MLSW2` | `10.30.10.9` |

- **Utilisateurs** : `irera`, `kodjo`, `kossi`, `mireille`.

### 2. Configuration AAA sur les équipements d'interconnexion

La configuration est **identique sur les multilayer switches et les routeurs** :

```cisco
! Activation du modèle AAA
aaa new-model

! Authentification : TACACS+ en priorité, base locale en secours
aaa authentication login hospital group tacacs+ local

! Autorisation : TACACS+ puis local
aaa authorization exec hospital group tacacs+ local

! Comptabilité : journalisation début/fin de session sur TACACS+
aaa accounting exec hospital start-stop group tacacs+

! Déclaration du serveur TACACS+
tacacs-server host 10.20.10.11 key <CLE_PARTAGEE>

! Comptes locaux de secours, avec niveaux de privilège
username irera    privilege 10 secret <MOT_DE_PASSE>
username kodjo    privilege 5  secret <MOT_DE_PASSE>
username kossi    privilege 15 secret <MOT_DE_PASSE>
username mireille privilege 7  secret <MOT_DE_PASSE>

! Application sur les lignes d'administration à distance
line vty 0 4
 login authentication hospital
 transport input ssh

! Bonnes pratiques
service password-encryption
banner motd ^C NO UNAUTHORIZED ACCESS IS PUNISHABLE !!! ^C
```

> ⚠️ Les clés partagées et mots de passe réels du laboratoire ont volontairement été remplacés par des *placeholders* dans ce README.

### 3. Gestion des privilèges

Les niveaux de privilège permettent d'appliquer le principe du **moindre privilège** : chaque administrateur n'accède qu'aux commandes nécessaires à son rôle.

| Utilisateur | Niveau de privilège | Rôle |
|---|:---:|---|
| `kossi` | **15** | Ingénieur sécurité (accès complet) |
| `irera` | 10 | Privilèges intermédiaires |
| `mireille` | 7 | Privilèges limités |
| `kodjo` | 5 | Technicien réseau (consultation) |

Exemple de commandes autorisées au niveau 5 (technicien réseau) sur `CORE-ROUTER` :

```cisco
privilege exec level 5 show running-config
privilege exec level 5 show ip interface brief
privilege exec level 5 show interfaces
privilege exec level 5 show version
```

### 4. Principe de fonctionnement

```
 Administrateur ──SSH──► Équipement réseau (client TACACS+)
                              │
                              │ 1. Requête d'authentification
                              ▼
                      Serveur TACACS+ (DMZ)
                              │
        ┌─────────────────────┼─────────────────────┐
   Authentification      Autorisation          Accounting
   (identité valide ?)   (quels droits ?)      (qui a fait quoi ?)
                              │
              Serveur injoignable ? ──► Repli sur la base locale
```

---

## ✅ Tests et vérifications

### 1. Connectivité

| Test | Résultat |
|---|:---:|
| Ping d'un PC interne → réseau **AWS** | ✅ |
| Ping d'un PC interne → serveur **NetApp** (DMZ) | ✅ |
| Ping entre PC internes (inter-VLAN) | ✅ |

### 2. Vérification de TACACS+

| # | Scénario | Résultat attendu | Résultat obtenu |
|:-:|---|---|:---:|
| a | Accès à `CORE-ROUTER` depuis le PC de l'**ingénieur sécurité** | Accès autorisé via TACACS+ | ✅ |
| b | Accès à `CORE-ROUTER` depuis le PC du **technicien réseau** | Accès autorisé avec privilèges restreints | ✅ |
| c | Accès à `CORE-MLSW1` et `CORE-MLSW2` par `kossi` | Accès autorisé ; le compteur de sessions TACACS+ augmente à chaque connexion | ✅ |
| d | **Serveur TACACS+ indisponible** (interface `Fa0/2` du switch `DMZ-SW` désactivée) | Connexion possible via le **compte local** (`kodjo`) | ✅ |
| e | Accès avec un **utilisateur non déclaré** | Accès **refusé** | ✅ |

---

## 📊 Résultats

- ✅ **Authentification et autorisation sélectives** des administrateurs.
- ✅ **Administration centralisée** : un seul point de gestion des comptes pour tous les équipements d'interconnexion.
- ✅ **Traçabilité** des connexions (accounting `start-stop`).
- ✅ **Résilience** : la méthode de repli `local` garantit l'accès d'urgence si le serveur TACACS+ est hors service.
- ✅ **Sécurité renforcée** : SSH uniquement, mots de passe chiffrés, bannière d'avertissement, contrôle des privilèges.
- ✅ **Réseau hautement disponible** grâce à HSRP, EtherChannel, STP et OSPF.

---

## 🚀 Bonnes pratiques et pistes d'amélioration

Pistes d'évolution identifiées pour aller plus loin que le périmètre de la simulation :

- 🔒 Utiliser des **mots de passe et clés partagées complexes et uniques**, jamais réutilisés d'un compte ou d'un équipement à l'autre.
- 👥 Créer des **groupes d'utilisateurs / profils de commandes** (autorisation par commande avec `aaa authorization commands`).
- 📝 Activer l'**accounting des commandes** (`aaa accounting commands`) pour un audit plus fin.
- 🛡️ Mettre en place un **second serveur TACACS+** pour la redondance.
- 🪪 Ajouter une **authentification multifacteur (MFA)**.
- 📡 Centraliser les journaux dans un **serveur Syslog / SIEM**.
- 🔐 Déployer **802.1X** et la **Port-Security** sur la couche accès.
- 📋 Renforcer les **ACL** et durcir les VLANs de gestion (VLAN de management dédié).

---

## 💻 Prérequis et reproduction

### Prérequis

- **Cisco Packet Tracer** (version 8.x recommandée)
- Notions de base en configuration Cisco IOS

### Reproduire le projet

1. Cloner le dépôt :
   ```bash
   git clone https://github.com/kossibihaho/Implementation_du_AAA.git
   cd Implementation_du_AAA
   ```
2. Ouvrir le fichier de simulation `.pkt` dans **Cisco Packet Tracer** (si présent dans le dépôt).
3. Consulter le **rapport PDF** pour la topologie, le plan d'adressage et les configurations.
4. Vérifier que le service **AAA** est activé (*On*) sur le serveur `10.20.10.11` et que les clients et utilisateurs y sont déclarés.
5. Appliquer la configuration AAA sur les équipements, puis exécuter les tests de la section [Tests et vérifications](#-tests-et-vérifications).

---

## 📁 Structure du dépôt

```
Implementation_du_AAA/
├── Rapport de projet_sécuritéIT.pdf   # Rapport complet du projet (33 pages)
└── README.md                          # Ce fichier
```

---

## 👥 Équipe et encadrement

**Réalisé par :**

| Nom | GitHub |
|---|---|
| AGBEVO Kodjo Honoré | — |
| BIHAHO Kossi | [@kossibihaho](https://github.com/kossibihaho) |
| IRERA Furaha Jospine | — |
| RAKOTOMANGA Manohisoa Mireille | — |

**Encadrante :** Mme **ABBASS Wissam**

**Établissement :** École Nationale des Sciences Appliquées de Marrakech (ENSA Marrakech) — Université Cadi Ayyad
**Filière :** Réseaux, Systèmes et Services Programmables (**RSSP**) — 2025-2026

---

## 📚 Références

- Cours de *Sécurité des infrastructures IT* — Mme ABBASS Wissam
- [Cisco.com](https://www.cisco.com)
- [Cisco Networking Academy](https://www.netacad.com)

---

## 📖 Glossaire

| Sigle | Signification |
|---|---|
| **TACACS+** | Terminal Access Controller Access-Control System Plus |
| **AAA** | Authentication, Authorization, Accounting |
| **VLAN** | Virtual Local Area Network |
| **OSPF** | Open Shortest Path First |
| **HSRP** | Hot Standby Router Protocol |
| **STP** | Spanning Tree Protocol |
| **DMZ** | Demilitarized Zone |
| **ACL** | Access Control List |
| **SSH** | Secure Shell |
| **DHCP** | Dynamic Host Configuration Protocol |
| **FAI / ISP** | Fournisseur d'Accès Internet |

---

<p align="center">⭐ Si ce projet vous a été utile, n'hésitez pas à laisser une étoile sur le dépôt ! ⭐</p>
