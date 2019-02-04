# B1 Réseau 2018 - TP4

## 1 - Création des réseaux

Voici les deux cartes pour les réseaux `net1` et `net2`, avec le DHCP désactivé.
VirtualBox Host-only Ethernet Adaptater

## 2 - Création des VMs

### CheckList

VM cliente --> `tp4.client`:

*ip a* :

    inet 10.1.0.10/24 brd 10.1.0.255

*ip statique* :

    BOOTPROTO=static
    IPADDR=10.1.0.10
    NETMASK=255.255.255.0
    NAME=enp0s8
    DEVICE=enp0s8
    ONBOOT=yes

*SSH* :

    C:\Users\asus>ssh 10.1.0.10
    host@10.1.0.10's password:
    Last login: Sun Feb  3 19:13:13 2019
    [host@tp4 ~]$

*Définition nom de domaine fichier:*

    tp4.client

*Remplissage des noms de domaines externes fichier: /etc/hosts*:

    10.2.0.10 serveur tp4.serveur
    10.1.0.254 routeur tp4.routeur

***

VM serveur --> `tp4.serveur`:

*ip a* :

    inet 10.2.0.10/24 brd 10.2.0.255 scope global noprefixroute enp0s8

*fichier d'ip statique : /etc/sysconfig/network-scripts/ifcfg-enp0s8* :

    BOOTPROTO=static
    IPADDR=10.2.0.10
    NETMASK=255.255.255.0
    NAME=enp0s8
    DEVICE=enp0s8
    ONBOOT=yes

*SSH* :

    C:\Users\asus>ssh 10.2.0.10
    host@10.2.0.10's password:
    Last login: Sun Feb  3 19:13:09 2019
    [host@tp4 ~]$

*Définition nom de domaine fichier: /etc/hostname*

    tp4.serveur

*Remplissage des noms de domaines externes fichier: /etc/hosts*:

    10.2.0.254 routeur tp4.routeur
    10.1.0.10 client tp4.client

***

VM Routeur--> `tp4.routeur`:

*ip a* :
  
     inet 10.1.0.254/24 brd 10.1.0.255 scope global noprefixroute enp0s8
     inet 10.2.0.254/24 brd 10.2.0.255 scope global noprefixroute enp0s9

*fichier d'IP statique : /etc/sysconfig/network-scripts/ifcfg-enp0s8* :

    BOOTPROTO=static
    IPADDR=10.1.0.254
    NETMASK=255.255.255.0
    NAME=enp0s8
    DEVICE=enp0s8
    ONBOOT=yes

*fichier d'ip statique : /etc/sysconfig/network-scripts/ifcfg-enp0s8* :

    BOOTPROTO=static
    IPADDR=10.2.0.254
    NETMASK=255.255.255.0
    NAME=enp0s9
    DEVICE=enp0s9
    ONBOOT=yes

*SSH* :

    C:\Users\host>ssh 10.1.0.254
    host@10.1.0.254's password:
    Last login: Sun Feb  3 19:13:18 2019
    [host@tp4 ~]$

*Définition nom de domaine fichier: /etc/hostname*

    tp4.routeur

*Remplissage des noms de domaines externes fichier: /etc/hosts*:

    10.2.0.10 serveur tp4.serveur
    10.1.0.10 client tp4.client

### Tests de ping

`client1`  ping  `tp4.routeur`  sur `10.1.0.254`:

    [host@tp4 ~]$ ping tp4.routeur
    PING routeur (10.1.0.254) 56(84) bytes of data.
    64 bytes from routeur (10.1.0.254): icmp_seq=1 ttl=64 time=0.770 ms
    *64 bytes from routeur (10.1.0.254): icmp_seq=2 ttl=64 time=0.281 ms
    64 bytes from routeur (10.1.0.254): icmp_seq=3 ttl=64 time=0.294 ms
    64 bytes from routeur (10.1.0.254): icmp_seq=4 ttl=64 time=0.262 ms
    64 bytes from routeur (10.1.0.254): icmp_seq=5 ttl=64 time=0.348 ms
    ^C
    --- routeur ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 4000ms
    rtt min/avg/max/mdev = 0.262/0.391/0.770/0.191 ms

