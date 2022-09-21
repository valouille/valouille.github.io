---
title: Inverser le sens de défilement de la molette sous Windows
author: VaLouille
type: post
date: 2014-08-14T10:16:35+00:00
url: /2014/08/inverser-le-sens-de-defilement-de-la-molette-sous-windows/
categories:
  - windows
tags:
  - défilement
  - direction
  - inverser
  - molette
  - natural
  - scroll
  - scrolling
  - sens
  - windows

---
Sous Mac OS X, par défaut, le sens de défilement est inversé. Cela s&rsquo;appelle le « natural scrolling » ou « défilement naturel ». Quand on passe d&rsquo;OS X à Windows régulièrement, on peut préférer avoir le même comportement sous Windows.

Afin d&rsquo;activer cette option, il faut lancer la commande Powershell ci-dessous (à lancer dans l&rsquo;application « Windows Powershell » en tant qu&rsquo;Administrateur)

```
Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Enum\HID\*\*\Device` Parameters FlipFlopWheel -EA 0 | ForEach-Object { Set-ItemProperty $_.PSPath FlipFlopWheel 1 }
```

Le redémarrage de l&rsquo;ordinateur est nécessaire afin d&rsquo;appliquer les modifications.

La deuxième solution est de le faire à la main. Il faut lancer l&rsquo;éditeur de registre (Touche Windows + R, et entrer « regedit »)

Il faut éditer la clé suivante :

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\HID\VID_***\VID_***\Device Parameters
```

Changer la valeur de FlipFlopWheel de 0 à 1 et redémarrer l&rsquo;ordinateur.

Afin de trouver le VID, il faut aller voir dans le « Panneau de configuration » (Afficher par : Grandes icones), et ouvrir l&rsquo;outil « Souris ». Ensuite, dans l&rsquo;onglet « Matériel », cliquer sur « Propriétés ». Dans l&rsquo;onglet « Détails », choisir la propriété « Chemin d&rsquo;accès à l&rsquo;instance du périphérique ».
