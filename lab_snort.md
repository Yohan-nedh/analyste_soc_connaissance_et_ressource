# 🐳  On va construire un vrai mini-lab SOC avec Docker

L'objectif : **apprendre vite** avec Docker ET **simuler un vrai environnement SOC**. On va créer un lab avec deux containers qui communiquent.

---

## 🗺️ Architecture du lab

```
┌──────────────────────────────────────────────────────┐
│                  VM Ubuntu 22.04                     │
│                                                      │
│   ┌─────────────────┐      ┌─────────────────────┐  │
│   │  Container      │      │  Container          │  │
│   │  🐷 Snort 3     │◀─────│  💀 Kali Linux      │  │
│   │  (Défenseur)    │      │  (Attaquant)        │  │
│   │  172.20.0.2     │      │  172.20.0.3         │  │
│   └────────┬────────┘      └─────────────────────┘  │
│            │                                         │
│            ▼                                         │
│   ┌─────────────────┐                                │
│   │  📋 Logs/Alertes│                                │
│   │  /var/log/snort │                                │
│   └─────────────────┘                                │
│                                                      │
│   Réseau Docker : soc-network (172.20.0.0/24)       │
└──────────────────────────────────────────────────────┘
```

---

## ⚙️ ÉTAPE 1 — Créer la structure du projet

```bash
# Créer le dossier principal du lab
mkdir -p ~/soc-lab/snort/{rules,logs,config}
cd ~/soc-lab
```

---

## ⚙️ ÉTAPE 2 — Créer le Dockerfile Snort

```bash
nano ~/soc-lab/Dockerfile.snort
```

Colle ce contenu :

```dockerfile
FROM ubuntu:22.04

# Éviter les interactions pendant l'installation
ENV DEBIAN_FRONTEND=noninteractive

# Installer les dépendances
RUN apt-get update && apt-get install -y \
    snort \
    net-tools \
    iputils-ping \
    tcpdump \
    && rm -rf /var/lib/apt/lists/*

# Créer les dossiers nécessaires
RUN mkdir -p /var/log/snort \
             /etc/snort/rules

# Copier la config et les règles
COPY snort/config/snort.conf /etc/snort/snort.conf
COPY snort/rules/local.rules /etc/snort/rules/local.rules

# Lancer Snort
CMD ["snort", "-c", "/etc/snort/snort.conf", \
     "-i", "eth0", \
     "-A", "fast", \
     "-l", "/var/log/snort", \
     "-k", "none", "-v"]
```
# 📁 Comprendre la structure des dossiers du lab

Très bonne question ! Un analyste SOC doit **toujours savoir pourquoi** il crée ce qu'il crée. On ne tape jamais des commandes en aveugle.

---

## 🔍 Décomposons la commande

```bash
mkdir -p ~/soc-lab/snort/{rules,logs,config}
```

| Morceau | Signification |
|---|---|
| `mkdir` | **Make Directory** — crée un dossier |
| `-p` | Crée tous les **dossiers parents** si ils n'existent pas |
| `~/soc-lab` | Dossier racine de ton lab dans ton **home directory** |
| `snort/{rules,logs,config}` | Crée **3 sous-dossiers** en une seule commande |

C'est équivalent à écrire :
```bash
mkdir ~/soc-lab
mkdir ~/soc-lab/snort
mkdir ~/soc-lab/snort/rules
mkdir ~/soc-lab/snort/logs
mkdir ~/soc-lab/snort/config
```
> 💡 Le `{}` est une **expansion bash** — ça évite de répéter le chemin 3 fois.

---

## 🗂️ Rôle de chaque dossier

```
~/soc-lab/                  ← Racine du projet
│
├── docker-compose.yml      ← Orchestre tous les containers
├── Dockerfile.snort        ← Instructions pour builder Snort
│
└── snort/
    │
    ├── 📁 rules/           ← Tes règles de détection
    │       └── local.rules     "Si tu vois X → génère une alerte"
    │
    ├── 📁 logs/            ← Les alertes générées par Snort
    │       └── alert.log        "Voici ce que j'ai détecté"
    │
    └── 📁 config/          ← La configuration de Snort
            └── snort.conf       "Voici comment je dois fonctionner"
```

