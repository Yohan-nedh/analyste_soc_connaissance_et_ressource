## Qu'est ce que Nagios ?
Nagios est une plateforme open-source de surveillance d'infrastructure informatique (systèmes, réseaux, applications) qui détecte les problèmes en temps réel, envoie des alertes, 
et fournit une visibilité complète pour réduire les temps d'arrêt et améliorer la productivité, avec une architecture modulaire basée sur un moteur de planification, 
une interface web et des plugins (sondes) pour une personnalisation étendue.


Et pour l'installation veuillez sur la documention et j'oubliais ici nous allons parler de **Nagios core**. Bon j'avais déjà fait l'installation de nagios donc ce sera plus orienté configuration et 
aide mémoire.


Voici le lien de la documentation de nagios core: https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/4/en/toc.html


Pour lancer le server nagios

```zsh
└─$ sudo systemctl start nagios
```

Bon nous allons ajouter un vm linux à notre server nagios et pour cela nous allons proceder étapes par étapes:

**1. Installation de NRPE**

**NRPE** est un module complémentaire qui vous permet d'exécuter des plugins sur des hôtes Linux/Unix
distants. (pour en savoir plus sur NRPE https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/4/en/addons.html#nrpe)

Alors commençons l'installation de celui-ci voici la commande a tapé sur la machine à surveiller:

```zsh
~$ sudo apt install nagios-nrpe-server
```

et nous procedons à l'installation des plugins

```zsh
~$ sudo apt install nagios-plugins nagios-plugins-contrib -y
```

puis déplaçons nous dans le fichier nrpe.cfg en tapant cette commande :

```zsh
~$ sudo nano /etc/nagios/nrpe.cfg
```
De là nous insérons l'adresse ip de notre server après ::1, 🩹

<img width="756" height="107" alt="image" src="https://github.com/user-attachments/assets/337d515d-e99c-41c4-854c-70d445914012" />

  Après cela nous nous dirigerons du côté de la machine ou il y a le servur nagios.

  **2. Ajout de la machine à surveiller**

  Pour plus de faciliter nous nommerons la machine à surveiller **nedh-vm**, brel let' go.
  Tout d'abord allons dans le dossier /objets Les objets désignent tous les éléments impliqués dans la logique de surveillance et de notification. 
  Nous allons crée le fichier de configuration avec cette commande
   
```zsh
sudo nano /usr/local/nagios/etc/objects/nedh-vm.cfg
```
Nous allons commencer par définr le host qui ce bloc **define host { ... }** sert à identifier et déclarer la machine distante (ou locale) dans le serveur nagios.

```zsh                                 
define host {
        use     linux-server
        host_name   nedh-first-vm
        alias   nedh-vm #on verra ce nom dans l'interface web
        address        #INSERER L'ADRESSE IP DE LA MACHINE QUE VOUS VOULEZ SURVEILLER 
}
```
Puis vérifions si la configuration est correct (dans mon cas j'ai mis l'adresse IP alors c'est bon) avec cette commande:

```zsh
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

<img width="689" height="625" alt="image" src="https://github.com/user-attachments/assets/44bc76dc-59a9-4ed0-9f9a-d793136e08de" />

 Ce qui est encadrer en rouge montre qu'il n'y pas d'erreur lors de la configuration. Maintenant insérons le chemin de notre fichier de configuration dans le fichiers de configuration princpal **nagios.cfg**. Nous insérons alors le fichier sous ce format **cfg_file=/usr/local/nagios/etc/objects/nedh-vm.cfg** dans le **nagios.cfg**

 ```zsh
cd  /usr/local/nagios/etc/
puis
sudo nano nagios
```

 <img width="700" height="700" alt="image" src="https://github.com/user-attachments/assets/677f353f-9d20-412b-bd21-ffb472681a0b" />

 Puis lançons ou relançons le server nagios avec cette commande:
 
 ```zsh
sudo service nagios restart
```

 puis connectez vous à votre interface web nagios et vérifier si la machine est là:

 <img width="1920" height="336" alt="image" src="https://github.com/user-attachments/assets/cc040fd6-c47e-4d2e-9590-04578667fe4e" />

Et voilà mais toute fois le travaille n'est pas fini nous allons maintenant configurer les services 

 
