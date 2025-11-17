
# Cisco Troubleshooting Lab: Inter-VLAN Communication Failure
<img width="724" height="350" alt="vlan-trunking-issues" src="https://github.com/user-attachments/assets/63e2966a-ad1d-447a-9293-6379f01e8894" />

This repository documents the troubleshooting process for a network lab scenario. The initial task was to solve a communication failure between two PCs on different VLANs.

This analysis is broken into two parts:
1.  The Intended Lab Solution (Layer 2): Identifying and fixing the specific configuration error as intended by the lab.
2.  Further Analysis (Layer 3): A deeper discovery of why the lab's scenario is fundamentally flawed given the specified hardware, and what a correct real-world solution would be.

## Lab Overview

* Topology: `[PC1]` --- `[SW1]` ===(Trunk)=== `[SW2]` --- `[PC2]`
* Hardware: 2x Cisco 2960 Switches (Layer 2)
* PC1 (VLAN 10):
    * IP: `192.168.10.10/24`
    * Gateway: `192.168.10.1` (SW1's SVI)
* PC2 (VLAN 20):
    * IP: `192.168.20.10/24`
    * Gateway: `192.168.20.1` (SW1's SVI)
* Problem: `PC1` cannot ping `PC2`.

---

## Part 1: The Intended L2 Troubleshooting

This section follows the expected troubleshooting path of the lab.

### 1.1. Problem Verification
A ping from PC1 to PC2 fails.


PC1\> ping 192.168.20.10
Request timed out.


This confirms a lack of connectivity.

### 1.2. Investigation
Since the PCs are on different switches, the trunk link is the primary suspect for inter-VLAN issues. I checked the trunk status on SW1.


SW1\# show interfaces trunk

Port        Mode         Encapsulation  Status        Native vlan
Gi0/1       on           802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/1       10

Port        Vlans allowed and active in management domain
Gi0/1       10
...


### 1.3. L2 Root Cause
The output immediately revealed the problem:
The trunk port `Gi0/1` on SW1 was only configured to allow VLAN 10. All traffic for VLAN 20 was being blocked at this port, preventing it from ever reaching SW2.

### 1.4. L2 Fix
The solution was to add VLAN 20 to the allowed list on the trunk.



SW1(config)\# interface GigabitEthernet0/1
SW1(config-if)\# switchport trunk allowed vlan add 20

### 1.5. L2 Verification
Running the command again confirmed the fix:


SW1\# show interfaces trunk

Port        Vlans allowed on trunk
Gi0/1       10,20  \<-- OK

At this point, the Layer 2 portion of the lab objective was complete.
---

## Part 2: Deeper Analysis & The Real L3 Problem

Despite applying the "fix," I noted that the ping would *still* fail in this environment. The lab's design itself was flawed.

### 2.1. The Contradiction
The lab guide instructed us to use Cisco 2960 switches, which are Layer 2 devices. However, the configuration also specified:
1.  Setting the PCs' default gateways to the switch SVI addresses (`192.168.10.1` and `192.168.20.1`).
2.  Using the `ip routing` command on the switches to handle inter-VLAN routing.

### 2.2. The L3 Discovery
When I attempted to apply the configuration, the switch returned a clear error:

```

SW1(config)\# ip routing
^
% Invalid input detected at '^' marker.

```
This is the core problem. A Cisco 2960 switch with the standard LANBASEK9 image does not support Layer 3 routing. It is not a router, and the `ip routing` command does not exist.

### 2.3. L3 Root Cause
For a PC in VLAN 10 to talk to a PC in VLAN 20, the traffic must be routed between the `192.168.10.0/24` and `192.168.20.0/24` subnets.

The lab's design relies on the switch to be the router, which it cannot be. Therefore, even with the L2 trunk fixed, PC1's ping would fail because its default gateway (`192.168.10.1`) has no capability to route the packet to the `192.168.20.0/24` network.

## Correct Real-World Solutions

To *actually* solve this problem and enable inter-VLAN communication, one of two design changes would be required:

### Solution A: Router-on-a-Stick (ROAS)
* How: Keep the 2960 switches. Add a dedicated router.
* Action:
    1.  Configure a port on SW1 as a trunk (e.g., `Gi0/2`) and connect it to a router interface (e.g., `G0/0`).
    2.  On the router, create sub-interfaces for each VLAN (`G0/0.10` and `G0/0.20`).
    3.  Assign the gateway IPs (`192.168.10.1` and `192.168.20.1`) to these sub-interfaces.
    4.  The router, which *can* route, will handle all inter-VLAN traffic.

### Solution B: Layer 3 Switch
* How: Replace the 2960 (L2) switches with Layer 3 switches (e.g., Cisco 3560, 3750, or modern 9300 series).
* Action:
    1.  The lab configuration would now work as intended.
    2.  The command `ip routing` would be accepted by the switch.
    3.  The switch's SVIs (`interface Vlan10` and `interface Vlan20`) would function as true gateways, and the switch would route traffic between them internally at high speed.

## Key Takeaways
* Always verify Layer 2 connectivity (like trunks) first.
* A problem can have multiple "root causes" at different OSI layers.
* Crucially, know your hardware's capabilities. The inability of a Layer 2 switch to perform Layer 3 routing was the true, unresolvable issue in this lab scenario.
