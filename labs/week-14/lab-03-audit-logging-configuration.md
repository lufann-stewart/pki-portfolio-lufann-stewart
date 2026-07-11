# Lab 03: Audit Logging Configuration and Verification *(Stretch)*

Lufann Stewart
July 12, 2026
**Phase:** 2 | **Week:** 14  
**Submission Path:** `labs/week-14/lab-03-audit-logging-configuration.md`

---

## Overview

In this stretch lab, you configure full CA audit logging on PKI-SRV01, generate test events, and verify that the correct Windows Security log events appear. After this lab, your CA will record certificate issuance, revocation, CRL publication, configuration changes, and key operations as auditable Security log events.

This configuration is non-destructive. Audit logging can be disabled at any time with `certutil -setreg CA\AuditFilter 0`. No CA data is modified — you are only changing what the CA records about its own actions.

**This lab requires two configuration steps — both are required:**
1. Set the CA AuditFilter registry value (tells the CA what to log)
2. Enable Audit Object Access in Local Security Policy (tells Windows where to write the events)

---

## Lab Environment

| Component | Details |
|-----------|---------|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| Login Account | CORP\pki.admin |
| Domain | corp.cvilab.local |

---

## Pre-Lab Check

### Step 1 — Confirm Login and CA Status

```powershell
whoami
Get-Service CertSvc
certutil -ping
```

```
corp\pki.admin

Status   Name               DisplayName                           
------   ----               -----------                           
Running  CertSvc            Active Directory Certificate Services 

```

---

## Part A — Document the Pre-Configuration State

Before making any changes, record the current audit configuration. This creates a baseline for the before/after comparison in Part D.

### Step 1 — Check Current CA AuditFilter Value

```powershell
certutil -getreg CA\AuditFilter
```

**Expected output (default — no audit logging):**
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\CVI Issuing CA 1
  AuditFilter REG_DWORD = 0 (0)
CertUtil: -getreg command completed successfully.
```

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\CVI Issuing CA 1\AuditFilter:
CertUtil: -getreg command FAILED: 0x80070002 (WIN32: 2 ERROR_FILE_NOT_FOUND)
CertUtil: The system cannot find the file specified.
```

**Current AuditFilter value:** `Not configured — AuditFilter registry value not found (expected before configuration)`

### Step 2 — Check Local Security Policy — Audit Object Access

Run the following to check the current audit policy state:

```powershell
auditpol /get /subcategory:"Certification Services"
```

```
System audit policy

Category/Subcategory                      Setting
Object Access
  Certification Services                  No Auditing
```

Alternatively, navigate manually:
- Run: `secpol.msc`
- Navigate: Security Settings → Local Policies → Audit Policy → Audit object access
- Record the current setting:

**Current Audit Object Access setting:**
- [X] Not Configured / No Auditing
- [ ] Success Only
- [ ] Failure Only
- [ ] Success and Failure

### Step 3 — Verify No CA Events in Security Log (Pre-Configuration)

```powershell
# Check for any existing CA-related events in the Security log
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = @(4887, 4870, 4872, 4880, 4881)
} -ErrorAction SilentlyContinue | Select-Object TimeCreated, Id -First 10
```

```

PS C:\Windows\system32> Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = @(4887, 4870, 4872, 4880, 4881)
} -ErrorAction SilentlyContinue | Select-Object TimeCreated, Id -First 10

PS C:\Windows\system32> 
```

**CA-related Security log events found before configuration:**
- [X] None — baseline confirmed
- [ ] Some found — record count and Event IDs:

---

## Part B — Configure CA Audit Logging

### Step 1 — Set the CA AuditFilter

This command sets the CA to log four categories: certificate issuance (0x4), revocation (0x8), configuration changes (0x40), and key operations (0x80). Combined value: 0xCC.

Run from an elevated PowerShell prompt on PKI-SRV01:

```powershell
certutil -setreg CA\AuditFilter 0xCC
```

**Expected output:**
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\CVI Issuing CA 1
Old Value:
  AuditFilter REG_DWORD = 0 (0)
