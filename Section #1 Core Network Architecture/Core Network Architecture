
# Phase 1: Primary Edge Device
## Problem Description
The network was experiencing a complete routing failure/deadlock when trying to integrate a **SonicWall TZ80** alongside an existing residential router. 

### Core Issues Identified:
1. **IP & Gateway Conflict:** Both the old router and the SonicWall were competing for the same `192.168.8.x` subnet simultaneously without sequential routing.
2. **Asymmetric Routing / Loops:** Client devices were pointing to `192.168.8.1` (the old router) as their gateway, while the SonicWall was sitting passively on `192.168.8.5`, resulting in dropped packets and broken internet paths.
3. **Spectrum MAC Binding:** The Spectrum wireline modem required a full power-cycle cache clear to accept the new WAN MAC address of the SonicWall interface (`X1`).

## Final Network Topology (Sequential Design)

Traffic now flows linearly, establishing the SonicWall as the definitive Layer 3 Edge Routing device.

[ Spectrum Modem ] 
│ 
│ (Public IP via DHCP)
▼ [ SonicWall TZ80 ] 
• X1 (WAN): Public IP 
• X2 (LAN): 192.168.8.1 <-- New Default Gateway & DHCP Server
│ 
▼
[ Main Network Switch ]
│
├─► Wireless Access Point ──► (Laptops, Phones, Smart TVs) 
├─► Home Lab Server (192.168.8.200) └─► Desktop PC (192.168.8.180)

## Migration Steps Taken (Step-by-Step)

### Phase 1: SonicWall Interface Configuration
1. Logged into SonicWall and navigated to **Network > Interfaces**.
2. Modified the **X2** interface configuration:
   * **Zone:** LAN
   * **IP Address:** Changed from `192.168.8.5` to `192.168.8.1`
   * **Subnet Mask:** `255.255.255.0`

### Phase 2: DHCP Migration
1. Navigated to **Network > DHCP Server**.
2. Enabled the SonicWall native DHCP server.
3. Configured a dynamic IP pool for the `X2` subnet (`192.168.8.20` - `192.168.8.254`) ensuring the default gateway handed to clients was defined as `192.168.8.1`.

### Phase 3: Hardware Cutover & MAC Flush
1. Powered down the Spectrum Modem completely to flush its ARP/MAC cache table.
2. Powered off and removed the legacy home router.
3. Patched the **Spectrum Modem Ethernet** straight into SonicWall **X1 (WAN)**.
4. Patched the **Core Network Switch** straight into SonicWall **X2 (LAN)**.
5. Booted the Spectrum modem first (allowed 2 minutes for solid sync), followed by booting the SonicWall TZ80.

## Verification & Key Takeaways
* **Layer 3 Consolidation:** The old router was completely eliminated. The SonicWall TZ80 now operates as the sole Layer 3 Routing Gateway, NAT boundary, and DHCP server for the `192.168.8.0/24` network.
* **Result:** All legacy home devices instantly retained their original IP configurations and successfully routed out to the WAN through the new firewall architecture.

---
# Phase 2: DMZ Isolation Configuration (Port X3)

An isolated DMZ zone was provisioned to segment publicly accessible or untrusted lab servers from the core home LAN. Port X0 was intentionally left unconfigured to preserve it as an emergency management recovery port.

### DMZ Architecture:
* **Physical Port:** SonicWall `X3` (Zone: DMZ)
* **Subnet:** `192.168.20.0/24`
* **Gateway IP:** `192.168.20.1`
* **DHCP Range:** `192.168.20.20` - `192.168.20.254`

### Security Policy Logic Applied:
1. **LAN (X2) ➔ DMZ (X3):** Allowed (Administrative control of lab targets from trusted desktops).
2. **DMZ (X3) ➔ WAN (X1):** Allowed (Allows lab systems to pull packages/OS updates from the internet).
3. **DMZ (X3) ➔ LAN (X2):** **DENIED (Strictly Blocked)** (Prevents lateral movement into the home environment in the event of a DMZ host compromise).

---

# Phase 3 Appendix: Layer 2 Hardware Constraints & Validation

VLANS were created. 

