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

cf. Sources

## Configuration

### Adressage

Management : 10.0.0.0/24
- TPME25 : 10.0.0.1/24
- TPME27 : 10.0.0.2/24
- vCenter: 10.0.0.3/24
- FreeNAS: 10.0.0.4/24
- Ad1	 : 10.0.0.5/24
- Ad2    : 10.0.0.6/24
- TPME26 : 10.0.0.10/24
- gateway- pfsense : 10.0.0.254/24

Stockage : 172.16.0.0/24
- FreeNAS : 172.16.0.1/24


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

Part2

1 Next
2 Synchro du temps avec ESXi et activer ssh
3 Create new sso domain
    Single Sign-On domain name : vsphere.local
    username : administrator
    Passwoed : (c'est un secret)
4 Refuser le truc (programme experience truc)
5 Résumé -> Finish

Part3

1 Ajout d'une zone DNS cpe.local
2 Ajout d'une entrée bacula-dir sur l'adresse IP du Vcenter

### VSphere

1 Cliquer sur action, et créer un nouveau centre de donnée.
2 Dans le centre de donnée, clique sur action et créer un nouveau cluster

:warning: Il vaut mieux éteindre toutes les VMs, avant de procéder à la suite

3 Dans le cluster clisquer sur action, et ajouter les ESXi :
  adresse ip
  Nom d'utilisateur ESXi
  Mot de passe ESXi
  
### FreeNAS

Lancer iso
mettre 2 interfaces reseaux

:warning: Mettre 2 disques à la vm (un pour os/install et 1 pour stockage) sinon pas possible de créer un volume
Lancer l'installation, choisir l'espace d'install
mot de passe root (Pa$$word)
System de boot : via BIOS
Swap : no swap
attendre ....
...
remove cd et reboot

aller sur interface web
configurer dans :
  System : langue et fuseau horaire
  Réseau : ip des interfaces
  Stockage : 
    Volume - Créer un volume (Suivre la procédure)
    Sur le volume (les 3 petits points) ajouter 2 zvol (en suivant la procédure)
  Service :
    Activer l'iscsi
    En haut à droite "Wizard" pour la Configuration
      nom
      portail - nouveau - ip interface stockage
      autoriser réseau stockage uniquement
      valider
      
      

      
## Sources

Doc FreeNAS : https://www.ixsystems.com/documentation/freenas/11.3-RELEASE
    

Doc pfSense : 
    Install pfSense https://www.certilience.fr/2018/03/tutoriel-deploiement-pfsense-pare-feu-opensource/
    DNS source : https://www.arsouyes.org/blog/2019/23_DNS_Personnel/