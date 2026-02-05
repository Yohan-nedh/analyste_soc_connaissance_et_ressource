# Note sur snort
https://chatgpt.com/share/698473cb-a08c-800a-a5c3-fe1cd935edb3

SNORT est un puissant système de détection des  intrusions (IDS)  et un système de prévention des  intrusions (IPS) open  sourcequi fournit une analyse du trafic réseau et un journalisation des paquet de données en temps réel. Il analyse le trafic IP en temps réel, compare les paquets à des règles prédéfinies, et détecte les activités malveillantes comme les scans de ports, les exploits, et les attaques DoS. 

Caractéristiques clés de Snort :

**Fonctionnement :** Analyse en temps réel, enregistrement de paquets (sniffer) et détection/prévention d'intrusions (IDS/IPS).

**Détection :** Utilise des signatures, des protocoles et des anomalies pour identifier les menaces.

**Polyvalence :** Compatible avec Linux et Windows, il agit en mode passif (alerte) ou actif (blocage).

**Règles :** Basé sur un langage de règles personnalisables permettant aux administrateurs de définir ce qui est suspect. 

## Installation de snort 

Vous pouvez suivre les étapes énocez ici: https://youtu.be/CkH4_WIPr4I?si=ZDmI6ldNjtcj7uCU

**1. installation de snort avec cette commande: **

```zsh
sudo apt install snort
```
puis vous installez les dépendances:

```zsh
sudo apt install -y \
  build-essential \
  libpcap-dev \
  libpcre3-dev \
  libnet1-dev \
  zlib1g-dev \
  luajit \
  hwloc \
  libdumbnet-dev \
  bison \
  flex \
  liblzma-dev \
  openssl \
  libssl-dev \
  pkg-config \
  libhwloc-dev \
  cmake \
  cpputest \
  libsqlite3-dev \
  uuid-dev \
  libcmocka-dev \
  libnetfilter-queue-dev \
  libmnl-dev \
  autotools-dev \
  libluajit-5.1-dev \
  libunwind-dev \
  libfl-dev
```

puis on crée un répertoire qui vas contenir les fichiers de snort:

```zsh
mkdir snort-file
```

Vous pouvez suivre les étapes énocez ici pour l'installation de la bibliothèque LibDAQ: https://docs.snort.org/start/installation

LibDAQ est une bibliothèque d'abstraction de données (Data Acquisition library) modulaire, principalement développée pour le système de détection d'intrusion Snort 3. Elle permet de charger et de configurer des modules (base ou wrapper) pour capturer et traiter le trafic réseau à partir de diverses interfaces matérielles ou logicielles. D'où j'ai commencé par cloner le dépot git:

```zsh
git clone https://github.com/snort3/libdaq.git
```

dont le résultat est ceci:

<img width="654" height="131" alt="image" src="https://github.com/user-attachments/assets/55b883fc-2162-4fd5-bd74-70edf6099d90" />

Ensuite, exécutez les commandes suivantes pour générer le script de configuration, configurer avec un préfixe spécifié, compiler et enfin installer :

```zsh
 cd libdaq
 ./bootstrap
 ./configure --prefix=/usr/local/lib/daq_s3
 make install
```

puis on installe **tcmalloc** pour optimiser la mémoire de snort :

```zsh
git clone https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbEtzR1FkWWxlUElfbFdtTHNnMXk0eGFRMy1VQXxBQ3Jtc0ttSlVFb1d2TXZWQXp6d1VPLV9XMnAwSnhDTWJzeVltSGF2RU1TOWVwajlzWXBnQUNDVFF3MXkzZ0RwN241VnBUSVFUcmFYR09zNUZNS24yclExU0xKbzk2bjJYb3FidlZWOUpRZW14MFRGTUdMVUdmNA&q=https%3A%2F%2Fgithub.com%2Fgperftools%2Fgperftools%2Freleases%2Fdownload%2Fgperftools-2.9.1%2Fgperftools-2.9.1.tar.gz&v=CkH4_WIPr4I
```

