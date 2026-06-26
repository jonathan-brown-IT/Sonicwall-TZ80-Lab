## Phase 5: Stateful Access Control Rules & Logging Matrix

To enforce the architectural rationale, explicit stateful access policies have been configured. Default implicit rules have been overridden with explicit logging rules to ensure visibility within the SonicOS packet monitor and syslog server.

### Configured Policy Matrix:

| Direction | Source | Destination | Service | Action | Logging | Intent / Rationale |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **LAN ➔ DMZ** | `192.168.8.180` | DMZ Subnet | HTTP, HTTPS, SSH | **ALLOW** | Enabled | Restricts DMZ management capabilities strictly to the administrator's primary desktop. |
| **LAN ➔ DMZ** | Any | Any | Any | **DENY** | Enabled | Prevents standard household nodes from interacting with lab infrastructure. |
| **DMZ ➔ LAN** | Any | Any | Any | **DENY** | Enabled | **Hard Boundary:** Absolute drop and log of any traffic attempting lateral movement from untrusted DMZ to LAN. |
| **DMZ ➔ WAN** | DMZ Subnet | Any | DNS, NTP, HTTP, HTTPS | **ALLOW** | Enabled | Permits necessary egress baselines for OS updating, repository pulling, and time syncing. |
| **DMZ ➔ WAN** | Any | Any | Any | **DENY** | Enabled | Drops malicious protocol egress attempts (e.g., rogue scanning, outbound SMTP spamming). |

### Validation Command (From DMZ Host):
* To verify the DMZ ➔ LAN deny rule is functioning, run a continuous ping from a DMZ host to the LAN gateway. It should fail cleanly while generating alert drops in the SonicWall log monitor:

  ping 192.168.8.1

---
## Phase 6: Geo-IP Perimeter Hardening & Region Restrictions

To minimize the firewall's exposed attack surface against automated global botnets and malicious wide-area scanning, country-level Layer 7 Geo-IP filtering was implemented.

### Geo-IP Global Configuration:
* **Status:** Enabled
* **Scope:** Applied strictly to Egress/Ingress boundaries on `X1` (WAN Zone).
* **Block Profile:** All Connections (Bidirectional drop).

### Documented Blocked Regions (Baseline):
* **Asia:** China (CN), North Korea (KP), Iran (IR)
* **Europe:** Russia (RU)
* *Note: Adjust this repository list based on active lab project requirements or known threat intelligence feeds.*

### Log Inspection Protocol:
* In the event an anomalous connection drop occurs from an external asset, logs can be verified via **MONITOR ➔ Logs ➔ System Logs** filtering for `Geo-IP Blocked`. The firewall will output the country ISO code alongside the dropped packet metadata.

## Phase 6 Appendix: Empirical Edge Security Validation (Log Capture)

To validate the efficacy of the perimeter security policies, the firewall's System Logs were audited following a period of active deployment. The edge appliance successfully intercepted and dropped unrequested inbound connections from high-risk geopolitical zones.

### Log Analysis Breakdown:

- **Event 1 Summary:** Intercepted an unsolicited connection attempt targeting port `8081` from source host `182.112.97.44` located in **China**. The firewall statefully identified the country profile threat and dropped the traffic.
    
- **Event 2 Summary:** Intercepted a wide-area scan attempt targeting port `42700` from source host `195.98.80.253` originating from the **Russian Federation**. The threat vector was immediately dropped and flagged with an `Alert` priority rating.
    

### Lab Takeaway:

This telemetry confirms that residential internet connections are subject to continuous, automated global scanning within minutes of provisioning a public IP. Deploying the SonicWall TZ80 at the absolute Layer 3 edge provides critical defensive visibility that standard consumer routers fail to log or mitigate.

## Phase 8: Time-Based Access Control & Schedule Object Profiling

To enforce rigorous operational boundaries on experimental infrastructure, a schedule-based policy was introduced to bound outbound wide-area networking strictly to an administrative maintenance window.

### 1. Schedule Object Profile
* **Object Name:** `Lab_Maint_Window`
* **Schedule Class:** Weekly Recurring
* **Active Window:** Monday – Friday | 17:00 – 22:00 (5:00 PM – 10:00 PM Local)

### 2. Access Policy Integration
The existing egress permission rule was bound directly to the new time schedule object:

| Direction | Source Object | Destination Object | Service Group | Schedule | Action |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **DMZ ➔ WAN** | `DMZ_Lab_Subnet` | Any | `Lab_Egress_Services` | `Lab_Maint_Window` | **ALLOW** |

### Policy Verification Check:
* **In-Window Execution:** Attempts to reach external public repositories resolve successfully during the designated maintenance period.
* **Out-of-Window Execution:** Traffic sent outside specified hours fails statefully. The firewall's **MONITOR ➔ Logs ➔ System Logs** console flags these out-of-window attempts as drops via the default/cleanup deny policy due to the primary rule expiring schedule validity.



