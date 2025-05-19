# Labo 3 - ZBPF

## Fonctionnement général

1. On définit des zones (INSIDE, OUTSIDE, DMZ, SELF, ...)
2. Assigner les zones aux interfaces: par défaut, le traffic dans une zone est allowed, et le traffic entre zones est denied.
3. Créer une `class-map` pour matcher le traffic
4. Créer une `policy-map` pour agir sur le traffic matché: inspect, drop, accept.
5. Appliquer la `policy-map` à une `zone-pair`

## Exemple avec FTP allowed pour 192.168.1.12

1. Create Zones

```
zone security INSIDE
zone security OUTSIDE
```

2. Assign Interfaces to Zones

```
interface GigabitEthernet0/0
 description Inside LAN
 ip address 192.168.1.1 255.255.255.0
 zone-member security INSIDE

interface GigabitEthernet0/1
 description Internet
 ip address <your.public.ip> <mask>
 zone-member security OUTSIDE
```

3. Create a Class Map for FTP FROM THAT HOST

```
class-map type inspect match-all ALLOW_FTP_HOST
 match protocol ftp
 match access-group name FTP_ONLY
```

We’re matching FTP and an access list. Let’s create that:

```
ip access-list extended FTP_ONLY
 permit tcp host 192.168.1.12 any eq 21
```

4. Create a Policy Map

```
policy-map type inspect POLICY_INSIDE_TO_OUTSIDE
 class type inspect ALLOW_FTP_HOST
  inspect
 class class-default
  drop
```

5. Create Zone Pair and Apply Policy

```
zone-pair security ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE
 service-policy type inspect POLICY_INSIDE_TO_OUTSIDE
```

## Remerciements

- ChatGPT pour ses loyaux services
- Et tous les maintainers de ce repository
