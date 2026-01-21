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

**Active le démarrage auto avec systemd**

```zsh
sudo /opt/splunk/bin/splunk enable boot-start -systemd-managed 1 -user splunk
```

```zsh
sudo systemctl daemon-reload
```
