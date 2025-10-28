# Cisco Packet Tracer Lab: Router-on-a-Stick (ROAS) Configuration

This README summarizes the configuration steps for implementing a **Router-on-a-Stick** topology on a single switch (`SW1`) and a router (`R1`) to enable Inter-VLAN Routing and dynamic routing using EIGRP.

## Topology Overview

<img width="756" height="621" alt="router-on-a-stick" src="https://github.com/user-attachments/assets/e51e69d3-23ce-419f-96be-e63128539515" />


The lab utilizes the Router-on-a-Stick principle to allow communication between three segmented VLANs using only one physical link (trunk) between the Switch and the Router9, 10. The router acts as the default gateway for each VLAN via logical sub-interfaces10.

| VLAN ID | VLAN Name | Network (CIDR) | Sub-Network ID | Default Gateway (R1 Sub-interface) |
| :---: | :---: | :---: | :---: | :---: |
| 10 | Sales | 192.168.0.0/25  | 192.168.0.0 51 | 192.168.0.1  |
| 20 | Admin | 192.168.0.128/25 | 192.168.0.128 51 | 192.168.0.129  |
| 30 | IT | 192.168.1.0/25  | 192.168.1.0 51 | 192.168.1.1 |

---

## Configuration Commands Summary

### 1. Switch Configuration (`SW1`)

The switch creates the VLANs, assigns access ports to them, and configures the uplink port (`Gig0/1`) to the router as a **trunk** to carry all VLAN traffic9, 20, 27.

| Command | Interface(s) | Purpose |
| :--- | :--- | :--- |
| `vlan 10` | Global | Creates VLAN 10 (Sales)  |
| `vlan 20` | Global | Creates VLAN 20 (Admin)  |
| `vlan 30` | Global | Creates VLAN 30 (IT)  |
| `int range fa0/1-2` | Interface Range | Selects ports for VLAN 10  |
| `switchport mode access` | Interface Config | Sets ports to **access** mode  |
| `switchport access vlan 10` | Interface Config | Assigns ports to VLAN 10  |
| `int gig0/1` | Interface Config | Selects the physical uplink to the router  |
| `switchport mode trunk` | Interface Config | Sets the link to **trunking** mode  |
| `switchport trunk allowed vlan 10,20,30` | Interface Config | **Crucial:** Allows VLANs 10, 20, and 30 to traverse the trunk link. |

### 2. Router Configuration (`R1`)

The router sets up the logical sub-interfaces (one per VLAN) on the physical uplink (`Gig0/0`) and configures EIGRP.

| Command | Interface(s) | Purpose |
| :--- | :--- | :--- |
| `int gig0/0` | Interface Config | Selects the physical uplink interface. |
| `no shutdown` | Interface Config | Activates the physical link. |
| `int gig0/0.10` | Sub-interface | Creates sub-interface for VLAN 10. |
| `encapsulation dot1q 10` | Sub-interface | Enables 802.1Q tagging for VLAN 10 traffic. |
| `ip add 192.168.0.1 255.255.255.128` | Sub-interface | Sets the default gateway for VLAN 10. |
| `int gig0/0.20` | Sub-interface | Creates sub-interface for VLAN 20. |
| `encapsulation dot1q 20` | Sub-interface | Enables 802.1Q tagging for VLAN 20 traffic. |
| `ip add 192.168.0.129 255.255.255.128` | Sub-interface | Sets the default gateway for VLAN . |
| `int gig0/0.30` | Sub-interface | Creates sub-interface for VLAN 30. |
| `encapsulation dot1q 30` | Sub-interface | Enables 802.1Q tagging for VLAN 30 traffic270. |
| `ip add 192.168.1.1 255.255.255.128` | Sub-interface | Sets the default gateway for VLAN . |
| `router eigrp 100` | Global | Starts the EIGRP routing process with AS **100**. |
| `no auto-summary` | Router Config | Prevents classful route summarization to ensure subnets are advertised correctly. |
| `network 192.168.0.0 0.0.0.127` | Router Config | Advertises the 192.168.0.0/25 network (VLAN 10). |
| `network 192.168.0.128 0.0.0.127` | Router Config | Advertises the 192.168.0.128/25 network (VLAN 20). |
| `network 192.168.1.0 0.0.0.127` | Router Config | Advertises the 192.168.1.0/25 network (VLAN 30). |

---

## Verification

After the configuration, Inter-VLAN communication was verified by successful Pings between hosts in different VLANs429, 431.

### Key Verification Commands

| Command | Purpose |
| :--- | :--- |
| `show vlan brief` (SW1) | Confirms ports are correctly assigned to VLANs (10, 20, 30). |
| `show int trunk` (SW1) | Confirms `Gig0/1` is trunking and allows VLANs 10, 20, and 30. |
| `show ip int brief` (R1) | Confirms R1's sub-interfaces are up/up and have the correct gateway IPs. |
| `show ip route` (R1) | Confirms the VLAN networks are installed as **Connected** routes (AD 0). |
| `show ip protocols` (R1) | Confirms EIGRP AS 100 is running and the networks are being advertised. |

---

