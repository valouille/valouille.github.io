---
title: Utiliser ovftool sous linux avec un domaine
author: VaLouille
type: post
date: 2018-01-29T13:07:06+00:00
url: /2018/01/utiliser-ovftool-sous-linux-avec-un-domaine/
categories:
  - linux
  - sysadmin
  - vmware

---
J&rsquo;ai suivi la [documentation officielle][1] pour exporter une VM en OVA avec ovftool, mais j&rsquo;ai été confronté aux erreurs suivantes :

Sans quotes :

```
# ovftool --noSSLVerify vi://vsphere.local\administrator@vcenter.fqdn/DC/vm/web1 web1_export.ova
Enter login information for source vi://vcenter.fqdn/
Username: vsphere.localadministrator
Password:
Terminate process signal received - aborting operation
Removing temporary objects and files
*Error: Execution aborted
Completed with errors
```

Avec quotes :

```
# ovftool --noSSLVerify "vi://vsphere.local\administrator@vcenter.fqdn/DC/vm/web1" web1_export.ova
Error: Internal error: Failed to connect to server
Completed with errors
```

La solution que j&rsquo;ai trouvé consiste à remplacer le backslash par le code hexadécimal correspondant : **%5c**

```
# ovftool --noSSLVerify "vi://vsphere.local%5cadministrator@vcenter.fqdn/DC/vm/web1" web1_export.ova
Enter login information for source vi://vcenter.fqdn/
Username: vsphere.local%5cadministrator
Password: ************
Opening VI source: vi://vsphere.local%5cadministrator@vcenter.fqdn:443/PA3/vm/web1
Opening OVA target: web1_export.ova
Writing OVA package: web1_export.ova
Disk progress: 29%
```

 [1]: https://kb.vmware.com/s/article/1038709
