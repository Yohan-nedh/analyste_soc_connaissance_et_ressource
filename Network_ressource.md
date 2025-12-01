# Networking

## Tryhackme
### Networking Concepts

<img width="968" height="367" alt="image" src="https://github.com/user-attachments/assets/fe08ba88-8453-41d3-9ad9-2bb641302749" />

dont le lien est: https://tryhackme.com/room/networkingconcepts, https://tryhackme.com/room/osimodelzi

Il est très important de connaître les différents concepts liés aux réseaux informatiques tels que le modèle OSI, le modèle TCP/IP, les protocoles réseau, etc.
Dans cette room, il est principalement décrit le modèle OSI, le modèle TCP/IP ainsi que les protocoles IP, UDP et TCP.

Mais comme ici je tiens à clarifier quelques concepts vu que c'est aussi un dépôt de note:

## Modèle OSI

Le modèle **OSI (Open Systems Interconnection)** est un modèle conçu par l’Organisation internationale de normalisation **(ISO)** pour décrire le fonctionnement des communications au sein d’un réseau informatique.
Bien qu’il ne soit qu’un modèle de référence, il permet de mieux comprendre et structurer la communication dans un réseau informatique. Il se compose de **sept couches** qui sont:

### 1. Couche Physique

Cette couche permet de gérer la connexion physique entre les appareils. À ce niveau, les données sont converties en séquences binaires (bits), qui sont ensuite transmises sur un support physique sous forme de signaux électriques, optiques ou sans fil (ondes radio), à l’aide de câbles ou d’antennes.

### 2. Couche de Liaison de données

Elle permet l’établissement d’une liaison fiable entre deux appareils directement connectés sur un même réseau physique afin de faciliter le transfert des données. Elle prend les paquets provenant de la couche réseau et les divise en fragments plus petits appelés trames. Cette couche détecte et, dans certains cas, corrige les erreurs de transmission, gère le contrôle d’accès au support (pour éviter les collisions) et identifie les appareils grâce à leurs adresses MAC.

### 3. Couche Réseau

Cette couche gère le transfert de données entre deux réseaux différents, en assurant la connectivité inter-réseaux. Elle assure l’adressage logique (par exemple, via des adresses IP v4 ou v6) et le routage des paquets, en déterminant le meilleur chemin pour acheminer les informations d’un réseau à un autre.

### 4. Couche Transport

Cette couche assure le transport de bout en bout des données entre les applications des hôtes source et destination. Elle segmente les données provenant des couches supérieures en unités plus petites (comme des segments pour TCP ou des datagrammes pour UDP), gère le contrôle de flux pour éviter les surcharges, détecte et corrige les erreurs, et permet le multiplexage via des numéros de ports pour distinguer les différentes communications.

### 5. Couche Session

Il s'agit de la couche responsable de l'ouverture et de la fermeture de la communication entre les deux appareils. L'intervalle entre l'ouverture et la fermeture de la communication est appelé **session**. La couche de session garantit que la session reste ouverte(ou plus simplement maintient la session ouverte) suffisamment longtemps pour transférer toutes les données échangées, puis ferme rapidement la session afin d'éviter le gaspillage de ressources.

### 6. Couche de Présentation

La couche de présentation garantit que les données sont fournies dans un format compréhensible par la couche application. La couche 6 gère l'encodage, la compression et le chiffrement des données. L'encodage des caractères, comme l'ASCII ou l'Unicode, en est un exemple.

### 7. Couche Application

C'est la seule couche qui interagit directement avec les données de l'utilisateur. Les applications logicielles comme les navigateurs web et les clients e-mail se servent de la couche applicative pour initier des communications. 

**Image du modèle OSI**

<img width="1018" height="764" alt="image" src="https://github.com/user-attachments/assets/cb272f34-8b76-4498-ad62-be536b8ac639" />

#### Tableau récapitulatif

<img width="1243" height="519" alt="image" src="https://github.com/user-attachments/assets/7c2ccd96-9934-4663-8923-76ce17dc19d0" />


