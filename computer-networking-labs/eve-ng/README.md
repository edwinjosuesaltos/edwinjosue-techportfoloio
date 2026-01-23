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
