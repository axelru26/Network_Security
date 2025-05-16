# Labo 4 - 802.1X Wireless

## Topologie

![](Images/Lab5_Topology.png)

---
## Configuration du switch L3

- Configurer un adresse et un pool DHCP sur le VLAN 1 pour PC1
- Configurer un trunk entre le switch et l'AP avec 3 VLANs:  
	- Un VLAN natif pour la communication CAPWAP entre le vWLC et l'AP
	- Deux autres pour différents groupes de clients
- Configurer des pools DHCP pour ces trois VLANs
	- Le pool DHCP du VLAN natif a besoin d'une option spéciale pour que l'AP puisse contacter son controlleur: `option 43 hex f104c0a802cc`

> [!NOTE]
> les 8 derniers chiffres hexadécimaux correspondent à l'adresse IP du vWLC

## Configuration du vWLC

- *Configuration > Static Routing*: Ajouter un route statique vers le subnet de l'AP, avec l'IP du switch comme next hop.
- *Monitoring > Wireless > AP Statistics*: Vérifier que l'AP est bien en état registered

### Sans AAA

### Avec AAA

## Configuration de l'ISE

## Connection au réseau
