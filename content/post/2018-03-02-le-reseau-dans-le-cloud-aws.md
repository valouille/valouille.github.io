---
title: Le réseau dans le cloud AWS
author: VaLouille
type: post
date: 2018-03-02T20:32:16+00:00
url: /2018/03/le-reseau-dans-le-cloud-aws/
categories:
  - AWS
tags:
  - AWS
  - network
  - diagram
  - schema
  - reference
  - architecture
  - réseau
  - VPC
  - multi-AZ
  - HA

---
Il est nécessaire de bien comprendre plusieurs concepts réseau afin de déployer une application dans le cloud AWS de manière optimale. Comme je n&rsquo;ai pas trouvé de schéma qui regroupe l&rsquo;ensemble des concepts, j&rsquo;ai décidé d&rsquo;en faire un, afin d&rsquo;avoir une architecture de référence à utiliser pour décrire ces concepts. Évidemment, étant donné la relative complexité de l&rsquo;infrastructure, le schéma est assez grand. Vous pouvez cliquer sur l&rsquo;image pour ouvrir le fichier original.

[<img class="alignnone wp-image-426 size-full" src="https://blog.valouille.fr/wp-content/uploads/2018/03/AWS_Architecture-1.png" alt="" width="3570" height="3132" srcset="https://blog.valouille.fr/wp-content/uploads/2018/03/AWS_Architecture-1.png 3570w, https://blog.valouille.fr/wp-content/uploads/2018/03/AWS_Architecture-1-342x300.png 342w, https://blog.valouille.fr/wp-content/uploads/2018/03/AWS_Architecture-1-768x674.png 768w, https://blog.valouille.fr/wp-content/uploads/2018/03/AWS_Architecture-1-700x614.png 700w" sizes="(max-width: 3570px) 100vw, 3570px" />][1]

Voici les principaux concepts à bien intégrer afin de comprendre ce schéma :

  * Il existe plusieurs **régions** dans AWS. Une **région** correspond à une zone géographique, par exemple Dublin ou Paris. La liste complète des régions est disponible à [la page suivante.][2]
  * Chaque région est composée d&rsquo;une ou plusieurs « zone de disponibilité », ou **Availability Zones**. Chaque **AZ** est indépendante l&rsquo;une de l&rsquo;autre, et il est nécessaire de placer au moins une instance dans chaque **AZ** pour assurer la haute disponibilité.
  * Un **VPC** est un réseau privé virtuel qui peut s&rsquo;étendre sur plusieurs **AZ**s. On lui attribue un réseau ainsi qu&rsquo;un masque associé.
  * Dans ce **VPC**, on crée des sous-réseaux, ou **subnets**, qui eux sont créés dans une **AZ**. Ces subnets communiquent via un routeur virtuel entre eux. Entre chaque subnet, il existe donc un routeur virtuel. Dans ce schéma, tous les routeurs ne sont pas représentés pour des soucis de lisibilité
  * Il est possible d&rsquo;associer à chaque subnet une **Network ACL**, ou NACL, afin d&rsquo;effectuer du filtrage « [_stateless_][3]« .
  * À chaque subnet est associée une table de routage, ou **Route Table**.
  * Dans ces subnets, on crée des instances **EC2**, c&rsquo;est à dire des machines virtuelles.
  * On associe à chaque instance EC2 (comme à la plupart des services AWS) un **Security Group**, qui contient des règles de filtrage.
  * Un subnet est peut être considéré comme public ou privé. 
  * Un **subnet public** contient une table de routage dans la passerelle par défaut (gateway) envoie vers une **Internet Gateway** (**IGW**).
  * Un **subnet privé** contient une table de routage qui ne renvoie pas vers une **IGW**.
  * Si l&rsquo;on souhaite accéder à Internet à partir d&rsquo;un subnet privé, il est nécessaire de créer une **NAT Gateway** dans le subnet public, et de configurer la passerelle par défaut dans la route table du subnet privé vers la NAT Gateway de l&rsquo;AZ correspondante. Uniquement l&rsquo;accès vers Internet est alors possible, la NAT Gateway ne faisant que du « Source NAT »
  * Si l&rsquo;on souhaite accéder à une instance à partir de l&rsquo;extérieur, il est nécessaire de créer un **Application Load Balancer** (**ALB**) ou un **Elastic Load Balancer** (**ELB**).
  * Si on souhaite rendre accessible une instance EC2 d&rsquo;un subnet public à partir de l&rsquo;extérieur, il est nécessaire de lui attribuer une **Elastic IP** (**EIP**).
  * Il est possible de faire communiquer entre eux deux VPC, en mettant en place du **VPC Peering** et en adaptant les tables de routage.
  * Généralement on configure entre le réseau des bureaux et le VPC un VPN pour accéder aux instances de manière sécurisée. Une **Virtual Private Gateway** (**VGW**) est alors utilisée pour établir la connexion VPN entre le boitier de l&rsquo;entreprise (**Customer Gateway**), et AWS. Les tables de routage doivent également être mises à jour.
  * Il est également possible à partir de la VGW de passer par un prestataire pour relier son data center à AWS via une liaison dédiée **Direct Connect**.
  * Certains services, comme **S3** (**Simple Storage Service**), ne sont pas directement accessibles à partir du VPC. On peut donc créer un **VPC Endpoint** pour éviter de passer par le public pour accéder à ces services.
  * Certains services, comme **RDS** (**Relational Database Service**) fonctionnement sur des instances EC2. Ces services sont donc placés dans un subnet, comme n&rsquo;importe quelle instance EC2.

J&rsquo;espère que ce schéma va vous permettre d&rsquo;y voir un peu plus clair dans le fonctionnement interne du réseau dans AWS. Les sources sont disponibles [ici][4] et peuvent être modifiées avec [Draw.io][5]

 [1]: https://blog.valouille.fr/wp-content/uploads/2018/03/AWS_Architecture-1.png
 [2]: https://aws.amazon.com/about-aws/global-infrastructure/
 [3]: https://fr.wikipedia.org/wiki/Protocole_sans_%C3%A9tat
 [4]: https://blog.valouille.fr/wp-content/uploads/2018/03/AWS_Architecture.xml
 [5]: https://draw.io
