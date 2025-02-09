+++
title = "Hugo: Afficher les dates en français"
date = "2025-02-09"
categories = ["adminsys"]
tags = ["hugo","date"]
+++

Hugo est un générateur de sites statiques rapide et moderne écrit en Go, conçu pour rendre la création de sites web à nouveau amusante.
<!--more-->
Cependant à chaque fois je rencontre le même problème sur le système de date. Hugo s’appuie sur des librairies de format de date du langage go, 100% américain, 
du coup je me retrouve avec les dates à l’envers en mode YYYY-DD-MM à la place du logique YYYY-MM-DD et les mois en langue Anglaise. 
Pour site entièrement francophone cela pose un problème de lecture.

Pour y remidier j'ai mis en place une petite routine :

* Je créé deux petits fichier en `YAML` appellé : `mois.yml` et `moishort.yml`. Ces deux fichiers font le pont entre le numéro de mois et l’affichage en français. 
Comme leur nom l'indique l'un affiche les mois en entier et l'autre les noms tronqués, en fonction des besoins et du theme utilisé.
* J'utilise ensuite le numéro de mois comme un index et stocke la valeur correspondante dans une variable locale : `{{ $mymonths := index $.Site.Data.mois }}`
* Pour finir avec `printf` j'affiche cette valeur : `{{ index $mymonths (printf "%d" .Date.Month) }} {{ .Date.Year }}`

Ainsi, mes dates seront bien formattées en français, il me suiffit de mettre à jour toutes les parties de mon template qui affichent des dates ainsi que dans les partials. 

## Le Code

mois.yml :
```yaml
1: "Janvier"
2: "Février"
3: "Mars"
4: "Avril"
5: "Mai"
6: "Juin"
7: "Juillet"
8: "Août"
9: "Septembre"
10: "Octobre"
11: "Novembre"
12: "Décembre"
```

moisshort.yml :
```yaml
1: "Jan"
2: "Fév"
3: "Mar"
4: "Avr"
5: "Mai"
6: "Jui"
7: "Juil"
8: "Aoû"
9: "Sept"
10: "Oct"
11: "Nov"
12: "Déc"
```
Le code en go pour traduire les dates :
``` go
{{ $mymonths := index $.Site.Data.mois }}{{ .Date.Day }}
{{ index $mymonths (printf "%d" .Date.Month) }} {{ .Date.Year }}
```