Et voilà ce que nous avons:

<img width="438" height="148" alt="image" src="https://github.com/user-attachments/assets/c3a82989-6f14-483d-8b07-da6f4dce9551" />

Et nous executons ces commandes:

```zsh
cd gperftools-2.9.1/
./configure
sudo make
sudo make install
```

puis nous clonons le dépôt git de snort :

```zsh
git clone https://github.com/snort3/snort3.git
```

Après clonage on fait:

```zsh
cd snort3
./configure_cmake.sh --prefix=/usr/local --with-pcre2-includes=/usr/include --with-pcre2-libraries=/usr/lib/x86_64-linux-gnu
```

puis on se déplace dans le fichier build

```zsh
cd build
sudo make
sudo make install
```
Et pour l'installation c'est terminé.

## Surveillance du réseau

**Mode promiscous**

Le mode promiscuous permet à la machine sur laquelle tourne Snort (ou un autre outil de capture) de voir et d'analyser tout le trafic du réseau (y compris celui qui ne lui est pas destiné), ce qui rend possible un scanning / monitoring complet du segment réseau.

Donc vous devez activer le mode promiscuous sur l'interface qui reçoit le trafic que vous voulez surveiller avec Snort, dans mon cas c'est **wlan0** pour activer ce mode nous utiliserons cette commande:

```zsh
sudo ip link set wlan0 promisc on (pour désactiver mettez off)
```
**Receive offload**

Receive offload (ou plus précisément Generic Receive Offload – GRO et Large Receive Offload – LRO) est une optimisation réseau qui permet à ta carte réseau (NIC) ou au pilote Linux de fusionner plusieurs petits paquets entrants (venant du même flux TCP/UDP) en un seul gros paquet avant de les remonter au noyau et aux applications.

**Pourquoi c'est important pour Snort (et pourquoi on le désactive souvent) ?**

Quand tu configures Snort en mode IDS/IPS (surtout avec promiscuous mode activé), GRO/LRO casse les choses :

Snort reçoit un gros paquet "reconstitué" au lieu des paquets originaux → il ne voit pas les fragments, les anomalies de taille, les attaques fragmentées, les malwares en petits bouts, etc.
Les règles Snort (surtout celles sur la fragmentation, les tailles bizarres, les checksums, etc.) deviennent inefficaces ou faussent les alertes.
Wireshark/tcpdump (si tu captures en parallèle) voit aussi des paquets "artificiels" pas conformes au fil réseau réel.

Solution classique pour Snort (recommandée dans tous les tutos) :


```zsh
sudo ethtool -K wlan0 gro off   # Désactive GRO
sudo ethtool -K wlan0 lro off   # Si LRO est présent (souvent off [fixed] déjà)
# Puis vérifie :
ethtool -k wlan0 | grep receive-offload
```

Et voilà le résutats

<img width="438" height="201" alt="image" src="https://github.com/user-attachments/assets/0cf29d56-71c8-4465-9294-206d78f2247d" />


Si vous éteignez ou redémarrer votre machine cela ramenera le gro à on pour contrer cela on peut éditer un service (moi dans mon cas je ne l'ai pas fait car c'est ma machine physique que j'utilise donc j'ai pas en que le gro soit toujours **on**) voici ce qu'il faut faire:

```zsh
sudo nano /etc/systemd/system/snort-nic.service
```

```zsh
[Unit]
Description=mode promiscous
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/ip link set dev eth0 promisc on
ExecStart=/usr/sbin/ethtool -K ens33(à remplacer avec votre interface par wlan0) gro off lro off
TimeoutStartSec=0
RemainAfterExit=yes

[Install]
WantedBy=default.target
```

## Installation des régles

Nous installerons bien évidemment installer les règles il y a trois types mais nous utiliserons la **Community** bien sur gratuite.

Bon créeons un repertoire ou mettre les règles, utilisons cette commande:

