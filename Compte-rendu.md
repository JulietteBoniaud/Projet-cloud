# Projet cloud privé

## Prérequis

### Images

- Win2k12
- Win10
- vCenter
- pfSense
- freeNAS
- VMware

### Matériel

Au minimum
- 1 switch
- 2 serveurs
- 1 poste client

Dans l'idéal un troisième serveur serait bien

## Configuration réseau

Mettre les interfaces du switch avec les esxi en trunk native vlan *vlanManagement*

### Adressage

*Les adresses utilisées sont en exemples* 

Management : 10.0.0.0/24 - Réseau utilisé pour la configuration/gestion des serveurs, contient les serveurs et éventuellement une machine admin 
> Les ESXi ont besoin de 2 interfaces dans ce réseau pour la haut disponibilité
- ESX1 : 10.0.0.1/24
	: 10.0.0.7/24
- ESX2 : 10.0.0.2/24
	: 10.0.0.8/24
- vCenter: 10.0.0.3/24
- FreeNAS: 10.0.0.4/24
- Ad1	 : 10.0.0.5/24
- Ad2    : 10.0.0.6/24
- pfsense : 10.0.0.254/24

Stockage : 172.16.0.0/24 - Réseau utilisé pour accéder au stockage des VMs 
- FreeNAS : 172.16.0.1/24
- ESX2   : 172.16.0.2/24
- ESX1	  : 172.16.0.3/24

Prod : 10.20.30.0/24 - Réseau de production/utilisation, pour les clients, les AD et la passerelle
*adresses réparties en dhcp pour les clients*
- AD1	: 10.20.30.200
- AD2	: 10.20.30.201
- Gateway : 10.20.30.254

vMotion : 10.50.100.0/24 - Réseau pour transférer les VMs d'un ESXi à l'autre
- ESX1	: 10.50.100.1
- ESX2 : 10.50.100.2

## Installation

### ESXi

Sur les serveurs
Mettre la clé USB
Boot
Choisir le disque, la langue et le mot de passe (celui du sujet)
Reboot quand demandé

### PfSense

Sur le poste client, avec VirtualBox créer une VM base FreeBSD
La démarrer avec l'ISO pfsense et suivre l'installation

### Controlleur de domain

Sur un des ESXi créer une VM avec une image Windows Server 2012
Ajouter les rôles AD DNS 
Configurer le DNS
Ajouter quelques utilisateurs

> Pour tester, installer une VM Windows10 et se connecter avec un compte de l'AD

### vCenter

> Pour cela, il faut une vm client win10 et un DNS avec un nom de domaine qui fonctionne

Sur une machine win10 monter le cd et aller dans /vcsa-ui-installer/win32 et lancer installer.exe

Part 1 
  
1. Choisir l'installation (deploy)
2. accepter les conditions
3. type de déploiment : Embedded Platform Service Controller
4. ESXi : 10.0.0.1
  Port HTTPS: 443 (default)
  username : root
  password : (ESXi root password)
5. VMname : vCenter
  root password : (comme on veut)
6. Deployment size : tiny (default)
  Storage size : (default)
7. Choix du stockage (default)
8. Network settings
    ip: 10.0.0.3
    netmask: /24
    gateway: 10.0.0.254
    dns: 10.0.0.5    
    reste : default    
9. Resumé -> cliquer sur Finish et attendre

Part 2

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

:warning: Il est conseillé éteindre toutes les VMs, avant de procéder à la suite

3 Dans le cluster clisquer sur action, et ajouter les ESXi :
  adresse ip
  Nom d'utilisateur ESXi
  Mot de passe ESXi
  
#### HA
  Activer la HA permet de surveiller l'activité des machines. Pour cela 

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
      
### Interfacer FreeNAS et ESXi

#### Ajouter un adaptateur de stockage

Sur vCenter, Cluster - IP ESXi - Configurer - Stockage - Adaptateur de stockage
Ajouter adaptateur iSCSI, mettre en cible l'ip du freeNAS (cf portail configuré avant)
Tout mettre sur vlan stockage (s'assurer que tout ping)

#### Ajout des des LUNs

Cluster - Action - Stockage - Nouvelle banque de données
    Type : VMFS
Selectionner un des ESXi
Selectionner le LUN 
Recommencer pour les autres LUNs

:warning: Pour utiliser le vMotion, il faut mettre le stockage des vm sur le stockage partagé créer sinon c'est caca (l'opération se fait à froid)

### Update Manager
Menu - Update Manager - Parametre - modifier - tu mets ce que tu veux t'es grand

L'update manger permet de mettre à jour les VMwareTools mais aussi les serveurs ESXi.
Pour cela il faut d'abors importer une image d'ESXi, puis définir une ligne de base.
VSphere va vérifier quelle ESX ont une versions inférieurs, et les mettre à jour.
On peut procéder manuellement ou avec une tâche planifié.

## Sources


Doc VMware : https://docs.vmware.com/en/VMware-vSphere/index.html
Doc FreeNAS : https://www.ixsystems.com/documentation/freenas/11.3-RELEASE

vSphere HA : https://docs.vmware.com/fr/VMware-vSphere/6.7/com.vmware.vsphere.avail.doc/GUID-AC35EFDD-F8B7-4FAF-B946-6553D7BDBF31.html
vSphere DRS: https://www.mchelgham.com/partie3-configuration-de-vsphere-ha-drs-evc-video/

Doc pfSense : 
    Install pfSense https://www.certilience.fr/2018/03/tutoriel-deploiement-pfsense-pare-feu-opensource/
    DNS source : https://www.arsouyes.org/blog/2019/23_DNS_Personnel/