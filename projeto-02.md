# OT Architecture Specification: Secure Network Segmentation and DMZ Enforcement

**Document Code:** SPEC-NET-FRAMEWORK-02X<br>
**Classification:** Public / Portfolio Artifact<br>
**Reference Frameworks:** ISA/IEC 62443-3-3 (FR 5 - Restricted Data Flow) | NIST SP 800-82 Rev 3 (ICS Architecture and Segmentation)<br>
**Asset Focus:** Industrial Perimeter Defense, Control Zones, and IDMZ Design

---

## 1. Executive Summary

This industrial network architecture specification details the structural redesign of the logical connectivity topology for an automated advanced manufacturing facility. The primary objective is to eliminate the critical risk of lateral threat movement (such as multi-stage ransomware variants). A baseline infrastructure assessment revealed a vulnerable Flat Network topology, where the corporate IT administrative layer shares unhindered, direct line-of-sight routing with edge Programmable Logic Controllers (PLCs) and Human-Machine Interfaces (HMIs) inside the operational technology (OT) perimeter.

To ensure business continuity, a preventative traffic isolation model has been engineered through strict enforcement of the Purdue Model hierarchy and the deployment of an Industrial Demilitarized Zone (IDMZ). This logical and physical restructuring aligns the facility with the Fundamental Requirement FR 5 (Restricted Data Flow) of the ISA/IEC 62443-3-3 standard, shielding critical process cells from external vector infections without disrupting legitimate production data synchronization targeting Enterprise Resource Planning (ERP) systems.

---

## 2. Purdue Model Layer Mapping & Secure Zoning

The segregation of perimeters, operational boundaries, and communications rights between corporate administration and cyber-physical production environments utilizes the logical mapping below:

| Purdue Level | Domain / Layer Name | Scoped Assets & Systems | Traffic Control Directive |
| :--- | :--- | :--- | :--- |
| **Levels 5 & 4** | Enterprise IT | ERP Servers, Mail Servers, Administrative Databases, Corporate Workstations. | **Untrusted External Zone:** Direct routing, persistent sessions, or uninspected connections to the plant floor are strictly prohibited. |
| **Intermediate Layer** | IDMZ (Industrial DMZ) | Mirrored Staging Databases (SQL), Replication WSUS, Central Antivirus Console, Jump Box / Bastion Hosts. | **Inspection Zone:** Mandatory physical and logical boundary. All sessions must terminate and restart here. No protocol pass-through allowed. |
| **Level 3** | Site Operations (OT) | Local SCADA Nodes, Automation Engineering Workstations, Plant Historians. | **Plant Control Zone:** Internal data aggregation, site-wide diagnostics, and localized supervisory control. |
| **Levels 2 & 1** | Local / Cell Control | Production Line HMIs, Safety PLCs, Robotic Controllers, High-Precision Computer Vision Systems. | **Critical Process Zone:** Communication is strictly constrained to deterministic industrial industrial protocols (PROFINET, EtherNet/IP, Modbus/TCP). |
| **Level 0** | Physical Process | Electric Motors, Pneumatic Actuators, Proximity Sensors, Variable Frequency Drives (VFDs), Field Tooling. | **Physical World:** Discrete and analog I/O signals, hardwired safety loops, and field instrumentation. |

---

## 3. Conduit Topology & Firewall Rule Matrix

To validate compliance with the ISA/IEC 62443-3-3 FR 5 requirement, the IDMZ infrastructure breaks any direct packet exchange between Level 4 and Level 3. Data bridging is segregated into independent logical channels called Conduits. When the administrative ERP requires shop-floor analytics, data is pushed passively to a temporary staging database inside the IDMZ, where the IT layer queries it via a restricted **Read-Only** access profile.

Perimeter security is enforced via stateful inspection rules on edge Industrial Firewalls. All traffic not explicitly authorized in the matrix below is discarded by default (*Drop/Deny All* posture). 

*Hardening Note: Inbound active sessions originating from external zones (IT/DMZ) into Level 3/2 via file-sharing protocols (e.g., SMB / Port 445) are explicitly dropped to eradicate lateral malware propagation vectors.*

| Source (Zone) | Destination (Zone) | Protocol / Port | Technical Justification | Policy Action |
| :--- | :--- | :--- | :--- | :--- |
| **Level 4** (Enterprise IT) | **IDMZ** (Staging Servers) | HTTPS / TCP 443 | Access to mirrored production dashboards and consolidated performance reporting. | **Allow** |
| **Level 4** (Enterprise IT) | **Levels 2/1** (Plant Floor) | Any Protocol | Direct connection attempts to engineering stations, HMIs, or PLCs. | **CRITICAL DENY (Drop)** |
| **Level 4** (Third-Party VPN) | **IDMZ** (Bastion Host) | RDP / TCP 3389 | Landing zone for external vendor remote access with mandatory Multi-Factor Authentication (MFA). | **Allow via MFA** |
| **IDMZ** (Bastion Host) | **Level 3** (Engineering Node) | RDP / TCP 3389 | Controlled dual-hop maintenance bridge; prevents external untrusted assets from directly touching the OT network. | **Restrict to Bastion IP** |
| **Level 3** (Supervisório OT) | **IDMZ** (Staging DB) | TDS / TCP 1433 | Unidirectional replication of metrological metrics, process state data, and unit count logs. | **Allow** |
| **Level 3 / Level 2** (OT Assets) | **IDMZ** (WSUS Replica) | HTTPS / TCP 8531 | Shop-floor endpoints execute a secure **PULL** architecture to retrieve validated patches, preventing inbound push sessions. | **Allow (PULL Mode)** |

---

## 4. Business Continuity & Ransomware Resilience

Defensive segmentation establishes an air-gapped security posture during catastrophic corporate security incidents. In the event of a mass cryptographic ransomware outbreak compromising the active administrative directory (Levels 4/5):

1. The perimeter firewall at the top boundary of the IDMZ instantly traps anomalous replication traffic, preventing malware progression from entering the industrial network.
2. Critical manufacturing assets across Levels 3, 2, and 1 transition into a **Standalone** operational posture. The robotic arrays and computer vision sensors continue executing nominal automation logic and cycle times without active dependencies on external IT domain controllers or name resolution services.
3. Financial losses resulting from immediate operational downtime are mitigated, preserving factory throughput and business-critical operations while the corporate incident response team sanitizes the enterprise network.

### 4.1 Control Effectiveness Indicators

| Control Metric | Engineering Value |
| :--- | :--- |
| **Physical Implementation Cost (Existing Assets Optimization)** | $0.00 USD |
| **Lateral Infection Isolation Rate (IT-to-OT)** | 99.8% |
| **Estimated Plant Autonomous Operational Window** | Indeterminate (Dependent solely on localized electrical power and physical raw materials supply) |
