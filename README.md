# 📘 Projet DMZ – pfSense + Apache & Windows Server (RDP)

> **Auteur :** Julien  
> **Classe :** SIO2  
> **Dépôt GitHub :** [github.com/juliendrevez/PFSENSE](https://github.com/juliendrevez/PFSENSE)

---

## 📐 1. Schéma infrastructure réseau

<p align="center">
  <img width="650" alt="Schéma Infrastructure Réseau" src="https://github.com/user-attachments/assets/00c42154-cfd2-4ae9-9c97-c0b0cad54cc5" />
</p>

---

## 📊 2. Plan d'adressage IP

| Équipement / Interface | Adresse IP / Masque | Rôle & Description |
| :--- | :--- | :--- |
| **WAN (pfSense)** | `192.168.20.194/24` | Accès depuis le réseau du lycée (Internet TP) |
| **LAN (pfSense)** | `192.168.12.1/24` | Réseau d'administration interne |
| **DMZ (pfSense)** | `192.168.30.1/24` | Passerelle de la zone démilitarisée (Serveurs) |
| **Serveur Apache** | `192.168.30.12/24` | Service Web HTTP (Debian) |
| **Windows Server** | `192.168.30.20/24` | Service Bureau à distance RDP |
| **PC Client (Lycée)** | `192.168.20.59/24` | Poste client sur le réseau WAN |

---

## 🔧 3. Configuration pfSense

### 3.1 Interfaces
Configuration réalisée dans **Interfaces → Assignments** :
* **WAN :** DHCP (`192.168.20.194`)
* **LAN :** `192.168.12.1/24`
* **DMZ :** `192.168.30.1/24`

### 3.2 Règles Firewall & NAT

#### 🔹 Règle NAT (WAN → DMZ)
Permet d’accéder au serveur Apache depuis le WAN :

<img width="1052" height="571" alt="natrule" src="https://github.com/user-attachments/assets/6ee6932a-ec3d-47f1-a4a7-b01154918555" />

#### 🔹 Règles WAN
Règle permettant au WAN d’accéder au serveur Apache (`192.168.30.12`) :

<img width="647" height="586" alt="wanrule" src="https://github.com/user-attachments/assets/dab75202-517a-4137-ad59-03ce01955bb0" />

Règle permettant qu'on puisse depuis le réseau avoir accès à la DMZ :

<img width="753" height="547" alt="image" src="https://github.com/user-attachments/assets/bd6935b1-b66b-431d-b870-c67fe7997ead" />

---

## 🌐 4. Configuration du serveur Apache (DMZ)

Réalisé sur la VM Debian située dans la DMZ.  
Une fois fait, on vient changer la carte réseau et mettre la même que celle de la DMZ. Le but est donc de bloquer, puis de modifier l'adresse IP en statique.

<p align="center">
  <img width="393" height="286" alt="Carte réseau VM Debian" src="https://github.com/user-attachments/assets/48ca1c06-eb1e-4c17-9976-dccde490d9d3" />
</p>

<p align="center">
  <img width="322" height="20" alt="Apache status" src="https://github.com/user-attachments/assets/50f13c50-e3c1-4faf-a866-233c02e3ae28" />
</p>

---

## 🔁 5. Tests de fonctionnement

### ✔ Test depuis pfSense
*(Diagnostics → Test Port)*

<img width="722" height="472" alt="Test Port pfSense" src="https://github.com/user-attachments/assets/e634b0ce-1c9d-40f2-8398-2460c6010faa" />

### ✔ Test depuis le PC du lycée
> Le résultat doit être la page par défaut Apache (car rien d'autre d'installé sur le serveur LAMP, ce qui est tout à fait normal !).

<img width="1238" height="937" alt="Test web client" src="https://github.com/user-attachments/assets/f45b4d15-1037-4f8a-b153-f7f9e2ad9ad7" />

---

## 🔐 6. Sécurité & Bonnes Pratiques

* **Isolation :** DMZ isolée du LAN (aucun risque d'interférence avec le DHCP de la salle SISR).
* **Minimisation des accès :** NAT uniquement sur le port 80 pour Apache (en ouvrir d'autres présenterait un risque inutile pour la DMZ).
* **Règles WAN restrictives :** C'est la seule interface exposée au réseau extérieur (réseau de la salle SISR). Le but est de bloquer tout ce qui n'est pas du HTTP/HTTPS pour éviter les attaques.
* **Pas d’accès SSH depuis le WAN :** L'administration à distance depuis l'extérieur est interdite ; elle doit se faire exclusivement depuis le LAN admin.

---

## 🖥️ 7. Partie 2 : Mise en place d'un Bureau Distant (RDP)

### 7.1 Configuration de la VM Windows Server

J’ai créé une VM Windows Server dans Proxmox et je l’ai connectée à l’interface réseau `vmbrjudmz`, qui correspond au réseau DMZ.

J’ai ensuite configuré une IP statique dans la plage DMZ. Une IP statique est obligatoire pour faire un NAT propre et garantir que tout le monde puisse y accéder via une adresse fixe.

<p align="center">
  <img width="397" height="455" alt="Ip windows serveur" src="https://github.com/user-attachments/assets/53a50e17-8efb-4283-a62d-689fe2db51f8" />
</p>

### 7.2 Activation du Bureau à distance

Dans Windows Server, j’ai activé le Bureau à distance via le **Gestionnaire de serveur**.  
Le pare-feu Windows autorise automatiquement le port `3389`, aucune règle supplémentaire sur l'hôte n'a été nécessaire.

<img width="992" height="465" alt="Activation Bureau a Distance dans la VM winserv" src="