`server1` ping `tp4.routeur` sur `10.2.0.254`

    [host@tp4 ~]$ ping tp4.routeur
    PING routeur (10.2.0.254) 56(84) bytes of data.
    64 bytes from routeur (10.2.0.254): icmp_seq=1 ttl=64 time=0.657 ms
    64 bytes from routeur (10.2.0.254): icmp_seq=2 ttl=64 time=0.357 ms
    64 bytes from routeur (10.2.0.254): icmp_seq=3 ttl=64 time=0.367 ms
    64 bytes from routeur (10.2.0.254): icmp_seq=4 ttl=64 time=0.326 ms
    64 bytes from routeur (10.2.0.254): icmp_seq=5 ttl=64 time=0.327 ms
    ^C
    --- routeur ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 4002ms
    rtt min/avg/max/mdev = 0.326/0.406/0.657/0.128 ms


## Mise en place du routage statique

*Routes connues de la VM Routeur ver `net1`et `net2`: ip route show* :

    10.1.0.0/24 dev enp0s8 proto kernel scope link src 10.1.0.254 metric 100
    10.2.0.0/24 dev enp0s9 proto kernel scope link src 10.2.0.254 metric 101

### Mise en place des routes VMs Client / Serveur

*Mise en place de la route Client --> Serveur fichier: /etc/sysconfig/network-scripts/route-enp0s8* :

    10.2.0.0/24 via 10.1.0.254 dev enp0s8

*Mise en place de la route Serveur--> Client fichier: /etc/sysconfig/network-scripts/route-enp0s8* :

    10.1.0.0/24 via 10.2.0.254 dev enp0s8

### Test de ping 

Ping `tp4.serveur` depuis la VM Cliente:

    [host@tp4 ~]$ ping tp4.serveur
    PING serveur (10.2.0.10) 56(84) bytes of data.
    64 bytes from serveur (10.2.0.10): icmp_seq=1 ttl=63 time=0.796 ms
    64 bytes from serveur (10.2.0.10): icmp_seq=2 ttl=63 time=0.683 ms
    64 bytes from serveur (10.2.0.10): icmp_seq=3 ttl=63 time=0.684 ms
    64 bytes from serveur (10.2.0.10): icmp_seq=4 ttl=63 time=0.690 ms
    64 bytes from serveur (10.2.0.10): icmp_seq=5 ttl=63 time=0.602 ms
    ^C
    --- serveur ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 4003ms
    rtt min/avg/max/mdev = 0.602/0.691/0.796/0.061 ms

Ping `tp4.client`depuis la VM Serveur:

    [host@tp4 ~]$ ping tp4.client
    PING client (10.1.0.10) 56(84) bytes of data.
    64 bytes from client (10.1.0.10): icmp_seq=1 ttl=63 time=0.994 ms
    64 bytes from client (10.1.0.10): icmp_seq=2 ttl=63 time=0.632 ms
    64 bytes from client (10.1.0.10): icmp_seq=3 ttl=63 time=0.618 ms
    64 bytes from client (10.1.0.10): icmp_seq=4 ttl=63 time=0.887 ms
    64 bytes from client (10.1.0.10): icmp_seq=5 ttl=63 time=0.669 ms
    ^C
    --- client ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 4002ms
    rtt min/avg/max/mdev = 0.618/0.760/0.994/0.152 ms

On effectue un `traceroute` vers le serveur pour savoir le chemin pris pas un packet :

    [host@tp4 ~]$ traceroute tp4.serveur
    traceroute to tp4.serveur (10.2.0.10), 30 hops max, 60 byte packets
     1  routeur (10.1.0.254)  0.342 ms  0.235 ms  0.228 ms
     2  serveur (10.2.0.10)  0.512 ms !X  0.318 ms !X  0.255 ms !X

## Spéléologie réseau : ARP

### Manipulation 1

Après avoir fait un `sudo ip neigh flush all` sur toutes les machines:

Table ARP de la VM Cliente:

    [host@tp4 ~]$ ip neigh
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:0f REACHABLE

Table ARP de la VM Serveur:

    [host@tp4 ~]$ ip neigh
    10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:08 REACHABLE

Ces lignes représentent toutes les deux la connexion SSH avec les cartes réseaux Virtual Box du PC Hôte.