```zsh
sudo mkdir /usr/local/etc/snort-rules
```
je tiens précise que vous êtes dans votre dossier personnel **~** mais que pour le téléchargement je l'ai d'abord fais dans le dossier Téléchargement bah parce que j'y étais mais bon les commandes suivantes seront plus clairs merci

```zsh
wget https://www.snort.org/downloads/community/snort3-community-rules.tar.gz

sudo tar -xzf snort3-community-rules.tar.gz -C /usr/local/etc/snort-rules/

```
Et pour voir les règles qu'il y a utilison cette commande

```zsh
nano /usr/local/etc/snort-rules/snort3-community-rules/snort3-community.rules
```

et les voicis:

<img width="1920" height="1029" alt="image" src="https://github.com/user-attachments/assets/410162de-4717-4412-913c-fb91eabd1e5d" />

**NB: ON PEUT AUSSI CRÉER CES PROPRES RÈGLES **

**snort.lua** est le fichier de configuration principal de Snort 3 il permet de spécifier le réseau à surveiller et pour cela modifier la variable **HOME_NET = PAR VOTRE RÉSEAU** 

```zsh
sudo nano /usr/local/etc/snort/snort.lua
```

<img width="963" height="567" alt="image" src="https://github.com/user-attachments/assets/14dad4c3-a6ff-4bdf-bef5-eac1ee7a43b5" />

Puis allons sur **snort_defaults.lua** pour vérifier sile chemin vers le fichier des règles est correct

```zsh
sudo nano /usr/local/etc/snort/snort_defaults.lua
```

Remplacer par le chemin ou vos règles se trouvent 

<img width="1188" height="991" alt="image" src="https://github.com/user-attachments/assets/1516cd7f-8972-43eb-bed4-faa43d599fff" />

dans notre cas voici notre chemin

<img width="1029" height="933" alt="image" src="https://github.com/user-attachments/assets/cde01767-b0bd-482b-abaa-97d84502ae73" />

crée un utilisateur système dédié (appelé snort) pour faire tourner le service Snort de manière sécurisée.

```zsh
sudo useradd -r -s /usr/sbin/nologin -M -c snort_svc snort
```

puis on crée ceci

```zsh
sudo nano /etc/systemd/system/snort_svc.service
```
```zsh
[Unit]
Description=Snort NIDS/IPS Service
After=network.target syslog.target

[Service]
Type=simple
ExecStart=/usr/local/bin/snort -c /etc/snort/snort.lua -i wlan0 -u snort -g snort --daq-dir /usr/local/lib/daq --create-pidfile -D -l /var/log/snort
ExecStop=/bin/kill -TERM $MAINPID
Restart=on-failure
User=snort
Group=snort
PIDFile=/var/run/snort/snort.pid   # optionnel, si tu utilises --create-pidfile

[Install]
WantedBy=multi-user.target
```

mais vous pouvez personnaliser ensuite :
```zsh
sudo systemctl daemon-reload          # systemd relit les nouveaux fichiers
sudo systemctl enable snort_svc       # démarrage auto au boot
sudo systemctl start snort_svc        # lance maintenant
sudo systemctl status snort_svc       # vérifie (regarde les logs avec journalctl -u snort_svc -e si besoin)
```

Créeons le répertoire pour les logs pour que le compte snort puisse ecrire ça là-bas:
```zsh
sudo mkdir /var/log/snort

sudo chmod -R 5775 /var/log/snort

sudo chown -R snort:snort /var/log/snort

```

Démarrage du server 
```zsh
sudo systemctl enable --now snort_svc
```

Activation du logging

```zsh
sudo nano /usr/local/etc/snort/snort.lua 
```

et nous allons à cette section -- 7. configure outputs et nous décommentons

<img width="938" height="933" alt="image" src="https://github.com/user-attachments/assets/d59dcdad-aabe-41fa-abd6-706704c3299e" />

Et on faut ce petit ajout

<img width="938" height="933" alt="image" src="https://github.com/user-attachments/assets/4accacb6-9512-4547-93a6-c0b0c6fad94d" />

Commande de vérifications

