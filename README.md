# High-Availability Enterprise Campus & Dual-Edge Redundant WAN Architecture
An enterprise-grade, multi-site infrastructure topology engineered inside Cisco Packet Tracer, showcasing full-mesh core redundancy, predictive path convergence, and robust defense-in-depth security perimeters.

## 🗺️ Architectural Topology Overview
- **Enterprise Campus Core:** Dual Multi-Layer Switches deployed in a high-availability Redundant Triangle topology, acting as localized L3 SVI routing engines for corporate segments.
- **HQ Resilient Edge:** Dual Cisco 2911 Edge Gateways cross-connected via a Full-Mesh Diamond backbone utilizing modular Fiber SFP transceivers out to the public internet core.
- **WAN Backplane:** A dedicated Service Provider Transit Core handling public internet routing networks and housing a simulated global DNS directory (8.8.8.8).
- **Remote Branch Office:** Scaled remote branch office utilizing an optimized Router-on-a-Stick architecture to dynamically bridge distinct private subnets to the enterprise core across the ISP cloud.

---

## 🚀 Key Engineering Implementations

### 1. High-Availability Layer 2/3 Switching Core
- **Deterministic Spanning Tree Calculations:** Hardcoded Rapid PVST+ priorities (`4096` on Core-1, `28672` on Core-2) to establish a predictable, loop-free backup path tree, completely eliminating macro-election convergence delays.
- **Link Aggregation via Active LACP:** Bundled the inter-core backbone trunks into a logical 2-Gbps EtherChannel using Link Aggregation Control Protocol (LACP), allowing simultaneous multi-path forwarding with sub-second failover redundancy.
- **Hot Standby Router Protocol (HSRP):** Configured redundant Virtual Gateways (`.1`) across all corporate subnets. Core-1 is hardcoded as the Active Master (Priority 110 with preemption), while Core-2 sits as the standby engine to handle seamless, hands-free client gateway failover.
- **Symmetrical Trunk Allowed Profiles:** Standardized and mirrored allowed VLAN strings (`10,20,30,99`) symmetrically across all core downlinks and access uplinks, preventing Spanning Tree blocking traps and HSRP heartbeat deadlocks.

### 2. Full-Mesh OSPF Edge Routing Matrix
- **Neighbor Adjacency Symmetry:** Harmonized all point-to-point transit link declarations inside the OSPF Area 0 architecture, ensuring perfect link-state database (LSDB) cross-site synchronization.
- **Predictive Path Cost Tuning:** Explicitly manipulated interface OSPF costs (`10` on primary paths, `500` on backup cross-connects) to force the network to systematically choose HQ-WAN-EDGE1 as the master outbound highway, preventing routing tables from flapping.
- **Core Security Hardening:** Applied `passive-interface` constraints to all internal user and server SVIs to silence OSPF multicast traffic on client ports, killing potential broadcast loop vectors and freeing up internal switch CPU cycles.

### 3. Redundant WAN Transit & Perimeter Engineering
- **Floating Service-Provider Routes:** Deployed static destination routing rules on the ISP backbone integrated with floating backup Administrative Distance metrics (AD 1 to Edge-1, AD 10 to Edge-2) to achieve automatic multi-hub failover when an edge hardware crash occurs.
- **Top-Down Named NAT Control Access Lists:** Structured prioritized, named extended access lists on all edge routers to handle Port Address Translation (PAT) overload mechanics, placing strict cross-site NAT exceptions at the top to protect private data packets.
- **Static PAT Forwarding Port Mapping:** Configured identical static port-forwarding rules (TCP 80/443) on both public edge gateways to grant continuous, high-availability external access to the centralized corporate Web Server.

### 4. Enterprise Defense-In-Depth Security
- **Native VLAN Hardening:** Shifted the untagged native trunking language away from the default factory setting over to a dedicated **VLAN 99** across all inter-switch links, fully mitigating VLAN-hopping security vectors.
- **Access-Layer Port Protection:** Enforced `switchport port-security` using `sticky` MAC-address binding parameters and strict `restrict` violation rules on active user ports, while keeping all unassigned interfaces administratively `shutdown`.
- **Spanning Tree Guard Perimeters:** Deployed `spanning-tree guard root` on critical core downlinks to prevent rogue switches from hijacking the Root Bridge path tree, paired with access-port `bpduguard` definitions.
- **Secure Cryptographic Remote Management:** Completely banned unencrypted plaintext Telnet access paths and globally enforced **SSHv2 RSA Cryptographic management policies** with distinct local database administrative profiles to protect management traffic from sniffing attacks.

---

## 📊 Physical & Logical Addressing Matrix

