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