---

## 📂 Rôle détaillé de chaque dossier

### 📁 `rules/` — Le cerveau de la détection
```
C'est ici que tu écris TES règles Snort.
→ Chaque règle dit à Snort QUOI surveiller
→ C'est le fichier que tu modifieras le plus souvent
→ Exemple : détecter un scan Nmap, un bruteforce SSH...
```

### 📁 `logs/` — La mémoire de Snort
```
C'est ici que Snort écrit toutes ses alertes.
→ Chaque fois qu'une règle matche → une ligne s'écrit ici
→ C'est CE fichier que tu liras en tant qu'analyste SOC
→ C'est aussi ce fichier qui sera envoyé à ton SIEM plus tard
```

### 📁 `config/` — Le comportement de Snort
```
C'est ici que tu configures Snort globalement.
→ Quel réseau surveiller ?
→ Où écrire les logs ?
→ Quelles règles charger ?
→ Comment formater les alertes ?
```

---

## 🧠 L'analogie pour retenir

> Imagine Snort comme un **agent de sécurité** dans un bâtiment :
>
> 📁 `config/` → **Son contrat de travail** — ses horaires, son périmètre, ses instructions générales
>
> 📁 `rules/` → **Son manuel de menaces** — la liste de tout ce qu'il doit considérer comme suspect
>
> 📁 `logs/` → **Son carnet de rapport** — tout ce qu'il a observé et signalé pendant son service

---

## 💡 Pourquoi séparer en dossiers distincts ?

C'est une **bonne pratique professionnelle** :

```
✅ Clarté        → chaque dossier a UN seul rôle
✅ Maintenance   → tu modifies les règles sans toucher à la config
✅ Sauvegarde    → tu peux sauvegarder rules/ séparément
✅ Docker        → on monte chaque dossier comme volume séparé
✅ Travail équipe → un analyste gère rules/, un admin gère config/
```

---

## ⚙️ ÉTAPE 3 — Créer la configuration Snort

```bash
nano ~/soc-lab/snort/config/snort.conf
```

Colle ce contenu :

```bash
# =============================================
# SNORT.CONF — Configuration Lab SOC
# =============================================

# Définition du réseau à surveiller
var HOME_NET 172.20.0.0/24
var EXTERNAL_NET !$HOME_NET

# Chemins importants
var RULE_PATH /etc/snort/rules
var LOG_DIR /var/log/snort

# Inclure nos règles personnalisées
include $RULE_PATH/local.rules

# Output — format des alertes
output alert_fast: /var/log/snort/alert.log
output log_tcpdump: /var/log/snort/snort.log
```

---

## ⚙️ ÉTAPE 4 — Créer les règles de détection

```bash
nano ~/soc-lab/snort/rules/local.rules
```

Colle ces règles :

```bash
# =============================================
# RÈGLES SOC LAB — local.rules
# =============================================

# --- RÈGLE 1 : Détection de ping (ICMP) ---
alert icmp any any -> $HOME_NET any (
  msg:"[LAB] Ping detecte";
  itype:8;
  classtype:attempted-recon;
  priority:3;
  sid:1000001;
  rev:1;
)

# --- RÈGLE 2 : Scan de port SYN ---
alert tcp any any -> $HOME_NET any (
  msg:"[LAB] Scan de port detecte";
  flags:S;
  threshold: type threshold, track by_src, count 15, seconds 3;
  classtype:attempted-recon;
  priority:2;
  sid:1000002;
  rev:1;
)

# --- RÈGLE 3 : Bruteforce SSH ---
alert tcp any any -> $HOME_NET 22 (
  msg:"[LAB] Bruteforce SSH detecte";
  flags:S;
  threshold: type both, track by_src, count 5, seconds 60;
  classtype:attempted-admin;
  priority:1;
  sid:1000003;
  rev:1;
)

# --- RÈGLE 4 : Tentative SQL Injection ---
alert tcp any any -> $HOME_NET 80 (
  msg:"[LAB] Tentative SQL Injection";
  content:"SELECT"; nocase;
  content:"FROM"; nocase;
  classtype:web-application-attack;
  priority:1;
  sid:1000004;
  rev:1;
)

# --- RÈGLE 5 : Scan Nmap détecté ---
alert tcp any any -> $HOME_NET any (
  msg:"[LAB] Scan Nmap detecte";
  flags:SF;
  classtype:attempted-recon;
  priority:2;
  sid:1000005;
  rev:1;
)
```

