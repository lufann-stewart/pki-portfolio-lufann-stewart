# Lab 01: Configure Autoenrollment via Group Policy

Lufann Stewart
**Date Completed:**
**Phase:** 2 | **Week:** 15
**Submission Path:** `labs/week-15/lab-01-autoenrollment-gpo.md`

---NOTES  left off at Then confirm from the command line:

```powershell
certutil -viewstore -enterprise "MY"
```
## Overview

In this lab, you configure real, hands-on certificate autoenrollment in your OVA environment — the mechanism that makes enterprise PKI practical at scale. You will grant autoenrollment permissions on a certificate template, create and link a Group Policy Object to enable the Certificate Services Client, force a policy update on CLIENT01, and verify successful autoenrollment using both the Certificates MMC and certutil.

This is the foundation Lesson 1 builds on to explain why certificate lifecycle management (CLM) platforms exist as a separate, complementary layer — you need to understand this mechanism firsthand before you can explain what it doesn't cover.

**Prerequisite:** A published certificate template from Week 10 (e.g., a duplicated Workstation Authentication template). If you no longer have one published, republish it from the Certification Authority console before proceeding.

---

## Lab Environment

| Component | Details |
|-----------|---------|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| Domain | corp.cvilab.local |
| Client | CLIENT01 |
| Target OU | CVI Workstations |
| Login Account | CORP\pki.admin |

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
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (16ms)
CertUtil: -ping command completed successfully.
```

**CA is running and responding:**
- [X] Yes — proceed to Pre-Lab Step 2
- [ ] No — resolve before continuing

### Step 2 — Confirm the Target OU Exists

```powershell
Get-ADOrganizationalUnit -Filter 'Name -eq "CVI Workstations"'
```

```

City                     : 
Country                  : 
DistinguishedName        : OU=CVI Workstations,DC=corp,DC=cvilab,DC=local
LinkedGroupPolicyObjects : {}
ManagedBy                : 
Name                     : CVI Workstations
ObjectClass              : organizationalUnit
ObjectGUID               : ab24dd36-cc01-4792-ad8d-eebd8fbcc587
PostalCode               : 
State                    : 
StreetAddress            : 
```

**CVI Workstations OU exists and CLIENT01 is a member:**
- [X] Yes
- [ ] No — create the OU and move CLIENT01's computer object into it before continuing

### Step 3 — Confirm a Published Template Exists

```powershell
certutil -CATemplates
```

```
CVI-WebServer11: CVI-WebServer11 -- Auto-Enroll: Access is denied.
CVICodeSigning1: CVI Code Signing1 -- Auto-Enroll: Access is denied.
CVI-WebServer1: CVI-WebServer1 -- Auto-Enroll
CodeSigning: Code Signing -- Auto-Enroll: Access is denied.
OCSPResponseSigning: OCSP Response Signing -- Auto-Enroll
CVI-ServiceAccount: CVI Service Account -- Auto-Enroll
CVICodeSigning: CVI Code Signing -- Auto-Enroll: Access is denied.
CVI-WebServer: CVI-WebServer -- Auto-Enroll: Access is denied.
DirectoryEmailReplication: Directory Email Replication -- Auto-Enroll: Access is denied.
DomainControllerAuthentication: Domain Controller Authentication -- Auto-Enroll: Access is denied.
KerberosAuthentication: Kerberos Authentication -- Auto-Enroll: Access is denied.
EFSRecovery: EFS Recovery Agent -- Auto-Enroll: Access is denied.
EFS: Basic EFS -- Auto-Enroll: Access is denied.
DomainController: Domain Controller -- Auto-Enroll: Access is denied.
WebServer: Web Server -- Auto-Enroll: Access is denied.
Machine: Computer -- Auto-Enroll: Access is denied.
User: User -- Auto-Enroll: Access is denied.
SubCA: Subordinate Certification Authority -- Auto-Enroll: Access is denied.
Administrator: Administrator -- Auto-Enroll: Access is denied.
CertUtil: -CATemplates command completed successfully.
```

Template to use for this lab: `CVI-WebServer1`

---

## Part A — Configure Template Permissions for Autoenrollment

### Step 1 — Open the Certificate Templates Console

```
certtmpl.msc
```

Locate your chosen template, right-click → Properties → Security tab.

### Step 2 — Grant Autoenroll, Read, and Enroll Permissions

Add the **Domain Computers** group (or a dedicated security group if you've configured one) and check all three boxes:

- [X] Read
- [X] Enroll
- [X] Autoenroll

```
- Group: Domain Computers 
- Access Permissions Granted:
  - Read: Allow
  - Enroll: Allow
  - Autoenroll: Allow
