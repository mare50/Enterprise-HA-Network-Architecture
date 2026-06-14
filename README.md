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

---

## 📊 Complete Subnet & SVI Addressing Matrix

| Device Identity | Management IP / SVI | Subnet Mask | Default Gateway | Primary Functional Role |
| :--- | :--- | :--- | :--- | :--- |
| **HQ-CORE-SW1** | `192.168.30.2` (V30) | `255.255.255.0` | *N/A (L3 Routing)* | Primary Gateway (IT/HR/Server Farm) |
| **HQ-CORE-SW2** | `192.168.30.3` (V30) | `255.255.255.0` | *N/A (L3 Routing)* | Backup Gateway / Redundant Transit |
| **ACCESS-SW1** | `192.168.10.251` (V10) | `255.255.255.0` | `192.168.10.1` | HQ IT Access Switch Layer |
| **ACCESS-SW2** | `192.168.20.252` (V20) | `255.255.255.0` | `192.168.20.1` | HQ HR Access Switch Layer |
| **ACCESS-SW3** | `192.168.30.253` (V30) | `255.255.255.0` | `192.168.30.1` | Corporate Server Farm Access Node |
| **BR-EDGE** | `192.168.100.1` (Sub) | `255.255.255.0` | *N/A (WAN Route)* | Branch Router Perimeter Gateway |
| **BR-DIST-SW1** | `192.168.100.2` (V100) | `255.255.255.0` | `192.168.100.1` | Branch Layer 2 Access Switch |

---

## 🔑 Topology Access Credentials

To access and audit the operational command lines (CLI) of the switches and routers within the simulation workspace, use the following standardized cryptographic access credentials:

* **Administrative Username:** `admin`
* **Secure VTY / Console Password:** `CiscoPass123!`
* **Privileged EXEC Mode (Enable Secret):** `CiscoPass123!`

*Note: All local database profiles have been symmetrically hardened with background cryptographic reversible hash formatting utilizing `service password-encryption` to align with enterprise security baseline compliance standards.*


## 🛠️ Deployment & Verification Commands

To verify route injection, address translation tables, and security enforcement across this topology, utilize the following Cisco IOS diagnostic commands:

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
```

---
*Maintained by [Your Name/GitHub Username] — Verified fully functional on Cisco Packet Tracer.*


