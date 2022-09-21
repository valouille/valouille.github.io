---
title: Mettre en place du bonding sous Linux avec un switch Cisco
author: VaLouille
type: post
date: 2013-02-13T17:46:11+00:00
url: /2013/02/mettre-en-place-du-bonding-sous-linux-avec-un-switch-cisco/
categories:
  - sysadmin
tags:
  - cisco
  - linux
  - réseau

---
Le bonding, ou agrégation de lien, permet d&rsquo;agréger deux liens afin de doubler le débit et d&rsquo;avoir de la redondance.
  
Sur la machine, il faut installer le paquet suivant :

```
apt-get install ifenslave-2.6
```

Puis il faut éditer le fichier /etc/network/interfaces afin de commenter eth0 et eth1 et de configurer une interface bond0 :

```
auto bond0
iface bond0 inet static
 slaves eth0 eth1
 bond-mode 802.3ad
 bond-miimon 100
 bond-downdelay 200
 bond-updelay 200
 bond-lacp-rate fast

 address IP.IP.IP.IP
 netmask IP.IP.IP.IP
 gateway IP.IP.IP.IP
```

Ensuite, sur le switch Cisco, on regarde quelles interfaces Port-channel ont déjà été configurées :

```
sh int status | inc Po
```

Si une ou plusieurs interfaces existent, il faudra la créer en utilisant un numéro de port inexistant. Sinon, on peut utliser le 1. Les interfaces Port-Channel se configurent comme les autres

```
interface Port-channel1
 description Machine:bond0 Gi0/17,Gi0/20
 switchport access vlan 3
 switchport mode access
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
 spanning-tree guard root
```

Puis pour les deux ports de switch, on configue le channel-group :

```
interface GigabitEthernet0/17
 channel-group 1 mode active
 channel-protocol lacp
```

```
interface GigabitEthernet0/18
 channel-group 1 mode active
 channel-protocol lacp
```

Puis on vérifie :

```
sh lacp internal
```

```
Channel group 1
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/17    SA      bndl      32768         0x1       0x1     0x11        0x3D  
Gi0/20    SA      bndl      32768         0x1       0x1     0x14        0x3D
```

Il ne reste plus qu&rsquo;à sauvegarder la configuration et redémarrer la machine.
