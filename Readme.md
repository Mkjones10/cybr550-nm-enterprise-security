# Northwestern Medicine — Enterprise Cybersecurity Program
**CYBR 550 | Maxine Jones | April–May 2026**

A four-assignment progressive build of a production-realistic enterprise cybersecurity program for Northwestern Medicine — one of the largest healthcare systems in the Chicago area, with 11 hospital campuses, 200+ outpatient sites, and 33,000+ employees.

Each assignment builds directly on the last, culminating in a complete architecture that spans asset management, network segmentation, layered security controls, and incident response.

---

## Assignments

### Assignment 1 — Axonius Enterprise Asset Management Proposal
Identified Northwestern Medicine's core asset visibility problem: disconnected security tools operating in silos with no unified inventory across endpoints, cloud, OT, and biomedical devices. Proposed Axonius as a cybersecurity asset management platform to aggregate data from CrowdStrike, Cisco ISE, Azure AD, Qualys, Intune, and ServiceNow into a single correlated inventory hosted on VLAN 30 (10.1.30.50).

**Key outcomes:** Unified asset graph across 7+ security tools · Real-time compliance gap detection · Foundation for all subsequent assignments

---

### Assignment 2 — Enterprise Network Architecture (Purdue Model)
Designed a six-zone Purdue Model network architecture adapted for a healthcare environment, with dedicated VLANs, IP subnets, and segmentation controls aligned to HIPAA, NIST SP 800-66, and HICP guidelines.

| Zone | Name | Subnet | VLANs |
|------|------|--------|-------|
| Zone 5 | External / Internet (untrusted) | 165.124.x.x | — |
| Zone 4 | DMZ / Screened Subnet | 10.0.0.0/24 | 10 |
| Zone 3 | Enterprise IT / Corporate | 10.1.0.0/16 | 20–50 |
| Zone 2 | Clinical Network / EHR | 10.2.0.0/16 | 60–90 |
| Zone 1 | Biomedical / OT / IoT | 10.3.0.0/16 | 100–140 |
| Zone 0 | Physical + Cloud (Azure Gov) | 172.16.0.0/16 | — |

**Key outcomes:** Full multi-campus topology across 5 hospital sites · Air-gap adjacent Zone 1 for biomedical devices · Azure ExpressRoute hybrid connectivity

---

### Assignment 3 — Security Controls & Defense Strategy
Overlaid a layered security architecture across all six zones, grounded in five core principles: Zero Trust, Least Privilege, Segmentation, Defense in Depth, and Visibility Everywhere.

**Controls by layer:**
- **Network:** Palo Alto NGFW HA · Cisco Firepower IPS (Snort 3) · F5 BIG-IP WAF · Zscaler ZTNA · Cisco ISE NAC
- **Endpoint:** CrowdStrike Falcon EDR · SCCM + Intune + JAMF · Passive NDR (Zone 1)
- **Identity:** Azure AD + Okta SSO · CyberArk PAM · MFA · RBAC on Epic EHR
- **Monitoring:** Splunk SIEM · Microsoft Sentinel · Axonius · H-ISAC threat intel

**Key outcomes:** Every zone boundary treated as a patient safety boundary · Dual-vendor firewall strategy · Zero standing privileges for third-party accounts

---

### Assignment 4 — Incident Response Playbook: Ransomware
Developed a full incident response playbook for a ransomware scenario originating on VLAN 40 (Admin workstations, 10.1.40.x) in Zone 3, with lateral movement attempts toward the Zone 2 clinical network.

**Playbook phases:**
1. **Triage** — CrowdStrike + Splunk alert correlation, Axonius asset pull, scope determination
2. **Containment** — CrowdStrike network isolation, ISE NAC switchport quarantine, Azure AD account disable, CyberArk credential revocation
3. **Investigation** — Forensic disk image, Splunk lateral movement analysis, M365 phishing source, NGFW C2 log review
4. **Recovery** — SCCM/Intune reimaging, ISE NAC posture check, MFA re-enrollment, VLAN 40 Axonius sweep

**Key finding:** The Zone 3 → Zone 2 segmentation firewall was the single most critical architectural control — the difference between an IT recovery event and a HIPAA breach affecting Epic EHR and ~1,200 clinical workstations.

---

## Architecture Gap Analysis

| Gap | Risk | Recommendation |
|-----|------|----------------|
| VLAN 40 internal flatness (~4,000 hosts) | Ransomware can move laterally within the subnet before zone-level controls engage | Implement intra-VLAN microsegmentation on VLAN 40 |
| Email attachment prevention | Malicious payloads reach the endpoint before any detection layer fires | Deploy Microsoft Defender for Office 365 attachment sandboxing at the M365 layer |
| Zone 1 EDR coverage | Legacy biomedical devices cannot run agents | Passive NDR + NAC isolation + Axonius tracking; evaluate hardware security modules for critical devices |

---

## Compliance Alignment

- **HIPAA Security Rule** — 45 C.F.R. § 164.306
- **NIST SP 800-66 Rev. 2** — Implementing the HIPAA Security Rule
- **HICP** — Health Industry Cybersecurity Practices (HHS 405(d))
- **ISA/IEC 62443** — Purdue Model zone segmentation reference

---

## Tools & Technologies Referenced

`Axonius` `CrowdStrike Falcon` `Splunk Enterprise` `Microsoft Sentinel` `Palo Alto NGFW` `Cisco Firepower IPS` `F5 BIG-IP WAF` `Zscaler ZTNA` `Cisco ISE NAC` `Cisco Catalyst 9000` `CyberArk PAM` `Azure Active Directory` `Okta SSO` `Epic EHR` `Sectra PACS` `Microsoft Azure Government` `Azure ExpressRoute` `Azure Site Recovery` `SCCM` `Intune` `ServiceNow`

---

## Repository Structure