New Value:
  AuditFilter REG_DWORD = 0xcc (204)
CertUtil: -setreg command completed successfully.
The CertSvc service may need to be restarted for changes to take effect.
```

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\CVI Issuing CA 1\AuditFilter:

New Value:
  AuditFilter REG_DWORD = cc (204)
CertUtil: -setreg command completed successfully.
The CertSvc service may need to be restarted for changes to take effect.
```

**AuditFilter set to 0xCC without errors:**
- [X] Yes
- [ ] No — error:

### Step 2 — Restart the CA Service to Apply the Change

```powershell
net stop certsvc
net start certsvc
```

```
The Active Directory Certificate Services service is stopping.
The Active Directory Certificate Services service was stopped successfully.

The Active Directory Certificate Services service is starting.
The Active Directory Certificate Services service was started successfully.
```

**CA service restarted:**
- [X] Yes
- [ ] No — error:

### Step 3 — Verify the New AuditFilter Value

```powershell
certutil -getreg CA\AuditFilter
```

**Expected:**
```
AuditFilter REG_DWORD = 0xcc (204)
```

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\CVI Issuing CA 1\AuditFilter:

  AuditFilter REG_DWORD = cc (204)
CertUtil: -getreg command completed successfully.
```

**AuditFilter is now 0xcc (204):**
- [X] Yes — confirmed
- [ ] No — actual value:

### Step 4 — Enable Audit Object Access in Local Security Policy

**Method A — GUI (secpol.msc):**
1. Run: `secpol.msc`
2. Navigate: Security Settings → Local Policies → Audit Policy
3. Double-click: **Audit object access**
4. Check: **[✓] Success** and **[✓] Failure**
5. Click **OK**

**Method B — Command line:**
```powershell
auditpol /set /subcategory:"Certification Services" /success:enable /failure:enable
```

```
The command was successfully executed.
```

### Step 5 — Verify Audit Object Access Is Enabled

```powershell
auditpol /get /subcategory:"Certification Services"
```

**Expected:**
```
System audit policy
Category/Subcategory                      Setting
Object Access
  Certification Services                  Success and Failure
```

```
System audit policy

Category/Subcategory                      Setting
Object Access
  Certification Services                  Success and Failure
```

**Audit Object Access is set to Success and Failure:**
- [X] Yes — both steps confirmed
- [ ] No — describe what is missing:

---

## Part C — Generate Test Events

With audit logging configured, generate three types of auditable CA events: certificate issuance, certificate revocation, and CRL publication. Each will produce a corresponding Security log event.

### Step 1 — Issue a Test Certificate

Issue a certificate from one of the templates configured in Week 10. You can use the Certification Authority MMC or certreq.

**Option A — Using the Certification Authority MMC:**
1. Open: `certsrv.msc` (Certification Authority console)
2. Right-click the CA name → **All Tasks → Submit new request**
3. Or: expand **Certificate Templates** → right-click a template → **Issue** (if this option appears)
4. For a simpler approach: use the web enrollment page at `http://PKI-SRV01/certsrv`

**Option B — Using certreq (from an elevated PowerShell prompt):**
```powershell
# Create a minimal INF file for a test request
$inf = @"
[Version]
Signature="\$Windows NT\$"
[NewRequest]
Subject = "CN=audit-test.corp.cvilab.local"
KeySpec = 1
KeyLength = 2048
Exportable = TRUE
MachineKeySet = TRUE
SMIME = False
PrivateKeyArchive = FALSE
UserProtected = FALSE
UseExistingKeySet = FALSE
ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
ProviderType = 12
RequestType = CMC
[RequestAttributes]
CertificateTemplate=WebServer
"@
$inf | Out-File -FilePath C:\Temp\audit-test.inf -Encoding ASCII
New-Item -ItemType Directory -Path C:\Temp -Force | Out-Null

# Submit the request
certreq -new C:\Temp\audit-test.inf C:\Temp\audit-test.req
certreq -submit -config "PKI-SRV01\CVI Issuing CA 1" C:\Temp\audit-test.req C:\Temp\audit-test.cer
```