```

**All three permissions granted to the correct group:**
- [X] Yes
- [ ] No — describe what's missing:

### Step 3 — Confirm the Template Is Published on the CA

```powershell
certutil -CATemplates
```

```
CVI-WebServer11: CVI-WebServer11 -- Auto-Enroll: Access is denied.
CVICodeSigning1: CVI Code Signing1 -- Auto-Enroll: Access is denied.
CVI-WebServer1: CVI-WebServer1 -- Auto-Enroll
CodeSigning: Code Signing -- Auto-Enroll: Access is denied.
OCSPResponseSigning: OCSP Response Signing -- Auto-Enroll
CVI-ServiceAccount: CVI Service Account -- Auto-Enroll
CVICodeSigning: CVI Code Signing -- Auto-Enroll: Access is denied.
CVI-WebServer: CVI-WebServer -- Auto-Enroll: Access is denied.
DirectoryEmailReplication: Directory Email Replication -- Auto-Enroll: Access is denied.
DomainControllerAuthentication: Domain Controller Authentication -- Auto-Enroll: Access is denied.
KerberosAuthentication: Kerberos Authentication -- Auto-Enroll: Access is denied.
EFSRecovery: EFS Recovery Agent -- Auto-Enroll: Access is denied.
EFS: Basic EFS -- Auto-Enroll: Access is denied.
DomainController: Domain Controller -- Auto-Enroll: Access is denied.
WebServer: Web Server -- Auto-Enroll: Access is denied.
Machine: Computer -- Auto-Enroll: Access is denied.
User: User -- Auto-Enroll: Access is denied.
SubCA: Subordinate Certification Authority -- Auto-Enroll: Access is denied.
Administrator: Administrator -- Auto-Enroll: Access is denied.
CertUtil: -CATemplates command completed successfully.
```

**Part A Summary:**

| Check | Result |
|---|---|
| Template has Read + Enroll + Autoenroll for the target group | Yes |
| Template is published on the CA | Yes |

---

## Part B — Create and Link the Group Policy Object

### Step 1 — Create a New GPO

```
Group Policy Management Console (gpmc.msc)
  → corp.cvilab.local
    → Group Policy Objects → right-click → New
      → Name: "CVI Certificate Autoenrollment"
```

### Step 2 — Configure the Certificate Services Client — Auto-Enrollment Setting

Edit the GPO:

```
Computer Configuration
  → Policies
    → Windows Settings
      → Security Settings
        → Public Key Policies
          → Certificate Services Client - Auto-Enrollment
            → Configuration Model: Enabled
            → Check: Renew expired certificates, update pending certificates, and remove revoked certificates
            → Check: Update certificates that use certificate templates
```

```
Enabled the Certificate Services Client - Auto-Enrollment policy under Computer Configuration → Public Key Policies. Configured the Configuration Model to "Enabled" and checked both options to automatically renew expired certificates, update pending requests, remove revoked certificates, and update certificates using updated templates.
```

### Step 3 — Link the GPO to the CVI Workstations OU

```
Right-click "CVI Workstations" OU → Link an Existing GPO → CVI Certificate Autoenrollment
```

**Part B Summary:**

| Check | Result |
|---|---|
| GPO created with correct auto-enrollment settings | Yes |
| GPO linked to CVI Workstations OU | Yes |

---

## Part C — Force Policy Update and Verify Autoenrollment

### Step 1 — Force Group Policy Update on CLIENT01

On CLIENT01, run as an administrator:

```powershell
gpupdate /force
```

```
Updating policy...



Computer Policy update has completed successfully.

User Policy update has completed successfully.
```

### Step 2 — Verify Autoenrollment Policy Applied

```powershell
gpresult /r
```

```

COMPUTER SETTINGS
------------------
    CN=PKI-SRV01,OU=CVI Workstations,DC=corp,DC=cvilab,DC=local
    Last time Group Policy was applied: 7/23/2026 at 8:55:09 AM
    Group Policy was applied from:      DC01.corp.cvilab.local
    Group Policy slow link threshold:   500 kbps
    Domain Name:                        CORP
    Domain Type:                        Windows 2008 or later

    Applied Group Policy Objects
    -----------------------------
        CVI Certificate Autoenrollment
        Default Domain Policy
