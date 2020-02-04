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

Source https://www.certilience.fr/2018/03/tutoriel-deploiement-pfsense-pare-feu-opensource/


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