> **If the template requires manager approval:** The request will be in pending state. In the Certification Authority console, navigate to Pending Requests, right-click the request, and select **Issue**.

**Certificate issued — confirmation:**

```powershell
# Verify the certificate appears in the database
certutil -view -restrict "Disposition=20" -out "RequestID,CommonName,NotAfter" | tail -10
```

```
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  RequestID                     Issued Request ID             Long    4 -- Indexed
  CommonName                    Issued Common Name            String  8192 -- Indexed
  NotAfter                      Certificate Expiration Date   Date    8 -- Indexed

Row 1:
  Issued Request ID: 0x3
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 2:
  Issued Request ID: 0x9
  Issued Common Name: "Svc Autoenroll"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 3:
  Issued Request ID: 0xb (11)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/13/2026 12:25 PM

Row 4:
  Issued Request ID: 0xc (12)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/13/2026 12:26 PM

Row 5:
  Issued Request ID: 0xd (13)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/13/2026 12:39 PM

Row 6:
  Issued Request ID: 0xf (15)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 7:
  Issued Request ID: 0x10 (16)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 8:
  Issued Request ID: 0x11 (17)
  Issued Common Name: "CVI Issuing CA 1-Xchg"
  Certificate Expiration Date: 7/17/2026 9:14 AM

Row 9:
  Issued Request ID: 0x12 (18)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 10:
  Issued Request ID: 0x14 (20)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Maximum Row Index: 10

10 Rows
  30 Row Properties, Total Size = 622, Max Size = 54, Ave Size = 20
   0 Request Attributes, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Certificate Extensions, Total Size = 0, Max Size = 0, Ave Size = 0
  30 Total Fields, Total Size = 622, Max Size = 54, Ave Size = 20
CertUtil: -view command completed successfully.
```

Request ID of the issued test certificate: `20`
Serial Number: `4400000014178d21c3ed6a4575000000000014`

### Step 2 — Revoke the Test Certificate

```powershell
# Replace <serial_number> with the serial number from Step 1
certutil -revoke <serial_number> 5
# Reason 5 = Cessation of Operation
```

**Expected output:**
```
Certificate "1A 00 00 00 0x..." revoked
CertUtil: -revoke command completed successfully.
```

```
Revoking "4400000014178d21c3ed6a4575000000000014" -- Reason: Cessation of Operation
CertUtil: -revoke command completed successfully.
```

**Certificate revoked without errors:**
- [X] Yes
- [ ] No — error:

### Step 3 — Publish a Fresh CRL

```powershell
certutil -CRL
```

```
CertUtil: -CRL command completed successfully.
```

**CRL published without errors:**
- [X] Yes
- [ ] No — error:

### Step 4 — Wait 30 Seconds

The Security log may take a brief moment to write events after CA operations. Wait 30 seconds before proceeding to Part D.

---

## Part D — Verify Security Log Events

### Step 1 — Search for Issuance Event (4887)

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4887
    StartTime = (Get-Date).AddHours(-1)
} | Select-Object TimeCreated, Id, Message | Format-List
```

```
TimeCreated : 7/11/2026 11:57:41 AM
Id          : 4887
Message     : Certificate Services approved a certificate request and issued a certificate.
              	
              Request ID:	20
              Requester:	CORP\PKI-SRV01$
              Attributes:	
              cdc:DC01.corp.cvilab.local
              rmd:PKI-SRV01.corp.cvilab.local
              
              ccm:PKI-SRV01.corp.cvilab.local
              Disposition:	3
              SKI:		20 63 4c ec dd 25 88 df 47 41 a2 7e 42 33 69 81 29 df 66 15
              Subject:	CN=PKI-SRV01.corp.cvilab.local
```

**Event 4887 (Certificate Issued) found:**
- [X] Yes — timestamp: `7/11/2026 11:57:41 AM`
- [ ] No — troubleshoot: confirm both AuditFilter and Audit Object Access are configured

**Key fields from Event 4887:**
- Request ID: `20`
- Requester: `CORP\PKI-SRV01$`
- Certificate Template: `CVI-WebServer11 (verified from the CA database)`

### Step 2 — Search for Revocation Event (4870)

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4870
    StartTime = (Get-Date).AddHours(-1)
} | Select-Object TimeCreated, Id, Message | Format-List
```

