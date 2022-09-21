---
title: Résoudre l'erreur « SOAP request error – possibly a protocol issue » qui apparaît avec le SDK Perl de VMWare
author: VaLouille
type: post
date: 2015-08-28T12:56:50+00:00
url: /2015/08/resoudre-lerreur-soap-request-error-possibly-a-protocol-issue-qui-apparait-avec-le-sdk-perl-de-vmware/
categories:
  - linux
  - sysadmin
  - vmware
tags:
  - error
  - perl
  - soap
  - vmware
  - vsphere

---
Si vous utilisez le SDK Perl de VMWare, il est possible que vous ayez été confronté à cette erreur :

```
SOAP request error - possibly a protocol issue: 
 xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
```

Le souci provient de la lib LWP (libwww-perl) qui possède un [bug][1] à partir de la version 6.04 qui a été corrigé à la version 6.06 mais seulement pour `IO::Socket::SSL`, mais le SDK Perl de VMWare utilise Crypt::SSLeay.

Pour vérifier la version de la lib, il faut utiliser la commande suivante :

```
perl -MLWP -e 'print "$LWP::VERSION\n"';
```

Si la version est trop récente (> 6.03), on peut installer les .deb disponibles aux adresse suivantes avec dpkg :

  * http://snapshot.debian.org/archive/debian/20111023T031448Z/pool/main/libw/libwww-perl/libwww-perl\_6.03-1\_all.deb
  * http://snapshot.debian.org/archive/debian/20120602T160115Z/pool/main/libn/libnet-http-perl/libnet-http-perl\_6.03-2\_all.deb

Si vous n&rsquo;êtes pas sous Debian, ou que vous ne souhaitez pas toucher aux versions déjà installées, vous pouvez compiler à partir des sources :

```
wget -O libwww-perl_6.03.tar.gz https://github.com/libwww-perl/libwww-perl/archive/libwww-perl/6.03.tar.gz
tar xvzf libwww-perl_6.03.tar.gz
cd libwww-perl-libwww-perl-6.03
```

Il faudra ajouter à votre script la ligne suivante au début deu code:

```
export PERL5LIB=installation_path/lib/perl5
```

 [1]: https://rt.cpan.org/Public/Bug/Display.html?id=81237
