---
title: Scripts pour générer la liste des jours fériés en France et des week-ends
author: VaLouille
type: post
date: 2014-05-28T15:41:47+00:00
url: /2014/05/scripts-pour-generer-la-liste-des-jours-feries-en-france-et-des-week-ends/
categories:
  - linux
tags:
  - bash
  - date
  - easter
  - fériés
  - php
  - script
  - timestamp
  - week-end

---
Voici deux scripts que j&rsquo;ai créé qui permettent de générer la liste des jours fériés en France (hors Alsace/Moselle et DOM/TOM), ainsi que les jour qui tombent un samedi ou un dimanche.

Cela peut être très pratique pour éviter de lancer certains scripts ces jours là.

&#8211; La liste des jours fériés :

Certains jours tombent à des dates fixes, ce qui est pratique, mais 3 jours fériés ont une date variable. Pour calculer ces 3 jours (Lundi de Pâques, Lundi de Pentecôte, Jeudi de l&rsquo;Assomption), il faut connaître la date de Pâques.

Celle-ci correspond au dimanche après le 14e jour du premier mois lunaire du printemps, donc le dimanche après la première pleine lune pendant ou après l’équinoxe de printemps.

La fonction PHP [easter_date()][1] renvoie cette date.

Ensuite, le lundi de Pâques est 1 jour après, le lundi de Pentecôte 50 jours après, et le jeudi de l&rsquo;Assomption 39 jours après. Il suffit donc juste de calculer.

```bash
#!/bin/bash
# Valerian Beaudoin
# Genere
if [ -z $1 ] || [ "$1" == "--help" ] || [ "$1" == "-h" ] || [[ $1 != [1-2]* ]] || [ "$1" -lt "1970" ] || [ "$1" -gt "2037" ]  ; then
    echo "Description : Renvoie la liste des jours feries en France (hors Alsace/Moselle et DOM/TOM)"
    echo "Ce script ne fonctionne pas pour les dates hors des timestamps UNIX (avant 1970 ou apres 2037)"
    echo "Usage : $0 <Year>"
    echo "Exemple : $0 2015"
    exit 1
fi
# Premier Janvier
date --date="$1-01-01" +%Y%m%d
# Lundi Paques
date --date=@$(($(php -r "print(easter_date($1));") + 1*86400)) +%Y%m%d
# Fete du travail
date --date="$1-05-01" +%Y%m%d
# Victoire des allies
date --date="$1-05-08" +%Y%m%d
# Jeudi Ascension
date --date=@$(($(php -r "print(easter_date($1));") + 39*86400)) +%Y%m%d
# Lundi Pentecote
date --date=@$(($(php -r "print(easter_date($1));") + 50*86400)) +%Y%m%d
# Fete nationale
date --date="$1-07-14" +%Y%m%d
# Assomption
date --date="$1-08-15" +%Y%m%d
# Toussaint
date --date="$1-11-01" +%Y%m%d
# Armistice
date --date="$1-11-11" +%Y%m%d
# Noel
date --date="$1-12-25" +%Y%m%d
```

* La liste des jours non travaillés (samedis et dimanches) pour l&rsquo;année en cours :

Il s&rsquo;agit d&rsquo;une simple boucle qui tourne 360 fois, qui ajoute 1 jour à la date du jour, et qui regarde si le jour correspond à un samedi ou un dimanche.

```bash
#!/bin/bash
# Valerian Beaudoin
i=0
while [ "$i" -lt "360" ] ; do
    jour="`date +%Y%m%d:%u -d "+$i days"`"
    week_day=`echo $jour | cut -d: -f2`
    if [ "$week_day" -eq 6 ] || [ "$week_day" -eq 7 ]; then
        jour_ferme=`date +%Y%m%d -d "+$i days"`
        echo $jour_ferme
    fi
    i=$(($i+1))
done
```

[1]: http://www.php.net/manual/fr/function.easter-date.php
