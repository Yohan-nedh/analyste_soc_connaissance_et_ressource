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
