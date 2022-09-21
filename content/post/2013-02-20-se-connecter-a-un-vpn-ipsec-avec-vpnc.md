---
title: Se connecter à un VPN IPSEC avec vpnc
author: VaLouille
type: post
date: 2013-02-20T13:46:20+00:00
url: /2013/02/se-connecter-a-un-vpn-ipsec-avec-vpnc/
categories:
  - linux
  - sysadmin
  - vpn
tags:
  - cisco
  - ipsec
  - vpn
  - vpnc

---
Pour se connecter à un VPN IPSec, on peut utiliser l&rsquo;outil libre vpnc. Voici comment le configurer.
  
Installer vpnc et netcat:

```
apt-get install vpnc netcat
```

Ajouter les lignes suivantes dans le fichier /etc/vpnc/default.conf :

```
IPSec gateway <IP du VPN>
IPSec ID tgroup1
IPSec secret Start1
IKE DH Group dh2
Vendor cisco
Xauth username <Nom d'utilisateur>
Xauth password <Mot de passe>
Script /etc/vpnc/vpnc-script
```

Lancer vpnc :

```
vpnc /etc/vpnc/default.conf
```

Si ca ne fonctionne pas, ajouter aussi l&rsquo;option suivante a /etc/vpnc/default.conf :

```
NAT Traversal Mode cisco-udp
```

Si ça fonctionne, ajouter les lignes suivantes à ~/.ssh/config pour un montage du VPN automatique lorsque l&rsquo;on initie une connexion vers une machine dont le nom commence par « test »

```
Host test*
  ProxyCommand bash -c 'if ! pgrep vpnc; then sudo vpnc-connect 1>&2; fi; nc -w 60 %h %p'
```

Ajouter ensuite « ServerAliveInterval 60 » au fichier /etc/ssh/ssh_config afin que les connexions SSH ne « timeout » pas

Ajouter au fichier /etc/sudoers les droits suivants pour l&rsquo;utilisateur qui se connecte en SSH:

```
<user> ALL = NOPASSWD : /usr/sbin/vpnc-connect, /usr/sbin/vpnc-disconnect
```

Ajouter au fichier /etc/hosts les lignes suivantes, en indiquant les IP, afin de faire fonctionner le montage automatique du VPN :

```
<IP>   test-lamp.domaine.com test-lamp
<IP>   test-mysql.domaine.com test-mysql
```