To export your full access rule table from SonicOS 8, head up to the **POLICY** tab, navigate to **Rules > Access Rules**, and look for the **Export** button (usually an icon with an arrow pointing down or a gear menu at the top right of the table grid). You can export it as a CSV or PDF file to capture your live data.

For your final lab report, you want to combine that exported data with a deep-dive annotation explaining the security philosophy behind why every rule exists. Here is the fully annotated security matrix to paste directly into your final documentation.

# SonicOS 8 Access Rule Table & Security Philosophy Annotations

```
                                [ THE INTERNET (WAN) ]
                                          │
                  ┌───────────────────────┴───────────────────────┐
                  ▼                                               ▼
         [ Inbound Scans ]                               [ Authorized Replies ]
                  │                                               │
    Rule #4 (Geo/Threat Drops)                             Rule #3 (Stateful TCP)
                  │                                               │
                  ▼                                               ▼
              [ DROPPED ]                                  [ INTERNAL HOSTS ]
```

|**Rule ID**|**Direction**|**Source**|**Destination**|**Service**|**Action**|**Schedule**|**Logging**|
|---|---|---|---|---|---|---|---|
|**1**|LAN ➔ DMZ|`Authorized_Admin_Hosts`|`DMZ_Lab_Subnet`|`Lab_Management_Services`|**ALLOW**|Always On|Enabled|
|**2**|LAN ➔ DMZ|Any|Any|Any|**DENY**|Always On|Enabled|
|**3**|DMZ ➔ WAN|`DMZ_Lab_Subnet`|Any|DNS, NTP, HTTP, HTTPS|**ALLOW**|`Lab_Maint_Window`|Enabled|
|**4**|WAN ➔ Any|`Custom_Geo_Block_Group`|Any|Any|**DENY**|Always On|Enabled|
|**5**|DMZ ➔ LAN|Any|Any|Any|**DENY**|Always On|Enabled|
|**6**|Any ➔ Any|Any|Any|Any|**DENY**|Always On|Disabled|

### Architectural Design Annotations & Philosophy Notes

#### Philosophy Note A: Intentional Management Micro-Segmentation (Rules 1 & 2)

- **Design Rationale:** In a typical home lab, users allow the entire local subnet to talk to their lab systems. This architecture enforces **Micro-Segmentation**. Rule 1 explicitly locks the control path down to a single host definition (`Admin_Desktop_PC`). Rule 2 functions as an immediate structural block for everything else on the LAN.
    
- **Security Outcome:** If a household mobile device or IoT asset catches malware, it cannot run automated lateral scans against the lab control interfaces because the firewall drops the traffic before it can exit the LAN zone.
    

#### Philosophy Note B: The Principle of Least Privilege Egress (Rule 3)

- **Design Rationale:** Standard networks let internal hosts talk to any port on the internet unrestricted. Rule 3 applies strict boundaries to outgoing connections by limiting traffic down to a core service group (Web browsing, Name resolution, and Time sync) and pairing it with a time-bound schedule object (`Lab_Maint_Window`).
    
- **Security Outcome:** If a threat actor establishes an initial foothold on a lab device, standard command-and-control (C2) callback mechanisms running on non-standard ports will fail. Furthermore, any malicious automated data exfiltration attempts launched outside of your 5:00 PM – 10:00 PM maintenance window are completely neutralized.
    

#### Philosophy Note C: Threat Intel Edge Hardening (Rule 4)

- **Design Rationale:** Shifting from standard reactive security to proactive threat hunting. By leveraging manual threat intelligence feeds to populate the `Custom_Geo_Block_Group` at Priority 4, the firewall kills connections from known malicious infrastructures at the very perimeter.
    
- **Security Outcome:** Reduces processor overhead on the SonicWall by instantly discarding known bad packets before they can hit downstream evaluation engines or trigger deeper stateful processing rules.
    

#### Philosophy Note D: Absolute Blast Radius Containment (Rule 5)

- **Design Rationale:** Rule 5 acts as an unyielding sandbox boundary. It assumes that the DMZ zone _will_ eventually experience a compromise.
    
- **Security Outcome:** By explicitly blocking and logging any traffic initiated from the DMZ toward the home network, the blast radius of any security incident is restricted strictly to physical interface `X3`. Your personal storage, documents, and domestic assets remain fully air-gapped from your laboratory experiments.
    

#### Philosophy Note E: The Zero-Trust Cleanup Standard (Rule 6)

- **Design Rationale:** This is the foundational catch-all implicit rule made explicit.
    
- **Security Outcome:** Anything that was not explicitly defined as safe in Rules 1–5 is categorized as an implicit threat and dropped immediately. This ensures that changes to the network architecture down the line will always default to a secure, completely closed posture rather than accidentally leaving data lines open.


# Architectural Security Essay: Implementing Least-Privilege and Default-Deny in Edge Infrastructure

When architecting a secure network perimeter, relying on reactive security measures is insufficient. A robust security posture requires structural paradigms built into the network's foundational logic. This writeup details the implementation of two core cybersecurity methodologies executed on the SonicWall TZ80 gateway: **The Principle of Least Privilege (PoLP)** and the **Default-Deny Posture**.

