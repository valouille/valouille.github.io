---
title: 'NetApp : commandes utiles'
author: VaLouille
type: post
date: 2014-09-05T12:20:06+00:00
url: /2014/09/netapp-commandes-utiles/
categories:
  - netapp
tags:
  - netapp

---
Vérifier que l&rsquo;ACP est en place :

```
> storage show acp

Alternate Control Path:          Enabled
Ethernet Interface:              e0P
ACP Status:                      Active
ACP IP Address:                  192.168.1.125
ACP Subnet:                      192.168.1.1
ACP Netmask:                     255.255.252.0
ACP Connectivity Status:         Full Connectivity
```

Vérifier la santé du NetApp :

```
> system health status show
 
Status
---------------
ok
```

Voir les alertes du NetApp :

```
> system health alert show
 
               Node: nodename
           Resource: Shelf ID 1
           Severity: Minor
     Probable Cause: BlaBla
    Possible Effect: BlaBla
Corrective Actions: BlaBla
```

Dire à un contrôleur d&rsquo;arrêter le takeover :

```
cf giveback
```

Passer en mode console à partir du SP :

```
system console
```

Empêcher le takeover sur un contrôleur :

```
cf disable
```
