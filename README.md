# Multi-VLAN Office Network with Inter-VLAN Routing, Port Security & ACLs

A simulated small-office network built in Cisco Packet Tracer, demonstrating VLAN segmentation across two switches, inter-VLAN routing using Router-on-a-Stick, port security, and traffic filtering with an extended ACL.

## Project Overview

This project simulates a small company with four departments — **IT, Sales, Finance, and Servers** — each isolated on its own VLAN for security and traffic management. A single router provides inter-VLAN routing so departments can communicate where required, while an extended ACL enforces a specific business rule: **Sales must not be able to reach the Servers VLAN**, while IT and Finance can.

## Network Topology

![Network Topology] 

- **Router0** connects to **Switch0** via a single trunk link (Router-on-a-Stick)
- **Switch0** and **Switch1** are connected via a trunk link, extending all VLANs across both switches
- **IT** and **Sales** PCs connect to **Switch0**
- **Finance** and **Servers** PCs connect to **Switch1**

## IP Addressing Plan

Base network `192.168.100.0/24` subnetted into four `/26` blocks:

| VLAN | Department | Network | Usable Range | Broadcast | Gateway (Router) |
|------|-----------|---------|---------------|-----------|-------------------|
| 10 | IT | 192.168.100.0/26 | .1 – .62 | .63 | 192.168.100.1 |
| 20 | Sales | 192.168.100.64/26 | .65 – .126 | .127 | 192.168.100.65 |
| 30 | Finance | 192.168.100.128/26 | .129 – .190 | .191 | 192.168.100.129 |
| 40 | Servers | 192.168.100.192/26 | .193 – .254 | .255 | 192.168.100.193 |

## Device Configuration

### Switch0 — VLANs & Trunking
![Switch0 VLAN and Trunk]

```
vlan 10
 name IT
vlan 20
 name Sales
vlan 30
 name Finance
vlan 40
 name Servers

interface fa0/1
 switchport mode access
 switchport access vlan 10

interface fa0/2
 switchport mode access
 switchport access vlan 20

interface fa0/3
 switchport mode trunk
 switchport trunk allowed vlan 1,10,20,30,40

interface fa0/4
 switchport mode trunk
 switchport trunk allowed vlan 1,10,20,30,40
```

### Switch1 — VLANs & Trunking
![Switch1 VLAN and Trunk]

```
vlan 10
 name IT
vlan 20
 name Sales
vlan 30
 name Finance
vlan 40
 name Servers

interface fa0/1
 switchport mode access
 switchport access vlan 30

interface fa0/2
 switchport mode access
 switchport access vlan 40

interface fa0/4
 switchport mode trunk
 switchport trunk allowed vlan 1,10,20,30,40
```

### Router0 — Router-on-a-Stick (Sub-interfaces)
![Router0 Sub-interfaces]

```
interface gi0/0.10
 encapsulation dot1Q 10
 ip address 192.168.100.1 255.255.255.192

interface gi0/0.20
 encapsulation dot1Q 20
 ip address 192.168.100.65 255.255.255.192

interface gi0/0.30
 encapsulation dot1Q 30
 ip address 192.168.100.129 255.255.255.192

interface gi0/0.40
 encapsulation dot1Q 40
 ip address 192.168.100.193 255.255.255.192
```

### Port Security (Switch0 & Switch1)
![Switch0 Port Security]
![Switch1 Port Security]

```
interface fa0/1
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
```

Restricts each access port to one learned MAC address; any additional or different device triggers a shutdown violation, protecting against rogue devices.

### Extended ACL — Business Rule Enforcement

**Requirement:** Sales (VLAN 20) must not reach the Servers VLAN (VLAN 40). IT and Finance must retain access.

```
access-list 100 deny ip 192.168.100.64 0.0.0.63 192.168.100.192 0.0.0.63
access-list 100 permit ip any any

interface gi0/0.20
 ip access-group 100 in
```

Applied inbound on the Sales sub-interface only — traffic from IT and Finance is never inspected by this ACL and passes through normally.

## Verification & Testing

| Test | Source | Destination | Result |
|------|--------|-------------|--------|
| Inter-VLAN routing | PC0 (IT, VLAN 10) | PC2 (Finance, VLAN 30) | ✅ Success — 0% loss |
| Inter-VLAN routing | PC0 (IT, VLAN 10) | PC3 (Servers, VLAN 40) | ✅ Success — 0% loss |
| Inter-VLAN routing | PC2 (Finance, VLAN 30) | PC3 (Servers, VLAN 40) | ✅ Success — 0% loss |
| ACL enforcement | PC1 (Sales, VLAN 20) | PC3 (Servers, VLAN 40) | ❌ Blocked — Destination host unreachable (as designed) |

![IT to Finance]
![IT to Servers]
![Finance to Servers]
![Sales to Servers - Blocked by ACL])

The blocked test returns a reply from `192.168.100.65` (the Sales gateway on Router0) stating "Destination host unreachable" — confirming the router itself is enforcing the ACL rule rather than the packet simply timing out.

## Key Concepts Demonstrated

- VLAN segmentation across multiple switches using 802.1Q trunking
- Inter-VLAN routing via Router-on-a-Stick (sub-interfaces)
- Subnetting a /24 network into four equal /26 subnets (VLSM principles)
- Port security with sticky MAC learning and violation handling
- Extended ACLs for granular traffic control based on source and destination
- Structured verification and documentation of network changes

## Tools Used

- Cisco Packet Tracer
- Cisco IOS CLI

## Author

Faraz — IT Engineer (Networking, Systems & Cloud) in training, preparing for IT Infrastructure roles.