## 1. The Principle of Least Privilege (PoLP)

The Principle of Least Privilege dictates that any entity—be it a user, a workstation, an application, or an entire network segment—must be granted only the absolute minimum level of access necessary to complete its legitimate, authorized function. No more, no less.

### Implementation in the Lab Environment:

In a traditional, flat home network, all devices are granted omnidirectional access; a smart television can communicate with a network-attached storage device, and a guest laptop can scan a primary desktop. In this hardened architecture, PoLP was rigidly enforced to break this trust model:

- **Administrative Access Control:** Rather than allowing the entire Local Area Network (`192.168.8.0/24`) to access the lab management plane, named address objects were utilized to isolate management privileges strictly to the administrator's primary device (`Admin_Desktop_PC`).
    
- **Egress Traffic Throttling:** Lab hosts situated on the DMZ (`X3`) do not possess blanket access to the wide-area network. Instead, outbound traffic is limited to a strict array of essential infrastructure protocols grouped under `Lab_Management_Services` (such as DNS, NTP, and secure web repositories).
    
- **Temporal Limitations:** Access control is further bound by a schedule object (`Lab_Maint_Window`), ensuring that even permitted egress privileges are restricted outside of active administrative operational hours.
    

### Security Deficits Mitigated:

By limiting operational scope, PoLP minimizes the impact of an interior threat or configuration oversight. If an application running inside the lab segment is exploited, its capacity to download secondary malware payloads or beacon back to arbitrary external command-and-control (C2) servers is systemically throttled by the firewall engine.

## 2. The Default-Deny Security Posture

A Default-Deny posture (often referred to as an implicit deny standard) establishes that all network traffic crossing security zone boundaries is forbidden by default unless it matches an explicit, pre-defined rule permitting its passage.

```
       Incoming Packet
              │
              ▼
    Matches Explicit Allow? ──► YES ──► Forward Packet
              │
              ▼ NO
    Matches Explicit Deny?  ──► YES ──► Drop & Log Packet
              │
              ▼ NO
    [ IMPLICIT CLEANUP DENY ] ─────────► Drop Packet (Silent)
```

### Implementation in the Lab Environment:

This architecture incorporates an explicit cleanup rule at the bottom of the stateful access rule hierarchy. While SonicOS naturally executes an implicit drop between untrusted zones, creating an explicit **Any ➔ Any ➔ Deny** rule accomplishes two vital security goals:

- **Elimination of Blind Spots:** The default implicit behavior of network appliances often leaves traffic unlogged. By implementing an explicit cleanup rule and enabling stateful logging, all unmapped traffic attempts generate actionable log telemetry in the `System Logs` console.
    
- **Blast Radius Containment:** The absolute firewall boundary between the DMZ interface (`X3`) and the LAN interface (`X2`) is a physical manifestation of Default-Deny. The firewall assumes that any host sitting on the `192.168.20.0/24` subnet is inherently untrusted. Lateral movement attempts toward the local home network do not trigger an automated "allow internal routing" logic; instead, they hit an immutable block.
    

### Security Deficits Mitigated:

Default-Deny fundamentally changes the defense paradigm from reactive to proactive. If an administrator accidentally deploys a new service or forgets to secure a port on a lab target, the external world cannot discover or exploit that service. The firewall defaults to a closed posture, requiring an intentional, administrative action to grant visibility.

## 3. Threat Matrix & Defensive Interception Analysis

The intersection of these two principles creates a highly defensive environment. Based on empirical telemetry captured during the deployment phase, the table below maps real-world attack vectors against the architectural controls implemented on the SonicWall chassis:

|**Observed Threat Vector**|**Technical Mechanism**|**Structural Defense Component**|**Resulting Log Event**|
|---|---|---|---|
|**Lateral Pivot Attempt**|Compromised DMZ lab server attempts an internal port scan against the core LAN gateway (`192.168.8.1`).|**Default-Deny Boundary:** Zero explicit rules exist allowing DMZ-to-LAN initiation.|`ICMP packet dropped` via structural access policy.|
|**Automated WAN Perimeter Scan**|External malicious nodes launch automated wide-area scans against open ports on the public IP footprint.|**Threat Intel Hardening:** Manual objects block known scanning networks at the absolute edge.|`Alert: Responder from country blocked` (e.g., China/Russia mitigation).|
|**Unauthorized Management Attempt**|A non-administrative host on the internal network attempts an SSH or HTTP link to the lab hypervisor.|**Least-Privilege Isolation:** LAN-to-DMZ traffic drops all source nodes not inside the `Authorized_Admin_Hosts` group.|`Access Rule Deny` with immediate drop metrics.|

## Conclusion

Security cannot be treated as an additive feature; it must function as an architectural constraint. By implementing the Principle of Least Privilege and a strict Default-Deny posture, this network deployment successfully decouples trust from physical connectivity. The internal household assets are isolated from the lab playground, turning the SonicWall TZ80 into a highly secure, visible, and resilient edge gateway.
