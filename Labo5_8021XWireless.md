# Labo 4 - 802.1X Wireless

## Topologie

![](Images/Lab5_Topology.png)

---
## Configuration du switch L3

- Configurer un adresse et un pool DHCP sur le VLAN 1 pour PC1
```
ena
conf t

ip routing

! Vlan 1 configuration
interface fastEthernet1/0/1
no switchport
ip address 192.168.2.254 255.255.255.0
no shutdown
exit

! DHCP pool
ip dhcp pool vlan1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.254

! VLAN 1
interface vlan1
 ip address 192.168.1.254 255.255.255.0
 no shutdown

! Port 48 in VLAN 1
interface fastethernet1/0/48
 switchport mode access
 switchport access vlan 1
 no shutdown
```

- Configurer un trunk entre le switch et l'AP avec 3 VLANs:  

  - Un VLAN natif pour la communication CAPWAP entre le vWLC et l'AP
```
! Vlan 3 configuration
interface vlan 3
ip address 192.168.3.254 255.255.255.0
no shutdown
exit

! Trunk between Switch and AP
interface fast 1/0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 1,3,20,21
switchport trunk native vlan 3
no shutdown
exit
```

  - Deux autres pour différents groupes de clients
```
interface vlan 20
ip address 192.168.20.254 255.255.255.0
no shutdown
exit

interface vlan 21
ip address 192.168.21.254 255.255.255.0
no shutdown
exit
```

- Configurer des pools DHCP pour ces trois VLANs
```
ip dhcp pool VLAN20
network 192.168.20.0 255.255.255.0
default-router 192.168.20.254
exit

ip dhcp pool VLAN21
network 192.168.21.0 255.255.255.0
default-router 192.168.21.254
exit
```

- Le pool DHCP du VLAN natif a besoin d'une option spéciale pour que l'AP puisse contacter son controlleur: `option 43 hex f104c0a802cc`
```
! DHCP Pool between AP and cWLC
ip dhcp pool VLAN_VWLC
network 192.168.3.0 255.255.255.0
default-router 192.168.3.254
option 43 hex f104c0a802CC
exit
```

> [!NOTE]
> les 8 derniers chiffres hexadécimaux correspondent à l'adresse IP du vWLC

## Configuration du vWLC

- *Configuration > Static Routing*: Ajouter un route statique vers le subnet de l'AP, avec l'IP du switch comme next hop.
- *Monitoring > Wireless > AP Statistics*: Vérifier que l'AP est bien en état registered

### Sans AAA

- *Configuration > Tags & Profiles > Tags > Site*: Dans le *default-site-tag*, décocher "Local Site"
- *Configuration > Wireless > Access-pointS*: Vérifier que l'AP est bien en mode "Flex"
- *Configuration > Tags & Profiles > Flex > default-flex-profile*: Spécifier le VLAN natif
- Dans le submenu VLAN: Spécifier les VLANs utilisateurs.

- *Configuration > Wireless Setup > Advanced*: Cliquer sur "Start Now". On configurera les trois premières tuiles (WLAN Profile, Policy Profile, Policy Tag)
- *WLAN Profile*: Spécifier un SSID, passer le statut en "Enabled".
- Dans l'onglet "Security", on le met à None pour le moment.
- *Policy Profile*: Choisir un nom, Status enabled, Central DHCP et Switching disabled.
- Dans l'onglet "Access Policies", spécifier l'ID d'un VLAN utilisateur.

> [!WARNING]
> C'est normal que les VLANs n'apparaissent pas dans le menu déroulant. C'est suffisant de juste écrire l'ID du VLAN dans la zone texte.

- *Policy Tag*: Lier les deux profils précédemment créés dans une nouvelle "WLAN-POLICY Map"
- Apply

- *Monitoring > Wireless > AP Statistics*: Vérifier que tout s'est bien appliqué en cliquant sur le petit icone bleu à côté du nom de l'AP

On peut maintenant se connecter au SSID configuré.
 
### Sécurisation avec AAA

- *Configuration > Security > AAA > Server/Groups*: Ajouter le serveur ISE, et l'ajouter dans un server group
- *Configuration > Security > AAA > AAA Method List*: Dans authentication, ajouter un entrée avec dot1x et sélectionner le groupe créé précedemment
- Dans authorization, ajouter une entrée avec Type = network et notre groupe

- On peut maintenant modifier la sécurité dans Wireless Setup
- *WLAN Profile*: Bien cocher AES & 802.1X:  
![](Images/Lab5_WLANProfileSecured.png)
- Dans l'onglet AAA, sélectionner la liste d'authentification crée précedemment
- *Policy Profile > Advanced*: Allow AAA Override 

## Configuration de l'ISE

- Ajouter le vWLC dans les network devices
- Idem que pour le labo wired, sauf que `NAS-Port-Type EQUALS Wireless-IEEE802.11`
