---
title: "Installation d'un serveur OpenVPN et automatisation"
author: VaLouille
type: post
date: 2013-02-18T22:27:39+00:00
url: /2013/02/installation-dun-serveur-openvpn-et-automatisation/
categories:
  - linux
  - sysadmin
tags:
  - client
  - debian
  - openvpn
  - script
  - serveur

---
OpenVPN est un logiciel permettant de monter un serveur VPN SSL. Le but est de créer un LAN qui passe sur le WAN, et dont les données sont cryptées. Cela est utile quand on ne souhaite pas ouvrir de ports à partir de tout internet par exemple. Ce tutorial présente l&rsquo;installation d&rsquo;un serveur OpenVPN avec une identification par certificats, qui sont uniques à chaque personne.

  * Installation d&rsquo;un serveur OpenVPN

Il faut installer le paquet OpenVPN

```
apt-get install openvpn
```

Puis on copie les samples dans le répertoire de configuration

```
mkdir /etc/openvpn/easy-rsa/
cp -r /usr/share/doc/openvpn/examples/easy-rsa/2.0/* /etc/openvpn/easy-rsa/
```

Il faut éditer les variables suivantes

```
vim /etc/openvpn/easy-rsa/vars
```

```
export KEY_COUNTRY="FR"
export KEY_PROVINCE="75"
export KEY_CITY="Paris"
export KEY_ORG="Nom de la societé"
export KEY_EMAIL="email@domaine.fr"
```

On lance les commandes suivantes pour générer les clés et certificats (ne pas tenir compte des messages qui s&rsquo;affichent, ceux-ci sont normaux)

```
cd /etc/openvpn/easy-rsa/
source vars
./clean-all
./build-dh
./pkitool --initca
./pkitool --server server
openvpn --genkey --secret keys/ta.key
```

On copie les clés dans le repertoire de configuration

```
cp keys/ca.crt keys/ta.key keys/server.crt keys/server.key keys/dh1024.pem /etc/openvpn/
```

On chroot OpenVPN afin de limiter la casse en cas de faille dans OpenVPN

```
mkdir /etc/openvpn/jail
mkdir /etc/openvpn/clientconf
```

On crée le fichier de configuration du serveur, en adaptant avec l&rsquo;adresse IP publique et le port sur lequel on souhaite faire écouter le serveur VPN.

```
vim /etc/openvpn/server.conf
```

```
mode server
proto tcp
port <port> # A adapter
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh1024.pem
tls-auth ta.key 
cipher AES-256-CBC
server 10.8.0.0 255.255.255.0 # On peut changer le réseau du LAN ici
keepalive 10 120
user nobody
group nogroup
chroot /etc/openvpn/jail
persist-key
persist-tun
comp-lzo
verb 3
mute 20
status openvpn-status.log
```

On lance ensuite le serveur

```
cd /etc/openvpn
openvpn server.conf
```

Si la ligne suivante s&rsquo;affiche à la fin, le serveur est correctement configuré

```
Tue Oct 16 09:13:00 2012 Initialization Sequence Completed
```

On peut donc l&rsquo;arréter avec un CTRL+C

On ajoute la ligne suivante à la fin du fichier de configuration /etc/openvpn/server.conf

```
log-append /var/log/openvpn.log
```

Puis on lance le serveur

```
/etc/init.d/openvpn start
```

Il faut activer l&rsquo;ip_forward

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Puis le mettre en dur dans la configuration en cas de reboot

```
vim /etc/sysctl.conf
```

```
net.ipv4.ip_forward = 1
```

Il faut créer la règle iptables, à adapter en fonction du LAN, et à ajouter à /etc/init.d/firewall

```
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```

  1. Ajouter une nouvelle route aux clients dès leur connexion

On modifie le fichier /etc/openvpn/server.conf

Ajouter la ligne après « server <IP> <mask> » :

```
push 'route 192.168.3.0 255.255.255.0'
```

On relance le serveur OpenVPN :

```
/etc/init.d/openvpn restart
```

Dans le cas où la chaine « FORWARD » d&rsquo;iptables est en « Policy DROP », il faut autoriser le routage

vers une machine en particlulier :

```
iptables -A FORWARD -i tun0 -s 10.8.0.0/24 -d 192.168.3.121 -j ACCEPT
```

vers un réseau entier :

```
iptables -A FORWARD -i tun0 -s 10.8.0.0/24 -d 192.168.3.0/24 -j ACCEPT
```

  * <span style="font-family: 'Droid Sans', 'Helvetica Neue', 'Nimbus Sans L', sans-serif; font-size: 15px; line-height: 1.62em;">Script d&rsquo;ajout d&rsquo;utilisateurs

Voici le script d&rsquo;ajout automatique d&rsquo;utilisateurs, ne pas oublier de change l&rsquo;adresse IP et le numéro de port :