```zsh
snort -c /usr/local/etc/snort/snort.lua 
```

message de succès

<img width="938" height="71" alt="image" src="https://github.com/user-attachments/assets/7e0e11d8-8479-442d-aa8b-b300230c4886" />


## confiuguration de mail
Super, YOHAN ! On passe directement à **l'étape 4** : configurer Snort pour envoyer des alertes en temps réel par e-mail (ou SMS si tu veux pousser plus loin) quand une activité suspecte est détectée.

Tu as raison : le paquet s'appelle **swatchdog** (et non swatch) sur les distributions modernes comme Kali Linux. C'est le même outil, mais il a été renommé "swatchdog" (Simple WATCH DOG) pour éviter les confusions. C'est un script Perl qui surveille les fichiers de logs en temps réel, cherche des patterns (regex), et déclenche des actions comme envoyer un e-mail.

**Pourquoi swatchdog ?**  
Snort génère des alertes dans un fichier (ex. `/var/log/snort/alert.fast` ou via syslog), mais il n'envoie pas d'email tout seul. Swatchdog "regarde" ce fichier continuellement (comme un tail -f intelligent) et réagit immédiatement quand une ligne match un pattern (ex. : une alerte Snort). C'est léger, simple, et parfait pour un TP sur Kali.

### Étape 4 détaillée : Alertes par e-mail avec swatchdog (grok https://grok.com/share/c2hhcmQtMw_3ff41f0b-8270-409b-9fba-78296299f82e)

#### 1. Installer swatchdog sur Kali
Kali a le paquet dans ses dépôts (souvent nommé **swatch** mais installe swatchdog en réalité).

```bash
sudo apt update
sudo apt install swatch -y   # Ça installe swatchdog + ses dépendances Perl
```

Vérifie l'installation :
```bash
which swatchdog   # Devrait retourner /usr/bin/swatchdog
swatchdog --version   # Ou --help pour confirmer
```

Si pas trouvé, installe depuis source (rare sur Kali 202x) :
```bash
sudo apt install git perl libfile-tail-perl libtimedate-perl -y
git clone https://github.com/ToddAtkins/swatchdog.git
cd swatchdog
perl Makefile.PL
make
sudo make install
```

#### 2. Configurer Snort pour envoyer les alertes dans un format facile à parser
Dans `/etc/snort/snort.lua` (ou ton fichier config principal), assure-toi d'avoir une sortie d'alertes claire :

```lua
alert_fast = { file = true, limit = 0 }   -- Écrit dans /var/log/snort/alert.fast (format simple : timestamp [**] message [**] etc.)
-- OU pour syslog (plus standard) :
alert_syslog = { facility = 'local5', level = 'info' }   # Les alertes vont dans /var/log/syslog ou /var/log/messages
```

- **Pourquoi alert_fast ?** Format lisible et rapide pour swatchdog (pas de JSON compliqué).  
- Si tu utilises syslog, swatchdog peut surveiller `/var/log/syslog` à la place.

