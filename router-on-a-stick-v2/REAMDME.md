# README: Router on a Stick (Inter-VLAN Routing) Lab

## Lab Objective

The goal of this lab is to implement **Inter-VLAN Routing** using the **"Router on a Stick" (RoaS)** method. This configuration allows devices in different **VLANs (Virtual Local Area Networks)**, which are all connected to the same switch, to communicate with each other via a single physical link to a central router.

## Equipment and Topology

<img width="645" height="654" alt="router-on-a-stick" src="https://github.com/user-attachments/assets/b379b63d-c298-460b-b4e4-bd23d2e7df40" />


This lab utilizes industry-standard Cisco equipment:

| Device | Model | Role | Connection |
| :--- | :--- | :--- | :--- |
| **Router (R1)** | Cisco 1941 ISR | Routing between VLANs (Gateway) | Gi0/0 |
| **Switch (SW1)** | Cisco 2960 Catalyst | VLAN Segmentation and Trunking | Gi0/1 (Trunk) |
| **Client 1 (Laptop1)** | Laptop/End Device | Belongs to **VLAN 10** | Fa0/1–12 (Access) |
| **Client 2 (Laptop2)** | Laptop/End Device | Belongs to **VLAN 20** | Fa0/13–24 (Access) |

### **Cabling**

  * **Router (R1) Gi0/0** $\rightarrow$ **Switch (SW1) Gi0/1**
      * **Cable Type:** Straight-Through Ethernet Cable
      * **Purpose:** To create a single **Trunk link** that carries traffic for all defined VLANs.

## Configuration Summary

### **Addressing Scheme**

| VLAN ID | Name | Network | Router Subinterface (Gateway) | Client IP Example |
| :--- | :--- | :--- | :--- | :--- |
| **10** | Sales | 192.168.10.0/24 | **192.168.10.1** | 192.168.10.10 |
| **20** | HR | 192.168.20.0/24 | **192.168.20.1** | 192.168.20.20 |

-----

## 1. Switch Configuration (SW1)

The goal is to create the VLANs, configure the trunk link to the router, and assign the access ports for the clients.

```cisco
SW1>en
SW1#conf t
SW1(config)#hostname SW1

! 1. Create VLANs
SW1(config)#vlan 10
SW1(config-vlan)#name Sales
SW1(config-vlan)#vlan 20
SW1(config-vlan)#name HR
SW1(config-vlan)#exit

! 2. Configure Trunk Port (connecting to R1 Gi0/0)
SW1(config)#int gigabitEthernet 0/1
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport trunk allowed vlan 10,20
SW1(config-if)#no shutdown
SW1(config-if)#exit

! 3. Configure Access Ports (for client devices)
! VLAN 10 (Sales)
SW1(config)#int range fastEthernet 0/1 - 12
SW1(config-if-range)#switchport mode access
SW1(config-if-range)#switchport access vlan 10
SW1(config-if-range)#no shutdown
SW1(config-if-range)#exit

! VLAN 20 (HR)
SW1(config)#int range fastEthernet 0/13 - 24
SW1(config-if-range)#switchport mode access
SW1(config-if-range)#switchport access vlan 20
SW1(config-if-range)#no shutdown
SW1(config-if-range)#exit

SW1(config)#end
SW1#copy run start
```

-----

## 2. Router Configuration (R1)

The goal is to enable the physical interface and create logical subinterfaces for each VLAN, defining the Default Gateway for each network.

```cisco
R1>en
R1#conf t
R1(config)#hostname R1

! 1. Enable the physical interface (connected to SW1 Gi0/1)
R1(config)#int gigabitEthernet 0/0
R1(config-if)#no ip address
R1(config-if)#no shutdown
R1(config-if)#exit

! 2. Create Subinterface for VLAN 10
R1(config)#int gigabitEthernet 0/0.10
R1(config-subif)#encapsulation dot1Q 10
R1(config-subif)#ip address 192.168.10.1 255.255.255.0
R1(config-subif)#exit

! 3. Create Subinterface for VLAN 20
R1(config)#int gigabitEthernet 0/0.20
R1(config-subif)#encapsulation dot1Q 20
R1(config-subif)#ip address 192.168.20.1 255.255.255.0
R1(config-subif)#exit

R1(config)#end
R1#copy run start
```

-----

## 3. Client Configuration (Laptop1 and Laptop2)

Each client must be configured with a static IP address, subnet mask, and the **Default Gateway** address pointing to the respective router subinterface.

| Client | VLAN | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **Laptop in VLAN 10** | 10 (Sales) | 192.168.10.X | 255.255.255.0 | **192.168.10.1** |
| **Laptop in VLAN 20** | 20 (HR) | 192.168.20.Y | 255.255.255.0 | **192.168.20.1** |

-----

## 4. Verification

The success of the "Router on a Stick" configuration is confirmed when devices in different VLANs can communicate (i.e., route traffic through R1).

  * **Test Command:** From a Laptop in VLAN 10, **`Ping 192.168.20.Y`** (the IP of a Laptop in VLAN 20).
  * **Expected Result:** The ping should be **successful**, proving that Inter-VLAN routing is active.

-----
