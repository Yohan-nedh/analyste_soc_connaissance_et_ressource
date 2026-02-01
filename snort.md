# Note sur snort


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
