---
title: Installer SNMP sur VCSA 6.5
author: VaLouille
type: post
date: 2016-12-08T15:38:14+00:00
url: /2016/12/installer-snmp-sur-vcsa-6-5/
categories:
  - vmware
tags:
  - 6.5
  - appliance
  - linux
  - net-snmp
  - oid
  - photon
  - snmp
  - snpmd
  - vcenter
  - vcsa
  - vmware
  - vmware-snmp
  - vsphere

---
Pour installer net-snmp sur le version 6.5 de l&rsquo;appliance linux vCenter 6.5 basée sur photon OS, voici la procédure à suivre. Toutes ces commandes sont à exécuter en root sur le vCenter.

Si vous avez un proxy, il faut paramétrer la variable https_proxy.

On désactive le SNMP de VMware :

```
vicfg-snmp -D
```

On supprime le SNMP de VMware :

```
rpm -e vmware-snmp
```

On installe SNMP via tdnf (Tiny Dandified Yum) :

```
tdnf install net-snmp
```

On créer le répertoire contenant le fichier de configuration :

```
mkdir -p /usr/etc/snmp
```

On crée le fichier de configuration /usr/etc/snmp/snmpd.conf

```
syslocation myCompany
syscontact report@company.com
sysservices 72
rocommunity public
agentuser root
agentaddress 161
# Extensions
######################
# System
########
pass .1.3.6.1.4.1.2021.XXX.XXX /path/to/custom/script/check-custom.pl
```

On prend en compte le fichier systemd snmpd :

```
systemctl daemon-reload
```

On start snmpd :

```
systemctl start snmpd
```

On autorise le port 161 dans le firewall :

```
iptables -A port_filter -p udp -m udp --dport 161 -j ACCEPT
```

On crée le fichier /etc/vmware/appliance/firewall/snmpd avec le contenu suivant :

```
{
 "firewall": {
    "enable": true,
    "rules": [
      {
        "comment" : "snmp_port",
        "name" : "snmp.ext.port1",
        "direction": "inbound",
        "protocol": "udp",
        "porttype": "dst",
        "port": "161",
        "portoffset": 0
      }    ]
  }
}
```

On autorise la connexion sur le port 161 :

```
echo "snmpd: ALL : ALLOW" >> /etc/hosts.allow
```
