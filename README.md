# Enterprise Network Segmentation & Access Control List (ACL) Implementation

A simulated enterprise network built using **Cisco Packet Tracer** demonstrating **Router-on-a-Stick** inter-VLAN routing, dynamic network configuration (DHCP), and network access control lists (ACLs) to enforce zero-trust security between department subnets.

---

## 📌 Project Overview

In a modern corporate infrastructure, isolating broadcast domains and securing confidential departments (e.g., IT and Management) from guest or non-employee networks is critical to preventing unauthorized access and reducing attack vectors.

This project demonstrates:
* **Logical Segmentation:** Creating virtual local area networks (VLANs) to separate IT, Corporate Staff, and Guest traffic.
* **Inter-VLAN Routing:** Configuring 802.1Q encapsulation on a single physical link ("Router-on-a-Stick") to route traffic between authorized subnets.
* **Network Access Control:** Applying an Extended Access Control List (ACL) at the default gateway level to isolate guest devices while preserving internet/gateway access.

---

## 📐 Network Topology & Addressing Scheme

![Network Topology](images/topology.png)

### Core Components
* **Core Router:** Cisco 2911 Router (`Router0`)
* **Distribution Switch:** Cisco Catalyst 2960 Switch (`Switch0`)
* **Department Endpoints:** `IT-PC` (VLAN 10), `Staff-PC` (VLAN 20), `Guest-PC` (VLAN 30)

### Subnet & Addressing Table

| Department / VLAN | VLAN ID | Network Subnet | Subnet Mask | Default Gateway | Assigned Switch Port |
|---|---|---|---|---|---|
| **IT / Management** | `VLAN 10` | `192.168.10.0/24` | `255.255.255.0` | `192.168.10.1` | `FastEthernet 0/1` |
| **Corporate Staff** | `VLAN 20` | `192.168.20.0/24` | `255.255.255.0` | `192.168.20.1` | `FastEthernet 0/2` |
| **Guest Wi-Fi** | `VLAN 30` | `192.168.30.0/24` | `255.255.255.0` | `192.168.30.1` | `FastEthernet 0/3` |

---

## ⚙️ Configuration Walkthrough

### 1. Switch VLAN & Trunk Configuration (`Switch0`)
Virtual networks were defined and assigned to corresponding access ports. The uplink connecting `Switch0` (`Gi0/1`) to `Router0` (`Gi0/0`) was set as an **802.1Q Trunk Link** to transport traffic across all VLANs.

```text
Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name IT_Management
Switch(config-vlan)# vlan 20
Switch(config-vlan)# name Corporate_Staff
Switch(config-vlan)# vlan 30
Switch(config-vlan)# name Guest_WiFi
Switch(config-vlan)# exit

! Assigning Access Ports
Switch(config)# interface FastEthernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10

Switch(config)# interface FastEthernet 0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20

Switch(config)# interface FastEthernet 0/3
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 30

! Configuring Trunk Uplink
Switch(config)# interface GigabitEthernet 0/1
Switch(config-if)# switchport mode trunk
2. Router-on-a-Stick Sub-Interface Configuration (Router0)Sub-interfaces were established on GigabitEthernet 0/0 to serve as default gateways for each VLAN.PlaintextRouter# configure terminal
Router(config)# interface GigabitEthernet 0/0
Router(config-if)# no shutdown

! VLAN 10 Sub-Interface
Router(config)# interface GigabitEthernet 0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0

! VLAN 20 Sub-Interface
Router(config)# interface GigabitEthernet 0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0

! VLAN 30 Sub-Interface
Router(config)# interface GigabitEthernet 0/0.30
Router(config-subif)# encapsulation dot1Q 30
Router(config-subif)# ip address 192.168.30.1 255.255.255.0
3. Access Control List (ACL) ImplementationTo satisfy corporate security requirements, an Extended ACL (100) was defined to filter inbound traffic from the guest network (192.168.30.0/24) destined for the sensitive IT subnet (192.168.10.0/24), while allowing all other standard communication.PlaintextRouter(config)# access-list 100 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
Router(config)# access-list 100 permit ip any any

! Binding ACL inbound on the Guest Gateway
Router(config)# interface GigabitEthernet 0/0.30
Router(config-subif)# ip access-group 100 in
🧪 Verification & Testing ResultsTest CaseSourceDestinationCommandResultScreenshotInter-VLAN RoutingIT-PC (192.168.10.2)Staff-PC (192.168.20.2)ping 192.168.20.2SUCCESS (0% Loss)Security ACL IsolationGuest-PC (192.168.30.2)IT-PC (192.168.10.2)ping 192.168.10.2BLOCKED (Destination host unreachable)