```

TimeCreated : 7/11/2026 1:14:43 PM
Id          : 4870
Message     : Certificate Services revoked a certificate.
              	
              Serial Number:	4400000014178d21c3ed6a4575000000000014
              Reason:	5
```

**Event 4870 (Certificate Revoked) found:**
- [X] Yes — timestamp: `7/11/2026 1:14:43 PM`
- [ ] No — troubleshoot:

**Key fields from Event 4870:**
- Serial Number: `4400000014178d21c3ed6a4575000000000014`
- Reason Code: `5` (should be 5 — Cessation of Operation)

### Step 3 — Search for CRL Publication Event (4872)

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4872
    StartTime = (Get-Date).AddHours(-1)
} | Select-Object TimeCreated, Id, Message | Format-List
```

```

TimeCreated : 7/11/2026 1:16:31 PM
Id          : 4872
Message     : Certificate Services published the certificate revocation list (CRL).
              	
              Base CRL:	No
              CRL Number:	31
              Key Container:	CVI Issuing CA 1
              Next Publish:	7/11/2125 8:16 PM 22.165s
              Publish URLs:	

TimeCreated : 7/11/2026 1:16:31 PM
Id          : 4872
Message     : Certificate Services published the certificate revocation list (CRL).
              	
              Base CRL:	Yes
              CRL Number:	31
              Key Container:	CVI Issuing CA 1
              Next Publish:	7/11/2125 8:16 PM 22.165s
              Publish URLs:	C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl;
```

**Event 4872 (CRL Published) found:**
- [X] Yes — timestamp: `7/11/2026 1:16:31 PM`
- [ ] No — troubleshoot:

### Step 4 — Event Viewer Verification (Screenshot or Copy)

Open Event Viewer → Windows Logs → Security. Filter for Event IDs 4887, 4870, and 4872.

Provide either a screenshot of the filtered Security log showing all three events, or paste the full Message field for each event.

```
![Event Viewer showing Events 4887, 4870, and 4872](../../assets/week-14/lab03-security-events.png)
```

---

## Part E — Before/After Comparison and Analysis

### Configuration State Comparison

| Configuration Item | Before | After |
|---|---|---|
| CA AuditFilter value | Not configured (registry value not found) | 0xCC (204) |
| Audit Object Access policy | Not Configured | Success and Failure |
| CA-related Security log events | None | 4887, 4870, 4872 present |

### Analysis Questions

**1. The AuditFilter value 0xCC enables four categories. List the four categories and their individual hex values that combine to produce 0xCC. (Hint: 0xCC = 0x4 + 0x8 + 0x40 + 0x80)**

```
0x4 – Issue and manage certificate requests – Records certificate issuance and related request management activities.

0x8 – Revoke certificates and publish CRLs – Records certificate revocations and CRL publication.

0x40 – Change CA configuration – Records changes to the Certification Authority configuration.

0x80 – Store and retrieve archived keys – Records key archival and key recovery operations.

These values combine to produce 0xCC (0x4 + 0x8 + 0x40 + 0x80 = 0xCC).

```

**2. You configured Audit Object Access in Local Security Policy as a required second step. Why was the AuditFilter change alone insufficient to produce Security log events? What does the Local Security Policy setting control?**

```
The AuditFilter change alone was not enough because it only tells the Certification Authority which events should be audited. The Local Security Policy controls whether Windows records those audit events in the Security log. Both settings have to be enabled. If only the AuditFilter is configured, the CA generates the audit events but they are not written to the Security log. If only the Local Security Policy is enabled, Windows is ready to log events, but the CA is not generating them.
```

**3. Event 4887 (Certificate Issued) includes the Requester Name, Certificate Template, and Serial Number. Compare this to what you found in the Application event log in Lab 01. What information does Event 4887 provide that the Application log does not — and why does that matter for an audit trail?**

```
In Lab 01, the Application log mostly showed that the CA service was running and recorded general operational events. Event 4887 gives much more detail because it includes the Requester Name, Certificate Template, Request ID, and Serial Number for the certificate that was issued. That matters for an audit trail because you can see exactly who requested the certificate, what template was used, and identify the specific certificate that was issued. It makes it much easier to track certificate activity and investigate issues if something goes wrong.