```

**GPO confirmed applied to CLIENT01:**
- [X] Yes
- [ ] No — troubleshoot: check OU membership, GPO link, and replication (`repadmin /syncall`)

### Step 3 — Verify the Certificate Was Issued

Open the Certificates MMC for the **Computer account** on CLIENT01:

```
certlm.msc
  → Personal → Certificates
```

```
* **Subject:** `CN = PKI-SRV01.corp.cvilab.local`
* **Issuer:** `CN = CVI Issuing CA 1, DC = corp, DC = cvilab, DC = local`
* **Template:** `CVI-WebServer1 (1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.14614516.7651276)`
* **Template Version:** `Major: 100, Minor: 4`
* **Valid From:** `Wednesday, July 22, 2026 9:38:26 AM`
* **Valid To:** `Sunday, April 25, 2027 7:36:58 PM`
```

Then confirm from the command line:

```powershell
certutil -viewstore -enterprise "MY"
```

```
MY "Personal"
CertUtil: -viewstore command FAILED: 0x80070002 (WIN32: 2 ERROR_FILE_NOT_FOUND)
CertUtil: The system cannot find the file specified.
```

**Certificate successfully autoenrolled on CLIENT01:**
- [ ] Yes — subject name, template, and validity dates recorded above
- [ ] No — describe what you observed instead:

### Step 4 — Confirm Issuance on the CA Side

Back on PKI-SRV01:

```powershell
certutil -view -restrict "Disposition=20","CommonName=CLIENT01*"
```

```
(paste output here)
```

**Part C Summary:**

| Check | Tool Used | Result |
|---|---|---|
| GPO applied to CLIENT01 | gpresult | Yes / No |
| Certificate present in computer store | certlm.msc / certutil -viewstore | Yes / No |
| Certificate record on CA matches | certutil -view | Yes / No |

---

## Part D — Autoenrollment Procedure (Reusable)

Write the complete autoenrollment configuration as a repeatable procedure — the steps another administrator could follow to enable autoenrollment for a new template and OU in this environment.

```
(document the procedure as a numbered list — cover template permissions, GPO configuration and linking, and client-side verification)
```

---

## Lab Report Questions

**1. You granted Read, Enroll, and Autoenroll permissions in Part A. What happens, specifically, if only Enroll and Read are granted but Autoenroll is missing? Why does autoenrollment require all three rather than just Enroll?**

```
(your answer here)
```

**2. In Part C, you verified the certificate using both the Certificates MMC and certutil -viewstore. What does each tool confirm that the other doesn't, and why is checking both sides — the client's certificate store and the CA's database — good practice rather than redundant?**

```
(your answer here)
```

**3. Autoenrollment, as you've just configured it, only applies to computers in the CVI Workstations OU within corp.cvilab.local. Name two categories of certificate-bearing assets in a typical enterprise that this exact mechanism could never reach, and explain why — this sets up Lesson 1's discussion of where autoenrollment stops.**

```
(your answer here)
```

**4. If this GPO were misconfigured — say, linked to the wrong OU — what would the symptom look like from CLIENT01's perspective, and what would you check first to diagnose it? Reference the requirements table from Lesson 1 (template permissions, GPO link, connectivity, renewal threshold) in your answer.**

```
(your answer here)
```

**5. Now that you've configured autoenrollment by hand, explain in your own words why an organization running a CLM platform (which you'll explore in Labs 02–04) still needs this exact mechanism running underneath it. What would break if the CLM platform tried to replace autoenrollment instead of building on top of it?**

```
(your answer here)
```

---

## Submission Checklist

- [ ] Logged in as CORP\pki.admin — whoami output included
- [ ] CA running and responding — Get-Service and certutil -ping output included
- [ ] CVI Workstations OU confirmed, CLIENT01 membership verified
- [ ] Published template identified — certutil -CATemplates output included
- [ ] Template permissions configured — Read, Enroll, and Autoenroll all granted and documented
- [ ] GPO created with correct Auto-Enrollment configuration model and options
- [ ] GPO linked to CVI Workstations OU
- [ ] gpupdate /force run on CLIENT01 — output included
- [ ] gpresult /r confirms GPO applied — relevant section included
- [ ] Certificate verified in Certificates MMC (computer account) — details recorded
- [ ] Certificate verified via certutil -viewstore — output included
- [ ] Certificate record confirmed on CA via certutil -view — output included
- [ ] Part C summary table completed
- [ ] Reusable autoenrollment procedure written
- [ ] All five lab report questions answered in complete sentences
- [ ] File committed to `labs/week-15/lab-01-autoenrollment-gpo.md`
