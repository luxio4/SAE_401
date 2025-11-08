# SAE4.Cyber.01 – Sécuriser un système d’information

Projet réalisé dans le cadre de la SAE4.Cyber.01 : **maquette de réseau d’entreprise sécurisée** répartie sur deux sites distants, avec tunnel sécurisé et services critiques durcis (DNS, Web).

## 🎯 Objectifs du projet

- Concevoir et configurer une **infrastructure réseau d’entreprise** sur deux sites :
  - Siège
  - Succursale
- Séparer les usages via :
  - 3 VLAN par site : `Service` / `Production` / `Admin`
  - 1 DMZ par site
- Mettre en place un **tunnel IPsec GRE sécurisé** entre les deux sites.
- Sécuriser :
  - le **DNS interne** (DNSSEC + DNS over TLS)
  - un **serveur web HTTPS** (Nginx + PHP + MariaDB)
  - les accès via **pare-feux, ACL, NAT, ANSSI**, etc.
- Tester la sécurité et documenter les configurations dans un **Write-Up détaillé**.

---

## 🏗️ Architecture globale

- Deux sites distants reliés par un réseau « Internet » (VLAN 800 dans la consigne).
- Par site :
  - Un **routeur principal**
  - Un **firewall ASA** vers Internet / tunnel
  - Un **switch L3** pour l’inter-VLAN
  - Une **DMZ** (serveur web / mail, etc.)
  - Trois réseaux LAN :
    - `Service`
    - `Production`
    - `Admin` (a accès à tous les autres réseaux, y compris le site distant)
- Entre les deux sites :
  - **Tunnel GRE encapsulé dans IPsec**
  - Routage dynamique via OSPF
  - NAT + ACL pour contrôler et filtrer les flux

---

## 🔐 Sécurisation mise en place

### 1. DNS interne sécurisé (Windows Server 2019)

- Serveur DNS avec zone interne `societeDDLS.pepiniere.rt`
- Activation de **DNSSEC** :
  - Génération de la KSK et ZSK
  - Signature de la zone
  - Activation des réponses sécurisées
- Mise en place de **DNS over TLS (DoT)** via *stunnel* :
  - Ecoute sur le port 853
  - Tunnel TLS vers le port 53 local
- Protection avancée :
  - Limitation des requêtes DNS (anti-amplification / anti-DDoS)
  - Blocage des requêtes `ANY`
  - Restriction aux IP internes
  - Journalisation et supervision des logs

➡️ Tous les détails sont dans la section **« Configuration DNSSEC et DoT – Windows Serveur 2019 »** du write-up.

---

### 2. Serveur Web sécurisé (Nginx + PHP + MariaDB)

- OS : Ubuntu / Debian
- Installation de :
  - `nginx`
  - `php-fpm`
  - `mariadb-server`
- Génération d’un **certificat SSL auto-signé**.
- Configuration Nginx :
  - Forçage **HTTPS** (redirection HTTP → HTTPS)
  - Protocoles et suites de chiffrement conformes aux recommandations **ANSSI**
  - En-têtes de sécurité :
    - HSTS
    - X-Frame-Options
    - X-Content-Type-Options
    - X-XSS-Protection
    - Content-Security-Policy
  - Désactivation de la compression sur les pages sensibles
- Application web :
  - Formulaire de **connexion sécurisée (PHP)** avec :
    - Hash de mot de passe (`password_hash`)
    - Protection CSRF
    - Session sécurisée (`cookie_secure`, `httponly`)
  - Connexion à MariaDB via **SSL** (PDO + SSL_CA).
- Base de données :
  - Utilisateur applicatif à privilèges limités
  - Compte admin DB séparé pour la maintenance
- Pare-feu :
  - `ufw` configuré pour n’autoriser que le nécessaire
  - Limitation basique anti-DDoS via `limit_req` dans Nginx

---

### 3. Maquette réseau Cisco Packet Tracer

Le projet inclut une maquette complète sous **Cisco Packet Tracer** :

- **Segmentation réseau** par VLAN sur les switches L3 :
  - VLAN 10 : Service
  - VLAN 20 : Production
  - VLAN 30 : Admin
- **Inter-VLAN routing** via interfaces `int vlan X` sur les L3.
- **ACL** pour contrôler les flux entre VLAN :
  - Admin peut accéder à tous les VLAN.
  - Service / Production **ne peuvent pas** accéder aux VLAN Admin ni à certains réseaux distants.
- **Firewall ASA** :
  - Inspection du trafic (DNS, HTTP, ICMP…)
  - Interfaces `inside` / `dmz` / `outside`
  - NAT dynamique pour sorties vers l’extérieur
- **Routeurs de site + routeur « Internet »** :
  - Routage OSPF
  - NAT
  - Mise en place du **tunnel IPsec GRE** :
    - IKE Phase 1 & 2 (AES, SHA, PSK, DH group 2)
    - `crypto map`, `transform-set`, ACL intéressée GRE
    - Interface `Tunnel0` / routes statiques vers les réseaux distants

Les commandes complètes pour chaque équipement sont listées dans le **Write-Up**.

---

### 4. Tests de sécurité

- Découverte réseau :
  - `netdiscover`
  - `nmap -sS -A`
- Exploitation de failles :
  - Scan SMBv1 et vulnérabilité **MS17-010 (EternalBlue)**
  - Exploitation via Metasploit (`msfconsole`)
- Tests DNS :
  - Vérification DNSSEC et DoT (`dig +dnssec +tls`, `Resolve-DnsName`)
  - Tentatives d’attaques (ANY, amplification, spoofing avec Bettercap)
- Validation des protections pare-feu, NAT, ACL et tunnel.

---

## 📁 Contenu du dépôt

Proposition d’arborescence :

```text
.
├── README.md                       # Ce fichier
├── writeup/
│   └── Write-Up-SAE4-Cyber-01.pdf  # Rapport complet (config + explications)
├── packettracer/
│   ├── schema_final.pkt
│   └── schema_final_sans_mdp.pkt   # Version sans mots de passe
└── docs/
    └── schema_reseau.png           # Schéma de l’énoncé (optionnel)