*Après ping VM Cliente sur VM Serveur*:

    [host@tp4 ~]$ ping serveur
    PING serveur (10.2.0.10) 56(84) bytes of data.
    64 bytes from serveur (10.2.0.10): icmp_seq=1 ttl=63 time=1.68 ms
    ^C
    --- serveur ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 1.684/1.684/1.684/0.000 ms
    
    [host@tp4 ~]$ ip neigh
    10.1.0.254 dev enp0s8 lladdr 08:00:27:0f:67:71 STALE
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:0f DELAY

*Apès ping VM Serveur sur VM Cliente*:

    [host@tp4 ~]$ ping client
    PING client (10.1.0.10) 56(84) bytes of data.
    64 bytes from client (10.1.0.10): icmp_seq=1 ttl=63 time=0.808 ms
    ^C
    --- client ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.808/0.808/0.808/0.000 ms
    
    [host@tp4 ~]$ ip neigh
    10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:08 REACHABLE
    10.2.0.254 dev enp0s8 lladdr 08:00:27:c5:a6:79 DELAY

La ligne qui s'est ajoutée dans la table ARP de chaque VM est l'IP de la VM Routeur, car c'est la seule VM qui à accès aux réseaux `net1` et `net2`, on ne voit pas les IPs des VM Client et Serveur car c'est la VM Routeur qui s'occupe de redistribuer le message qui ne lui est pas destiné.

### Manipulation 2

Après avoir fait un `sudo ip neigh flush all` sur toutes les machines:

*Avant le ping de VM Cliente sur VM Serveur*:

[host@tp4 ~]$ ip neigh
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:0f REACHABLE

*Après le ping de la VM Cliente sur la VM Serveur*:

[host@tp4 ~]$ ip neigh
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:0f REACHABLE
10.2.0.10 dev enp0s9 lladdr 08:00:27:40:5e:56 REACHABLE
10.1.0.10 dev enp0s8 lladdr 08:00:27:b5:a5:05 REACHABLE

On peut remarquer que les adresses de deux machines se sont ajoutés sur la table ARP de la VM routeur, car c'est elle qui agis comme pont entre les réseaux `net1` et `net2` .

 1. Client ping serveur en passant par la route 10.1.0.254 qui est le routeur.
	 - [x] Routeur ajoute donc la MAC de la VM Cliente.
 2. Client ne connais pas la route pour aller vers le réseau `net2` mais Routeur la connais, et connais même l'adresse directe de Serveur pour donner le message qui lui est destiné.
	 - [x] Routeur ajoute donc la MAC de la VM Serveur.

### Manipulation 3

*Après avoir vidé la table ARP avec la commande : netsh interface ip delete arpcache*

    Interface : 10.2.0.1 --- 0x8
      Adresse Internet      Adresse physique      Type
      224.0.0.22            01-00-5e-00-00-16     statique
    
    Interface : 192.168.1.36 --- 0x9
      Adresse Internet      Adresse physique      Type
      192.168.1.254         6c-38-a1-06-bf-04     dynamique
      224.0.0.2             01-00-5e-00-00-02     statique
    
    Interface : 10.1.0.1 --- 0xf
      Adresse Internet      Adresse physique      Type
      224.0.0.22            01-00-5e-00-00-16     statique
    
    Interface : 192.168.56.1 --- 0x15
      Adresse Internet      Adresse physique      Type
      224.0.0.22            01-00-5e-00-00-16     statique

