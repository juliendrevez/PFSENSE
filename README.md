# DMZ pfSense – Apache & Windows Server (RDP)

**Auteur :** Julien (SIO2)  
**Dépôt GitHub :** https://github.com/juliendrevez/PFSENSE

---

## 1. Schéma de l'infrastructure

Voici le schéma réseau global de la maquette :

<p align="center">
  <img width="650" alt="Schéma réseau" src="https://github.com/user-attachments/assets/00c42154-cfd2-4ae9-9c97-c0b0cad54cc5" />
</p>

---

## 2. Adressage IP

| Équipement / Interface | IP / Masque | Rôle |
| :--- | :--- | :--- |
| **WAN (pfSense)** | `192.168.20.194/24` | Réseau du lycée (simule Internet) |
| **LAN (pfSense)** | `192.168.12.1/24` | Réseau d'administration |
| **DMZ (pfSense)** | `192.168.30.1/24` | Passerelle de la DMZ |
| **Serveur Apache** | `192.168.30.12/24` | Serveur Web Debian |
| **Windows Server** | `192.168.30.20/24` | Serveur Bureau à distance (RDP) |
| **Client Lycée** | `192.168.20.59/24` | Client de test sur le WAN |

---

## 3. Configuration pfSense

### Interfaces
Assignation classique dans **Interfaces → Assignments** :
- **WAN :** DHCP (`192.168.20.194`)
- **LAN :** `192.168.12.1/24`
- **DMZ :** `192.168.30.1/24`

### Rules & NAT

#### NAT pour Apache (WAN → DMZ)
Redirection du port 80 du WAN vers le serveur Web :

<img width="1052" height="571" alt="Règle NAT Apache" src="https://github.com/user-attachments/assets/6ee6932a-ec3d-47f1-a4a7-b01154918555" />

#### Règles WAN
Règle qui autorise le flux HTTP vers le serveur Apache en `192.168.30.12` :

<img width="647" height="586" alt="Règle WAN Apache" src="https://github.com/user-attachments/assets/dab75202-517a-4137-ad59-03ce01955bb0" />

Règle pour ouvrir l'accès vers la DMZ depuis le réseau :

<img width="753" height="547" alt="Accès DMZ" src="https://github.com/user-attachments/assets/bd6935b1-b66b-431d-b870-c67fe7997ead" />

---

## 4. Config du serveur Apache (DMZ)

Sur la VM Debian en DMZ : on change la carte réseau dans Hyperviseur/Proxmox pour la basculer sur le vSwitch de la DMZ. On passe ensuite la VM en IP statique (`192.168.30.12`).

<p align="center">
  <img width="393" height="286" alt="Carte réseau VM" src="https://github.com/user-attachments/assets/48ca1c06-eb1e-4c17-9976-dccde490d9d3" />
</p>

Vérification du service Apache :

<p align="center">
  <img width="322" height="20" alt="Status Apache" src="https://github.com/user-attachments/assets/50f13c50-e3c1-4faf-a866-233c02e3ae28" />
</p>

---

## 5. Tests de validation

### Test depuis pfSense
Validation avec l'outil **Diagnostics → Test Port** sur le port 80 :

<img width="722" height="472" alt="Test Port pfSense" src="https://github.com/user-attachments/assets/e634b0ce-1c9d-40f2-8398-2460c6010faa" />

### Test depuis un PC du lycée
Depuis le poste client (`192.168.20.59`), en tapant l'IP WAN de pfSense (`192.168.20.194`), on tombe bien sur la page par défaut d'Apache.

<img width="1238" height="937" alt="Test navigateur client" src="https://github.com/user-attachments/assets/f45b4d15-1037-4f8a-b153-f7f9e2ad9ad7" />

---

## 6. Sécurité mise en place

- **DMZ isolée :** Aucun flux direct possible vers le LAN pour éviter de perturber le réseau de la salle SISR (DHCP, etc.).
- **Moindre privilège :** Seul le port 80 est ouvert vers le serveur Web.
- **Règles WAN strictes :** On bloque tout par défaut sauf les ports spécifiquement redirigés.
- **Pas de SSH depuis le WAN :** L'administration se fait uniquement depuis le réseau LAN.

---

## 7. Partie 2 : Configuration du Bureau Distant (RDP)

### VM Windows Server
J'ai créé une VM Windows Server connectée à l'interface `vmbrjudmz` (DMZ).  
J'ai configuré une IP fixe (`192.168.30.20`) afin de pouvoir cibler correctement la VM lors de la création du NAT.

<p align="center">
  <img width="397" height="455" alt="IP Windows Server" src="https://github.com/user-attachments/assets/53a50e17-8efb-4283-a62d-689fe2db51f8" />
</p>

### Activation du RDP
Dans le **Gestionnaire de serveur**, activation du Bureau à distance. Le pare-feu Windows ouvre automatiquement le port `3389`.

<img width="992" height="465" alt="Activation RDP" src="https://github.com/user-attachments/assets/849b9b14-900b-41b9-ad25-066e2ff5b0c7" />

### Règle NAT sur pfSense
Pour accéder au RDP depuis le WAN (réseau lycée), ajout d'une règle NAT Port Forward.  
J'ai utilisé un port personnalisé (`50000`) côté WAN qui redirige vers le port `3389` de la VM en DMZ.

<img width="916" height="571" alt="NAT RDP pfSense" src="https://github.com/user-attachments/assets/cb7e7dca-40df-40da-aa69-b8b405f3573a" />

### Validation de la connexion
Tentative de connexion RDP depuis un PC client sur `192.168.20.194:50000` :

<p align="center">
  <img width="406" height="241" alt="Connexion MSTSC" src="https://github.com/user-attachments/assets/7473852d-cab4-48a7-b05d-fd2603c2ed2d" />
</p>

La connexion s'établit correctement :

<img width="1272" height="810" alt="RDP OK" src="https://github.com/user-attachments/assets/3909dde0-3c01-4fea-b689-5fa13eb18836" />