```

**4. You enabled audit logging for four categories (0xCC). The full set of categories would be 0xFF. In a production CA issuing 500 certificates per day, what would be the operational consequence of enabling all categories (0xFF) vs. only the production minimum (0xCC)? What specific high-volume category would you likely want to exclude?**

```
If all audit categories (0xFF) were enabled on a production CA issuing 500 certificates a day, it would generate a lot more Security log events. That would make the logs grow faster and make it harder to find the events that actually matter during an audit or investigation. Using the production minimum (0xCC) records the important certificate and CA events without creating as much unnecessary log data. The high-volume category I would likely exclude is certificate request processing (0x20), because a busy CA receiving hundreds or thousands of requests can generate a large number of audit events. Logging every request may create unnecessary volume compared to focusing on issued certificates, revocations, and CA changes.
```

---

## Lab Report Questions

**1. Explain why both the CA AuditFilter setting and the Local Security Policy Audit Object Access setting are required for Security log events to appear. What happens if only one of the two is configured?**

```
Both settings are required because they do different jobs. The CA AuditFilter tells the Certification Authority which actions to audit, and the Local Security Policy tells Windows to write those audit events to the Security log. If only the AuditFilter is configured, the CA generates the audit events but they are not written to the Security log. If only the Local Security Policy is configured, Windows is ready to log events, but the CA is not generating them.

```

**2. A junior administrator tells you: "I can see CRL publication events in the Application log, so I know the CA is being audited." What is wrong with this statement, and what would you tell them to check to determine whether the CA is producing a proper security audit trail?**

```
That statement is wrong because seeing CRL publication events in the Application log only shows that the CA is performing its normal operations. It does not mean security auditing is enabled. To make sure the CA is producing a proper security audit trail, I would check that the AuditFilter is configured, Audit Object Access is enabled in the Local Security Policy, and verify that Security log events such as 4887, 4870, and 4872 are being recorded.

```

**3. The AuditFilter value 0xCC does not include CA service start/stop events (0x1) or backup/restore events (0x2). In what operational scenario would you add these categories — and what would you be looking for in those events?**

```
I would enable these categories if I was troubleshooting the CA or investigating a security or operational issue. For example, if the CA service kept stopping unexpectedly, or after a backup or restore, or if certificates suddenly stopped being issued. I would look for events showing when the CA service started or stopped and whether a backup or restore was performed so I could figure out what happened and when.

```

---

## Submission Checklist

- [X] Logged in as CORP\pki.admin — whoami output included
- [X] Pre-configuration AuditFilter value documented (certutil -getreg output)
- [X] Pre-configuration Audit Object Access state documented
- [X] Pre-configuration Security log check performed (no CA events)
- [X] certutil -setreg CA\AuditFilter 0xCC output included
- [X] CA service restarted — net stop/start output included
- [X] Post-configuration certutil -getreg confirms 0xCC (204)
- [X] Audit Object Access enabled — auditpol /get output confirms Success and Failure
- [X] Test certificate issued — Request ID and Serial Number recorded
- [X] Test certificate revoked with reason code 5 — certutil -revoke output included
- [X] CRL published — certutil -CRL output included
- [X] Event 4887 (Issued) located and documented — timestamp and key fields recorded
- [X] Event 4870 (Revoked) located and documented — timestamp and key fields recorded
- [X] Event 4872 (CRL Published) located and documented — timestamp recorded
- [X] Event Viewer screenshot or full message content provided for all three events
- [X] Before/after comparison table completed
- [X] All four Part E analysis questions answered
- [X] All three lab report questions answered in complete sentences
- [X] File committed to `labs/week-14/lab-03-audit-logging-configuration.md`
