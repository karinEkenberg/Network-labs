# Cisco Packet Tracer Lab: Basic RIPv2 Configuration

This README documents the configuration of a simple Wide Area Network (WAN) topology using three routers and the **RIP version 2** routing protocol to ensure end-to-end connectivity.

## Topology Overview

<img width="852" height="586" alt="ripv2" src="https://github.com/user-attachments/assets/53738c65-982d-435c-8d20-c4039649d476" />

The lab consists of three routers (`R1`, `R2`, `R3`) connected sequentially, forming a chain. Each endpoint router (`R1` and `R3`) also connects to a local LAN segment represented by a PC.

| Component | IP Network | Interface | Gateway/Router IP |
| :--- | :--- | :--- | :--- |
| **R1 LAN** | 192.168.1.0/24 | Gig0/0/0 | 192.168.1.1 |
| **R1-R2 WAN** | 192.168.12.0/24 | Serial 0/0/0 | R1: 192.168.12.1 |
| **R2-R3 WAN** | 192.168.23.0/24 | Serial 0/0/0 & 0/0/1 | R2: 192.168.23.1 |
| **R3 LAN** | 192.168.3.0/24 | Gig0/0/0 | 192.168.3.1 |

> **Hardware Note:** Since the default 1941 routers lack serial ports, the **HWIC-2T** module was installed on each router to enable the necessary Serial DCE/DTE connections between them.

## Lab Objectives

1.  Successfully deploy and cable the three-router topology in Cisco Packet Tracer.
2.  Assign correct IP addresses and subnet masks to all router interfaces and end devices.
3.  Activate and configure **RIP version 2 (RIPv2)** on all three routers.
4.  Verify that all routing tables are dynamically populated.
5.  Confirm end-to-end network connectivity between the two PCs.

## Configuration Steps (Router CLI)

The following commands were used to configure the RIPv2 routing protocol on each router.

### Router R1 Configuration

```cli
R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# no auto-summary
R1(config-router)# network 192.168.1.0    // R1's LAN
R1(config-router)# network 192.168.12.0   // R1-R2 Link
R1(config-router)# end
```

### Router R2 Configuration (Transit Router)

```cli
R2(config)# router rip
R2(config-router)# version 2
R2(config-router)# no auto-summary
R2(config-router)# network 192.168.12.0   // R2-R1 Link
R2(config-router)# network 192.168.23.0   // R2-R3 Link
R2(config-router)# end
```

### Router R3 Configuration

```cli
R3(config)# router rip
R3(config-router)# version 2
R3(config-router)# no auto-summary
R3(config-router)# network 192.168.3.0    // R3's LAN
R3(config-router)# network 192.168.23.0   // R3-R2 Link
R3(config-router)# end
```

## Verification

Connectivity was verified on `R1` and `R3` after the RIP updates had been exchanged.

### 1\. Check Routing Protocol Status

```cli
R1# show ip protocols
```

> **Expected Output Snippet:** `Routing Protocol is "rip"`. `Sending/Receiving version 2`.

### 2\. Check Dynamic Routes

```cli
R1# show ip route rip
```

> **Expected Output:** The routing table on R1 should display the **192.168.23.0/24** and **192.168.3.0/24** networks, both marked with an **R** (RIP).

### 3\. End-to-End Ping

A final test was performed from the PC in R1's LAN (IP: `192.168.1.10`) to the PC in R3's LAN (IP: `192.168.3.10`).

```cli
C:\> ping 192.168.3.10
```

> **Result:** Successful replies, confirming that the RIPv2 configuration and all inter-router links are fully functional.

## R1 Configuration Summary (Commands and Purpose)

This table outlines every configuration step performed on **Router R1**, detailing the command used and its function within the network setup.

| Configuration Mode | Command | Purpose |
| :--- | :--- | :--- |
| **Privileged EXEC** | `enable` | Enters the Privileged Exec mode (from User Exec mode). |
| **Privileged EXEC** | `configure terminal` | Enters Global Configuration Mode to make system-wide changes. |
| **Global Config** | `hostname R1` | Assigns the router the hostname `R1`. |
| **Global Config** | `interface gigabitethernet 0/0/0` | Enters configuration mode for the Local Area Network (LAN) interface. |
| **Interface Config** | `ip address 192.168.1.1 255.255.255.0` | Assigns the IP address and subnet mask for the R1 LAN segment. |
| **Interface Config** | `no shutdown` | Activates the interface (brings the interface status to "up"). |
| **Interface Config** | `exit` | Returns to Global Configuration Mode. |
| **Global Config** | `interface serial 0/1/0` | Enters configuration mode for the Wide Area Network (WAN) interface connecting to R2. |
| **Interface Config** | `ip address 192.168.12.1 255.255.255.0` | Assigns the IP address and subnet mask for the R1-R2 link. |
| **Interface Config** | `clock rate 128000` | **(Only on the DCE side):** Sets the clock speed for synchronous communication (in this case, on R1). |
| **Interface Config** | `no shutdown` | Activates the WAN interface. |
| **Interface Config** | `exit` | Returns to Global Configuration Mode. |
| **Global Config** | `router rip` | Initiates the configuration process for the RIP routing protocol. |
| **Router Config** | `version 2` | Sets RIP to use version 2 (which supports Variable Length Subnet Masking - VLSM/CIDR). |
| **Router Config** | `no auto-summary` | Disables automatic network summarization, crucial for classless routing (RIPv2). |
| **Router Config** | `network 192.168.1.0` | Instructs RIP to advertise the network directly connected to R1's LAN interface. |
| **Router Config** | `network 192.168.12.0` | Instructs RIP to advertise the R1-R2 WAN link. |
| **Router Config** | `end` | Returns to Privileged Exec mode. |
| **Privileged EXEC** | `write memory` (or `copy running-config startup-config`) | Saves the current running configuration to NVRAM so it is persistent across reboots. |

---
