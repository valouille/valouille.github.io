---
title: Compiler un paquet Debian DKMS pour un module noyau
author: VaLouille
type: post
date: 2013-05-21T12:55:23+00:00
url: /2013/05/compiler-un-paquet-debian-dkms-pour-un-module-noyau/
categories:
  - linux
  - sysadmin
tags:
  - build
  - debian
  - dkms
  - kermel
  - module
  - paquet
  - semtex
  - wrapper

---
Voici un exemple, pour le module wrapper qui permet de corriger la faille CVE-2013-2094 qui peut être exploitée avec l&rsquo;outil semtex.c

Il faut installer les dépendances suivantes :

```
apt-get install debhelper dkms linux-headers-`uname -r`
```

Puis on télécharge les sources du module dans /urs/src:

```
cd /usr/src
wget https://www.develer.com/~arighi/linux/fix/CVE-2013-2094/perf-bug-fix.tar.gz
tar xvzf perf-bug-fix.tar.gz
```

Il faut nommer le dossier en utilisant la syntaxe « module-version »

```
mv perf-bug-fix wrapper-0.1
```

Puis on se place dans le dossier

```
cd wrapper-0.1
```

Il faut créer un fichier dkms.conf contenant les lignes suivantes, en remplacant le nom du module et la version :

```
PACKAGE_NAME="perl-bug-fix"
PACKAGE_VERSION="0.1"
CLEAN="make clean"
MAKE[0]="make all KVERSION=$kernelver"
BUILT_MODULE_NAME[0]="wrapper"
DEST_MODULE_LOCATION[0]="/updates"
AUTOINSTALL="yes"
POST_BUILD="exit 0"
```

Ensuite on ajoute le module à DKMS (toujours adapter nom/version) :

```
# dkms add -m wrapper -v 0.1

Creating symlink /var/lib/dkms/wrapper/0.1/source ->
                 /usr/src/wrapper-0.1

DKMS: add completed.
```

Dans mon cas, j&rsquo;ai dû adapter le fichier Makefile (c&rsquo;est probablement très moche):

```
ifndef KDIR
KDIR:=/lib/modules/`uname -r`/build
endif
PWD := $(shell pwd)
obj-m := wrapper.o

all:
        $(MAKE) -C $(KDIR) SUBDIRS=$(PWD) modules
install:
        $(MAKE) -C $(KDIR) SUBDIRS=$(PWD) modules_install
clean:
        rm -f *.o *.ko *.mod.* .*.cmd *.ko.unsigned Module.symvers modules.order
        rm -rf .tmp_versions
```

Puis on lance la compilation :

```
# dkms build -m wrapper -v 0.1

Kernel preparation unnecessary for this kernel.  Skipping...

Building module:
cleaning build area....
make KERNELRELEASE=3.2.0-4-amd64 all KVERSION=3.2.0-4-amd64....
cleaning build area....

DKMS: build completed.
```

On crée le paquet debian source :

```
# dkms mkdsc -m wrapper -v 0.1 --source-only
Using /etc/dkms/template-dkms-mkdsc
copying template...
modifying debian/changelog...
modifying debian/compat...
modifying debian/control...
modifying debian/copyright...
modifying debian/dirs...
modifying debian/postinst...
modifying debian/prerm...
modifying debian/README.Debian...
modifying debian/rules...
copying legacy postinstall template...
Copying source tree...
Building source package... dpkg-source --before-build wrapper-dkms-0.1
 debian/rules clean
 dpkg-source -b wrapper-dkms-0.1
dpkg-source: warning: no source format specified in debian/source/format, see dpkg-source&#40;1&#41;
 dpkg-genchanges -S >../wrapper-dkms_0.1_source.changes
dpkg-genchanges: including full source code in upload
 dpkg-source --after-build wrapper-dkms-0.1


DKMS: mkdsc completed.
Moving built files to /var/lib/dkms/wrapper/0.1/dsc...
Cleaning up temporary files...
```

Puis on crée le paquet Debian:

```
# dkms mkdeb -m wrapper -v 0.1 --source-only
Using /etc/dkms/template-dkms-mkdeb
copying template...
modifying debian/changelog...
modifying debian/compat...
modifying debian/control...
modifying debian/copyright...
modifying debian/dirs...
modifying debian/postinst...
modifying debian/prerm...
modifying debian/README.Debian...
modifying debian/rules...
copying legacy postinstall template...
Copying source tree...
Building binary package...dpkg-buildpackage: warning: using a gain-root-command while being root
 dpkg-source --before-build wrapper-dkms-0.1
 fakeroot debian/rules clean
 debian/rules build
 fakeroot debian/rules binary
 dpkg-genchanges -b >../wrapper-dkms_0.1_amd64.changes
dpkg-genchanges: binary-only upload - not including any source code
 dpkg-source --after-build wrapper-dkms-0.1


DKMS: mkdeb completed.
Moving built files to /var/lib/dkms/wrapper/0.1/deb...
Cleaning up temporary files...
```

Il faut déplacer le dossier puis on peut tester :

```
# mv /var/lib/dkms/wrapper/ /var/lib/dkms/wrapper-old/
# dpkg -i /var/lib/dkms/wrapper-old/0.1/deb/wrapper-dkms_0.1_all.deb 
Selecting previously unselected package wrapper-dkms.
&#40;Reading database ... 154687 files and directories currently installed.&#41;
Unpacking wrapper-dkms &#40;from .../deb/wrapper-dkms_0.1_all.deb&#41; ...
Setting up wrapper-dkms &#40;0.1&#41; ...
Loading new wrapper-0.1 DKMS files...
First Installation: checking all kernels...
Building for 3.2.0-4-amd64 and 3.8-2-amd64
Building for architecture amd64
Building initial module for 3.2.0-4-amd64
Done.

wrapper:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/3.2.0-4-amd64/updates/dkms/

depmod....

DKMS: install completed.
Module build for the currently running kernel was skipped since the
kernel source for this kernel does not seem to be installed.
```

Il ne reste qu&rsquo;à charger le module :

```
# modprobe wrapper
# lsmod | grep wrapper
wrapper                12421  0
```
