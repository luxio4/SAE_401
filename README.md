# SAE4.Cyber.01 – Sécurisation d’un système d’information

Projet réalisé dans le cadre de la SAE4.Cyber.01 du BUT Réseaux & Télécommunications – Semestre 4.  
Objectif : concevoir, configurer et sécuriser une maquette réseau d’entreprise répartie sur deux sites distants, intégrant un tunnel sécurisé et des services critiques durcis.

---

## 🎯 Objectifs

- Concevoir une architecture réseau complète sur deux sites :
  - **Siège**
  - **Succursale**
- Mettre en place :
  - Un **tunnel IPsec GRE** entre les deux sites
  - Trois **VLAN par site** (`Service`, `Production`, `Admin`)
  - Une **DMZ** par site
- Sécuriser les services principaux :
  - **DNS interne** (DNSSEC + DNS over TLS)
  - **Serveur Web HTTPS** (Nginx + PHP + MariaDB)
- Appliquer les bonnes pratiques de sécurité :
  - Pare-feux ASA, ACL, NAT
  - Conformité aux recommandations de l’**ANSSI**
- Documenter l’ensemble du projet dans un **rapport technique (write-up)**.

---

## 🧩 Architecture réseau

Chaque site comprend :

- Un **routeur principal** connecté à un routeur “Internet”
- Un **pare-feu ASA** (interface `inside`, `dmz`, `outside`)
- Un **switch L3** pour l’inter-VLAN
- Trois VLAN :
  - VLAN 10 : Service
  - VLAN 20 : Production
  - VLAN 30 : Admin
- Une **DMZ** pour les serveurs publics (web, DNS…)

Les deux sites sont reliés par un **tunnel GRE encapsulé dans IPsec**, permettant le routage OSPF entre eux et la communication sécurisée des sous-réseaux internes.

---

## 🔐 Sécurisation des services

### DNS interne (Windows Server 2019)

- Zone interne : `societeDDLS.pepiniere.rt`
- Configuration **DNSSEC** :
  - Signature de zone (KSK/ZSK)
  - Validation des réponses signées
- Activation de **DNS over TLS (DoT)** via *stunnel*
- Sécurisation :
  - Blocage des requêtes `ANY`
  - Limitation aux IP internes
  - Journalisation et supervision

### Serveur Web (Nginx + PHP + MariaDB)

- Serveur HTTPS avec **certificat SSL** auto-signé
- Application web sécurisée :
  - Authentification PHP avec hash (`password_hash`)
  - Protection CSRF et cookies sécurisés
  - Connexion MariaDB via SSL
- Configuration Nginx conforme ANSSI :
  - Forçage HTTPS
  - HSTS, CSP, XSS protection
  - Filtrage des méthodes HTTP
- Pare-feu `ufw` limitant les ports ouverts aux seuls nécessaires

---

## 🧱 Technologies et outils utilisés

- **Cisco Packet Tracer** : conception du réseau et configuration Cisco  
- **Windows Server 2019** : DNS interne, DNSSEC et DoT  
- **Debian / Ubuntu Server** : serveur web et base de données  
- **Kali Linux** : tests d’intrusion et validation de la sécurité  
- **Nmap**, **Metasploit**, **Bettercap** : audit et vérification des protections  

---

## ⚙️ Contenu du dépôt

```text
.
├── README.md
├── packettracer/
│   ├── schema_final.pkt
│   └── schema_final_sans_mdp.pkt
├── writeup/
│   └── Write-Up - SAE4.Cyber.01.pdf
└── docs/
    └── schema_reseau.png