*Et après avoir un peut attendu* :

    Interface : 10.2.0.1 --- 0x8
      Adresse Internet      Adresse physique      Type
      10.2.0.255            ff-ff-ff-ff-ff-ff     statique
      224.0.0.22            01-00-5e-00-00-16     statique
      224.0.0.252           01-00-5e-00-00-fc     statique
      239.255.255.250       01-00-5e-7f-ff-fa     statique
    
    Interface : 192.168.1.36 --- 0x9
      Adresse Internet      Adresse physique      Type
      192.168.1.39          f0-79-60-03-75-a6     dynamique
      192.168.1.254         6c-38-a1-06-bf-04     dynamique
      192.168.1.255         ff-ff-ff-ff-ff-ff     statique
      224.0.0.2             01-00-5e-00-00-02     statique
      224.0.0.251           01-00-5e-00-00-fb     statique
      224.0.0.252           01-00-5e-00-00-fc     statique
      239.255.255.250       01-00-5e-7f-ff-fa     statique
    
    Interface : 10.1.0.1 --- 0xf
      Adresse Internet      Adresse physique      Type
      10.1.0.255            ff-ff-ff-ff-ff-ff     statique
      224.0.0.22            01-00-5e-00-00-16     statique
      224.0.0.252           01-00-5e-00-00-fc     statique
      239.255.255.250       01-00-5e-7f-ff-fa     statique
    
    Interface : 192.168.56.1 --- 0x15
      Adresse Internet      Adresse physique      Type
      192.168.56.255        ff-ff-ff-ff-ff-ff     statique
      224.0.0.22            01-00-5e-00-00-16     statique
      224.0.0.252           01-00-5e-00-00-fc     statique
      239.255.255.250       01-00-5e-7f-ff-fa     statique

 1. On peut remarquer l'IP `239.255.255.250` qui correspond au protocole SSDP (Simple Service Discovery Protocol).
 2. Ensuite on à aussi l'IP `224.0.0.22`, `224.0.0.252` ou encore `224.0.0.2` qui correspond 
	 - 224.0.0.2 correspond à l'adresse multicast, pour que les routeurs d'un sous-réseau communiques entre eux.
	 - 224.0.0.22 est l'IP utilisé par le protocole IGMP (Internet Group Management Protocol).
	 - 224.0.0.252 et 224.0.0.251 qui sont des IPs servant au LLMNR (Link-Local Multicast Name Resolution).

### Manipulation 4

Après avoir fait un `sudo ip neigh flush all` sur toutes les machines:

*Sur VM Cliente*

    [host@tp4 ~]$ ip neigh
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:0f REACHABLE
    
    [host@tp4 ~]$ curl google.com
    <HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
    <TITLE>301 Moved</TITLE></HEAD><BODY>
    <H1>301 Moved</H1>
    The document has moved
    <A HREF="http://www.google.com/">here</A>.
    </BODY></HTML>
    
    [host@tp4 ~]$ ip neigh
    10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 REACHABLE
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:0f REACHABLE

On peut remarquer que l'IP `10.0.2.2` s'est ajoutée a la table, elle correspond à l'IP locale de la VM CentOS (comme quand on veut se connecter sur un serveur qui est sur la machine en cours d'utilisation, on utilise 127.0.0.1 sur Windows), c'est donc cette IP acceptée dans le pare-feu CentOS, qui fait le lien entre la VM et la carte NAT.

## Wireshark

*Ping depuis La VM Cliente vers la VM Serveur* :

    [host@tp4 ~]$ ping -c 4 serveur
    PING serveur (10.2.0.10) 56(84) bytes of data.
    64 bytes from serveur (10.2.0.10): icmp_seq=1 ttl=63 time=0.918 ms
    64 bytes from serveur (10.2.0.10): icmp_seq=2 ttl=63 time=0.660 ms
    64 bytes from serveur (10.2.0.10): icmp_seq=3 ttl=63 time=0.530 ms
    64 bytes from serveur (10.2.0.10): icmp_seq=4 ttl=63 time=0.626 ms
    
    --- serveur ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3003ms
    rtt min/avg/max/mdev = 0.530/0.683/0.918/0.145 ms

*Capture des packets sur la VM Routeur* :

    [host@tp4 ~]$ sudo tcpdump -i enp0s9 -w ping.pcap
    tcpdump: listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
    ^C12 packets captured
    12 packets received by filter
    0 packets dropped by kernel

Pour le transfert de fichiers, j'utilise Filezilla :

<img src="filezilla.PNG">

On peut donc voir plus clairement tous les protocoles et packets envoyés durant ce ping : 

<img src="ping_wireshark.PNG">

<img src="echo_request.PNG">

<img src="echo_reply.PNG">

En effet il manque beaucoup de trames de ping, car le routeur aussi envois des ping `request` et `reply`, normalement il manque 8 ping entre chaque ping entre VM Cliente et VM Serveur.
