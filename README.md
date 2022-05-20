- [Livrables](#livrables)

- [Échéance](#%c3%89ch%c3%a9ance)

- [Quelques éléments à considérer](#quelques-éléments-à-considérer-pour-les-parties-2-et-3-)

- [Travail à réaliser](#travail-%c3%a0-r%c3%a9aliser)

# Sécurité des réseaux sans fil

## Laboratoire 802.11 Sécurité WPA Entreprise

__A faire en équipes de deux personnes__

### Objectif :

1.	Analyser les étapes d’une connexion WPA Entreprise avec une capture Wireshark
2.	Implémenter une attaque WPE (Wireless Pwnage Edition) contre un réseau WPA Entreprise
1.  Implémenter une attaque GTC Dowgrade contre un réseau WPA Entreprise


## Quelques éléments à considérer pour les parties 2 et 3 :

Les parties 2 et 3 nécessitent du matériel particulier. Si vous avez travaillé jusqu'ici avec l'interface WiFi interne de votre laptop, il y a des fortes probabilités qu'elle puisse aussi être utilisée pour les attaques Entreprise. Cela dépendra de la capacité de votra interface d'être configurée en mode AP. Ces attaques ne fonctionnent pas avec toutes les interfaces Alfa. Il faudra utiliser le bon modèle.

En principe, il devrait être possible de démarrer vos machines en Kali natif (à partir d'une clé USB, avec une distro live par exemple) ou d'employer une autre version de Linux si vous voulez utiliser votre propre interface 

## Voici quelques informations qui peuvent vous aider :

- Solution à l’erreur éventuelle « ```Could not configure driver mode``` » :

```
nmcli radio wifi off
rfkill unblock wlan
```
-	Pour pouvoir capturer une authentification complète, il faut se déconnecter d’un réseau et attendre 1 minute (timeout pour que l’AP « oublie » le client) 
-	Les échanges d’authentification entreprise peuvent être facilement trouvés utilisant le filtre d’affichage « ```eap``` » dans Wireshark
-   Il est __impératif__ de bien fixer le cannal lors de vos captures


## Travail à réaliser

### 1. Analyse d’une authentification WPA Entreprise

Dans cette première partie (la moins fun du labo...), vous allez capturer une connexion WPA Entreprise au réseau de l’école avec Wireshark et fournir des captures d’écran indiquant dans chaque capture les données demandées.

A tittre d'exemple, voici [une connexion WPA Entreprise](files/auth.pcap) qui contient tous les éléments demandés. Vous pouvez utiliser cette capture comme guide de ce que la votre doit contenir. Vous pouvez vous en servir pour votre analyse __comme dernière ressource__ si vos captures ne donnent pas le résultat désiré ou s'il vous manquent des éléments importants dans vos tentatives de capture.

Pour réussir votre capture, vous pouvez procéder de la manière suivante :

- Identifier l'AP le plus proche, en identifiant le canal utilisé par l’AP dont la puissance est la plus élevée (et dont le SSID est HEIG-VD...). Vous pouvez faire ceci avec ```airodump-ng```, par exemple

- Lancer une capture avec Wireshark

- Etablir une connexion depuis un poste de travail (PC), un smartphone ou n'importe quel autre client WiFi. __Attention__, il est important que la connexion se fasse à 2.4 GHz pour pouvoir sniffer avec les interfaces Alfa

- Comparer votre capture au processus d’authentification donné en théorie (n’oubliez pas les captures d'écran pour illustrer vos comparaisons !). En particulier, identifier les étapes suivantes :
	
	Nous avons eu quelques soucis avec les captures, nous en avons donc fait plusieurs afin de récupérer un maximum de trames. Les captures d'écran proviennent donc de différentes captures Wireshark, dont celle fournie.
	
	- Requête et réponse d’authentification système ouvert
	
	  ![image-20220519153019383](images/image-20220519153019383.png)
	  
	  Le client (IntelCor_da:a2:87) envoie une requête d'authentification à l'AP (Cisco_60:c2:b0) qui lui retourne une réponse d'authentification.
	
 	- Requête et réponse d’association (ou reassociation)
	
	![image-20220519154224793](images/image-20220519154224793.png)
	
	![image-20220519154251210](images/image-20220519154251210.png)
	
	Ces captures ont été récupérées de deux captures Wireshark différentes. Mais on voit bien le requête et la réponse de l'association, toujours entre le même client et le même AP.
	
	- Négociation de la méthode d’authentification entreprise (TLS?, TTLS?, PEAP?, LEAP?, autre?)
	
	  ![image-20220519154836938](images/image-20220519154836938.png)
	
	  Nous n'avons pas réussi à récupérer l'échange complet, nous avons donc utilisé la capture fournie. Cet échange s'est fait en 3 requêtes, la première est l'AP qui indique vouloir utiliser la méthode `TLS`. La deuxième est le client qui répond négativement à l'AP, et demande une autre méthode (`PEAP`). La dernière est l'AP qui réitère sa requête en utilisant la méthode `PEAP`.
	
	- Phase d’initiation
	
	  ![image-20220519155004508](images/image-20220519155004508.png)
	
	  Ici, il s'agit d'un simple requête et réponse entre l'AP et le client.
	
	- Phase hello :
		
		Pour cette phase, nous avons pris en fonction de ce que nous avons récupérer de nos propres captures, et compléter avec la capture fournie.
		
		![image-20220519155041014](images/image-20220519155041014.png)
		
		![image-20220520133510299](images/image-20220520133510299.png)
		
		On voit ici la requête du client et du serveur.
		
		- Version TLS
		
		  ![image-20220519155517269](images/image-20220519155517269.png)
		
		  ![image-20220520133612256](images/image-20220520133612256.png)
		
		  La version TLS utilisée est la `1.2`. Comme confirmée par le serveur.
		
		- Suites cryptographiques et méthodes de compression proposées par le client et acceptées par l’AP
		
		  Suites proposées par le client :
		
		  ![image-20220519155605935](images/image-20220519155605935.png)
		
		  Méthode de compression proposée par le client :
		
		  ![image-20220520134425111](images/image-20220520134425111.png)
		
		  Suite et méthode de compression acceptée par le serveur :
		
		  ![image-20220520134238158](images/image-20220520134238158.png)
		
		- Nonces
		
		  Random du client :
		
		  ![image-20220519155652682](images/image-20220519155652682.png)
		
		  Random du serveur :
		
		  ![image-20220520134627243](images/image-20220520134627243.png)
		
		- Session ID
		
		  Client :
		  
		  ![image-20220519155931889](images/image-20220519155931889.png)
		  
		  Serveur :
		  
		  ![image-20220520134731468](images/image-20220520134731468.png)
		
	- Phase de transmission de certificats
	
	    - Echanges des certificats
	
	  ![image-20220519160859014](images/image-20220519160859014.png)
	
	  Avec la méthode `PEAP`, seulement le serveur envoie ses certificats.
	
	  Certificats transmis :
	
	  ![image-20220520135314525](images/image-20220520135314525.png)
	
	  - Change cipher spec
	
	    ![image-20220519161126191](images/image-20220519161126191.png)
	    
	    Ce message indique que les transmissions entre le client et le serveur seront chiffrées dès cette trame.
	
	- Authentification interne et transmission de la clé WPA (échange chiffré, vu par Wireshark comme « Application data »)
	
	  ![image-20220519161301026](images/image-20220519161301026.png)
	
	- 4-way handshake
	
	  ![image-20220519161326769](images/image-20220519161326769.png)

### Répondez aux questions suivantes :

> **_Question :_** Quelle ou quelles méthode(s) d’authentification est/sont proposé(s) au client ?
> 
> **_Réponse :_** Le serveur propose en premier temps la méthode `TLS`, le client répond négativement et demande la méthode `PEAP`. Le serveur repropose donc cette méthode au client.

---

> **_Question:_** Quelle méthode d’authentification est finalement utilisée ?
> 
> **_Réponse:_** La méthode `PEAP`.

---

> **_Question:_** Arrivez-vous à voir l’identité du client dans la phase d'initiation ? Oui ? Non ? Pourquoi ?
>
> **_Réponse:_** Oui, on arrive, car le contenu est en clair :
>
> ![image-20220520140007858](images/image-20220520140007858.png)

---

> **_Question:_** Lors de l’échange de certificats entre le serveur d’authentification et le client :
>
> - a. Le serveur envoie-t-il un certificat au client ? Pourquoi oui ou non ?
>
> **_Réponse:_** Oui, avec la méthode `PEAP`, le serveur envoie ses certificats, dans notre cas 3, pour s'authentifier auprès du client. Afin de vérifier la légitimité du serveur, le client va utiliser une autorité de certification.
>
> - b. Le client envoie-t-il un certificat au serveur ? Pourquoi oui ou non ?
>
> **_Réponse:_** Non, avec la méthode `PEAP`, seulement le serveur envoie ses certificats. Le serveur va authentifier le client lors de la phase d'authentification.

---

__ATTENTION__ : pour l'utilisation des deux outils suivants, vous __ne devez pas__ configurer votre interface en mode monitor. Elle sera configurée automatiquement par l'outil en mode AP.

### 2. Attaque WPA Entreprise (hostapd)

Les réseaux utilisant une authentification WPA Entreprise sont considérés aujourd’hui comme étant très surs. En effet, puisque la Master Key utilisée pour la dérivation des clés WPA est générée de manière aléatoire dans le processus d’authentification, les attaques par dictionnaire ou brute-force utilisés sur WPA Personnel ne sont plus applicables. 

Il existe pourtant d’autres moyens pour attaquer les réseaux Entreprise, se basant sur une mauvaise configuration d’un client WiFi. En effet, on peut proposer un « evil twin » à la victime pour l’attirer à se connecter à un faux réseau qui nous permette de capturer le processus d’authentification interne. Une attaque par dictionnaire ou même par brute-force peut être faite sur cette capture, beaucoup plus vulnérable d’être craquée qu’une clé WPA à 256 bits, car elle est effectuée sur le compte d’un utilisateur.

Pour faire fonctionner cette attaque, __il est impératif que la victime soit configurée pour ignorer les problèmes de certificats__ ou que l’utilisateur accepte un nouveau certificat lors d’une connexion. Si votre connexion ne vous propose pas d'accepter le nouveau certificat, faites une recherche pour configurer votre client pour ignorer les certificats lors de l'authentification.

Pour implémenter l’attaque :

- Installer [```hostapd-wpe```](https://www.kali.org/tools/hostapd-wpe/) (il existe des versions modifiées qui peuvent peut-être faciliter la tâche... je ne les connais pas mais si vous en trouvez une qui vous rend les choses plus faciles, vous pouvez l'utiliser et nous apprendre quelque chose ! Dans le doute, utiliser la version originale...). Lire la documentation [du site de l’outil](https://github.com/OpenSecurityResearch/hostapd-wpe), celle de Kali ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```hostapd-wpe``` pour proposer un réseau semblable (mais pas le même !!!) au réseau de l’école ou le réseau de votre préférence, sachant que dans le cas d'une attaque réelle, il faudrait utiliser le vrai SSID du réseau de la cible
- Lancer une capture Wireshark
- Tenter une connexion au réseau (ne pas utiliser vos identifiants réels)
- Utiliser un outil de brute-force (```john```, ```hashcat``` ou ```asleap```, par exemple) pour attaquer le hash capturé (utiliser un mot de passe assez simple pour minimiser le temps)

### Répondez aux questions suivantes :

> **_Question :_** Quelles modifications sont nécessaires dans la configuration de hostapd-wpe pour cette attaque ?
> 
> **_Réponse :_** 

---

> **_Question:_** Quel type de hash doit-on indiquer à john ou l'outil que vous avez employé pour craquer le handshake ?
> 
> **_Réponse:_** 

---

> **_Question:_** Quelles méthodes d’authentification sont supportées par hostapd-wpe ?
> 
> **_Réponse:_**


### 3. GTC Downgrade Attack avec [EAPHammer](https://github.com/s0lst1c3/eaphammer) 

[EAPHammer](https://github.com/s0lst1c3/eaphammer) est un outil de nouvelle génération pour les attaques WPA Entreprise. Il peut en particulier faire une attaque de downgrade GTC, pour tenter de capturer les identifiants du client __en clair__, ce qui évite le besoin de l'attaque par dictionnaire.

- Installer ```EAPHammer```. Lire la documentation du site de l’outil ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```EAPHammer``` pour proposer un réseau semblable au réseau de l’école ou le réseau de votre préférence. Le but est de réaliser une GTC Downgrade Attack.
- Lancer une capture Wireshark
- Tenter une connexion au réseau


### Répondez aux questions suivantes :

> **_Question :_** Expliquez en quelques mots l'attaque GTC Downgrade
> 
> **_Réponse :_** 

---

> **_Question:_** Quelles sont vos conclusions et réflexions par rapport à la méthode hostapd-wpe ?
> 
> **_Réponse:_** 


### 4. En option, vous pouvez explorer d'autres outils comme [eapeak](https://github.com/rsmusllp/eapeak) ou [crEAP](https://github.com/W9HAX/crEAP/blob/master/crEAP.py) pour les garder dans votre arsenal de pentester.

(Il n'y a pas de rendu pour cette partie...)

## Livrables

Un fork du repo original . Puis, un Pull Request contenant :

-	Captures d’écran + commentaires
-	Réponses aux questions

## Échéance

Le 2 juin 2022 à 23h59
