---
title: Installer Puppet 2.6 sous Debian 3.1 Sarge et Etch
author: VaLouille
type: post
date: 2013-03-28T12:23:15+00:00
url: /2013/03/installer-puppet-2-6-sous-debian-3-1-sarge-et-etch/
categories:
  - linux
  - sysadmin
tags:
  - debian
  - etch
  - libssl0.9.8
  - openssl
  - puppet
  - rubygems
  - sarge

---
A l&rsquo;installation de puppet sous Debian Sarge, on se retrouve avec l&rsquo;erreur suivante :

```
err: /File[/var/lib/puppet/lib]: Failed to generate additional resources using 'eval_generate': unknown message digest algorithm
err: /File[/var/lib/puppet/lib]: Could not evaluate: unknown message digest algorithm Could not retrieve file metadata for puppet://bushmills.nexen.net/plugins: unknown message digest algorithm
```

Il s&rsquo;agit d&rsquo;un bug de la libssl0.9.8, comme on le voit dans les bug reports suivants : [#648285][1], [#544819][2], ou [#541735][3]
  
Sous Debian Sarge, c&rsquo;est la version 0.9.7 qui est utilisée. Ce bug n&rsquo;a donc pas été résolu sous Sarge. J&rsquo;ai donc compilé des paquets backportés de Etch afin de bypasser ce bug sans perturber le système. Voici la procédure complète pour installer Puppet 2.6 sous Debian Sarge. Cette procédure est aussi compatible avec Etch.

Installer Ruby à partir des dépots :

```
apt-get install curl wget lsb-base lsb-release irb1.8 libreadline-ruby1.8 libruby libruby1.8 rdoc1.8 ruby ruby1.8 ruby1.8-dev libopenssl-ruby1.8 libshadow-ruby1.8
```

Installer RubyGems :

```
wget http://rubyforge.org/frs/download.php/20989/rubygems-0.9.4.tgz
tar xzf rubygems-0.9.4.tgz
cd rubygems-0.9.4
ruby setup.rb
```

Installer Puppet et Facter avec RubyGems :

```
wget -q http://rubygems.org/downloads/facter-1.6.18.gem
wget -q http://rubygems.org/downloads/puppet-2.6.2.gem
gem install --no-rdoc --no-ri facter-1.6.18.gem
gem install --no-rdoc --no-ri puppet-2.6.2.gem
```

Créer le répertoire /etc/puppet :

```
mkdir /etc/puppet
```

Créer le fichier de configuration /etc/puppet/puppet.conf en remplaçant l&rsquo;URL du serveur :

```
[main]
logdir=/var/log/puppet
vardir=/var/lib/puppet
ssldir=/var/lib/puppet/ssl
rundir=/var/run/puppet
factpath=\$vardir/lib/facter
templatedir=\$confdir/templates
pluginsync = true
server=server.puppet.com
report=true
```

On installe le fichier d&rsquo;init :

```
curl -k -s 'https://blog.valouille.fr/wp-content/uploads/2013/03/puppet.init' > /etc/init.d/puppet
chmod 755 /etc/init.d/puppet
update-rc.d puppet defaults 21
```

On met on place le fichier /etc/default/puppet avec le contenu suivante :

```
# Defaults for puppet - sourced by /etc/init.d/puppet

# Start puppet on boot?
START=yes

# Startup options
DAEMON_OPTS=""
```

On paramètre les droits :

```
chmod 644 /etc/default/puppet
```

**Uniquement pour Sarge :**
  
Télécharger l&rsquo;archive contenant les .deb de openssl [ssl-puppet-sarge.tar.gz][4] et les installer :

```
wget https://blog.valouille.fr/wp-content/uploads/2013/03/ssl-puppet-sarge.tar.gz
tar xvzf ssl-puppet-sarge.tar.gz
dpkg -i libopenssl-ruby1.8_1.8.5-4+awh_i386.deb libssl0.9.8_0.9.8c-4+awh_i386.deb openssl_0.9.8c-4+awh_i386.deb libruby1.8_1.8.5-4~bpo.1_i386.deb
```

Il ne reste plus qu&rsquo;à tester :

```
puppet agent --test
```

 [1]: http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=648285 "#648285"
 [2]: http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=544819 "#544819"
 [3]: http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=541735 "#541735"
 [4]: https://blog.valouille.fr/wp-content/uploads/2013/03/ssl-puppet-sarge.tar.gz
