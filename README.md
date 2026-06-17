# Dual-HQ & Branch Enterprise Network Architecture
### Fully Redundant Tier-3 Campus Core & Full-Mesh OSPF WAN Topology

## 📌 Project Overview
This repository contains the complete production-grade configuration scripts for a multi-site enterprise network infrastructure. The architecture features a highly available dual-headquarters campus network integrated with a remote branch office over a full-mesh WAN backplane. The design prioritizes zero-single-point-of-failure redundancy, sub-second failover path convergence, and strict security hardening across the access, distribution, core, and perimeter layers.

---

## 🗺️ Global Network Architecture Blueprint

### 🏢 1. Headquarters (HQ) Campus Core
* **High-Availability Gateway Layer:** Symmetrical **HSRP** implementation providing a virtual default gateway cluster (`.1`) for local subnets with sub-second failover preemption.
* **Core Switch Interconnect:** A 2-Gbps logical backbone formed via an active LACP EtherChannel bundle between Core Switches.
* **Deterministic Spanning Tree:** Controlled Layer 2 loops using **Rapid PVST+** with prioritized Root Bridge placement (`4096` on Core-1, `8192` on Core-2) and Root Guard protection on access downlinks.

### 🌐 2. WAN & Perimeter Edge
* **Full-Mesh Path Routing:** Multi-site path optimization crossing a central Service Provider core powered by single-area **OSPF Area 0** with explicit link-cost path engineering.
* **Secure Internet Boundaries:** Symmetrical **NAT/PAT Overload** mappings alongside dedicated Static PAT port-forwarding constraints to safely host corporate HTTPS web services (`192.168.30.10`) to the outside world.

### 🏬 3. Secured Branch Office (Router-on-a-Stick Evolution)
* **Sub-Interface Routing:** Provisioned a clean 802.1Q trunk gateway boundary via a Cisco `BR-EDGE` router using dedicated logical interfaces.
* **Isolated Management Plane:** Built an in-band management subnet entirely decoupled from user data layers to handle secure remote administration.

---

## 🔒 Security Hardening & Administrative Control

* **VTY Access-Class Control:** Standardized `RESTRICT_SSH` control filters applied directly to the virtual terminal lines (`line vty 0 15`) of all core switches, routers, and access blocks. This strictly limits cryptographic **SSHv2 RSA** administrative access to the **HQ IT Subnet (VLAN 10)** while dynamically dropping unauthenticated network requests at the TCP border.
* **Layer 2 Access Controls:** Enforced strict hardware **Sticky Port-Security** profiles across active end-user access ports, disabled unused interfaces, and applied **BPDU Guard** / **PortFast** on critical server farm downlinks.
* **Symmetrical NAT Exceptions:** Programmed top-down extended access control lists (`NAT_CONTROL` / `BR_NAT_CONTROL`) to completely bypass network translation layers for all internal cross-site traffic tunnels, ensuring seamless data symmetry.
* **Global Cryptographic Compliance:** Enforced global `service password-encryption` to mask local database profiles and active secrets from casual view.
* Role-Based SNMPv2c Network Telemetry Protection: Hardened network-wide infrastructure monitoring lines by implementing explicit segregation of telemetry access; restricted baseline Network Management Station (NMS) metrics to a Read-Only (RO) community string (Corporate1), while isolating administrative configuration capabilities behind a strict Read-Write (RW) credential block (Corporate2).
---

## 📊 Complete Subnet, Gateway, Loopback & Routing Boundary Matrix