### Hardware Limitation Discovery:
* **Current Infrastructure:** The deployed core switch is **Unmanaged**.
* **Impact:** Unmanaged switches cannot parse 802.1Q VLAN tags. Tagged packets flowing from the SonicWall parent interface (`X2`) will either be dropped entirely or stripped of their headers, causing segmentation failure


# Phase 4: Logical Layer 3 Network Diagram Specifications

The network documentation has been migrated from a physical component mapping (Excalidraw) to a logical security gateway schematic (draw.io) focusing exclusively on the firewall boundaries.

### Topology Diagram Checklist:
* [ ] **Edge Demarcation:** Spectrum WAN handoff explicitly mapped to physical port `X1`.
* [ ] **Interface Isolation:** `X0` isolated and designated as emergency rollback/out-of-band management.
* [ ] **Logical Segmenting:** Physical port `X2` defined as the parent trunk interface with explicit text blocks breaking out virtual sub-interface metrics (`X2:V10`).
* [ ] **DMZ Delineation:** Physical port `X3` mapped directly to untrusted server infrastructure with security zone boundary definitions.
* [ ] **Access Rule Legend:** Visual indicator showing directional traffic flows allowed or dropped by the firewall engine between zones.

# Network Segmentation Security Rationale

## 1. Perimeter Isolation (The Spectrum Demarcation)

- **Decision:** Migrating the SonicWall TZ80 to the absolute edge of the network as the sole Layer 3 gateway behind the Spectrum modem.

- **Rationale:** A standard consumer router lacks the deep packet inspection (DPI), intrusion prevention (IPS), and stateful firewalling capabilities required to defend a network containing lab experiments. By forcing all inbound and outbound traffic through the SonicWall `X1` interface, we establish a definitive security perimeter where malicious traffic can be inspected and dropped before it touches internal hosts.

## 2. Demilitarized Zone (DMZ) Isolation on Physical Port `X3`

- **Decision:** Assigning public-facing or experimental lab targets to a dedicated physical interface (`X3`) on the `192.168.20.0/24` subnet.
    
- **Rationale:** Based on the principle of **Least Privilege**, any device that hosts services accessible to the outside world—or runs untrusted lab code—is inherently a high-risk node. By placing these systems in a strictly isolated DMZ zone, the SonicWall automatically enforces a drop-all rule for traffic attempting to move from the DMZ back into the primary local home network (`X2`). If a lab server is compromised, the attacker is trapped inside a sandbox and cannot move laterally to target personal data, smart home devices, or family storage.
    

## 3. Logical VLAN Segmentation Architecture

- **Decision:** Designing a multiplexed 802.1Q trunking scheme for the local environment splitting traffic into Management (VLAN 10), Lab (VLAN 30), IoT (VLAN 40), and Guest (VLAN 50).
    
- **Rationale:** Traditional local networks operate as a single flat broadcast domain, meaning any device can freely talk to any other device at Layer 2. Slicing this environment into VLANs drastically reduces the network's attack surface:
    
    - **Management (VLAN 10):** Isolates the administrative interfaces of the firewall, switches, and server hypervisors so unauthorized users or malware on the main network cannot attempt brute-force or exploit attacks against infrastructure control planes.
        
    - **IoT Devices (VLAN 40):** Consumer smart devices (cameras, smart TVs, appliances) notoriously suffer from unpatched vulnerabilities and poor security standards. Giving them their own isolated VLAN prevents a compromised smart plug from being used to sniff local network traffic or attack trusted personal computers.
        
    - **Guest Network (VLAN 50):** Provides untrusted third-party devices with direct internet access while completely blocking them from discovering or communicating with internal household assets.
        

## 4. Operational Resiliency (Out-of-Band Management on `X0`)

- **Decision:** Deliberately preserving physical interface `X0` in its factory default configuration as an air-gapped, emergency management access port.
    
- **Rationale:** In an enterprise or advanced lab ecosystem, configuration errors (such as routing loops, broken VLAN tags, or overly restrictive firewall rules) can inadvertently lock an administrator out of the firewall via the primary LAN interfaces. Maintaining `X0` as an air-gapped recovery interface ensures that physical access to the chassis always grants an untampered management path to restore operations without needing a destructive factory reset.
