---
title: Nettoyer les logs IPMI sous linux en ligne de commande
author: VaLouille
type: post
date: 2013-12-16T12:53:45+00:00
url: /2013/12/nettoyer-les-logs-ipmi-sous-linux-en-ligne-de-commande/
categories:
  - linux
  - sysadmin
tags:
  - dell
  - drac
  - idrac
  - ipmi
  - ipmitool
  - linux
  - osma

---
Les logs systèmes sur les serveurs Dell peuvent ne plus s&rsquo;écrire au bout d&rsquo;un certain temps. En effet, l&rsquo;espace est limité.

Les erreurs en résultant seront de la forme :

```
Event Logging Disabled #0x72 | Log full | Asserted
```

ou

```
ESM log is 100% full
```

On peut utiliser la commande ipmitool suivante pour les effacer, et ainsi permettre à nouveau l&rsquo;écriture des logs :

```
ipmitool sel clear
```