Relance Snort pour appliquer (en mode test d'abord) :
```bash
sudo snort -c /etc/snort/snort.lua -T   # Vérifie la config
```

#### 3. Créer le fichier de configuration de swatchdog
Le fichier par défaut est `~/.swatchdogrc` (ou `/etc/swatchdogrc` si tu veux global).

Crée-le pour ton utilisateur :
```bash
mkdir ~/swatch
nano ~/.swatchdogrc
```

Contenu exemple adapté à Snort (très détaillé) :

```perl
# Surveille le fichier d'alertes Snort
watchfor /Snort/   # Ou /ALERT/ ou le mot que tu veux (ex: ton SID custom comme 1000001)
    echo bold yellow   # Affiche en console en jaune (optionnel, pour debug)
    mail addresses=tonadresse@email.com, subject=ALERTE SNORT - Activité suspecte détectée
    throttle 00:05:00   # Envoie max 1 email toutes les 5 minutes (évite flood si 100 alertes d'un coup)

# Exemple plus précis pour tes règles custom
watchfor /sid:1000001/   # Ton scan Nmap
    mail addresses=tonadresse@email.com, subject=SCAN RESEAU DETECTE (Nmap?)

watchfor /sid:1000003/   # Ta brute force SSH
    mail addresses=tonadresse@email.com, subject=BRUTE FORCE SSH EN COURS !

# Option : exécuter un script (ex: bloquer IP)
# watchfor /src_ip:(\d+\.\d+\.\d+\.\d+)/
#     exec /home/yohan/block_ip.sh $1
```

- **Explications des lignes** :
  - `watchfor /pattern/` : Regex Perl pour matcher les lignes du log. Plus le pattern est précis, moins de faux positifs.
  - `mail addresses=...` : Envoie un email avec le sujet et le corps = la ligne entière du log.
  - `throttle HH:MM:SS` : Limite le spam d'emails (très important !).
  - Tu peux ajouter `exec "commande $line"` pour des actions plus avancées.

Pour ton email : remplace `tonadresse@email.com` par ton vrai mail (ex. Gmail, mais attention aux filtres spam).

#### 4. Configurer l'envoi d'email sur Kali (très important !)
Kali n'a pas de serveur mail par défaut. Utilise **ssmtp** ou **msmtp** (léger) pour envoyer via Gmail ou autre.

Option simple : msmtp (recommandé)

```bash
sudo apt install msmtp msmtp-mta ca-certificates -y
```

Crée `/etc/msmtprc` :
```bash
sudo nano /etc/msmtprc
```

Contenu (pour Gmail) :
```
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
account        gmail
host           smtp.gmail.com
port           587
from           toncomptegmail@gmail.com
user           toncomptegmail@gmail.com
password       tonmotdepasse_app   # Utilise un "mot de passe d'application" Google (pas ton mot de passe normal !)
account default : gmail
```

Crée un mot de passe d'application Google : https://myaccount.google.com/apppasswords (active "Accès aux applications moins sécurisées" ou utilise App Password).

Teste l'envoi :
```bash
echo "Test Snort" | mail -s "Test" tonadresse@email.com
```
Ou directement :
```bash
echo "Test" | msmtp tonadresse@email.com
```

Si ça marche → parfait.

#### 5. Lancer swatchdog en mode daemon (surveillance continue)
```bash
swatchdog --config-file=~/.swatchdogrc --tail-file=/var/log/snort/alert.fast --daemon
# OU si tu utilises syslog :
# swatchdog --config-file=~/.swatchdogrc --tail-file=/var/log/syslog --daemon
```

- `--daemon` : tourne en arrière-plan.
- Pour le lancer au boot (optionnel pour TP) : ajoute dans crontab ou systemd service.

Vérifie qu'il tourne :
```bash
ps aux | grep swatchdog
```

#### 6. Tester l'ensemble
- Génère une alerte Snort (ex. : lance ton Nmap depuis une autre machine : `nmap -sS ton_ip_kali`).
- Regarde `/var/log/snort/alert.fast` → tu dois voir l'alerte.
- Dans 5-10 secondes, swatchdog devrait envoyer l'email.

Si pas d'email → vérifie :
- `tail -f /var/log/syslog` (erreurs msmtp ?)
- Logs swatchdog (il log parfois dans ~/.swatchdog.log ou console si pas daemon)

#### Bonus : Pour SMS (si tu veux aller plus loin)
Utilise un script exec dans swatchdog qui appelle une API SMS (ex. Twilio, Free Mobile, etc.). Exemple :
- Installe `curl`
- Dans .swatchdogrc :
  ```
  watchfor /sid:1000003/
      exec curl -X POST https://api.twilio.com/... --data-urlencode "Body=ALERTE BRUTE FORCE $line"
  ```
Mais pour le TP, l'email suffit amplement.

Dis-moi si l'installation de swatchdog/msmtp passe bien, ou si tu as une erreur précise. Une fois que les emails arrivent, on pourra passer à l'étape 5 (tests d'attaques) ! 😊
## Test
