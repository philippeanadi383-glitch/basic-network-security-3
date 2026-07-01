# 🔐 VLAN Isolation + ACL Inter-VLAN — Cisco Packet Tracer

## Objectif
Séparer deux départements (RH et IT) sur le même switch physique en utilisant des VLANs, puis bloquer la communication de RH vers IT avec une ACL étendue.

## Outils
- Cisco Packet Tracer

## Topologie
```
[PC-RH]  192.168.10.10 ──┐
[PC-RH2] 192.168.10.11 ──┤── Switch (2960) ── Routeur (ISR4331)
[PC-IT]  192.168.20.10 ──┤
[PC-IT2] 192.168.20.11 ──┘
```

## Table d'adressage

| Appareil | Interface | IP | Masque | Gateway | VLAN |
|---|---|---|---|---|---|
| PC-RH | Fa0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 | 10 |
| PC-RH2 | Fa0 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 | 10 |
| PC-IT | Fa0 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 | 20 |
| PC-IT2 | Fa0 | 192.168.20.11 | 255.255.255.0 | 192.168.20.1 | 20 |
| Routeur | g0/0/0.10 | 192.168.10.1 | 255.255.255.0 | — | 10 |
| Routeur | g0/0/0.20 | 192.168.20.1 | 255.255.255.0 | — | 20 |

## Configuration

### Switch — Création des VLANs
```
vlan 10
 name RH
vlan 20
 name IT

interface fa0/1
 switchport mode access
 switchport access vlan 10
interface fa0/2
 switchport mode access
 switchport access vlan 10
interface fa0/3
 switchport mode access
 switchport access vlan 20
interface fa0/4
 switchport mode access
 switchport access vlan 20
interface fa0/5
 switchport mode trunk
```

### Routeur — Sous-interfaces
```
interface g0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface g0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

### ACL — Bloquer RH vers IT
```
access-list 110 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
access-list 110 permit ip any any

interface g0/0/0.10
 ip access-group 110 in
```

## Résultats des tests

| Test | Résultat |
|---|---|
| PC-RH → PC-RH2 (même VLAN) | ✅ Autorisé |
| PC-RH → PC-IT (inter-VLAN) | ❌ Bloqué par ACL |
| PC-IT → PC-RH (sens inverse) | ✅ Autorisé |

## Concepts démontrés
- Création et assignation de VLANs sur un switch Cisco
- Trunk link entre switch et routeur
- Router-on-a-Stick (routage inter-VLAN sur sous-interfaces)
- ACL étendue unidirectionnelle
- Principe du moindre privilège
