📘 Projet DMZ – pfSense + Apache
Auteur : Julien  
Classe : SIO2
Dépôt GitHub :  https://github.com/juliendrevez/PFSENSE
🧩 1. Objectif du TP
Ce TP consiste à mettre en place une architecture réseau sécurisée comprenant :
un pare‑feu pfSense
une DMZ
un serveur Apache dans la DMZ
une règle NAT permettant d’accéder au serveur depuis le WAN
une isolation stricte entre LAN / DMZ / WAN

 Interface  IP  Rôle 
 WAN ->  192.168.20.194/24 -> Accès depuis le réseau du lycée 
 LAN -> 192.168.12.1/24 -> Administration
 DMZ -> 192.168.30.1/24 -> Hébergement du serveur Apache
 Serveur Apache -> 192.168.30.12 ->  Service web|
 PC client (lycée) -> 192.168.20.59 -> Accès WAN 

 🔧 3. Configuration pfSense
3.1 Interfaces
Configuration réalisée dans Interfaces → Assignments :
WAN : DHCP (192.168.20.194)
LAN : 192.168.12.1/24
DMZ : 192.168.30.1/24
3.2 Règles Firewall
🔹 Règle NAT (WAN → DMZ)
Permet d’accéder au serveur Apache depuis le WAN :
<img width="1052" height="571" alt="natrule" src="https://github.com/user-attachments/assets/6ee6932a-ec3d-47f1-a4a7-b01154918555" />
🔹 Règles WAN
Règle permettant au WAN d’accéder au serveur Apache qui a pour ip 192.168.30.12
<img width="647" height="586" alt="wanrule" src="https://github.com/user-attachments/assets/dab75202-517a-4137-ad59-03ce01955bb0" />

🌐 4. Configuration du serveur Apache (DMZ)
Sur la VM Debian dans la DMZ :
<img width="322" height="20" alt="apache" src="https://github.com/user-attachments/assets/50f13c50-e3c1-4faf-a866-233c02e3ae28" />

🔁 5. Tests de fonctionnement
✔ Test depuis pfSense (Diagnostics → Test Port)

<img width="722" height="472" alt="image" src="https://github.com/user-attachments/assets/e634b0ce-1c9d-40f2-8398-2460c6010faa" />

✔ Test depuis le PC du lycée
Le résultat doit donc être la page apache (car rien d'autre installer sur le lamp donc c'est normal que l'on arrive sur sa !)

<img width="1238" height="937" alt="image" src="https://github.com/user-attachments/assets/f45b4d15-1037-4f8a-b153-f7f9e2ad9ad7" />

🔐 6. Sécurité
DMZ isolée du LAN donc pas de risque avec le DHCP de la salle SISR.

NAT uniquement sur le port 80 car Apache n'a besoin que de celui la, en ouvrir d'autre serait une prise de risque pour la DMZ

Règles WAN restrictives car c'est la seule interface exposée au réseau exterieur à celui ou est la DMZ ( donc le réseau de la salle SISR par exemple). Le but est donc de bloquer tout ce qui n'est pas du HTTP/HTTPS pour éviter les risques d'attaques


Pas d’accès SSH depuis WAN car il pourrait permettre d'administrer le serveur depuis l'extérieur et ce n'est pas ce que l'on veut et doit donc se faire uniquement depuis le LAN

Partie 2 : Mise en place d'un bureau distant sous PFSense

J’ai créé une VM Windows Server dans Proxmox et je l’ai connectée à l’interface réseau vmbrjudmz, qui correspond au réseau DMZ.

J’ai ensuite configuré une IP statique dans la plage DMZ . Une IP statique est obligatoire pour pouvoir faire un NAT propre ensuite et que tout le monde puisse y accéder avec une IP donner. 
<img width="397" height="455" alt="Ip windows serveur " src="https://github.com/user-attachments/assets/53a50e17-8efb-4283-a62d-689fe2db51f8" />


Activation du Bureau à distance : 

Dans Windows Server, j’ai activé le Bureau à distance via le Gestionnaire de serveur.
Le pare‑feu Windows autorise automatiquement le port 3389, donc aucune règle supplémentaire n’a été nécessaire.
<img width="992" height="465" alt="Activation Bureau a Distance dans la VM winserv" src="https://github.com/user-attachments/assets/849b9b14-900b-41b9-ad25-066e2ff5b0c7" />

Règle NAT sur pfsense ( WAN -> DMZ)

Pour rendre le serveur accessible depuis le réseau du lycée (considéré comme Internet dans le TP), j’ai créé une règle NAT sur l’interface WAN.
J’ai choisi un port WAN différent du port RDP par défaut, comme demandé dans le cahier des charges.
<img width="916" height="571" alt="Configuration WAN dans le DMZ" src="https://github.com/user-attachments/assets/cb7e7dca-40df-40da-aa69-b8b405f3573a" />

Une fois cela fait, on peut donc depuis une machine du réseau essayer de se connecter en bureau a distance .
<img width="406" height="241" alt="tentative connexion a distance" src="https://github.com/user-attachments/assets/7473852d-cab4-48a7-b05d-fd2603c2ed2d" />
<img width="1272" height="810" alt="Connexion a distance réussi ! " src="https://github.com/user-attachments/assets/3909dde0-3c01-4fea-b689-5fa13eb18836" />
