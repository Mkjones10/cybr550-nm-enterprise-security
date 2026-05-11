# Northwestern Medicine — Enterprise Cybersecurity Program
**CYBR 550 | Maxine Jones | April–May 2026**

A four-assignment progressive build of a production-realistic enterprise cybersecurity program for Northwestern Medicine — one of the largest healthcare systems in the Chicago area, with 11 hospital campuses, 200+ outpatient sites, and 33,000+ employees.

Each assignment builds directly on the last, culminating in a complete architecture that spans asset management, network segmentation, layered security controls, and incident response.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Assignment 1 — Axonius Asset Management](#assignment-1--axonius-enterprise-asset-management-proposal)
- [Assignment 2 — Network Architecture](#assignment-2--enterprise-network-architecture-purdue-model)
- [Assignment 3 — Security Controls](#assignment-3--security-controls--defense-strategy)
- [Assignment 4 — Incident Response](#assignment-4--incident-response-playbook-ransomware)
- [Architecture Gap Analysis](#architecture-gap-analysis)
- [Compliance Alignment](#compliance-alignment)
- [Tools & Technologies](#tools--technologies-referenced)
- [Repository Structure](#repository-structure)

---

## Project Overview

Northwestern Medicine operates one of the most complex healthcare IT environments in the Midwest. The network spans an on-premises data center at the Galter Pavilion in Chicago, five satellite hospital campuses interconnected via managed MPLS WAN, an expanding telehealth platform, a Microsoft Azure Government cloud environment, and a large biomedical and OT device estate that includes infusion pumps, patient monitors, imaging systems, and building management infrastructure.

This project applies the Purdue Model for industrial control system segmentation — adapted for a healthcare context — to logically and physically separate Northwestern Medicine's clinical systems, enterprise IT, biomedical devices, and cloud infrastructure into six discrete security zones. Every design decision is grounded in the principle that in a healthcare environment, a cybersecurity failure is not just a data breach — it is a direct threat to patient care delivery and safety.

### Organization at a Glance

| Attribute | Detail |
|-----------|--------|
| Hospitals | 11 campuses |
| Outpatient sites | 200+ |
| Employees | 33,000+ |
| Primary data center | Galter Pavilion, Chicago (Tier III, N+1) |
| WAN | Managed MPLS, 100 Mbps – 1 Gbps per site |
| Cloud | Microsoft Azure Government |
| Cloud connectivity | Azure ExpressRoute 10 Gbps |
| EHR platform | Epic (Hyperdrive) |
| Security zones | 6 (Purdue Model adapted) |

---

## Assignment 1 — Axonius Enterprise Asset Management Proposal

### Problem Statement

Northwestern Medicine's security tooling was operationally siloed. CrowdStrike, Cisco ISE, Azure AD, Qualys, Intune, and ServiceNow each maintained independent asset records with no automated correlation between them. The result was a fragmented view of the environment that created four distinct risk conditions:

- **Lack of asset visibility** — no single source of truth for what devices existed on the network
- **Shadow IT and unmanaged devices** — devices present on the network with no security agent, no compliance state, and no owner
- **Security tools operating in silos** — analysts manually pivoting between tools during incidents, increasing response time
- **Expanding attack surface** — a growing biomedical device estate, remote access expansion, and cloud adoption adding assets faster than they could be tracked

### Proposed Solution: Axonius

Axonius was proposed as a cybersecurity asset management platform to serve as the unified inventory layer across all Northwestern Medicine environments. Rather than replacing existing tools, Axonius connects to them via API adapters and aggregates their asset data into a single correlated record per device — combining endpoint, identity, network, cloud, and vulnerability data into one view.

**How it works:**
1. Adapter connectors pull asset data from CrowdStrike, Cisco ISE, Azure AD, Qualys, Intune, and ServiceNow
2. Data is correlated and deduplicated into a unified asset inventory
3. The inventory is continuously updated in real time as device state changes
4. Enforcement Center actions can trigger automated responses — quarantine, notification, or ticketing — when devices fall out of compliance

**Deployment in this architecture:**
- Axonius host: `10.1.30.50` (VLAN 30 — SOC zone)
- Aggregates asset data from all six Purdue Model zones
- Feeds enriched asset context to Splunk SIEM for correlated alerting
- Feeds compliance state to Cisco ISE NAC for dynamic quarantine decisions

### Business Value

| Benefit | Detail |
|---------|--------|
| Unified asset inventory | Single correlated record across 7+ security tools |
| Real-time visibility | Continuous updates as device state changes |
| Faster incident response | SOC analysts get full asset context without manual pivoting |
| Compliance support | NIST SP 800-66, HIPAA, ISO 27001 asset inventory requirements |
| Shadow IT detection | Flags devices present on network but absent from security tools |

### Cost Estimate

| Deployment size | Estimated annual cost |
|----------------|-----------------------|
| Small | ~$50,000 |
| Mid-size (typical) | ~$90,000 |
| Large enterprise | $300,000+ |

Pricing is subscription-based. A 30-day trial is available.

---

## Assignment 2 — Enterprise Network Architecture (Purdue Model)

### Design Philosophy

The network architecture is modeled on the Purdue Model for industrial control system segmentation, adapted for a healthcare environment to logically and physically separate Northwestern Medicine's clinical systems, enterprise IT, biomedical and OT devices, and cloud infrastructure. The goals of this segmentation are to:

- Reduce Northwestern Medicine's attack surface
- Support regulatory compliance with HIPAA, NIST SP 800-66, and HICP
- Limit lateral movement in the event of a compromise
- Isolate high-value assets — Epic EHR, PACS, biomedical devices — from general enterprise traffic

Each zone is assigned dedicated IP subnets, VLAN ranges, and security controls aligned with Zero Trust segmentation and least privilege principles.

### Zone Architecture

#### Zone 5 — External / Internet (Untrusted)
**Public IP space: 165.124.x.x**

The untrusted perimeter. All external entities interact with Northwestern Medicine through this zone before any traffic reaches internal systems.

| Entity | Access method |
|--------|--------------|
| Remote clinicians | ZTNA / Ivanti Connect, MFA enforced |
| Telehealth patients | NM Virtual Visit (Zoom), TLS 1.3 + MFA |
| 3rd-party vendors | Epic/GE/Siemens support, PAM + time-limited access |
| Patient portal | MyNM (Epic MyChart), WAF + HTTPS |
| ISP / BGP | ATT / Comcast, dual-homed WAN |

All inbound and outbound traffic passes through a Palo Alto NGFW HA pair with IPS, URL filtering, SSL inspection, and anti-malware before reaching Zone 4.

---

#### Zone 4 — DMZ / Screened Subnet
**10.0.0.0/24 | VLAN 10**

A controlled buffer between the internet and internal systems. A dual-vendor firewall strategy (Palo Alto + Cisco Firepower) reduces single-vendor dependency and improves detection coverage.

| Component | Details |
|-----------|---------|
| Palo Alto NGFW | PA-5450 HA pair, deep packet inspection, App-ID + User-ID — `10.0.0.1 / 10.0.0.2` |
| F5 BIG-IP WAF | OWASP rule sets, API gateway (HL7/FHIR), TLS termination — `10.0.0.10` |
| Cisco Firepower IPS | Inline blocking mode, Snort 3 signatures, SSL decryption — `10.0.0.20` |
| Zscaler ZTNA | Zero trust proxy, app-level access only, no lateral movement — `10.0.0.30` |

---

#### Zone 3 — Enterprise IT / Corporate Network
**10.1.0.0/16 | VLANs 20–50**

Northwestern Medicine's enterprise IT environment. All enterprise governance, authentication, and device compliance functions originate from this zone. Logs from all other zones are forwarded here for centralized incident response and monitoring.

| System | Details |
|--------|---------|
| Identity / IAM | MS Azure AD, Okta SSO + MFA, CyberArk PAM — VLAN 20, `10.1.20.0/24` |
| SOC / SIEM | Splunk Enterprise, CrowdStrike EDR, Axonius asset mgmt — VLAN 30, `10.1.30.0/24` |
| Admin systems | MS 365 / Teams, Workday HR / Finance, staff workstations (~4,000 hosts) — VLAN 40, `10.1.40.0/22` |
| Core switching / Wi-Fi | Cisco Catalyst 9000, Cisco ISE NAC, Cisco Meraki Wi-Fi — VLAN 50, `10.1.50.0/24` |

---

#### Zone 2 — Clinical Network / EHR Systems
**10.2.0.0/16 | VLANs 60–90**

Critical clinical systems that support direct patient care. Traffic is heavily controlled and monitored to maintain the low latency and high availability required for clinical operations while enforcing ePHI protection requirements.

| System | Details |
|--------|---------|
| Epic EHR cluster | Epic Hyperdrive, RBAC + audit logging, PHI encrypted AES-256 — VLAN 60, `10.2.60.0/23` |
| Telehealth platform | NM Virtual Visit, Zoom for Healthcare, E2E encrypted sessions — VLAN 70, `10.2.70.0/24` |
| PACS / Imaging | Sectra PACS, MRI/CT/X-ray data, DICOM protocol only — VLAN 80, `10.2.80.0/24` |
| Clinical workstations | Nurse / MD stations (~1,200 hosts), smart card + MFA, CrowdStrike EDR — VLAN 90, `10.2.90.0/23` |

---

#### Zone 1 — Biomedical / OT / IoT
**10.3.0.0/16 | VLANs 100–140 | Air-gap adjacent**

Biomedical, OT, and IoT devices supporting patient care and facility operations. These devices frequently run proprietary or legacy operating systems that cannot be patched, making isolation — not patching — the primary security control.

| Device category | Details |
|----------------|---------|
| Infusion pumps | Baxter / BD Alaris, legacy firmware, NAC quarantine, no EHR direct link — VLAN 100 |
| Patient monitors | Philips IntelliVue, vitals telemetry, read-only feeds, encrypted HL7 — VLAN 110 |
| Imaging devices | Siemens MRI/CT, DICOM to PACS only, Win10 LTSC patching, isolated subnet — VLAN 120 |
| OT / BMS / HVAC | Siemens Desigo CC, air/power/elevators, Purdue L1 isolation, no IT crossover — VLAN 130 |
| IoT / smart devices | Smart beds (Hill-Rom), RTLS asset trackers, badge/access readers, Cisco ISE NAC enforced — VLAN 140 |

---

#### Zone 0 — Physical Infrastructure + Cloud
**Galter Pavilion Data Center + Satellite closets | Azure Gov: 172.16.0.0/16**

Physical infrastructure and cloud services that underpin all other zones.

**On-premises physical:**

| Component | Details |
|-----------|---------|
| Galter data center | Tier III, N+1 redundancy |
| UPS / generator | 72hr diesel, APC UPS |
| Badge / biometric | HID readers, mantrap |
| CCTV / env. sensors | Temp, humidity, water |

**Microsoft Azure Government cloud (172.16.0.0/16):**

| Service | Details |
|---------|---------|
| Epic cloud / HIE | Azure Health Data Svc, FHIR API gateway, AES-256 at rest, TLS 1.3 in transit — `172.16.10.0/24` |
| Backup / DR | Azure Site Recovery, immutable blob storage, geo-redundant (East US), RTO 4hr / RPO 1hr — `172.16.20.0/24` |
| Cloud SIEM / SOAR | Splunk Cloud, MS Sentinel, SOAR automation, threat intel (ISAC) — `172.16.30.0/24` |
| Telehealth SaaS | Zoom for Healthcare, HIPAA BAA in place, E2E encrypted, session logging — `172.16.40.0/24` |

Connectivity: Azure ExpressRoute 10 Gbps, private peering, SOC2 / HIPAA BAA signed.

### Multi-Campus Topology

All campuses are interconnected via managed MPLS WAN (100 Mbps – 1 Gbps per site):

- Prentice Women's Hospital
- Bluhm Cardiovascular Institute
- Lake Forest Hospital
- Delnor Hospital
- Kishwaukee Hospital

---

## Assignment 3 — Security Controls & Defense Strategy

### Core Security Principles

Every architectural decision in this design is grounded in five principles. In a healthcare environment, these are not optional best practices — a breach that disrupts Epic EHR or life-critical biomedical devices is a direct patient safety event.

#### Zero Trust (ZT)
No user, device, or application is trusted by default regardless of network location. Every access request must be authenticated, authorized, and continuously verified.

*Applied in this design:* Okta SSO + MFA on every login · Zscaler ZTNA replaces VPN · CyberArk enforces zero standing privileges for vendors and admins

#### Least Privilege (LP)
Every user, system, and application is granted only the minimum access required to perform its function.

*Applied in this design:* RBAC on Epic EHR limits PHI by role · ZTNA grants app-level access only · Vendor credentials expire on ticket close · Zone 1 devices cannot reach EHR directly

#### Segmentation (SG)
The network is divided into isolated zones with controlled communication paths between them, so a compromise in one area cannot automatically spread.

*Applied in this design:* Six Purdue Model zones with dedicated VLANs and subnets · Internal firewalls at every zone boundary · Zone 1 biomedical devices air-gap adjacent

#### Defense in Depth (DD)
Multiple overlapping layers of security controls are deployed so that if one layer fails, others remain in place to detect, contain, or stop a threat.

*Applied in this design:* NGFW → IPS → ZTNA → Segmentation FW → NAC → EDR → SIEM — an attacker must defeat every layer independently

#### Visibility Everywhere (VE)
The security team maintains continuous awareness of every asset, session, and event across all zones — on-premises and cloud — so no blind spots exist.

*Applied in this design:* Axonius unified inventory across all VLANs · Splunk aggregates logs from all zones · Sentinel forwards cloud alerts to on-prem SOC · H-ISAC threat intel integrated

---

### Security Controls by Layer

#### Network Security

| Control | Details |
|---------|---------|
| Palo Alto NGFW HA | PA-5450 pair at Zone 5/4 boundary |
| Cisco Firepower IPS | Inline blocking, Snort 3 signatures |
| F5 BIG-IP WAF | OWASP rules + FHIR/HL7 API gateway |
| Zscaler ZTNA | App-level access, no lateral movement |
| Cisco Catalyst 9000 + ISE NAC | Port-level enforcement |
| Microsegmentation | Between all Purdue Model zones |
| Dual-vendor firewall | Palo Alto + Cisco — no single-vendor risk |

#### Endpoint Security

| Control | Details |
|---------|---------|
| CrowdStrike Falcon EDR | All enterprise + clinical endpoints |
| SCCM + Intune + JAMF | Patch management, device compliance |
| Cisco ISE NAC | Device posture check before network access |
| Axonius | Continuous asset inventory across all VLANs |
| Smart card + MFA | All clinical workstations |
| Legacy biomedical isolation | Zone 1 VLANs |
| Passive NDR | Monitor OT/IoT without agents |

#### Identity & Access

| Control | Details |
|---------|---------|
| Azure AD + Okta SSO | Enterprise identity |
| MFA | Enforced for all remote access + vendor sessions |
| CyberArk PAM | Privileged + vendor session recording |
| RBAC on Epic EHR | Role-limited PHI access |
| Time-limited vendor credentials | Via CyberArk, expire on ticket close |
| Conditional access policies | Device posture required |
| Zero standing privileges | All third-party accounts |

#### Monitoring & Detection

| Control | Details |
|---------|---------|
| Splunk SIEM | On-prem log aggregation, all zones |
| Microsoft Sentinel | Cloud SIEM + SOAR automation |
| Axonius | Real-time asset state + security gap alerts |
| H-ISAC threat intel | Integrated into SIEM |
| Cloud telemetry | Azure forwarded to on-prem SOC via ExpressRoute |

---

### Risk Analysis

| Severity | Risk | Mitigation |
|----------|------|-----------|
| HIGH | Ransomware lateral movement | Microsegmentation + ISE NAC limits blast radius to one VLAN; Zone 1 air-gap prevents OT pivot |
| HIGH | Compromised vendor credential | ZTNA grants app-only access; CyberArk records session, auto-revokes on ticket close |
| HIGH | PHI exfiltration via EHR | RBAC limits PHI access by role; Splunk SIEM alerts on anomalous Epic access patterns |
| HIGH | Command-and-control callback | NGFW + IPS inspects all outbound; Zscaler proxy blocks unauthorized C2 communications |
| MED | Unmanaged / shadow IoT device | Cisco ISE NAC quarantines non-compliant devices; Axonius flags unknowns in real time |
| MED | Legacy biomedical device exploit | Zone 1 isolation + ACLs + passive NDR; containment is the control, not patching |

---

### Key Enforcement Points

| Boundary | Control | Function |
|----------|---------|----------|
| Z5 → Z4 | Palo Alto NGFW | First inspection; all traffic blocked by default |
| Z4 DMZ | IPS + WAF + ZTNA | Threat detection, API security, vendor proxy |
| Z4 → Z3 | Internal FW | Enforces trust boundary between DMZ and enterprise IT |
| Z3 → Z2 | Segmentation FW | Protects clinical zone from enterprise zone |
| Z2 → Z1 | Microsegment FW + NAC | Isolates biomedical devices from clinical IT |
| Z3 SOC | Splunk + Sentinel | Central log aggregation from all zones |
| Cloud | ExpressRoute + Defender | Private connectivity, cloud-side EDR + SIEM |

---

### Design Trade-offs

| Decision | Trade-off |
|----------|-----------|
| ZTNA vs. VPN | Higher complexity, but eliminates network-wide exposure from a single compromised VPN credential |
| Dual-vendor firewall | Increased operational overhead, but eliminates single-vendor risk and improves detection coverage |
| Agent-less Zone 1 | Lower endpoint visibility, compensated by passive NDR, NAC isolation, and Axonius tracking |
| ExpressRoute over public internet | Higher cost, but PHI never traverses untrusted networks |
| No perimeter-only trust | More tools to manage, but significantly reduces blast radius if the outer firewall is bypassed |

---

## Assignment 4 — Incident Response Playbook: Ransomware

### Incident Overview

**Scenario:** A staff member in the Enterprise IT zone (Zone 3) opens a phishing email attachment, executing a ransomware payload on an administrative workstation on VLAN 40 (Admin Systems, `10.1.40.0/22`). The malware begins encrypting local files and attempts lateral movement via SMB across the ~4,000-host subnet. Initial detection is generated by CrowdStrike Falcon EDR, with corroborating signals from Splunk SIEM and Cisco ISE NAC within minutes.

**Critical boundary:** The Zone 3 → Zone 2 segmentation firewall at `10.2.0.0/16` determines whether this incident remains an IT recovery event or escalates into a clinical disruption affecting Epic EHR, PACS, and ~1,200 clinical workstations — with associated HIPAA breach liability.

---

### Detection & Visibility

Multiple overlapping detection systems generate independent signals within minutes of initial execution, reflecting the defense-in-depth design from Assignment 3.

| System | Detection signal |
|--------|----------------|
| CrowdStrike Falcon EDR | Identifies malicious process tree (Office doc → PowerShell/cmd → mass file rename); generates high-severity alert |
| Splunk SIEM (`10.1.30.10`) | Correlates high-volume file modification events, anomalous SMB connection attempts, unusual authentication activity |
| Cisco ISE NAC | Identifies abnormal SMB connection attempts from a previously compliant device; flags posture violation; triggers switchport quarantine |
| Axonius (`10.1.30.50`) | Surfaces full asset profile of affected device across all integrated data sources instantly |
| Palo Alto NGFW + Zscaler ZTNA | Log and block outbound C2 callback attempts |
| Segmentation FW (Z3 → Z2) | Generates deny logs in Splunk for any cross-zone communication attempts, confirming clinical network was not reached |

---

### Response Playbook

#### Phase 1 — Initial Triage

- Confirm CrowdStrike high-severity alert and Splunk correlation alert are tied to the same source IP on `10.1.40.x` before escalating
- Pull full asset record from Axonius for affected device: hostname, assigned user, VLAN placement, installed agents, last-known compliance state
- Review Splunk for past 24 hours of endpoint activity: authentication logs, SMB connection attempts to adjacent hosts, outbound traffic flagged by Palo Alto NGFW
- Query Splunk for any other `10.1.40.x` devices showing similar process behavior or file encryption indicators within the last 30–60 minutes
- Escalate immediately to Incident Response lead and notify CISO if more than one host is confirmed affected

#### Phase 2 — Containment

- Activate CrowdStrike Falcon network containment on infected workstation to cut all network communication while preserving host for forensic analysis
- If CrowdStrike containment is unavailable, direct network team to disable device switchport on Cisco Catalyst 9000 via Cisco ISE NAC
- Verify through Splunk firewall deny logs that Zone 3 → Zone 2 segmentation firewall has blocked all lateral movement attempts toward `10.2.0.0/16`
- Query Axonius for all VLAN 40 devices sharing the compromised user's credentials or reflecting recent authentication from that identity; disable associated Azure AD account immediately
- If compromised user held privileged access, direct CyberArk PAM team to rotate and revoke all associated credentials and terminate active privileged sessions

#### Phase 3 — Investigation

- Collect memory image and forensic disk copy via CrowdStrike Falcon remote forensics before the device is reimaged
- Review Splunk for complete lateral movement path: which hosts the malware attempted to reach, which connections succeeded, which were blocked by ISE NAC or segmentation firewall
- Pull Microsoft 365 email logs to identify original phishing message, determine if forwarded to other devices, and identify other staff who received the same email
- Check Palo Alto NGFW outbound logs for any successful C2 callbacks before containment — if callback succeeded, elevate incident scope to include potential data exfiltration
- Review Azure AD sign-in logs for anomalous authentication activity from compromised account, including access to M365 services or SharePoint

#### Phase 4 — Communication

- Notify CISO and IT leadership within the first hour of confirmed ransomware activity — do not delay pending full scope determination
- If any clinical systems were reached or PHI is potentially at risk, notify Chief Medical Officer and initiate a HIPAA breach assessment per 45 C.F.R. § 164.306
- Coordinate network team, SOC, and IAM team in parallel — all three are needed simultaneously for effective containment
- Open a formal incident ticket in ServiceNow and maintain a running timeline of all actions, decisions, and personnel involved

#### Phase 5 — Recovery

- Restore affected workstation from most recent verified clean backup using SCCM and Intune, which support remote reimaging without physical access
- Require Cisco ISE NAC to perform a full posture check on any recovered device before network reconnection, confirming it is fully patched and running a current CrowdStrike Falcon agent
- Re-enable compromised Azure AD account only after infection vector is fully addressed, MFA re-enrolled on a clean device, and affected user has completed targeted phishing awareness review
- Conduct follow-up Axonius sweep across all of VLAN 40 for remaining devices with signs of compromise, missing agents, or compliance gaps
- Document complete incident timeline, detection-to-containment gap, all affected assets, and lessons learned in both Splunk and ServiceNow

---

### Post-Incident Reflection

The architecture performed well. Three independent detection signals (CrowdStrike, Splunk, ISE NAC) fired within minutes of initial infection. The Zone 3 → Zone 2 segmentation firewall proved to be the single most critical patient safety control in the entire architecture — without it, this incident would have escalated from an IT recovery event into a potential HIPAA breach affecting Epic EHR and 1,200 clinical workstations.

---

## Architecture Gap Analysis

| Gap | Risk level | Impact | Recommendation |
|-----|-----------|--------|----------------|
| VLAN 40 internal flatness (~4,000 hosts) | High | Ransomware can spread between workstations within the subnet before zone-level segmentation engages | Implement intra-VLAN microsegmentation on VLAN 40 so workstations cannot communicate directly outside explicitly approved traffic paths |
| No email attachment sandboxing | High | Malicious payloads reach the endpoint before any detection layer fires, requiring reactive containment rather than prevention | Deploy Microsoft Defender for Office 365 attachment sandboxing at the M365 layer to stop payloads before endpoint execution |
| Zone 1 EDR coverage gap | Medium | Legacy biomedical devices cannot run security agents, reducing endpoint visibility in the highest-risk device category | Current mitigation: passive NDR + NAC isolation + Axonius tracking; evaluate hardware security modules for highest-criticality devices |

---

## Compliance Alignment

| Framework | Application in this design |
|-----------|---------------------------|
| HIPAA Security Rule (45 C.F.R. § 164.306) | Zone segmentation, PHI encryption (AES-256), access controls, audit logging, breach notification procedures |
| NIST SP 800-66 Rev. 2 | Administrative, physical, and technical safeguard implementation across all zones |
| HICP (HHS 405(d)) | Email protection, endpoint protection, access management, network segmentation, incident response |
| ISA/IEC 62443 (Purdue Model) | Zone and conduit model for OT/biomedical device segmentation in Zone 1 |

---

## Tools & Technologies Referenced

### Security Operations
`CrowdStrike Falcon EDR` `Splunk Enterprise` `Microsoft Sentinel` `Axonius` `H-ISAC Threat Intel`

### Network Security
`Palo Alto NGFW (PA-5450)` `Cisco Firepower IPS` `F5 BIG-IP WAF` `Zscaler ZTNA` `Cisco ISE NAC` `Cisco Catalyst 9000` `Cisco Meraki Wi-Fi`

### Identity & Access
`Microsoft Azure Active Directory` `Okta SSO` `CyberArk PAM` `Microsoft Intune`

### Endpoint & Device Management
`SCCM` `JAMF` `Qualys`

### Clinical Systems
`Epic EHR (Hyperdrive)` `Sectra PACS` `Zoom for Healthcare` `Siemens MRI/CT` `Philips IntelliVue` `Baxter / BD Alaris` `Siemens Desigo CC`

### Cloud & Infrastructure
`Microsoft Azure Government` `Azure ExpressRoute` `Azure Site Recovery` `Azure Health Data Services` `Microsoft Defender for Cloud` `ServiceNow`

---

## Repository Structure

```
cybr550-nm-enterprise-security/
├── README.md
├── Assignment-1-Axonius/
│   └── Maxine_Jones_Enterprise_Assignment_1.pdf
├── Assignment-2-Network-Architecture/
│   ├── Maxine_Jones_Assignment_2_CYBR_550.pdf
│   ├── northwestern_medicine_network_map.jpg
│   └── northwestern_medicine_security_overlay.svg
├── Assignment-3-Security-Overlay/
│   └── Maxine_Jones_Assignment_3_CYBR_550.pdf
├── Assignment-4-Incident-Response/
│   └── Maxine_Jones_Assignment_4_CYBR_550.pdf
└── Executive-Summary/
    └── NM_CYBR550_Executive_Summary.pptx
```

---

## References

- American Hospital Association. (2025). *Cybersecurity and the healthcare sector.* https://www.aha.org/cybersecurity
- Cybersecurity and Infrastructure Security Agency. (2024). *#StopRansomware guide.* https://www.cisa.gov/stopransomware
- International Society of Automation. (n.d.). *ISA-95: Enterprise-Control System Integration.* https://www.isa.org
- National Institute of Standards and Technology. (2024). *Implementing the HIPAA Security Rule (SP 800-66 Rev. 2).* https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-66r2.pdf
- U.S. Department of Health and Human Services. (2023). *Health Industry Cybersecurity Practices (HICP).* https://405d.hhs.gov/Documents/HICP-Main-508.pdf
- U.S. Department of Health and Human Services. (2023). *HIPAA Security Rule: 45 C.F.R. § 164.306.* https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-C

---

*Prepared for CYBR 550 — Enterprise Cybersecurity. All network designs, IP addressing, and tool configurations are academic representations prepared for coursework purposes.*
