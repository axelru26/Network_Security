# Config Switch & Firewall
Rien à faire sur le Switch pour se connecter en RDP

## Connexion à l'ASAv
RDP sur la machine Windows 192.168.2.201

Via Internet Explorer aller sur l'url : https://192.168.2.253
Cliquer sur "Install ASDM Launcher"
User/Password : admin / P@ssw0rd
Save
Run

## Connexion ASDM Launcher
Device : 192.168.2.253
Username : admin
Password : P@ssw0rd

## Configuration de base ASAv
### Configuration Interfaces
Aller dans Menu Configuration
Interface Settings
Interfaces

Sur GigabitEthernet0/2 :
	Interface Name : inside
	Security Level : 100
	Enable Interface : Case à cocher
	IP Address : 192.168.4.253
	Subnet Mask : 255.255.255.0

Sur GigabitEthernet0/1 :
	Interface Name : dmz
	Security Level : 50 (entre 1 et 99)
	Enable Interface : Case à cocher
	IP Address : 192.168.3.253
	Subnet Mask : 255.255.255.0

Apply

### Configuration Access Rules
Aller dans l'onglet Firewall en bas à gauche de la fenêtre
Aller dans Access Rules

#### Nouvelles règles
Ajout nouvelle règle sur outside :
Règle 1 :
	Action : Permit
	Source : any
	Destination : dmz
	Service : RDP_TCP3389

Règle 2 :
	Action : Permit
	Source : any
	Destination : inside
	Service : RDP_TCP3389
#### Création du service
Lorsqu'on ajoute une règle cliquer sur service
Add (en haut a gauche)
Service Object :
	Name : RDP_TCP3389
	Service Type : TCP
	Destination Port/Range : 3389
	Source Port/Range : default
	Description : RDP

### Ajout de route
Pour pouvoir aller en RDP sur 3.1 et 4.1 il faut ajouter une route static :
Dans un CMD sur la machine Windows 192.168.2.201
```
route add 192.168.3.0 mask 255.255.255.0 192.168.2.253
route add 192.168.4.0 mask 255.255.255.0 192.168.2.253
```
Pour vérifier que les routes sont les bonnes :
	route print
# VPN clientless

## Configurer 192.168.3.1
Installer le server IIS via "add roles and features" => Server Role (Web server IIS)
## SSL VPN WIZARD (WOLOLO)
Aller dans Wizard > VPN Wizard > Clientless SSL VPN Wizard (onglet en haut à gauche)
Next
![[8_1.png]]
	Connexion profile name : ClientlessProfile
	Cocher caser "Connexion group alias/url" et compléter  : CLIENTLESS
Next
![[8_2.png]]
	Cocher authentication using the local user database
	Ajouter user : cisco | password : P@ssw0rd (confirmer le password)
	
Next
	Cocher create new group policy et écrire GPClientless
Next
	Bookmark list => Manage => add => Bookmark list name :  "List1" 
	add => Select bookmark type => url with get => ok
		bookmark title : WEB | URL : http://192.168.3.1 => ok
	add =>Select bookmark type => url with get => ok
		bookmark title : CIFS | URL : cifs://192.168.3.1/share => ok => ok (global)
![[8_3.png]]
	Sélection de la liste (on clique dessus) => "Assign", sélectionner GPClientless et le user cisco => ok
	![[8_4.png]]
Next
Finish
## Remote access VPN
Configuration > Remote access VPN (en bas à gauche de l'écran) > AAA/Local Users > Local Users
Cliquer sur le user cisco => edit
Cliquer sur VPN Policy en haut à gauche, décocher inherit sur Connection Profile (Tunnel Group) Lock et sélectionner "ClientlessProfile"

Cliquer sur le user cisco => edit
Cliquer sur Identity (en haut à gauche)=> Change user password

## Vérifier que ça fonctionne
ouvrir navigateur
http://192.168.2.253/CLIENTLESS (connexion group alias créé au début)
Rentrer user name et password
Sélectionner web
(Pour faire fonctionner le cifs, il faut sélectionner la feature FTP de IIS (je crois) et share un folder)

# Anyconnect
## Configuration du firewall pour le client Anyconnect 
Wizards > VPN Wizards > Anyconnect VPN Wizards
Next => entrer dans le champ Connection Profile Name "AnyConnectProfile"
Laisser l'interface d'acces sur "outside"
Next
Décocher IPsec (laisser uniquement SSL de coché)
Next
Ajouter une client image (Add) => Uploader le fichier "xxxanyconnectxxx.pkg" fournit par le prof => ok
Next
Ajouter un user local user : cisco2 | password : P@ssw0rd => add
Next
Address Pool => New pool
- Name : Pool_Lab 
- Starting address IP : Adresse de départ pour la pool dhcp du VPN (ici 192.168.4.100)
- Ending Ip address : (ici 192.168.4.150)
- Subnet Mask : 255.255.255.0 
ok => Next
Next
Ne rien faire pour le VPN => Cliquer ok sur la pop-up
Next
Cocher "Exempt VPN traffic from NAT", laisser les options par défaut (Nat inside)
Next
Next
Finish

## Configuration du user
Configuration > Remote Access VPN (en bas à gauche) > AAA/Local Users > Local Users > Cliquer sur cisco2
Edit
Sélection VPN policy
![[8_6.png]]
- Group Policy : Décocher inherit, sélectionner Profile Anyconnect 
- Connection Profile (Tunnel Group) Lock : Décocher Inherit et sélectionner Anyconnect Profile

Cliquer sur le user cisco => edit
Cliquer sur Identity (en haut à gauche)=> Change user password

## Tester le fonctionnement
Se rendre sur https://192.168.2.253 (adresse du firewall)
Sélectionner le profile Anyconnect, se login avec les creds de cisco2
Télécharger le client