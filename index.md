# TP3

### BY : Sakassa Rachid ,Lakhmiri Mohammed elias, Benzemroun Badr

## Simulations d’attaques(DNS,ARP,PROXY)


# Objectif

Les entrées ARP peut facilement être manipulées en utilisant des paquets de 
données falsifiées. On parle alors d’ARP spoofing (de l’anglais spoof, qui signifie 
échanger), un type d’attaque de l’homme du milieu, qui permet aux pirates 
d’échanger deux systèmes de communication en passant inaperçuS

## Prise en main de `bettercap` sur kali linux pour des simulation des 
attaques. 


Pour l’installation, je vous invite à suivre leur github, qui est très bien documentée : 
https://github.com/bettercap/bettercap

![WhatsApp Image 2022-01-20 at 09 56 19](https://user-images.githubusercontent.com/53974876/150309674-4b2112cf-139e-4498-9627-b9077d1e8e66.jpeg)


1- Tout d’abord, pour lancer bettercap, il suffit de lancer la commande suivante :

![WhatsApp Image 2022-01-20 at 09 56 03](https://user-images.githubusercontent.com/53974876/150309872-2383fcad-d62a-46f8-a9d6-67d1c344e3be.jpeg)


```bash
sudo bettercap
```

2- Avant de commencer à voir les différentes attaques, il faut savoir que nous allons 
utiliser des « modules », et ces derniers peuvent se configurer en utilisant la 
commande « set » : 

`set dns.spoof.domain toto.com`

Et pour lancer le module, il suffit simplement d’appeler le module et d’ajouter l’argument 
« on » ou « off »


```bash
dns.spoof on
```

![Capture d’écran 2022-01-20 095031](https://user-images.githubusercontent.com/53974876/150305317-9a5ded56-7dbd-4b82-b977-2a09c52d7b02.png)


## 2. Attaque ARP 

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

```bash
set arp.spoof.targets 192.168.5.99
```


**NB** : Par défaut, la cible est le réseau où se trouve l’attaquant. Dans notre cas, s’il y a pas du 
modification de arp.spoof.target, l’attaque ARP aurait été propagée sur le réseau 
` 192.168.5.0/24. `

**Verifier la table ARP de la victime (avant l’attaque) :**

![Capture d’écran 2022-01-20 095639](https://user-images.githubusercontent.com/53974876/150306015-36aa308d-74fb-4814-b7f8-4adf79571fdb.png)


L’adresse MAC commençant par « e4-5d » est la passerelle du réseau. L’adresse 
MAC commençant par « d8-cb » est la machine de l’attaquant 

Lorsque nous activons le module :

```bash
arp.spoof on
```

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


```bash
set dns.spoof.domains apache.org
set dns.spoof.all true
dns.spoof on
```

![Capture d’écran 2022-01-20 100331](https://user-images.githubusercontent.com/53974876/150307240-a648321a-d5b0-4cd4-acd4-c47b30128fb4.png)

**NB** : la commande « set dns.spoof.all true » permet de prendre en compte les requêtes 
provenant de l’extérieur (et non local à la machine).

## Rappel : 

```bash
set arp.spoof.targets 192.168.5.99
arp.spoof on
```

Vu que nous redirigeons les flux en provenance de « apache.org » vers l’ip de l’attaquant, nous 
allons monter un serveur web avec Bettercap : 

```bash
set http.server.path /var/www/html
http.server on
```
![Capture d’écran 2022-01-20 100517](https://user-images.githubusercontent.com/53974876/150307545-b799e339-febf-4f14-9c82-a8cbe63b5141.png)

Attention de ne pas avoir un serveur web déjà allumé, sinon il faut modifier le port dans la 
configuration du serveur dans Bettercap (set http.server.port)
Avant la modification de la table ARP de la victime, lorsqu’il se rend sur apache.org il atteint 
la page officielle :
![Capture d’écran 2022-01-20 100604](https://user-images.githubusercontent.com/53974876/150307675-596afc27-aa3d-4c75-a6af-6c4b2625c100.png)


Après l’empoisonnement du cache ARP, la page apache.org redirige vers la page web de 
l’attaquant : 

![image](https://user-images.githubusercontent.com/53974876/150307826-31b494d4-2abc-4ef1-8aa2-a63b185d72ee.png)

L’attaque DNS est maintenant terminée,

# Attaque Proxy


Une attaque Proxy est le fait de pouvoir récupérer les logs de toutes les requêtes web. Il est 
intéressant de mettre en place un proxy lorsque l’on souhaite bloquer certains sites web dans 
une entreprise. 
Dans notre cas, nous allons utiliser le proxy pour récupérer les pages que consulte la victime. 
Tout d’abord, il faut configurer le niveau de verbosité du sniffer (false équivaut à réduit) : 


```bash
set net.sniff.verbose false
net.sniff on
```
![image](https://user-images.githubusercontent.com/53974876/150307984-4b4c6db4-f6a6-4aa4-b19d-3abbec235bc4.png)

Puis on configure le proxy :

```bash
set http.proxy.sslstrip true
http.proxy on
```
![image](https://user-images.githubusercontent.com/53974876/150308224-6a9f001f-3f23-4752-96b2-51f1a7f0791e.png)


L’utilisation du paramètre `« sslstrip »` nous permet d’activer le proxy https, et de proposer au 
client un certificat généré par Bettercap. 
Là encore nous gardons en place l’attaque ARP afin de cibler la victime. 
Et lorsque la victime consulte des pages web, nous avons des messages sur bettercap :


Attention aux sites en HTTPS, certains ont une protection HSTS qui permet de bloquer (ou du 
moins complexifier l’attaque) :

![image](https://user-images.githubusercontent.com/53974876/150308364-54785db0-3465-426b-bbf4-26d6f65cb06a.png)


`voir dans Bettercap que la victime à bien été sur google comme ce suit:`

![image](https://user-images.githubusercontent.com/53974876/150308508-87f368e0-89dc-4633-b7af-0355f1d0c82f.png)


Lorsque la victime souhaite accéder à un site en HTTPS (sans HSTS) :

![image](https://user-images.githubusercontent.com/53974876/150308591-f6f2d46c-9a6c-4f4e-a8a9-a2eee72fb0bc.png)


Une erreur de certificat apparaît sur la page du navigateur car c’est un certificat non approuvé 
par l’autorité de certification. Si la victime continue sans faire attention, nous pourrons alors 
récupérer potentiellement des informations confidentielles. 
En effet, vu que nous avons généré le certificat pour le proxy, nous détenons la clef privée. 
Tous les flux peuvent donc être déchiffrés sans difficultés. 