```
#!/bin/bash
# Syntaxe: # ./create_vpn_user.sh <prenom-nom>
#
# Test que le script est lance en root
if [ $EUID -ne 0 ]; then
  echo "Le script doit etre lance en root: # $0 <prenom-nom>" 1>&2
  exit 1
fi

# Test parametre
if [ $# -ne 1 ]; then
  echo "Il faut saisir le nom de la personne: # $0 <prenom-nom>" 1>&2
  exit 1
fi

cd /etc/openvpn/easy-rsa

echo "Creation du client OpenVPN: $1"
echo "Veuillez choisir le type de certificat :"
echo "1) Certificat SANS mot de passe"
echo "2) Certificat AVEC mot de passe"
read key

case $key in
        1)
                echo "Creation du certificat SANS mot de passe pour le client $1"
                source vars
                ./build-key $1
                ;;

        2)
                echo "Creation du certificat AVEC mot de passe pour le client $1"
                source vars
                ./build-key-pass $1
                ;;

        *)
                echo "Choix non correct !"
                echo "Arret du script"
                exit 0
                ;;
esac

mkdir /etc/openvpn/clientconf/$1
cp /etc/openvpn/ca.crt /etc/openvpn/ta.key keys/$1.crt keys/$1.key /etc/openvpn/clientconf/$1/

cd /etc/openvpn/clientconf/$1
cat >> client.conf << EOF
# Client
client
dev tun
proto tcp-client
remote <IP> <port>
resolv-retry infinite
cipher AES-256-CBC
# Cles
ca ca.crt
cert $1.crt
key $1.key
tls-auth ta.key 1
# Securite
nobind
persist-key
persist-tun
comp-lzo
verb 3
EOF
cp client.conf client.ovpn

tar cvzf $1.tar.gz *

echo "Creation du client OpenVPN $1 termine"
echo "/etc/openvpn/clientconf/$1/$1.tar.gz"
echo "---"
```

Une fois le script exécuté, il crée une archive .tar.gz dans /etc/openvpn/clientconf/nom-du-client. Il faut lui fournir cette archive, qui contient clés et certificats, ainsi que le fichier de configuration pour windows et linux

  * Ajout manuel de client

On crée la clé

```
cd /etc/openvpn/easy-rsa
source vars
./build-key
```

Il faut répondre aux questions, et on peut paramétrer un mot de passe ou pas.

Puis on crée le repertoire correspondant au client et on déplace les fichiers générés dedans

```
mkdir /etc/openvpn/clientconf/<prenom-nom>
cp /etc/openvpn/ca.crt /etc/openvpn/ta.key keys/<prenom-nom>.crt keys/<prenom-nom>.key /etc/openvpn/clientconf/<prenom-nom>
cd /etc/openvpn/clientconf/<prenom-nom>/
```

Il faut ensuite créer le fichier de configuration, en pensant à adapter l&rsquo;IP et le port, ainsi que les noms des certificats :

```
vim client.conf
```

```
# Client
client
dev tun
proto tcp-client
remote <IP> <port>
resolv-retry infinite
cipher AES-256-CBC
ca ca.crt
cert <prenom-nom>.crt
key <prenom-nom>.key
tls-auth ta.key 1
nobind
persist-key
persist-tun
comp-lzo
verb 3
```

On copie le fichier en changeant l&rsquo;extention pour créer un fichier de configuration Windows

```
cp client.conf client.ovpn
```

Puis on génère l&rsquo;archive que l&rsquo;on va fournir au client

```
tar cvzf .tar.gz *
```

  * Configuration côté client
  * Sous Linux (Network-Manager) :

Il faut tout d&rsquo;abord installer openvpn et network-manager-openvpn-gnome

```
apt-get install openvpn resolvconf network-manager-openvpn-gnome
```

Puis déplacer l&rsquo;archive .tar.gz dans /etc/openvpn/ et l&rsquo;extraire

```
mv prenom-nom.tar.gz /etc/openvpn/
cd /etc/openvpn
tar xvzf prenom-nom.tar.gz
```

Aller dans Tableau de bord > Connexions VPN > Configurer le VPN.

Cliquer sur le bouton Importer.

Choisir le fichier /etc/openvpn/client.conf

Remplacer le nom de la connexion par un nom plus explicite

Cliquer sur « Editer », puis aller dans « Routes », et cocher la case « Utiliser cette connexion uniquement pour les ressources du réseau »

Cliquer sur « Appliquer », puis aller dans Tableau de bord > Connexions VPN > Client

Le VPN devrait être connecté.

  * Sous Windows

Télécharger OpenVPN for Windows :
  
L&rsquo;installer, accepter la création du nouveau périphérique « TAP ».
  
Extraire l&rsquo;archive et copier les fichiers dans C:\Programmes\OpenVPN\config
  
Lancer OpenVPN GUI (Avec les droits administrateurs, clique droit, Exécuter en tant qu&rsquo;administrateur »)
  
Une icône apparaitra en bas, et quelques secondes plus tard, une bulle informe de l&rsquo;IP assignée.

  * Sous Linux à la main

Aller dans le répertoire qui contient l&rsquo;archive extraite, puis lancer la commande suivante

```
openvpn --script-security 2 --config client.conf
```

La ligne suivante doit s&rsquo;afficher à la fin

```
Tue Oct 16 16:30:39 2012 Initialization Sequence Completed
```