**Site pour se docuementer sur le modèle OSI:** https://www.cloudflare.com/fr-fr/learning/ddos/glossary/open-systems-interconnection-model-osi/, https://www.frameip.com/osi/#24-8211-la-couche-reseau


## Modèle TCP/IP

Ce modèle rassemble certaines couches du modèle OSI, comme il est montré dans l’image ci-dessous:

<img width="1018" height="778" alt="image" src="https://github.com/user-attachments/assets/f4eced28-e932-485d-be67-bc286f6c2c6d" />

Le modèle TCP/IP se compose de 4 couches principales, conçues pour simplifier la communication réseau par rapport au modèle OSI (qui en a 7):

### 1. Couche accès réseau

Elle réunit les couches 1 (physique) et 2 (liaison de données) du modèle OSI. Cette couche gère la transmission physique des données sur le réseau local, incluant les connexions via câbles (comme Ethernet), antennes Wi-Fi ou autres supports sans fil. Elle organise les bits en trames, détecte les erreurs de transmission, gère le contrôle d’accès au medium (pour éviter les collisions, par exemple avec CSMA/CD), et utilise des adresses MAC pour identifier les appareils connectés directement

### 2. Couche Internet

Équivalente à la couche 3 (réseau) du modèle OSI. Elle assure le routage des données entre différents réseaux, en utilisant des protocoles comme IP (Internet Protocol). Ici, les données sont encapsulées en paquets avec des adresses IP logiques (IPv4 ou IPv6), et des algorithmes de routage déterminent le meilleur chemin pour traverser les routeurs et interconnecter les réseaux mondiaux, comme sur Internet. Elle gère aussi la fragmentation des paquets et des outils de diagnostic comme ICMP (pour les pings).

### 3. Couchz transport

Correspond à la couche 4 du modèle OSI. Elle fournit un transport de bout en bout, fiable ou non, entre les applications des hôtes. Des protocoles comme TCP (fiable, avec contrôle de flux, correction d’erreurs et réassemblage) ou UDP (rapide, sans garantie) segmentent les données, ajoutent des ports pour multiplexage (ex. : port 80 pour HTTP), et assurent que les données arrivent intactes d’une machine à l’autre.

### 4. Couche application

Elle regroupe les couches 5 (session), 6 (présentation) et 7 (application) du modèle OSI. C’est ici que les protocoles d’application opèrent directement, comme HTTP/HTTPS pour le web, SMTP pour les emails, FTP pour les transferts de fichiers, ou DNS pour la résolution de noms. Elle gère l’interface utilisateur-réseau, la conversion des données (encodage, compression), et les sessions de connexion.


## Encapsulation et désencapsulation



liens: https://waytolearnx.com/2019/06/encapsulation-et-decapsulation-tcp-ip.html


## Protocoles

### Telnet 

Le protocole **Telnet** est un protocole de **couche application utilisé** pour se connecter à un terminal virtuel d'un autre ordinateur. Grâce à Telnet, un utilisateur peut se connecter à un autre ordinateur et accéder à son terminal (console) pour exécuter des programmes, lancer des processus par lots et effectuer des tâches d'administration système à distance. Et il utilise le **port 23**

Le protocol en lui même nous permet d'intérargir avec plusieur autre protocole en indiquant le port pour exemple nous avons les proctocoles suivant: **HTTP, FTP, POP3, SMTP et IMAP**

<img width="833" height="336" alt="image" src="https://github.com/user-attachments/assets/be9f082c-ae2d-41be-bac9-34efecd1e75f" />

je vous renvoie vers cette resource pour plus de clarté: https://tryhackme.com/room/protocolsandservers
 
### HTTP
Le protocole de transfert hypertexte ( HTTP ) est utilisé pour transférer les pages web. Votre navigateur se connecte au serveur web et utilise HTTP pour demander des pages HTML, des images et d'autres fichiers, ainsi que pour soumettre des formulaires et télécharger divers fichiers. Lorsque vous naviguez sur le World Wide Web (WWW), vous utilisez le protocole HTTP. Et les ports utilisés pour **HTTP** sont : **80 et 8080**  et pour **HTTPS** nous avons **443 et 8443**


