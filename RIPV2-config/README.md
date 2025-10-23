# RIPv2 Basic Lab

This lab demonstrates dynamic routing using RIPv2 between three Cisco routers in Packet Tracer.

## Topology
[R1]──192.168.12.0/24──[R2]──192.168.23.0/24──[R3]
| |
LAN: 192.168.1.0/24 LAN: 192.168.3.0/24

<img width="565" height="178" alt="lab" src="https://github.com/user-attachments/assets/b4df338c-8227-400c-b948-d0bc9b00a704" />

## Objective
- Configure RIPv2 on all routers  
- Verify that routes are learned dynamically  
- Ensure version 2 is used (not RIPv1)

##  IP Addressing Plan
| Link / Network | R1 | R2 | R3 |
|----------------|----|----|----|
| R1–R2 | 192.168.12.1 | 192.168.12.2 | — |
| R2–R3 | — | 192.168.23.2 | 192.168.23.3 |
| R1 LAN | 192.168.1.1 | — | — |
| R3 LAN | — | — | 192.168.3.1 |

Subnet mask: 255.255.255.0 (/24)

##  Configuration Example
R1(config)# router rip

R1(config-router) version 2

R1(config-router) no auto-summary

R1(config-router) network 192.168.1.0

R1(config-router) network 192.168.12.0

(Repeat similar config on R2 and R3)

## Verification
- `show ip route` → routes learned via RIP (marked with “R”)  
- `show ip protocols` → confirms RIPv2 running  
- `ping 192.168.3.1` → end-to-end connectivity test

<img width="695" height="706" alt="cli" src="https://github.com/user-attachments/assets/8c45c242-a74d-4963-83ee-bdde873091f1" />


### Successful Connectivity
- **R1 → R2 (192.168.23.2)**: 100% ping success
- **R1 → R3 (192.168.23.3)**: 80% ping success (4/5 packets)

### Remaining Issue
- **R1 → R3 LAN (192.168.3.1)**: 0% ping success
- **R1 → R2-R3 link (192.168.12.3)**: 0% ping success

##  Analysis & Findings

### What's Working
- **Direct link routing** between R1-R2 and R2-R3 is functional
- **RIPv2 is partially operational** - some routes are being learned
- **Local interface communication** is successful

### Identified Issues
Based on the connectivity pattern, I suspect:
1. **Missing network advertisements** in RIPv2
2. **Incomplete RIP configuration** on R2 or R3
3. **Interface or routing problems** on R3's LAN interface

##  Troubleshooting Steps Performed

Här är en strukturerad command table med förklaringar av alla dina kommandon:

##  Command Reference Table

| Command | Mode | Description | Purpose |
|---------|------|-------------|---------|
| **`en`** | User → Privileged | Enable | Enter privileged EXEC mode for configuration changes |
| **`conf t`** | Privileged → Global Config | Configure Terminal | Enter global configuration mode |
| **`host R1`** | Global Config | Hostname | Set router name to R1 for identification |
| **`int g0/0`** | Global Config → Interface | Interface GigabitEthernet0/0 | Enter interface configuration mode for G0/0 |
| **`ip add 192.168.1.1 255.255.255.0`** | Interface Config | IP Address | Assign IP address 192.168.1.1/24 to interface |
| **`no shut`** | Interface Config | No Shutdown | Activate the interface (bring it up) |
| **`ip add 192.168.12.1 255.255.255.0`** | Interface Config | IP Address | **⚠️ ERROR**: Second IP on same interface (should be different interface) |
| **`int g0/1`** | Global Config → Interface | Interface GigabitEthernet0/1 | Enter interface configuration for G0/1 |
| **`ip add 192.168.1.1 255.255.255.0`** | Interface Config | IP Address | **⚠️ ERROR**: Duplicate IP address (same as G0/0) |
| **`rout rip`** | Global Config → RIP | Router RIP | Enable RIP routing protocol |
| **`vers 2`** | RIP Config | Version 2 | Configure RIPv2 (supports classless routing) |
| **`no auto-summary`** | RIP Config | No Auto-summary | Disable automatic network summarization |
| **`net 192.168.1.0`** | RIP Config | Network | Advertise network 192.168.1.0 in RIP |
| **`net 192.168.12.0`** | RIP Config | Network | Advertise network 192.168.12.0 in RIP |
| **`end`** | Any → Privileged | End Configuration | Exit configuration mode and return to privileged EXEC |
| **`show ip protocols`** | Privileged EXEC | Show IP Protocols | Display active routing protocol information |
| **`sh ip route rip`** | Privileged EXEC | Show IP Route RIP | Display only RIP-learned routes from routing table |
| **`ping 192.168.3.1`** | Privileged EXEC | Ping | Test connectivity to destination IP |
| **`ping 192.168.12.3`** | Privileged EXEC | Ping | Test connectivity to R2-R3 link interface |
| **`ping 192.168.23.3`** | Privileged EXEC | Ping | Test connectivity to R3's interface on R2-R3 link |
| **`ping 192.168.23.2`** | Privileged EXEC | Ping | Test connectivity to R2's interface on R2-R3 link |

```bash
# Verified RIP learned routes
R1# show ip route rip
# Output showed only partial routing table

# Checked RIP configuration
R1# show ip protocols
# Confirmed RIPv2 active but potentially missing networks

---

_All configurations are for educational use only._

