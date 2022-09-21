---
title: Migrer une VM Xen vers Proxmox KVM
author: VaLouille
type: post
date: 2013-02-08T17:19:28+00:00
url: /2013/02/migrer-une-vm-xen-vers-proxmox-kvm/
categories:
  - sysadmin

---
Voici la procédure pour miger une VM XEN vers un hyperviseur Proxmox KVM :
  
Sur le serveur ProxMox, il faut récupérer un ID libre :

```
echo $((`ls /etc/pve/nodes/*/qemu-server | sed s/.conf//g | grep -v /etc/pve/nodes|sort -n | tail -1` +1))
```

  * <ID> correspond au numéro renvoyé par cette ligne de commande
  * <VM> correspond au nom de la VM

On crée le répertoire et le fichier de configuration correspondant

```
mkdir /VMs/images/<ID>
touch /etc/pve/qemu-server/<ID>.conf
```

Sur le serveur Xen

```
lvcreate -s -n snap-<VM> -L 20g /dev/system/<VM>-disk
qemu-img convert -O qcow2 /dev/system/snap-<VM> /var/backup/<ID>.qcow2
scp /var/backup/<VM>.qcow2 serveur_proxmox:/VMs/images/<VM>/<ID>.qcow2
```

Sur le serveur ProxMox, on monte l&rsquo;image

```
modprobe nbd max_part=8
qemu-nbd -c /dev/nbd0 /VMs/images/<ID>/<VM>.qcow2
mkdir /mnt/temp
mount /dev/nbd0 /mnt/temp
mount -o bind /dev /mnt/temp/dev
mount -o bind /dev/pts /mnt/temp/dev/pts
mount -o bind /proc /mnt/temp/proc
```

On chroot dedans et on installe les paquets necessaires

```
chroot /mnt/temp
apt-get install grub2
# installer dans /dev/nbd0 quand il demande où l'installer
apt-get install -t squeeze-backports linux-image-3.2.0-.bpo.4-amd64
apt-get remove linux-image-2.6.32-5-xen-amd64
```

On adapte la configuration grub

```
vim /boot/grub/grub.cfg
```

Remplir comme ceci :

```
set root='(/dev/sda)'
linux /boot/vmlinuz-3.2.0-.bpo.4-amd64 root=/dev/sda ro console=tty0 cons
```

On adapte le fstab

```
vim /etc/fstab
```

Mettre juste ces deux lignes :

```
proc /proc proc defaults  
/dev/sda / ext3 errors=remount-ro  1
```

Ajouter les lignes suivantes dans /etc/inittab :

```
1:2345:respawn:/sbin/getty 38400 tty1
2:23:respawn:/sbin/getty 38400 tty2
3:23:respawn:/sbin/getty 38400 tty3
4:23:respawn:/sbin/getty 38400 tty4
5:23:respawn:/sbin/getty 38400 tty5
6:23:respawn:/sbin/getty 38400 ttyS0
```

On adapte la configuration réseau

```
vim /etc/network/interfaces
```

Mettre une autre IP ou 1.1.1.1 afin de ne pas avoir l&rsquo;IP montée deux fois

On sort du chroot et on démonte l&rsquo;image

```
exit
cd /
umount /mnt/temp/dev/pts
umount /mnt/temp/dev/
umount /mnt/temp/proc
umount /mnt/temp
qemu-nbd -d /dev/nbd0
```

Faire fichier de conf

```
vim /etc/pve/qemu-server/<ID>.conf
```

Remplir comme ceci :

```
args: -serial unix:/var/run/qemu-server/<ID>.serial,server,nowait -balloon virtio
boot: cdn
bootdisk: ide0
cores: 1
ide0: <NOM_storage>:<ID>/<VM>.qcow2
keyboard: fr
memory: 4096
balloon: 2048
name: <VM>
onboot: 1
ostype: l26
sockets: 1
```

On lance la VM :

```
qm start <ID> && minicom -D unix#/var/run/qemu-server/<ID>.serial
```

Pour ajouter une swap, il faut créer un disque :

```
cd /VMs/images/<VMID>/
qemu-img create -f qcow2 vm-<VMID>-disk-2.qcow2 2G
```

Sur la VM, ajouter la ligne suivante au fichier /etc/fstab

```
/dev/vda none swap sw 0 0
```

Charger les modules pour l&rsquo;ajout à chaud de disque

```
modprobe acpiphp pci_hotplug
```

Sur l&rsquo;hyperviseur

```
qm monitor <VMID>
pci_add auto storage file=/VMs/images/<VMID>/vm-<VMID>-disk-2.qcow2,if=virtio
```

Puis lancer les commandes suivantes pour activer la swap:

```
mkswap -f /dev/vda
swapon -a
```

Ajouter la ligne suivante dans le fichier de configuration <VMID>.conf

```  
virtio0: <Storage>/vm-<VMID>-disk-2.qcow2
```
