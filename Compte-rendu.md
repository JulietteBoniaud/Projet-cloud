# Projet cloud privé

## 1 Installation

### ESXi

Mettre la clé USB
Boot
Choisir le disque, la langue et le mot de passe (celui du sujet)
Reboot quand demandé

### Client

VM ubuntu desktop : clientesxi - Pa$$word

### PfSense

Install source https://www.certilience.fr/2018/03/tutoriel-deploiement-pfsense-pare-feu-opensource/
DNS source : https://www.arsouyes.org/blog/2019/23_DNS_Personnel/

## Configuration

### Adressage

Management : 10.0.0.0/24
- TPME25 : 10.0.0.1/24
- TPME27 : 10.0.0.2/24
- TPME26 : 10.0.0.10/24
- gateway- pfsense : 10.0.0.254/24

Stockage : 172.16.0.0/24


Prod : 10.20.30.0/24


vMotion : 10.50.100/24

# Besoin

ISO :
- Win2k12
- Win10
- vCenter

### vCenter

Sur une machine win10 monter le cd et aller dans /vcsa-ui-installer/win32 et lancer installer.exe

1 Choisir l'installation (deploy)
2 accepter les conditions
3 type de déploiment : Embedded Platform Service Controller
4 ESXi : 10.0.0.1
  Port HTTPS: 443 (default)
  username : root
  password : (ESXi root password)
5 VMname : vCenter
  root password : (comme on veut)
6 Deployment size : tiny (default)
  Storage size : (default)
7 Choix du stockage (default)
8 Network settings
    ip: 10.0.0.3
    netmask: /24
    gateway: 10.0.0.254
    dns: 10.0.0.5
    
    reste : default
    
9 Resumé -> cliquer sur Finish et attendre ....