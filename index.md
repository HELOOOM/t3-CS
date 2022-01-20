# Objectif

Les entrées ARP peut facilement être manipulées en utilisant des paquets de 
données falsifiées. On parle alors d’ARP spoofing (de l’anglais spoof, qui signifie 
échanger), un type d’attaque de l’homme du milieu, qui permet aux pirates 
d’échanger deux systèmes de communication en passant inaperçuS

1. Prise en main de `bettercap` sur kali linux pour des simulation des 
attaques. 

Pour l’installation, je vous invite à suivre leur github, qui est très bien documentée : 
https://github.com/bettercap/bettercap

1- Tout d’abord, pour lancer bettercap, il suffit de lancer la commande suivante :


`sudo bettercap`

2- Avant de commencer à voir les différentes attaques, il faut savoir que nous allons 
utiliser des « modules », et ces derniers peuvent se configurer en utilisant la 
commande « set » : 

`set dns.spoof.domain toto.com`

Et pour lancer le module, il suffit simplement d’appeler le module et d’ajouter l’argument 
« on » ou « off »


`dns.spoof on`

![Capture d’écran 2022-01-20 095031](https://user-images.githubusercontent.com/53974876/150305317-9a5ded56-7dbd-4b82-b977-2a09c52d7b02.png)


2. Attaque ARP 

Cette attaque consiste à empoisonner le cache ARP de la machine cible afin de permettre de 
router les paquets vers la machine pirate. 
En effet, dans un réseau local, toutes les communications se font via les adresses MAC des 
machines. L’attaque ARP va permettre de modifier la table d’adresse MAC de la victime. Il 
est judicieux de modifier l’adresse MAC de la passerelle par défaut de la victime afin de 
pouvoir récupérer tous les paquets transmis par la victime. 
Attention, si vous attaquez la victime sans configurer votre machine comme « routeur », 
l’attaque sera appelée DOS (Denial Of Service). Les requêtes réalisées par la victime ne sera 
pas transmis au « vrai » destinataire et seront donc « drop ». 


Il est donc judicieux d’activer le mode routeur du Kali en saisissant cette commande : `echo 
1 > /proc/sys/net/ipv4/ip_forward`

Pour trouver la cible que l’on souhaite attaquer, utiliser la commande suivante :

**net.show**

![Capture d’écran 2022-01-20 095224](https://user-images.githubusercontent.com/53974876/150305380-b63b0f07-0840-45e0-ad5d-803031f226d9.png)


Notre victime sera la machine possédant l’adresse IP : 192.168.5.99 : 

`set arp.spoof.targets 192.168.5.99`


**NB** : Par défaut, la cible est le réseau où se trouve l’attaquant. Dans notre cas, s’il y a pas du 
modification de arp.spoof.target, l’attaque ARP aurait été propagée sur le réseau 
` 192.168.5.0/24. `

**Verifier la table ARP de la victime (avant l’attaque) : **

![Capture d’écran 2022-01-20 095639](https://user-images.githubusercontent.com/53974876/150306015-36aa308d-74fb-4814-b7f8-4adf79571fdb.png)


L’adresse MAC commençant par « e4-5d » est la passerelle du réseau. L’adresse 
MAC commençant par « d8-cb » est la machine de l’attaquant 

Lorsque nous activons le module :

`arp.spoof on`

![Capture d’écran 2022-01-20 095839](https://user-images.githubusercontent.com/53974876/150306418-1bf1dc7b-d584-4f39-86cc-452e568f39c9.png)

Nous pouvons observer la modification de la table ARP de la victime : 

A partir de maintenant, nous avons effectué une attaque MITM (Man-In-The-Middle). Nous 
pouvons « sniffer » le réseau et nous verrons apparaître les paquets transmis par la victime. 
Nous pouvons dès à présent combiner cette attaque avec une attaque de type « Spoof DNS » 
pour vérifier son fonctionnement. 

# 3. Attaque DNS

Un « Spoof DNS » consiste à modifier la table DNS d’un client (ou d’un réseau entier). 
Lorsque la victime souhaitera contacter le site web http://apache.org (nous allons prendre le 
site apache.org comme exemple), il sera alors redirigé vers une autre adresse IP (privée ou 
public) en gardant le même nom de domaine. 


Nous commençons tout d’abord par configurer le spoof DNS :


```
set dns.spoof.domains apache.org
set dns.spoof.all true
dns.spoof on





