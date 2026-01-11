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

puis déplaçons nous dans le fichier nrpe.cfg

