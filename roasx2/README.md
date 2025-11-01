
# Cisco Network Lab - Inter-VLAN Routing with RIPv2 over Serial WAN

## Overview
This repository contains a fully functional Cisco packet tracer lab demonstrating inter-VLAN routing,
 RIPv2 dynamic routing,serial WAN connectivity (DCE/DTE)**, **subinterfaces**, **trunking**,
loopback interfaces, and cross-switch VLAN communication.

All devices are configured and tested. VLANs from different switches can communicate with each
other and with loopback interfaces. The WAN link between routers is up and routing via RIPv2.

---

## Topology Summary

<img width="1126" height="617" alt="Skärmklipp" src="https://github.com/user-attachments/assets/7d14d12d-fa7c-4333-a223-1ccd91aadd1f" />


---

## Key Features Implemented

| Feature               | Implementation |
|-----------------------|----------------|
| **Subinterfaces**     | `g0/0.10`, `g0/0.20` on R1<br>`g0/0.30`, `g0/0.40` on R2 |
| **802.1Q Trunking**   | Between R1 ↔ SW-Fa0/2 and R2 ↔ SW-Fa0/2 |
| **Inter-VLAN Routing**| Router-on-a-Stick on both routers |
| **RIPv2**             | Enabled on all routers with `no auto-summary` |
| **Serial WAN (DCE/DTE)** | R1 provides clocking (`clock rate 64000`) |
| **Loopback Interfaces**| Advertised in RIPv2 |
| **Cross-Site VLAN Comms** | VLAN 10 ↔ VLAN 30, VLAN 20 ↔ VLAN 40, etc. |
| **Reachability**      | All PCs can ping loopbacks and remote VLANs |

---

## Configuration Highlights

### Router R1 (Left)
```ios
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
!
interface Serial0/0/0
 ip address 10.10.10.1 255.255.255.252
 clock rate 64000
!
interface Loopback1
 ip address 1.1.1.1 255.255.255.255
!
router rip
 version 2
 network 1.0.0.0
 network 10.0.0.0
 network 192.168.10.0
 network 192.168.20.0
 no auto-summary
```

### Router R2 (Right)
```ios
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
!
interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
!
interface Serial0/0/0
 ip address 10.10.10.2 255.255.255.252
!
interface Loopback2
 ip address 2.2.2.2 255.255.255.255
!
router rip
 version 2
 network 2.0.0.0
 network 10.0.0.0
 network 192.168.30.0
 network 192.168.40.0
 no auto-summary
```

### Switch (Trunk Ports)
```ios
interface FastEthernet0/1
 switchport mode trunk
!
interface range FastEthernet0/2 - 3
 switchport mode access
 switchport access vlan X
```

---

## Verification Commands

```bash
# On any PC
ping 2.2.2.2          # Reach remote loopback
ping 192.168.40.10    # Reach remote VLAN

# On routers
show ip route
show ip rip database
show interfaces trunk
show vlan brief
```

All pings succeed. Routing table shows RIPv2 learned routes.

---

## Requirements

- Cisco Packet Tracer 7.3+
- Basic understanding of VLANs, routing, and serial links

---

## How to Use

1. Open `lab-topology.pkt` in Packet Tracer
2. Power on all devices
3. Test connectivity:
   - PC in VLAN 10 → ping PC in VLAN 30
   - Any PC → ping `1.1.1.1` and `2.2.2.2`
4. Check routing: `show ip route` on R1/R2

---

## Learning Outcomes

- Configure **router-on-a-stick** with subinterfaces
- Set up **DCE/DTE serial links** with clocking
- Enable **RIPv2** with proper network statements
- Use **loopback interfaces** for testing
- Achieve **full inter-site VLAN communication**

---

## Author
Karin Ekenberg – Networking Student / Lab Practice

---
*Everything works as expected. Ready for CCNA-level review.*


