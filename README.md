🔒 Enterprise-Grade Edge Security & Infrastructure Micro-Segmentation Lab

This repository documents the structural engineering, deployment, and security auditing of an enterprise-grade perimeter topology using a physical SonicWall TZ80 Next-Generation Firewall (NGFW) running SonicOS 8.

The core objective of this project is to successfully decouple a highly experimental laboratory environment from a trusted domestic production network, establishing an absolute zero-trust posture across critical network boundaries.
Key Architectural Implementations:

    Zone Micro-Segmentation: Designed and deployed physical interface mappings isolating the internal LAN Zone from an untrusted, sandbox DMZ Zone to ensure strict blast radius containment.

    Least-Privilege Egress Controls: Implemented customized Address Objects and Service Groups bound to recurring Schedule Objects, throttling outbound laboratory traffic strictly to specified maintenance windows and authorized services.

    Perimeter Hardening & Threat Intelligence: Configured proactive Geo-IP and Threat-Intel blocking profiles to drop unsolicited inbound malicious reconnaissance probes directly at the wide-area network edge.

    Defensive Workarounds & Compensating Controls: Addressed real-world hardware evaluation licensing constraints by designing and executing elite tier-2 compensating controls, including DNS-layer category filtering and endpoint antimalware verification (EICAR).

    Centralized Logging & SIEM Integration: Configured real-time, event-driven Enhanced Syslog streaming to an external open-source SIEM framework (Wazuh/Graylog) for unbuffered telemetry retention and log analytics.

    Bare-Metal Penetration Auditing: Conducted real-world, cross-zone lateral movement testing using a physical host to verify the integrity of the stateful packet inspection engine and management plane isolation.

    Architectural Philosophy: This project treats network security not as an additive feature, but as an absolute architectural constraint. It serves as a comprehensive portfolio demonstrating advanced skills in firewall routing matrices, threat mitigation, security auditing, and professional technical documentation.
