# 🏢 LAB 01: Enterprise Network Segmentation (L2/L3) & Internet Access

![Cisco Badge](https://img.shields.io/badge/Vendor-Cisco-blue?style=for-the-badge&logo=cisco) ![Lab Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge) ![Type](https://img.shields.io/badge/Type-Routing%20%26%20Switching-orange?style=for-the-badge)

## 📌 Project Overview
This project simulates a corporate branch network environment using **Cisco IOL** within **EVE-NG**. The primary objective was to design a scalable "Collapsed Core" architecture that segments departmental traffic, provides secure Inter-VLAN routing, and enables Internet access via NAT.

Additionally, a secure management plane was implemented using SSHv2 and a "Jump Host" methodology to access internal Layer 2 devices.

### 🎯 Key Objectives
- **Segmentation:** Isolate Sales and IT departments using VLANs (802.1Q).
- **Routing:** Implement **Router-on-a-Stick (ROAS)** for efficient Inter-VLAN communication.
- **Internet Access:** Configure **NAT Overload (PAT)** to allow internal private subnets to access the WAN.
- **Security:** Replace Telnet with **SSHv2** and enforce VLAN tagging on uplinks.

---

## 🗺️ Network Topology & Addressing

### IP Schema (Addressing Table)

| Device | Interface | Role | VLAN ID | IP Address / Mask | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **R1-Gateway** | `Eth0/0` | WAN | N/A | **DHCP** | Uplink to Internet Provider |
| **R1-Gateway** | `Eth0/1.10` | LAN Gateway | 10 | **10.1.10.1 /24** | Gateway for Sales Dept |
| **R1-Gateway** | `Eth0/1.20` | LAN Gateway | 20 | **10.1.20.1 /24** | Gateway for IT Dept |
| **R1-Gateway** | `Eth0/1.99` | LAN Gateway | 99 | **10.1.99.1 /24** | Management Gateway |
| **SW1-Access** | `Vlan 99` | Mgmt Interface | 99 | **10.1.99.2 /24** | SVI for SSH Access |
| **PC1** | `Eth0` | Endpoint | 10 | **10.1.10.2 /24** | Sales Workstation |
| **PC2** | `Eth0` | Endpoint | 20 | **10.1.20.2 /24** | IT Workstation |

---

## ⚙️ Configuration Snippets

### 1. Router Configuration (R1)
**Feature: Router-on-a-Stick & NAT**
This configuration enables the router to handle tagged traffic from the switch and translate private IPs to the public WAN IP.

```cisco
! -- INTER-VLAN ROUTING --
interface Ethernet0/1.10
 description Gateway-Sales
 encapsulation dot1Q 10
 ip address 10.1.10.1 255.255.255.0
 ip nat inside
!
interface Ethernet0/1.20
 description Gateway-IT
 encapsulation dot1Q 20
 ip address 10.1.20.1 255.255.255.0
 ip nat inside

! -- NAT OVERLOAD (PAT) --
! Access List defining allowed internal networks
access-list 1 permit 10.1.0.0 0.0.255.255

! Applying NAT to the WAN interface
ip nat inside source list 1 interface Ethernet0/0 overload
interface Ethernet0/0
 ip nat outside
```

### 2. Switch Configuration (SW1)
**Feature: VLANs & Trunking Ensures that the uplink to the router carries all VLAN tags and that end-user ports are assigned correctly.**
```cisco
! -- VLAN DATABASE --
vlan 10
 name SALES
vlan 20
 name IT
vlan 99
 name MGMT

! -- UPLINK TRUNK CONFIGURATION --
interface Ethernet0/0
 description Link-to-R1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99

! -- ACCESS PORT CONFIGURATION --
interface Ethernet0/1
 description PC-SALES
 switchport mode access
 switchport access vlan 10

! -- MANAGEMENT SVI --
interface Vlan99
 ip address 10.1.99.2 255.255.255.0
! Default gateway required for SSH replies
ip default-gateway 10.1.99.1
```
### 🔧 Troubleshooting Log
During the deployment, a connectivity issue was encountered.
This section details the diagnosis and resolution.

**🔴 Issue: PC1 unable to reach Gateway**
- **Symptom: PC1 (`10.1.10.2`) failed to ping its default gateway (`10.1.10.1`).**
- **Checked physical connectivity (OK).**
- **Ran `show vlan brief` on SW1.**
- **Finding: The uplink port `Eth0/0` was listed under VLAN 1 (Default) instead of being a Trunk.**
- **Root Cause: The link between the Switch and Router was operating in Access Mode. The router was expecting tagged frames (VLAN 10), but the switch was sending untagged frames (VLAN 1) or dropping tagged frames.**
- **Solution: Applied the following configuration to force Trunking:**
```cisco
SW1(config)# interface e0/0
SW1(config-if)# switchport trunk encapsulation dot1q
SW1(config-if)# switchport mode trunk
```
- **Outcome: Layer 3 connectivity was restored immediately.**

### ✅ Security Configuration (SSHv2)
**Feature: Secure Remote Management Applied to both R1 and SW1 to enforce encrypted management sessions.**
```cisco
! -- DOMAIN & CRYPTO KEYS --
ip domain-name corp.lab
! Generates RSA keys (required for SSH)
crypto key generate rsa modulus 2048

! -- LOCAL USER DATABASE --
username admin privilege 15 secret Cisco123!

! -- VTY LINE CONFIGURATION --
line vty 0 4
 transport input ssh
 login local
```

<img width="4480" height="1440" alt="image" src="https://github.com/user-attachments/assets/d7fe8f6f-bee3-4ffa-9e50-b0a831b17275" />

# 🚀LAB 02 Segmented Corporate Network Implementation (VLSM & Inter-VLAN Routing)
![Cisco Badge](https://img.shields.io/badge/Vendor-Cisco-blue?style=for-the-badge&logo=cisco) ![Lab Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge) ![Type](https://img.shields.io/badge/Type-Routing%20%26%20Switching-orange?style=for-the-badge)

## 📝 Project Overview
This project demonstrates the design and deployment of a scalable corporate network infrastructure. Utilizing a **$172.16.0.0/16$** base network, I implemented a **Variable Length Subnet Masking (VLSM)** scheme to optimize IP address allocation. The environment was emulated using **EVE-NG Professional** with authentic **Cisco IOL (IOS on Linux)** images to ensure production-grade behavior.

## 🎯 Key Objectives
* **Efficient Addressing**: Minimize IP waste using VLSM calculations based on host requirements.
* **Network Segmentation**: Deploy VLANs to isolate broadcast domains and enhance organizational security.
* **Inter-VLAN Routing**: Implement a "Router-on-a-Stick" configuration via IEEE 802.1Q trunking.
* **Secure Management**: Enforce SSHv2 and encrypted credentials for secure remote device administration.

---

## 🗺️ Network Topology
The topology follows a hierarchical design consisting of a Core Router, a Distribution Switch, and multiple departmental Virtual PCs (VPCS).
<img width="975" height="458" alt="image" src="https://github.com/user-attachments/assets/4d891cbf-9dc5-4d18-8a86-2a0f6d5a7f8e" />
<img width="975" height="458" alt="image" src="https://github.com/user-attachments/assets/0b97d299-3ae4-48d9-97d1-978d345a1947" />
<img width="975" height="993" alt="image" src="https://github.com/user-attachments/assets/8a4efd3e-f391-41e1-8f6e-34b571b66fcf" />
<img width="975" height="735" alt="image" src="https://github.com/user-attachments/assets/271783bd-f7e9-444d-8009-e9d6904d6fdb" />
<img width="975" height="546" alt="image" src="https://github.com/user-attachments/assets/7288352a-93ed-44c9-b52e-68cd7a8417c9" />
<img width="975" height="594" alt="image" src="https://github.com/user-attachments/assets/4068ab31-a318-4716-bc3a-437b49501b69" />
> **Note:**


---

## 🧮 VLSM Addressing Plan
The network was segmented based on specific host requirements, prioritizing the largest subnets to maintain contiguous addressing and efficiency.

| Department | VLAN | Required Hosts | Subnet ID | CIDR / Mask | Default Gateway |
| :--- | :---: | :---: | :--- | :--- | :--- |
| **Operations** | 10 | 500 | `172.16.0.0` | `/23` (255.255.254.0) | `172.16.0.1` |
| **Guest WiFi** | 20 | 100 | `172.16.2.0` | `/25` (255.255.255.128) | `172.16.2.1` |
| **Administration** | 30 | 50 | `172.16.2.128` | `/26` (255.255.255.192) | `172.16.2.129` |
| **IT Management** | 99 | 10 | `172.16.2.192` | `/28` (255.255.255.240) | `172.16.2.193` |

---

## 🛠️ Configuration Highlights

### 1. R1-CORE (Gateway & SSH Security)
Configured with logical sub-interfaces for routing and hardened with RSA-2048 encryption for secure SSH access.

```cisco
! Management Security & SSHv2
hostname R1-CORE
ip domain-name enterprise.local
username admin privilege 15 secret adminpass
crypto key generate rsa (2048 bits)
ip ssh version 2

! Inter-VLAN Routing (Router-on-a-Stick)
interface Ethernet0/0.10
 description GW_OPERATIONS
 encapsulation dot1Q 10
 ip address 172.16.0.1 255.255.254.0

interface Ethernet0/0.30
 description GW_ADMINISTRATION
 encapsulation dot1Q 30
 ip address 172.16.2.129 255.255.255.192

### 2. SW1-DIST (VLANs & Trunking) 
Configured to manage broadcast domains and bridge departmental traffic to the Core Router using the 802.1Q standard.
```cisco
! VLAN Database Creation
vlan 10
 name Operations
vlan 30
 name Administration

! Access Port Assignments
interface Ethernet0/1
 switchport mode access
 switchport access vlan 10

! 802.1Q Trunk Link
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 description TRUNK_TO_CORE
```

### 🧪 Verification & Proof of Concept
Successful deployment was validated through the following technical checks:Inter-VLAN Connectivity: Confirmed via ICMP Echo Requests (ping) between PC-Operations ($172.16.0.10$) and PC-Administration ($172.16.2.140$).

Routing Table Integrity: Verified using show ip route, confirming all VLSM subnets are correctly learned and reachable.Switchport Status: Verified via show vlan brief to ensure correct port-to-VLAN mapping.
