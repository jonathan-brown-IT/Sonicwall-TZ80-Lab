## Phase 17: Centralized Telemetry Export & SIEM Integration

To establish an immutable audit trail and enable long-term security analytics, the SonicWall TZ80 was configured to stream its real-time event logs to an enterprise open-source SIEM platform (Wazuh).

### 1. Syslog Forwarding Profile
* **Target SIEM Server IP:** `192.168.8.50` *(Adjust to match your specific SIEM deployment node)*
* **Transport Protocol / Port:** UDP `1514` (Wazuh Default Listener) / UDP `514` (Standard Graylog)
* **Log Payload Format:** Enhanced Syslog (Key-Value Pair Format)
* **Facility Code:** Local 0

### 2. Operational Flow Analysis
Unlike local firewall memory buffers which overwrite dynamically once capacity is reached, syslog streaming functions as an unbuffered, real-time push mechanism. 

```text
Sample Export Frame:
<134>id=firewall sn=1234567890AB time="2026-06-26 11:15:00" fw=192.168.8.1 pri=5 c=64 m=97 msg="Connection Denied" src=182.112.97.44:8081 dst=192.168.20.10:8081 proto=tcp