| Device Identity | Local / Virtual Interface | IP Address / Subnet Mask | Functional Boundary / Gateway | Architectural Subnet & Management Role |
| :--- | :--- | :--- | :--- | :--- |
| **HQ-WAN-EDGE1**| `Gig0/1` <br> `Gig0/2` <br> `Gig0/0` <br> `Gig0/3/0` <br> **`Loopback0`** | `192.168.1.1 /30` <br> `192.168.1.5 /30` <br> `192.168.1.9 /30` <br> `203.0.113.2 /30` <br> **`10.0.0.1 /32`** | **OSPF Link Cost 10** <br> **OSPF Link Cost 10** <br> **OSPF Link Cost 10** <br> **Next-Hop:** `203.0.113.1` <br> **OSPF RID:** `1.1.1.1` | **HQ Perimeter Edge 1 (Primary Path)** <br> Transit to HQ-CORE-SW1 <br> Transit to HQ-CORE-SW2 <br> Transit to HQ-WAN-EDGE2 <br> Public WAN Internet Uplink (NAT Outside) <br> **Syslog / SNMP Source Identifier** |
| **HQ-WAN-EDGE2**| `Gig0/1` <br> `Gig0/2` <br> `Gig0/0` <br> `Gig0/3/0` <br> **`Loopback0`** | `192.168.1.13 /30` <br> `192.168.1.17 /30` <br> `192.168.1.10 /30` <br> `203.0.113.6 /30` <br> **`10.0.0.2 /32`** | **OSPF Link Cost 500** <br> **OSPF Link Cost 500** <br> **OSPF Link Cost 10** <br> **Next-Hop:** `203.0.113.5` <br> **OSPF RID:** `2.2.2.2` | **HQ Perimeter Edge 2 (Backup Path)** <br> Transit to HQ-CORE-SW2 <br> Transit to HQ-CORE-SW1 <br> Transit to HQ-WAN-EDGE1 <br> Public WAN Internet Uplink (NAT Outside) <br> **Syslog / SNMP Source Identifier** |
| **HQ-CORE-SW1** | `Vlan10` <br> `Vlan20` <br> `Vlan30` <br> `Po1` <br> **`Loopback0`** | `192.168.10.2 /24` <br> `192.168.20.2 /24` <br> `192.168.30.2 /24` <br> *Layer 2 Trunk* <br> **`10.0.0.3 /32`** | **HSRP VIP:** `192.168.10.1` <br> **HSRP VIP:** `192.168.20.1` <br> **HSRP VIP:** `192.168.30.1` <br> *N/A (L3 Routing)* <br> **OSPF RID:** `3.3.3.3` | **Primary Active Campus Gateway** <br> IT Subnet Gateway <br> HR Subnet Gateway <br> Server Farm SVI / Core Backbone <br> Inter-Core Bundle Interconnect <br> **Stable Loopback Node Management IP** |
| **HQ-CORE-SW2** | `Vlan10` <br> `Vlan20` <br> `Vlan30` <br> `Po1` <br> **`Loopback0`** | `192.168.10.3 /24` <br> `192.168.20.3 /24` <br> `192.168.30.3 /24` <br> *Layer 2 Trunk* <br> **`10.0.0.4 /32`** | **HSRP VIP:** `192.168.10.1` <br> **HSRP VIP:** `192.168.20.1` <br> **HSRP VIP:** `192.168.30.1` <br> *N/A (L3 Routing)* <br> **OSPF RID:** `4.4.4.4` | **Hot Standby Campus Gateway** <br> IT Subnet Redundancy <br> HR Subnet Redundancy <br> Server Farm Redundancy / Core Backbone <br> Inter-Core Bundle Interconnect <br> **Stable Loopback Node Management IP** |
| **ACCESS-SW1**  | `Vlan10` | `192.168.10.251 /24` | **Gateway:** `192.168.10.1` | HQ IT Access Layer Management IP |
| **ACCESS-SW2**  | `Vlan20` | `192.168.20.252 /24` | **Gateway:** `192.168.20.1` | HQ HR Access Layer Management IP |
| **ACCESS-SW3**  | `Vlan30` | `192.168.30.253 /24` | **Gateway:** `192.168.30.1` | Server Farm Access Layer Management IP |
| **BR-EDGE**     | `Gig0/0/0` <br> `Gig0/0/1.40` <br> `Gig0/0/1.50` <br> `Gig0/0/1.100` | `203.0.113.10 /30` <br> `192.168.40.1 /24` <br> `192.168.50.1 /24` <br> `192.168.100.1 /24` | **Next-Hop:** `203.0.113.9` <br> *Direct Router Gateway* <br> *Direct Router Gateway* <br> *Direct Router Gateway* | **Branch Edge Router** <br> Public WAN Uplink (NAT Outside) <br> Branch Main LAN SVI Gateway <br> Branch Local Devices SVI Gateway <br> In-Band Branch Management Gateway |
| **BR-DIST-SW1** | `Vlan100` | `192.168.100.2 /24` | **Gateway:** `192.168.100.1` | Branch Layer 2 Switch Management IP |

---

## 🔑 Topology Access Credentials

To access and audit the operational command lines (CLI) of the switches and routers within the simulation workspace, use the following standardized cryptographic access credentials:

* **Administrative Username:** `admin`
* **Secure VTY / Console Password:** `CiscoPass123!`
* **Privileged EXEC Mode (Enable Secret):** `CiscoPass123!`

*Note: All local database profiles have been symmetrically hardened with background cryptographic reversible hash formatting utilizing `service password-encryption` to align with enterprise security baseline compliance standards.*


## 🛠️ Deployment & Verification Commands

To verify route injection, address translation tables, and security enforcement across this topology, utilize the following Cisco IOS diagnostic commands:

## 🛠️ Deployment & Verification Commands

To verify route injection, address translation tables, infrastructure logging, and security enforcement across this topology, utilize the following Cisco IOS diagnostic commands:

```text
! Verify Layer 3 Path Symmetrical Convergence
show ip route
show ip ospf neighbor

! Validate Redundant Active/Standby Gateway Matrix
show standby brief

! Inspect Operational Security & Translations
show ip nat translations
show access-lists
show port-security interface [interface_id]

! Check Management Plane Gatekeeper Isolation
show ip ssh

! Validate SNMPv2c Telemetry & Syslog Status
show logging
```

---
*Maintained by Marko Cvetković — Verified fully functional on Cisco Packet Tracer.*


