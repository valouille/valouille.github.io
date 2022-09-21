---
title: Supprimer un ancien aggregat (foreign aggregate) sur un shelf NetApp en CDOT 8.3
author: VaLouille
type: post
date: 2015-06-12T08:56:04+00:00
url: /2015/06/supprimer-un-ancien-aggregat-foreign-aggregate-sur-un-shelf-netapp-en-cdot-8-3/
categories:
  - netapp
tags:
  - 8.3
  - aggr
  - aggregate
  - c-mode
  - cdot
  - netapp

---
Afin de supprimer un foreign aggregate et transformer les disques en non-zeroed spares, la procédure est la suivante :

Ici, je souhaite supprimer l&rsquo;aggregat &lsquo;ag0&rsquo;, qui était encore présent sur un shelf que je viens d&rsquo;ajouter :

```
::> disk show
[...]
1.11.0              827.7GB    11   0 BSAS    aggregate   ag0       pa1-netapp-01b
1.11.1              827.7GB    11   1 BSAS    aggregate   ag0       pa1-netapp-01a
1.11.2              827.7GB    11   2 BSAS    aggregate   ag0       pa1-netapp-01b
1.11.3              827.7GB    11   3 BSAS    aggregate   ag0       pa1-netapp-01a
[...]
```

Pour voir si l&rsquo;aggregat &lsquo;ag0&rsquo; est foreign, on tape la commande suivante à la recherche de la ligne :

```
::> system node run -node * -command sysconfig -r
[...]
Aggregate ag0 (raid_dp, foreign) (block checksums)
[...]
```

Pour le supprimer, la commande est la suivante :

```
::> set -privilege advanced
::*> storage aggregate remove-stale-record -nodename [node_name] -aggregate [foreign aggregate name]
```

On vérifie que la suppression est effective :

```
::> disk show
[...]
1.11.0              827.7GB    11   0 BSAS    spare       Pool0     pa1-netapp-01b
1.11.1              827.7GB    11   1 BSAS    spare       Pool0     pa1-netapp-01a
1.11.2              827.7GB    11   2 BSAS    spare       Pool0     pa1-netapp-01b
1.11.3              827.7GB    11   3 BSAS    spare       Pool0     pa1-netapp-01a
[...]
```
