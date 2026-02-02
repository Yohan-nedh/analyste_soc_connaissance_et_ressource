Je tiens à dire que cette section de note est un peu perso et ne vous étonnez pas qu'il y aie des tatonnement mais cela
sera corriger au fur et à mesure. Mais je suis content car **splunk** est le premier SIEM que j'ai eu à utiliser, mais bon 
passons à la définition

## Splunk c'est quoi ?

Splunk est une plateforme puissante qui collecte, analyse et visualise des données machine en temps réel pour la sécurité, l'observabilité et l'analyse métier, transformant des logs et événements non structurés en informations exploitables via des tableaux de bord et des alertes, ce qui permet aux entreprises de détecter rapidement les menaces, résoudre les problèmes IT et prendre des décisions éclairées, souvent surnommé le "Google des données machine" pour son moteur de recherche puissant. 

Et ici voici la documentation que j'ai p consulter sur splunk (c'est bien de pratiquer mais connaître les fondamentaux c'est bien aussi): https://docs.splunk.com/Documentation/Splunk

NB: pour démarrer le server **splunk** j'ai utiliser une suite commande que je vous expliquerez mais bon comme ce sont des notes personnelles je fais ce que je veux et tac 😏😏 bref revenons à l'essentiel 

**Démarre Splunk manuellement**

```zsh
sudo /opt/splunk/bin/splunk start
```

Mais comme je l'ai dit précedemment ce sont des notes perso et j'ai pas envie de me faire chier à taper cette commande et vous non plus donc nous allons créer des alias vu que c'est un lab nous allons proceder commme cela mais en entreprise ne faite pas cela merci.

1. La plus simple et rapide: Créer un alias dans ton terminal
C'est idéal pour l'entraînement, tu tapes juste une courte commande au lieu de la longue.

Ouvre ton fichier de configuration du shell (comme tu es sur bash ou zsh) :
```zsh
nano ~/.bashrc ou ~/.zshrc # si tu utilises zsh
```

Ajoute cette ligne à la fin du fichier :
```zsh
alias splunkstart='sudo /opt/splunk/bin/splunk start'
alias splunkstop='sudo /opt/splunk/bin/splunk stop'
alias splunkstatus='sudo /opt/splunk/bin/splunk status'
alias splunkrestart='sudo /opt/splunk/bin/splunk restart'
```
Sauvegarde (Ctrl+O → Enter → Ctrl+X) puis recharge le fichier :
```zsh
source ~/.bashrc ou ~/.zshrc 
```
Et en cas de problème(surtout le shell **bash**) tel que celui-ci(er oui j'ai fait la capture d'écran dit moi merci) et pour résoudre ce problème veuillez taper **bash** comme encodrer ci-dessous:

<img width="946" height="289" alt="image" src="https://github.com/user-attachments/assets/84876902-1f75-4e40-a6ec-7bd70674db8c" />

À partir de maintenant, tu tapes simplement :

splunkstart  pour démarrer

splunkstatus pour vérifier

splunkstop  pour arrêter

splunkrestart pour redémarrer


## Installation du l' Universal Forwarder (sur linux)

On peut se poser la question légitime de savoir ce que c'est que l'universal forwarder?

L'Universal forwarders acheminent les données de votre machine vers un récepteur de données. Ce récepteur est généralement un index de la plateforme Splunk
où vos données sont stockées. Vous pouvez utiliser l'Universal forwarders pour surveiller vos données en temps réel.

Utilisez l'Universal forwarders pour vous assurer que vos données sont correctement formatées avant de les envoyer à Splunk. Vous pouvez également
manipuler vos données avant qu'elles n'atteignent les index ou les ajouter manuellement.


### En pratique comment l'installer sur une machine linux
**1-Connectez-vous sur la plateforme de splunk** et si vous n'avez pas encore créer de compte faite le. (Lien menant vers les universal forwarder https://www.splunk.com/en_us/download/universal-forwarder.html)

**2- Sélectionnez l'Universal forwarder selon votre os** nous ici nous somme sur linux

<img width="1906" height="900" alt="image" src="https://github.com/user-attachments/assets/32301c26-ad63-4b24-9102-1346ead55753" />

Vous avez le choix en fonction des caractéristique de votre système moi je suis 64 bits mais en cas de doute exécuter cette commande **uname -m** comme je suis sur un vm et que je l'untilise en ligne ce commande j'ai choisi l'option **copy wget link** et j'ai copier la commande et exécuter

<img width="1914" height="810" alt="image" src="https://github.com/user-attachments/assets/7585cab3-cc27-4d5b-acff-73d240384648" />

<img width="936" height="228" alt="image" src="https://github.com/user-attachments/assets/316d70fe-d159-4888-9f04-881bb771d165" />

OK après le téléchargement nous allons crée le dossier **splunkforwarder** dans le dossier **/opt/** qui est réservé aux logiciels additionels voici la commande

```zsh
sudo mkdir -p /opt/splunkforwarder
```

Si en téléchargeant le fichier de L'universal forwarder(UF pour plus de simplicité) se trouve dans un autre dossier c'est pas grave éxcuter cette commande

```zsh
sudo tar xvzf splunkforwarder-10.2.0-d749cb17ea65-linux-amd64.tgz -C /opt/splunkforwarder/
```

OK, après cette étape vérifions si il y a un utilisateuR. De façon automatique splunk crée un utilisateur nommé **splunkfwd** commande pour vérifier:
```zsh
id splunkfwd
```

dans le cas contraire utilisons ces commande pour crée un utilisateur:

```zsh
sudo useradd -m -s /bin/false splunkfwd
sudo groupadd splunkfwd
sudo chown -R splunkfwd:splunkfwd /opt/splunkforwarder
```