---

## ⚙️ ÉTAPE 5 — Créer le Docker Compose

```bash
nano ~/soc-lab/docker-compose.yml
```

Colle ce contenu :

```yaml
version: '3.8'

networks:
  soc-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/24

services:

  # ─── Container Snort (Défenseur) ───
  snort:
    build:
      context: .
      dockerfile: Dockerfile.snort
    container_name: snort-ids
    networks:
      soc-network:
        ipv4_address: 172.20.0.2
    volumes:
      - ./snort/logs:/var/log/snort
      - ./snort/rules:/etc/snort/rules
    cap_add:
      - NET_ADMIN
      - NET_RAW
    restart: unless-stopped

  # ─── Container Kali (Attaquant) ───
  kali:
    image: kalilinux/kali-rolling
    container_name: kali-attacker
    networks:
      soc-network:
        ipv4_address: 172.20.0.3
    command: sleep infinity
    cap_add:
      - NET_ADMIN
      - NET_RAW
    stdin_open: true
    tty: true
```

---

## ⚙️ ÉTAPE 6 — Lancer le lab

### 6.1 — Construire et démarrer
```bash
cd ~/soc-lab
docker compose up -d --build
```

### 6.2 — Vérifier que les containers tournent
```bash
docker ps
```
Tu dois voir :
```
CONTAINER ID   IMAGE     NAME             STATUS
xxxxxxxxxxxx   snort     snort-ids        Up
xxxxxxxxxxxx   kali      kali-attacker    Up
```

### 6.3 — Surveiller les alertes Snort en temps réel
```bash
# Dans un terminal dédié — ne le ferme pas !
tail -f ~/soc-lab/snort/logs/alert.log
```

---

## ⚙️ ÉTAPE 7 — Simuler des attaques depuis Kali !

Ouvre un **nouveau terminal** et entre dans le container Kali :

```bash
docker exec -it kali-attacker bash
```

### 🔴 Attaque 1 — Ping (ICMP)
```bash
ping 172.20.0.2
```
→ Alerte attendue : `[LAB] Ping detecte`

### 🔴 Attaque 2 — Installer et lancer Nmap
```bash
apt update && apt install -y nmap
nmap -sS 172.20.0.2
```
→ Alerte attendue : `[LAB] Scan de port detecte`

### 🔴 Attaque 3 — Bruteforce SSH
```bash
apt install -y hydra
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.20.0.2
```
→ Alerte attendue : `[LAB] Bruteforce SSH detecte`

---

## 📋 Commandes utiles pour le lab

```bash
# Voir les logs Snort en direct
tail -f ~/soc-lab/snort/logs/alert.log

# Entrer dans le container Snort
docker exec -it snort-ids bash

# Entrer dans le container Kali
docker exec -it kali-attacker bash

# Arrêter le lab
docker compose down

# Redémarrer le lab
docker compose restart

# Voir les logs du container Snort
docker logs snort-ids -f
```

---

## 🗺️ Récapitulatif

```
ÉTAPE 1 → Structure du projet   ✅
ÉTAPE 2 → Dockerfile Snort      ✅
ÉTAPE 3 → snort.conf            ✅
ÉTAPE 4 → local.rules           ✅
ÉTAPE 5 → docker-compose.yml    ✅
ÉTAPE 6 → Lancer le lab         🚀
ÉTAPE 7 → Simuler des attaques  💀
          → Voir les alertes    🎯
```

---

Lance l'**Étape 6** et dis-moi ce que tu obtiens ! Si tu as une erreur, copie-la ici — on debug ensemble comme en vrai SOC 🔍🚀
