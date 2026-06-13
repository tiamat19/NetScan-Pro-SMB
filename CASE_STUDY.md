 Case Study — Small Office Network Scan
 
**Environment:** Small office / home office (SOHO)  
**Devices:** 8  
**Scan date:** June 2026  
**Tool version:** NetScan Pro SMB v1.1.0  
**Scan duration:** ~5 minutes  
 
> All IP addresses, MAC addresses and hostnames have been anonymized.
> Device types and findings are real.
 
---
 
## Network Overview
 
| Metric | Value |
|---|---|
| Subnet | 192.168.x.0/24 |
| Devices discovered | 8 |
| Open ports | 10 |
| Overall score | 59/100 — Grade D |
| HIGH risks | 3 |
| MEDIUM risks | 2 |
| CVEs flagged | 3 |
| NIS2 compliance | 7/10 (70%) |
 
---
 
## Devices Found
 
| Device type | Vendor (detected) | Open ports | Score |
|---|---|---|---|
| Router/modem | Zyxel | 21, 53, 80, 139, 263, 443, 445 | 14/100 F |
| Windows PC | — | 135, 139, 445 | 33/100 F |
| IoT device | Tuya Smart | — | 67/100 C |
| IoT device | AMPAK Technology | — | 65/100 C |
| Smartphone | — | — | 75/100 C |
| Smartphone | — | — | 75/100 C |
| Smartphone | Nintendo | — | 75/100 C |
| Smartphone | — | — | 75/100 C |
 
---
 
## Key Findings
 
### 🔴 HIGH — SMB active on router (port 445)
The router exposes SMB on port 445.  
CVE mapped: CVE-2016-2110 (NTLM authentication bypass).  
NIS2 control: PATCH-01 (vulnerability & patch management).  
**Fix:** verify SMBv1 is disabled; block port 445 at the perimeter if SMB is not needed externally.
 
### 🔴 HIGH — SMB active on Windows PC (port 445)
The Windows workstation exposes SMB with NetBIOS (port 139).  
No CVE confirmed (version not fingerprinted), but SMBv1 status unknown.  
NIS2 control: PATCH-01.  
**Fix:** `Set-SmbServerConfiguration -EnableSMB1Protocol $false`
 
### 🔴 HIGH — Open DNS resolver (port 53)
The router acts as a recursive DNS resolver accessible on the LAN.  
Risk: DNS amplification, tunneling, data exfiltration.  
**Fix:** restrict recursive queries to internal clients only.
 
### 🟡 MEDIUM — FTP active on router (port 21)
FTP transmits credentials and file contents in cleartext.  
NIS2 control: ENC-01 (mandatory communication encryption).  
**Fix:** replace with SFTP (port 22) or FTPS (port 990).
 
### 🟡 MEDIUM — NetBIOS active on Windows PC (port 139)
NetBIOS exposes hostname and workgroup information.  
Low direct attack risk on internal networks, but increases reconnaissance surface.  
NIS2 control: NET-01 (network segmentation & protection).
 
---
 
## NIS2 Gap Analysis Results
 
| Control | Status |
|---|---|
| INV-01 Network asset inventory | ✅ Compliant |
| MON-01 Continuous network monitoring | ✅ Compliant |
| ACC-02 Remote access control | ✅ Compliant |
| ACC-03 Secure admin service authentication | ✅ Compliant |
| ENC-01 Mandatory communication encryption | ❌ Non-compliant (FTP) |
| ENC-02 HTTPS for web admin panels | ✅ Compliant |
| PATCH-01 Vulnerability & patch management | ❌ Non-compliant (SMB) |
| DB-01 Data & database protection | ✅ Compliant |
| NET-01 Network segmentation & protection | ❌ Non-compliant (NetBIOS) |
| NET-02 Attack surface reduction | ✅ Compliant |
 
---
 
## Economic Risk Estimate
 
| Risk scenario | Min (EUR) | Max (EUR) |
|---|---|---|
| Fixed costs (downtime, GDPR, crisis management) | 15,000 | 55,000 |
| Malware propagation (if SMBv1 active) | 15,000 | 90,000 |
| Data exfiltration via FTP | 7,000 | 40,000 |
| Network reconnaissance (info exposure) | 1,500 | 8,000 |
| **ESTIMATED TOTAL** | **38,500** | **193,000** |
 
*Source: Clusit 2025 · IBM Cost of Data Breach 2024 · Yarix Y-Report 2024*  
*Note: estimates based on Italian SMB average breach cost (EUR 95,000). Not a certified risk assessment.*
 
---
 
## Cyber Insurance Readiness
 
**Status: ELIGIBLE (80%)**
 
| Insurer-required control | Status |
|---|---|
| No exposed RDP | ✅ Compliant |
| Patch management (no critical CVEs) | ✅ Compliant |
| No obsolete protocols (Telnet/FTP) | ⚠️ FTP active |
| SMBv1 disabled / SMB not high-risk | ❌ Non-compliant |
| Documented vulnerability assessment | ✅ This report |
| Up-to-date asset inventory | ✅ Compliant |
 
> ⚠️ High-risk SMB detected: insurers typically exclude ransomware damage
> when SMBv1 is accessible. Resolve SMB before applying for a policy.
 
---
 
## What This Scan Found vs What It Did Not
 
**Found:**
- All 8 devices on the subnet
- 10 open ports across 2 devices
- 3 service-level CVE associations (heuristic, not version-confirmed)
- 3 NIS2 control gaps
- 1 unidentified service on port 263 (unknown process on router)
**Not confirmed:**
- Actual SMBv1 status (would require active test or router access)
- Whether CVE-2016-2110 is exploitable on this specific firmware version
- FTP credential exposure (would require active sniffing test)
---
 
## Scan Limitations (this environment)
 
- Router vendor identified (Zyxel) but firmware version not fingerprinted
- Windows PC role detected at 90% confidence, but OS version unknown
- IoT devices (Tuya, AMPAK) had no open ports — may have UDP services not tested
- Port 263 on router: unknown service, not matched to any known CVE
---
 
## Remediation Priority
 
| Priority | Action | Where |
|---|---|---|
| P1 — Fix immediately | Verify and disable SMBv1 | Router + Windows PC |
| P2 — Plan soon | Replace FTP with SFTP/FTPS | Router |
| P3 — Monitor | Disable NetBIOS if not needed | Windows PC |
| P3 — Monitor | Identify process on port 263 | Router |
| P4 — Monitor | Restrict DNS recursive queries | Router |
 
---
 
*This case study is based on a real scan performed with NetScan Pro SMB v1.1.0.  
All identifying information has been removed.  
The report was generated automatically in approximately 5 minutes.*
