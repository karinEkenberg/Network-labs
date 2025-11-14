# Lab 3: Multi-Area OSPF Configuration

<img width="575" height="204" alt="multi-area" src="https://github.com/user-attachments/assets/25ffc9cf-85e4-4890-8317-a1d04932ae21" />

This project demonstrates the configuration of a multi-area OSPF (Open Shortest Path First) network. The primary goal is to segment a simple OSPF domain into two areas (Area 0 and Area 1) to improve scalability, reduce Link-State Database (LSDB) size, and manage routing updates more efficiently.

## Learning Objectives

- Configure two distinct OSPF areas: **Area 0** (the backbone area) and **Area 1**.
    
- Correctly assign router interfaces to their respective areas.
    
- Establish an **Area Border Router (ABR)** that connects the two areas.
    
- Verify inter-area routing and inspect the OSPF database.




## topology Network Topology

The lab consists of three Cisco routers (R1, R2, and R3) connected in a series. R2 serves as the Area Border Router (ABR), connecting Area 1 and Area 0.

```
     <---------- Area 1 ---------->     <---------- Area 0 ---------->

[ R1 ] ---- (G0/0/0) ---- (G0/0/0) [ R2 ] (G0/0/1) ---- (G0/0/0) [ R3 ]
                                   (ABR)
```

### IP Addressing Scheme

|**Device**|**Interface**|**IP Address**|**Subnet Mask**|**OSPF Area**|
|---|---|---|---|---|
|**R1**|`GigabitEthernet0/0/0`|`192.168.1.1`|`255.255.255.0`|1|
|**R2**|`GigabitEthernet0/0/0`|`192.168.1.2`|`255.255.255.0`|1|
|**R2**|`GigabitEthernet0/0/1`|`10.0.0.1`|`255.255.255.252`|0|
|**R3**|`GigabitEthernet0/0/0`|`10.0.0.2`|`255.255.255.252`|0|

---

## Configuration

Here is the final, correct configuration for each router.

### Router 1 (R1)

R1 is an internal router in Area 1.

Cisco CLI

```
! Configure Hostname
hostname R1

! Configure Interface
interface GigabitEthernet0/0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit

! Configure OSPF
router ospf 1
 network 192.168.1.0 0.0.0.255 area 1
```

### Router 2 (R2 - The ABR)

R2 is the Area Border Router, with one interface in Area 1 and another in Area 0.

Cisco CLI

```
! Configure Hostname
hostname R2

! Configure Interface for Area 1
interface GigabitEthernet0/0/0
 ip address 192.168.1.2 255.255.255.0
 no shutdown
 exit

! Configure Interface for Area 0
interface GigabitEthernet0/0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown
 exit

! Configure OSPF
router ospf 1
 network 192.168.1.0 0.0.0.255 area 1
 network 10.0.0.0 0.0.0.3 area 0
```

### Router 3 (R3)

R3 is an internal router in Area 0.

Cisco CLI

```
! Configure Hostname
hostname R3

! Configure Interface
interface GigabitEthernet0/0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
 exit

! Configure OSPF
router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
```

---

## Verification

After configuration, OSPF adjacencies should form. The console logs should display messages similar to these, indicating neighbors have reached the **FULL** state:

```
%OSPF-5-ADJCHG: Process 1, Nbr 192.168.1.1 on GigabitEthernet0/0/0 from LOADING to FULL, Loading Done
%OSPF-5-ADJCHG: Process 1, Nbr 10.0.0.2 on GigabitEthernet0/0/1 from LOADING to FULL, Loading Done
```

### Verification Commands (on R2)

These commands were used to verify the multi-area configuration.

#### 1. `show ip ospf database`

This command is crucial on the ABR. The output confirms that R2 is maintaining a separate LSDB for both Area 0 and Area 1.

```
R2# sh ip ospf database

            OSPF Router with ID (192.168.1.2) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
192.168.1.2     192.168.1.2     34          0x80000001 0x002a49 1
...

                Router Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Link count
192.168.1.2     192.168.1.2     37          0x80000002 0x009f6f 1
...
```

#### 2. `show ip ospf border-routers`

This command shows the routes to other ABRs and ASBRs. In this simple topology, it shows the path to the other router in Area 0.

```
R2# sh ip ospf border-routers
OSPF Process 1 internal Routing Table

Codes: i - Intra-area route, I - Inter-area route

i 10.0.0.2 [1] via 10.0.0.2, GigabitEthernet0/0/1, ABR, Area 0, SPF 1
```

#### 3. `debug ip ospf events`

This debug command shows real-time OSPF events, such as Hello packets being received from neighbors in different areas.

```
R2# debug ip ospf events
OSPF events debugging is on
...
00:16:28: OSPF: Rcv hello from 192.168.1.1 area 1 from GigabitEthernet0/0/0 192.168.1.1
...
00:16:30: OSPF: Rcv hello from 10.0.0.2 area 0 from GigabitEthernet0/0/1 10.0.0.2
...
```

## Key Concepts

> OSPF areas are used to divide a large autonomous system into smaller, more manageable domains. This reduces the size of the Link-State Database (LSDB) on each router and minimizes the overhead of SPF (Shortest Path First) calculations. The **Area Border Router (ABR)**, R2 in this lab, connects different areas and is responsible for summarizing routes. All inter-area communication must pass through the backbone, **Area 0**.