| Infrastructure Device | Local Interface | Assigned IP Address | Subnet Mask | Configured Role / Description |
| :--- | :--- | :--- | :--- | :--- |
| **HQ-CORE-SW1** | Vlan 10 (SVI) | 192.168.10.2 | 255.255.255.0 | Active HSRP Virtual Gateway: 192.168.10.1 |
| **HQ-CORE-SW1** | Vlan 20 (SVI) | 192.168.20.2 | 255.255.255.0 | Active HSRP Virtual Gateway: 192.168.20.1 |
| **HQ-CORE-SW1** | Vlan 30 (SVI) | 192.168.30.2 | 255.255.255.0 | Active HSRP Virtual Gateway: 192.168.30.1 |
| **HQ-CORE-SW2** | Vlan 10 (SVI) | 192.168.10.3 | 255.255.255.0 | Standby HSRP Virtual Gateway: 192.168.10.1 |
| **HQ-CORE-SW2** | Vlan 20 (SVI) | 192.168.20.3 | 255.255.255.0 | Standby HSRP Virtual Gateway: 192.168.20.1 |
| **HQ-CORE-SW2** | Vlan 30 (SVI) | 192.168.30.3 | 255.255.255.0 | Standby HSRP Virtual Gateway: 192.168.30.1 |
| **HQ-WAN-EDGE1** | Gig0/3/0 (WAN) | 203.0.113.2 | 255.255.255.252 | Primary HQ Public Interface (Gateway: 203.0.113.1) |
| **HQ-WAN-EDGE2** | Gig0/3/0 (WAN) | 203.0.113.6 | 255.255.255.252 | Backup HQ Public Interface (Gateway: 203.0.113.5) |
| **BR-EDGE-ROUTER**| Gig0/0/0 (WAN) | 203.0.113.10 | 255.255.255.252 | Branch Public Interface (Gateway: 203.0.113.9) |
| **BR-EDGE-ROUTER**| Gig0/0/1.40 | 192.168.40.1 | 255.255.255.0 | Branch Main Office Router-on-a-Stick Gateway |
| **BR-EDGE-ROUTER**| Gig0/0/1.50 | 192.168.50.1 | 255.255.255.0 | Branch Local Devices Router-on-a-Stick Gateway |
| **ISP-ROUTER** | Loopback0 | 8.8.8.8 | 255.255.255.255 | Simulated Global Internet DNS Framework Node |

---

## 🔬 High-Availability Automated Failover Verification

### Test 1: Layer 2 EtherChannel Link Failure Simulation
1. Establish a continuous ping from **HQ-PC-IT** to **HQ-Server** (`ping 192.168.30.10 -n 1000`).
2. Administratively shut down or disconnect interface **`GigabitEthernet1/0/3`** on Core-1.
3. **Observation:** The LACP port-channel summary dynamically updates to show `Po1(SU)` with a single interface active. The continuous ping logs drop **0 packets**, proving successful sub-second link aggregation hardware failover.

### Test 2: Core Switch Blackout Simulation
1. Run a persistent gateway check from **HR PC**: `ping 192.168.20.1 -n 1000`.
2. Physically turn off the power switch on **`HQ-CORE-SW1`** (Primary Master Root Bridge).
3. **Observation:** Core-2 detects the missing HSRP hello heartbeats across the backup switch paths, moves its state from Standby to Active inside 3 seconds, and re-allocates the virtual MAC bindings. Client connectivity stays uninterrupted with a **100% network recovery rate**.

### Test 3: Dual-Edge WAN Edge Outage Simulation
1. Launch a cross-site connection check from the **Branch PC** straight to the HQ Server: `ping 192.168.30.10 -n 1000`.
2. Delete the physical fiber port link or shut down **`HQ-WAN-EDGE1`** entirely.
3. **Observation:** The internal campus network switches re-converge using OSPF path adjustments. Simultaneously, the ISP router recognizes the loss of the primary route, automatically activates its **floating static route (AD 10)**, and drops the incoming branch data traffic straight down into the backup gateway **`HQ-WAN-EDGE2`**, keeping your remote office seamlessly online.

## 🔐 Laboratory Access Credentials
To verify and interact with the live command-line interfaces (CLI) of the core switches and edge routing nodes, utilize the standardized non-production administrative profile embedded within the topology architecture:

- **SSH Username:** `admin`
- **Password / Enable Secret:** `CiscoPass123!`
- **Cryptographic Access Control:** Enforced via `transport input ssh` on all Virtual Terminal Lines (`line vty 0 15`), with legacy unencrypted Telnet protocols explicitly disabled.


