# DHCP Relay Configuration Lab

<img width="704" height="465" alt="dchp-relay-problem" src="https://github.com/user-attachments/assets/bd70cf9c-c854-4992-84b8-d8aea2a75c8d" />


## Overview
This lab demonstrates a common network segmentation issue where DHCP broadcasts fail to cross VLAN boundaries. 
The objective was to configure a **DHCP Relay Agent** to forward requests from clients in VLAN 10 and VLAN 20 to a 
centralized DHCP server located in VLAN 99.

## Technologies
* **Cisco IOS (Layer 3 Switch & Router)**
* **Inter-VLAN Routing**
* **DHCP & DHCP Relay**

## The Problem
* **Scenario:** PCs in VLAN 10 and 20 requested IP addresses via broadcast (DHCP Discover).
* **Constraint:** Routers and Layer 3 switches do not forward broadcasts by default. The DHCP server in VLAN 99 never received the requests.
* **Symptom:** Clients received APIPA addresses (`169.254.x.x`).

## The Solution
We configured the **Switch Virtual Interfaces (SVIs)** acting as gateways for the client VLANs to forward UDP broadcasts to the 
specific IP address of the DHCP server.

### Key Configuration (Layer 3 Switch)
The `ip helper-address` command was applied to the client-facing interfaces:

```cisco
interface vlan 10
 description Client Gateway
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.99.5  <-- Forwards request to DHCP Server
!
interface vlan 20
 description Client Gateway
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.99.5

Verification

    Before Fix: PCs self-assigned 169.254.x.x addresses.

    After Fix: PCs successfully obtained IP addresses in the 192.168.10.x and 192.168.20.x ranges.

    Server Check: show ip dhcp binding on the router confirmed active leases.
