---
link: https://moodle.org/mod/forum/discuss.php?d=466712
excerpt: Bonjour Séverin,
tags:
  - moodle
  - configuration
slurped: 2025-03-12T08:27
title: "Moodle en français: Fixer les réglages dans le fichier de configuration | Moodle.org"
---
Dans Moodle, il est possible de fixer certains réglages dans le [fichier de configuration](https://docs.moodle.org/4x/fr/Fichier_de_configuration) (config.php), et [spécifier leurs valeurs](https://docs.moodle.org/4x/fr/Fichier_de_configuration#Forcer_la_valeur_de_certains_r%C3%A9glages_d'administration), comme indiqué dans le [fichier config-dist.php](https://github.com/moodle/moodle/blob/MOODLE_403_STABLE/config-dist.php#L852-L870) (en 4.3 par exemple).

Cela permet d'éviter tout risque de modification inadaptée via l'interface graphique, étant donné que les valeurs spécifiées ainsi ne sont plus modifiables depuis l'interface graphique.

Je précise que les paramètres peuvent être forcés de la manière suivante, pour les paramètres généraux :  
```
`$CFG->nom_paramètre = "valeur";`  
```


Et pour les paramètres des plugins, soit un par un, sous la forme suivante :  
```
$CFG->forced_plugin_settings['plugin']['nom_paramètre'] = 'valeur';  

```
  
Soit de façon groupée, par exemple :  
```
$CFG->forced_plugin_settings = array(  
'moodlecourse' => array('format' => 'topics', 'maxsections' => 52),  
'url' => array('display' => 0, 'displayoptions' => '0,1,3,4,5,6'),  
[resource](https://moodle.org/mod/glossary/showentry.php?eid=3921&displayformat=dictionary "Terminologie francophone de Moodle : Resource") => array('displayoptions' => '0,1,4,5,6'), 
'backup' => array('backup_auto_storage' => 1, 'backup_auto_destination' => /chemin/vers/[sauvegardes](https://moodle.org/mod/glossary/showentry.php?eid=4067&displayformat=dictionary "Terminologie francophone de Moodle : Sauvegardes")),  
 logstore_standard' => array('loglifetime' => 730)  
);  
```
  
Sachant que pour les plugins, il est possible de cumuler les deux techniques, en autant d'occurrences que nécessaire.
Et les [requêtes](https://moodle.org/mod/glossary/showentry.php?eid=351&displayformat=dictionary "Terminologie francophone de Moodle : Requêtes") pour lister tout cela (sur le principe qu'indiquait Olivier).  
  
Pour les réglages généraux :  
Et les [requêtes](https://moodle.org/mod/glossary/showentry.php?eid=351&displayformat=dictionary "Terminologie francophone de Moodle : Requêtes") pour lister tout cela (sur le principe qu'indiquait Olivier).  
  
Pour les réglages généraux :  
`SELECT CONCAT('$CFG->', name, ' = \'', value, '\';') FROM mdl_config;`  
ou :

`SELECT CONCAT("$CFG->", name, " = '", value, "';") FROM mdl_config;`

Et on pourrait ajouter un ordre de tri, avec (juste avant le ; final) :

`ORDER BY name`

  
