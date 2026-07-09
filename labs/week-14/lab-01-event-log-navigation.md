# Lab 01: AD CS Event Log Navigation

Lufann Stewart    
July 9, 2026  
**Phase:** 2 | **Week:** 14  
**Submission Path:** `labs/week-14/lab-01-event-log-navigation.md`

---

## Overview

In this lab, you navigate the two data sources that record what your CA has been doing: the Windows Application event log and the CA certificate database. Both contain a record of the operations you performed across Weeks 10 through 13 — certificate issuance, revocation, CRL publication, and CA service lifecycle events.

This lab has three parts. Part A navigates the Application event log on PKI-SRV01. Part B queries the CA certificate database using certutil -view. Part C asks you to connect both sources to reconstruct an operational picture of the CA.

**No new CA operations are required.** The data you need already exists from prior lab work.

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

### Step 1 — Confirm Login

```powershell
whoami
```

**Expected:** `corp\pki.admin`

```
corp\pki.admin
```

### Step 2 — Confirm CA Is Running

```powershell
Get-Service CertSvc
certutil -ping
```

```
Status   Name               DisplayName                           
------   ----               -----------                           
Running  CertSvc            Active Directory Certificate Services 
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (0ms)
CertUtil: -ping command completed successfully.
```

**CA is running and responding:**
- [X] Yes — proceed to Part A
- [ ] No — describe issue:

---

## Part A — Application Event Log Navigation

The Windows Application event log records CA operational events by default — no configuration required. You will create a filtered view and document the CA event history on PKI-SRV01.

### Step 1 — Open Event Viewer

Log into PKI-SRV01. Open **Event Viewer**:
- Start → search "Event Viewer" → Open
- Or run: `eventvwr.msc`

### Step 2 — Navigate to the Application Log

In the left panel: **Windows Logs → Application**

### Step 3 — Create a Filtered View

Right-click **Application** → **Filter Current Log**

In the filter dialog:
- **Event sources:** Type `CertificationAuthority` and select it
- Leave all other fields at default
- Click **OK**

> **To save this filter as a Custom View (recommended):**
> Right-click **Custom Views** in the left panel → **Create Custom View** → same filter settings → Name it "CA Events" → OK

### Step 4 — Document Your Findings

Review the filtered event list. Identify at least **three distinct event types** present in your log. For each event, record the information below.

---

**Event 1**

Event ID:
```
17
```

Timestamp:
```
7/1/2026 2:16:19 PM
```

Source:
```
CertificationAuthority
```

Event Description (copy from the General tab of the event):
```
Active Directory Certificate Services did not start: Unable to initialize the database connection for CVI Issuing CA 1.  File not found 0xc8000713 (ESE: -1811 JET_errFileNotFound).
```

What this event represents (in your own words):
```
The CA database files in the CertLog folder could not be found, so the database was unable to initialize when AD CS tried to start. Since the database couldn't be opened, the CA service was unable to start.
```

---

**Event 2**

Event ID:
```
38
```

Timestamp:
```
6/30/2026 12:04:11 PM
```

Event Description:
```
Active Directory Certificate Services for CVI Issuing CA 1 was stopped.
```

What this event represents:
```
This event shows that the Active Directory Certificate Services service was stopped. The service may have been stopped as part of maintenance, troubleshooting, or another administrative task.
```

---

**Event 3**

Event ID:
```
94
```

Timestamp:
```
5/31/2026 3:03:17 PM
```

Event Description:
```
Active Directory Certificate Services CVI Issuing CA 1 can not open the certificate store at CN=NTAuthCertificates,CN=Public Key Services,CN=Services in the Active Directory's configuration container.
```

What this event represents:
```
This event shows that the CA was unable to open the NTAuthCertificates store in Active Directory. This indicates there was an issue accessing Active Directory when the event occurred.
```

---

**Additional events (optional — document as many as you find useful):**

```
Event ID: 44  
Logged: 5/29/2026 4:08:27 PM  
Description:  The "Windows default" Policy Module "Initialize" method returned an error. The specified domain either does not exist or could not be contacted. The returned status code is 0x8007054b (1355).  The Active Directory containing the Certification Authority could not be contacted.  
```

### Step 5 — Event Type Summary

Based on your filtered view, complete this table:

| Event Type | Present? | Approximate Count |
|---|---|---|
| CRL publication events | No | 0 |
| CA service start/stop events | Yes | 68 |
| Certificate template update events | No | 0 |
| CA configuration change events | Yes | 1 |
| Error events (Level = Error) | Yes | 39 |

---

## Part B — CA Certificate Database Query

The CA certificate database contains every certificate request, issuance, and revocation since the CA was stood up. You will query it using certutil -view from an elevated PowerShell prompt on PKI-SRV01.

### Step 1 — Open an Elevated PowerShell Prompt

Right-click **Windows PowerShell** → **Run as Administrator**

Confirm: `whoami` returns `corp\pki.admin`

### Step 2 — Query All Issued Certificates

```powershell
certutil -view -restrict "Disposition=20"
```

This returns all certificates with Disposition = 20 (Issued and active).

```
        Digital Signature, Key Encipherment (a0)

    1.3.6.1.4.1.311.21.10: Flags = 20000(Origin=Policy), Length = e
    Application Policies
        [1]Application Certificate Policy:
             Policy Identifier=Server Authentication

    2.5.29.14: Flags = 10000(Origin=Request), Length = 16
    Subject Key Identifier
        00393f76a6ab7fe0ea20ff357df61875b4702fb4

    2.5.29.35: Flags = 20000(Origin=Policy), Length = 18
    Authority Key Identifier
        KeyID=114d5edbf32dc1542007b8cd3f192d04bbe85fd1

    2.5.29.31: Flags = 40000(Origin=Server), Length = 125
    CRL Distribution Points
        [1]CRL Distribution Point
             Distribution Point Name:
                  Full Name:
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevoc
ationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PKI-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=
cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)
                       URL=http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl (http://pki-srv01.corp.cvilab.local/CertEnroll/CVI%20Issuing%20CA%201.crl)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = f9
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=cer
tificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?object
Class=certificationAuthority)
        [2]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=http://pki-srv01.corp.cvilab.local/ocsp

    2.5.29.19: Flags = 20003(Critical, Disabled, Origin=Policy), Length = 2
    Basic Constraints
        Subject Type=End Entity
        Path Length Constraint=None

    2.5.29.17: Flags = 20000(Origin=Policy), Length = 1f
    Subject Alternative Name
        DNS Name=PKI-SRV01.corp.cvilab.local


Maximum Row Index: 8

8 Rows
 273 Row Properties, Total Size = 36017, Max Size = 1643, Ave Size = 131
  31 Request Attributes, Total Size = 2334, Max Size = 132, Ave Size = 75
  84 Certificate Extensions, Total Size = 7850, Max Size = 319, Ave Size = 93
 388 Total Fields, Total Size = 46201, Max Size = 1643, Ave Size = 119
CertUtil: -view command completed successfully.
```

**Total number of issued certificates found:**
```
8
```

### Step 3 — Query All Revoked Certificates
        
```powershell
certutil -view -restrict "Disposition=21"
```

<details>
<summary>Click to expand certutil output</summary>

````
PS C:\Windows\system32> certutil -view -restrict "Disposition=21"
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  Request.RequestID             Request ID                    Long    4 -- Indexed
  Request.RawRequest            Binary Request                Binary  65536
  Request.RawArchivedKey        Archived Key                  Binary  65536
  Request.KeyRecoveryHashes     Key Recovery Agent Hashes     String  8192
  Request.RawOldCertificate     Old Certificate               Binary  16384
  Request.RequestAttributes     Request Attributes            String  32768
  Request.RequestType           Request Type                  Long    4
  Request.RequestFlags          Request Flags                 Long    4
  Request.StatusCode            Request Status Code           Long    4
  Request.Disposition           Request Disposition           Long    4 -- Indexed
  Request.DispositionMessage    Request Disposition Message   String  8192
  Request.SubmittedWhen         Request Submission Date       Date    8 -- Indexed
  Request.ResolvedWhen          Request Resolution Date       Date    8 -- Indexed
  Request.RevokedWhen           Revocation Date               Date    8
  Request.RevokedEffectiveWhen  Effective Revocation Date     Date    8 -- Indexed
  Request.RevokedReason         Revocation Reason             Long    4
  Request.RequesterName         Requester Name                String  2048 -- Indexed
  Request.CallerName            Caller Name                   String  2048 -- Indexed
  Request.SignerPolicies        Signer Policies               String  8192
  Request.SignerApplicationPolicies  Signer Application Policies   String  8192
  Request.Officer               Officer                       Long    4
  Request.DistinguishedName     Request Distinguished Name    String  8192
  Request.RawName               Request Binary Name           Binary  4096
  Request.Country               Request Country/Region        String  8192
  Request.Organization          Request Organization          String  8192
  Request.OrgUnit               Request Organization Unit     String  8192
  Request.CommonName            Request Common Name           String  8192
  Request.Locality              Request City                  String  8192
  Request.State                 Request State                 String  8192
  Request.Title                 Request Title                 String  8192
  Request.GivenName             Request First Name            String  8192
  Request.Initials              Request Initials              String  8192
  Request.SurName               Request Last Name             String  8192
  Request.DomainComponent       Request Domain Component      String  8192
  Request.EMail                 Request Email Address         String  8192
  Request.StreetAddress         Request Street Address        String  8192
  Request.UnstructuredName      Request Unstructured Name     String  8192
  Request.UnstructuredAddress   Request Unstructured Address  String  8192
  Request.DeviceSerialNumber    Request Device Serial Number  String  8192
  Request.AttestationChallenge  Attestation Challenge         Binary  4096
  Request.EndorsementKeyHash    Endorsement Key Hash          String  144 -- Indexed
  Request.EndorsementCertificateHash  Endorsement Certificate Hash  String  144 -- Indexed
  Request.RawPrecertificate     Binary Precertificate         Binary  16384
  RequestID                     Issued Request ID             Long    4 -- Indexed
  RawCertificate                Binary Certificate            Binary  16384
  CertificateHash               Certificate Hash              String  128 -- Indexed
  CertificateTemplate           Certificate Template          String  254 -- Indexed
  EnrollmentFlags               Template Enrollment Flags     Long    4
  GeneralFlags                  Template General Flags        Long    4
  PrivatekeyFlags               Template Private Key Flags    Long    4
  SerialNumber                  Serial Number                 String  128 -- Indexed
  IssuerNameID                  Issuer Name ID                Long    4
  NotBefore                     Certificate Effective Date    Date    8
  NotAfter                      Certificate Expiration Date   Date    8 -- Indexed
  SubjectKeyIdentifier          Issued Subject Key Identifier  String  128 -- Indexed
  RawPublicKey                  Binary Public Key             Binary  4096
  PublicKeyLength               Public Key Length             Long    4
  PublicKeyAlgorithm            Public Key Algorithm          String  254
  RawPublicKeyAlgorithmParameters  Public Key Algorithm Parameters  Binary  4096
  PublishExpiredCertInCRL       Publish Expired Certificate in CRL  Long    4
  UPN                           User Principal Name           String  2048 -- Indexed
  DistinguishedName             Issued Distinguished Name     String  8192
  RawName                       Issued Binary Name            Binary  4096
  Country                       Issued Country/Region         String  8192
  Organization                  Issued Organization           String  8192
  OrgUnit                       Issued Organization Unit      String  8192
  CommonName                    Issued Common Name            String  8192 -- Indexed
  Locality                      Issued City                   String  8192
  State                         Issued State                  String  8192
  Title                         Issued Title                  String  8192
  GivenName                     Issued First Name             String  8192
  Initials                      Issued Initials               String  8192
  SurName                       Issued Last Name              String  8192
  DomainComponent               Issued Domain Component       String  8192
  EMail                         Issued Email Address          String  8192
  StreetAddress                 Issued Street Address         String  8192
  UnstructuredName              Issued Unstructured Name      String  8192
  UnstructuredAddress           Issued Unstructured Address   String  8192
  DeviceSerialNumber            Issued Device Serial Number   String  8192

Row 1:
  Request ID: 0x4
  Binary Request:
-----BEGIN NEW CERTIFICATE REQUEST-----
MIIGWgYJKoZIhvcNAQcCoIIGSzCCBkcCAQMxCzAJBgUrDgMCGgUAMIIEtAYIKwYB
BQUHDAKgggSmBIIEojCCBJ4waTBnAgECBgorBgEEAYI3CgoBMVYwVAIBADADAgEB
MUowSAYJKwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxv
Y2FsDA5DT1JQXHBraS5hZG1pbgwHTU1DLkVYRTCCBCugggQnAgEBMIIEIDCCAwgC
AQAwADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANKBLS+0hBgzLcO5
i42pR/b/4Hzc10zIaMLAeskG+4rkHdDma/hX8Qyctc6+X8mjkBsxZBjwerHGrI+J
5DG34eHYsfNbA+ojP/Ft99J5e2TaCj12MmH3gyl+5iTf6O6T5R3vsErXgEuMFdWS
wHPzuoqZLT6sBUJHwlIvSfAgH/uCPRMrCS1qwocYopOkgJBpKAzeTlRn3n0iRVgw
iZxmTEYwGgJmdXgmPypDJ6yZKSVxnY6IqTOZiR1oGG7eWbVwrELUQcmuK0PdFDBP
TaGA1+JbcGZeo9HRRD43agU4+wCpIIS/GyidjwIWj66x7MaRr/RgDV1svLQJT7tw
Nb1gmlkCAwEAAaCCAdkwHAYKKwYBBAGCNw0CAzEOFgwxMC4wLjIwMzQ4LjIwSAYJ
KwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxvY2FsDA5D
T1JQXHBraS5hZG1pbgwHTU1DLkVYRTB0BgorBgEEAYI3DQICMWYwZAIBAR5cAE0A
aQBjAHIAbwBzAG8AZgB0ACAARQBuAGgAYQBuAGMAZQBkACAAQwByAHkAcAB0AG8A
ZwByAGEAcABoAGkAYwAgAFAAcgBvAHYAaQBkAGUAcgAgAHYAMQAuADADAQAwgfgG
CSqGSIb3DQEJDjGB6jCB5zA+BgkrBgEEAYI3FQcEMTAvBicrBgEEAYI3FQiHydJI
goaqPISljyiHissog+KoU4EpgrzrYIaT4yECAWQCAQUwEwYDVR0lBAwwCgYIKwYB
BQUHAwIwDgYDVR0PAQH/BAQDAgWgMBsGCSsGAQQBgjcVCgQOMAwwCgYIKwYBBQUH
AwIwRAYJKoZIhvcNAQkPBDcwNTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQC
AgCAMAcGBSsOAwIHMAoGCCqGSIb3DQMHMB0GA1UdDgQWBBRwHAG/cY/gAXNxOTsS
5H8z2zLSVTANBgkqhkiG9w0BAQUFAAOCAQEAGm25SvExJmKvVkZMA19ksMehBgSg
I43N1QMkyOJAZwYAt7EvdnB5F5fkfRiq9ZC52x8FbT/FcgwtCoDnTDXL3CndxNxL
c2y3rMEi5OvG3CHKKBHrYhCRsAQZx7IpZGffRsq/yGq4ZUggipaKZu2ZCSII24ah
LYLu1JVa3uUbDlwnXv04EjSDoL6NAqFRXR+20ODwxX2uxElSOjsN0Wfw+i0nl95k
pnplOV081DhBjzKstcZwqkGHDXm9Jh8ZhGEq8i5zVo9IqGHVitolq19Q8Eki6oqh
EyUC2loeisPp6JX5Ggl2VVKB/p0UqBqo7HV6VxjA9AGEBr8Ty577Zv/rmDAAMAAx
ggF7MIIBdwIBA4AUcBwBv3GP4AFzcTk7EuR/M9sy0lUwCQYFKw4DAhoFAKA+MBcG
CSqGSIb3DQEJAzEKBggrBgEFBQcMAjAjBgkqhkiG9w0BCQQxFgQUgUcMwJzvv3uj
evALu326lykKA0IwDQYJKoZIhvcNAQEBBQAEggEAqlKLBkRvmZHcGgEG5VZrNcio
Mi//IL7Y3FDpITJwHzkmZ3NOXHz8SfkM2CQQJshGXVdjuU2t4RzaXSJkCM66rLgY
k3kfbtG13iP2MmkymlZejq0oJL1Ek3GXeG2HIqnh0oHxmZlfTm03BlUByJ0K5bct
0c0fb+h8kfF1YSS/bz/f+5/SCfd5DJ384mTp3zF4jt9HqQcqKsb90HWCi4x9yoYB
hbs3FxMxBjM8t6egzTw6abTqhs8uuAOhCXugfDysgg0IOioflV3bqjnGebyqSZN0
GbgsvucKBWHGIqTXYmRRmDhmds5w4tsxDmxG+xEQxAo6ppH8VsDj3K8m7OpS7Q==
-----END NEW CERTIFICATE REQUEST-----

  Archived Key: EMPTY
  Key Recovery Agent Hashes: EMPTY
  Old Certificate: EMPTY
  Request Attributes: "
cdc:DC01.corp.cvilab.local
rmd:PKI-SRV01.corp.cvilab.local

ccm:PKI-SRV01.corp.cvilab.local"
  Request Type: 0x40400 (263168) -- CMC, Full Response
  Request Flags: 0x4 -- Force UTF-8
  Request Status Code: 0x0 (WIN32: 0) -- The operation completed successfully.
  Request Disposition: 0x15 (21) -- Revoked
  Request Disposition Message: "Revoked by CORP\pki.admin"
  Request Submission Date: 5/23/2026 5:39 PM
  Request Resolution Date: 5/23/2026 5:39 PM
  Revocation Date: 5/28/2026 3:34 PM
  Effective Revocation Date: 5/28/2026 3:34 PM
  Revocation Reason: 0x5 -- Reason: Cessation of Operation
  Requester Name: "CORP\pki.admin"
  Caller Name: "CORP\pki.admin"
  Signer Policies: EMPTY
  Signer Application Policies: EMPTY
  Officer: EMPTY
  Request Distinguished Name: EMPTY
  Request Binary Name:
0000	30 00                                              0.

  Request Country/Region: EMPTY
  Request Organization: EMPTY
  Request Organization Unit: EMPTY
  Request Common Name: EMPTY
  Request City: EMPTY
  Request State: EMPTY
  Request Title: EMPTY
  Request First Name: EMPTY
  Request Initials: EMPTY
  Request Last Name: EMPTY
  Request Domain Component: EMPTY
  Request Email Address: EMPTY
  Request Street Address: EMPTY
  Request Unstructured Name: EMPTY
  Request Unstructured Address: EMPTY
  Request Device Serial Number: EMPTY
  Attestation Challenge: EMPTY
  Endorsement Key Hash: EMPTY
  Endorsement Certificate Hash: EMPTY
  Binary Precertificate: EMPTY
  Issued Request ID: 0x4
  Binary Certificate:
-----BEGIN CERTIFICATE-----
MIIF/TCCBOWgAwIBAgITRAAAAAS3QrXauj1N6wAAAAAABDANBgkqhkiG9w0BAQsF
ADBgMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFjAUBgoJkiaJk/IsZAEZFgZjdmls
YWIxFDASBgoJkiaJk/IsZAEZFgRjb3JwMRkwFwYDVQQDExBDVkkgSXNzdWluZyBD
QSAxMB4XDTI2MDUyNDAwMjkxM1oXDTI3MDQyNjAyMzY1OFowFDESMBAGA1UEAxMJ
UEtJIEFkbWluMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0oEtL7SE
GDMtw7mLjalH9v/gfNzXTMhowsB6yQb7iuQd0OZr+FfxDJy1zr5fyaOQGzFkGPB6
scasj4nkMbfh4dix81sD6iM/8W330nl7ZNoKPXYyYfeDKX7mJN/o7pPlHe+wSteA
S4wV1ZLAc/O6ipktPqwFQkfCUi9J8CAf+4I9EysJLWrChxiik6SAkGkoDN5OVGfe
fSJFWDCJnGZMRjAaAmZ1eCY/KkMnrJkpJXGdjoipM5mJHWgYbt5ZtXCsQtRBya4r
Q90UME9NoYDX4ltwZl6j0dFEPjdqBTj7AKkghL8bKJ2PAhaPrrHsxpGv9GANXWy8
tAlPu3A1vWCaWQIDAQABo4IC+jCCAvYwPgYJKwYBBAGCNxUHBDEwLwYnKwYBBAGC
NxUIh8nSSIKGqjyEpY8oh4rLKIPiqFOBKYK862CGk+MhAgFkAgEFMBMGA1UdJQQM
MAoGCCsGAQUFBwMCMA4GA1UdDwEB/wQEAwIFoDAbBgkrBgEEAYI3FQoEDjAMMAoG
CCsGAQUFBwMCMEQGCSqGSIb3DQEJDwQ3MDUwDgYIKoZIhvcNAwICAgCAMA4GCCqG
SIb3DQMEAgIAgDAHBgUrDgMCBzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQUcBwBv3GP
4AFzcTk7EuR/M9sy0lUwHwYDVR0jBBgwFoAUEU1e2/MtwVQgB7jNPxktBLvoX9Ew
gd8GA1UdHwSB1zCB1DCB0aCBzqCBy4aByGxkYXA6Ly8vQ049Q1ZJJTIwSXNzdWlu
ZyUyMENBJTIwMSxDTj1QS0ktU1JWMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUy
MFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y29ycCxE
Qz1jdmlsYWIsREM9bG9jYWw/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNl
P29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHRBggrBgEFBQcBAQSB
xDCBwTCBvgYIKwYBBQUHMAKGgbFsZGFwOi8vL0NOPUNWSSUyMElzc3VpbmclMjBD
QSUyMDEsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZp
Y2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y29ycCxEQz1jdmlsYWIsREM9bG9jYWw/
Y0FDZXJ0aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRo
b3JpdHkwNgYDVR0RBC8wLaArBgorBgEEAYI3FAIDoB0MG3BraS5hZG1pbkBjb3Jw
LmN2aWxhYi5sb2NhbDANBgkqhkiG9w0BAQsFAAOCAQEAMUqgmlFn5HXcyj4w0fmF
vX9ddE9TrspSyg8B+5qG0GKboE1q4OQcH+u7kv1VPRdpr0ZVEC00WxT9AYMO4kC3
EO7S8VRGDqfv1TBCnttYRjc/s0n23T+2C6er/IvwG9uleL66hFW6fbL/mjHpikud
eFrhwBP9MCZZRqnjM+1QtFldx09YpK7Txz9QtzajV8e2xhfXrWdTvA5jLd7hlgu6
wFPoiLuyxsAAye5HCD/b11OaiUSCPNgzj9lxjOk4dAMpkXuxSxke2JwZCrltL1zJ
7lY2OCibCFq1UwZwkfTiweelqs/yRk57ivQd1pvL/eIk/JPNqaUHcbUGGp01sWby
2w==
-----END CERTIFICATE-----

  Certificate Hash: "4f 2a a3 df 00 7c 21 93 e7 19 a7 bf e7 5c f9 85 23 63 6d 54"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913"
  Template Enrollment Flags: 0x29 (41)
    CT_FLAG_INCLUDE_SYMMETRIC_ALGORITHMS -- 1
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
  Template General Flags: 0x2023a (131642)
    CT_FLAG_ADD_EMAIL -- 2
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_EXPORTABLE_KEY -- 10 (16)
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
    CT_FLAG_ADD_TEMPLATE_NAME -- 200 (512)
    CT_FLAG_IS_MODIFIED -- 20000 (131072)
  Template Private Key Flags: 0x5030010 (84082704)
    CTPRIVATEKEY_FLAG_EXPORTABLE_KEY -- 10 (16)
    CTPRIVATEKEY_FLAG_ATTEST_NONE -- 0
    TEMPLATE_SERVER_VER_2008R2<<CTPRIVATEKEY_FLAG_SERVERVERSION_SHIFT -- 30000 (196608)
    TEMPLATE_CLIENT_VER_WINBLUE<<CTPRIVATEKEY_FLAG_CLIENTVERSION_SHIFT -- 5000000 (83886080)
  Serial Number: "4400000004b742b5daba3d4deb000000000004"
  Issuer Name ID: 0x0 CA Version 0.0
  Certificate Effective Date: 5/23/2026 5:29 PM
  Certificate Expiration Date: 4/25/2027 7:36 PM
  Issued Subject Key Identifier: "70 1c 01 bf 71 8f e0 01 73 71 39 3b 12 e4 7f 33 db 32 d2 55"
  Binary Public Key:
0000	30 82 01 0a 02 82 01 01  00 d2 81 2d 2f b4 84 18
0010	33 2d c3 b9 8b 8d a9 47  f6 ff e0 7c dc d7 4c c8
0020	68 c2 c0 7a c9 06 fb 8a  e4 1d d0 e6 6b f8 57 f1
0030	0c 9c b5 ce be 5f c9 a3  90 1b 31 64 18 f0 7a b1
0040	c6 ac 8f 89 e4 31 b7 e1  e1 d8 b1 f3 5b 03 ea 23
0050	3f f1 6d f7 d2 79 7b 64  da 0a 3d 76 32 61 f7 83
0060	29 7e e6 24 df e8 ee 93  e5 1d ef b0 4a d7 80 4b
0070	8c 15 d5 92 c0 73 f3 ba  8a 99 2d 3e ac 05 42 47
0080	c2 52 2f 49 f0 20 1f fb  82 3d 13 2b 09 2d 6a c2
0090	87 18 a2 93 a4 80 90 69  28 0c de 4e 54 67 de 7d
00a0	22 45 58 30 89 9c 66 4c  46 30 1a 02 66 75 78 26
00b0	3f 2a 43 27 ac 99 29 25  71 9d 8e 88 a9 33 99 89
00c0	1d 68 18 6e de 59 b5 70  ac 42 d4 41 c9 ae 2b 43
00d0	dd 14 30 4f 4d a1 80 d7  e2 5b 70 66 5e a3 d1 d1
00e0	44 3e 37 6a 05 38 fb 00  a9 20 84 bf 1b 28 9d 8f
00f0	02 16 8f ae b1 ec c6 91  af f4 60 0d 5d 6c bc b4
0100	09 4f bb 70 35 bd 60 9a  59 02 03 01 00 01

  Public Key Length: 0x800 (2048)
  Public Key Algorithm: "1.2.840.113549.1.1.1" RSA (RSA_SIGN)
  Public Key Algorithm Parameters:
0000	05 00                                              ..

  Publish Expired Certificate in CRL: 0x0
  User Principal Name: "pki.admin@corp.cvilab.local"
  Issued Distinguished Name: "CN=PKI Admin"
  Issued Binary Name:
0000	30 14 31 12 30 10 06 03  55 04 03 13 09 50 4b 49   0.1.0...U....PKI
0010	20 41 64 6d 69 6e                                   Admin

  Issued Country/Region: EMPTY
  Issued Organization: EMPTY
  Issued Organization Unit: EMPTY
  Issued Common Name: "PKI Admin"
  Issued City: EMPTY
  Issued State: EMPTY
  Issued Title: EMPTY
  Issued First Name: EMPTY
  Issued Initials: EMPTY
  Issued Last Name: EMPTY
  Issued Domain Component: EMPTY
  Issued Email Address: EMPTY
  Issued Street Address: EMPTY
  Issued Unstructured Name: EMPTY
  Issued Unstructured Address: EMPTY
  Issued Device Serial Number: EMPTY

  Request Attributes:
    RequestOSVersion: "10.0.20348.2"
    RequestCSPProvider: "Microsoft Enhanced Cryptographic Provider v1.0"
    cdc: "DC01.corp.cvilab.local"
    rmd: "PKI-SRV01.corp.cvilab.local"
    ccm: "PKI-SRV01.corp.cvilab.local"

  Certificate Extensions:
    1.3.6.1.4.1.311.21.7: Flags = 20000(Origin=Policy), Length = 31
    Certificate Template Information
        Template=1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913
        Major Version Number=100
        Minor Version Number=5

    2.5.29.37: Flags = 20000(Origin=Policy), Length = c
    Enhanced Key Usage
        Client Authentication (1.3.6.1.5.5.7.3.2)

    2.5.29.15: Flags = 20001(Critical, Origin=Policy), Length = 4
    Key Usage
        Digital Signature, Key Encipherment (a0)

    1.3.6.1.4.1.311.21.10: Flags = 20000(Origin=Policy), Length = e
    Application Policies
        [1]Application Certificate Policy:
             Policy Identifier=Client Authentication

    1.2.840.113549.1.9.15: Flags = 20000(Origin=Policy), Length = 37
    SMIME Capabilities
        [1]SMIME Capability
             Object ID=1.2.840.113549.3.2
             Parameters=02 02 00 80
        [2]SMIME Capability
             Object ID=1.2.840.113549.3.4
             Parameters=02 02 00 80
        [3]SMIME Capability
             Object ID=1.3.14.3.2.7
        [4]SMIME Capability
             Object ID=1.2.840.113549.3.7

    2.5.29.14: Flags = 10000(Origin=Request), Length = 16
    Subject Key Identifier
        701c01bf718fe0017371393b12e47f33db32d255

    2.5.29.35: Flags = 20000(Origin=Policy), Length = 18
    Authority Key Identifier
        KeyID=114d5edbf32dc1542007b8cd3f192d04bbe85fd1

    2.5.29.31: Flags = 40000(Origin=Server), Length = d7
    CRL Distribution Points
        [1]CRL Distribution Point
             Distribution Point Name:
                  Full Name:
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevoc
ationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PKI-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=
cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=cer
tificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?object
Class=certificationAuthority)

    2.5.29.19: Flags = 20003(Critical, Disabled, Origin=Policy), Length = 2
    Basic Constraints
        Subject Type=End Entity
        Path Length Constraint=None

    2.5.29.17: Flags = 20000(Origin=Policy), Length = 2f
    Subject Alternative Name
        Other Name:
             Principal Name=pki.admin@corp.cvilab.local


Row 2:
  Request ID: 0x5
  Binary Request:
-----BEGIN NEW CERTIFICATE REQUEST-----
MIIGFAYJKoZIhvcNAQcCoIIGBTCCBgECAQMxCzAJBgUrDgMCGgUAMIIEbgYIKwYB
BQUHDAKgggRgBIIEXDCCBFgwaTBnAgECBgorBgEEAYI3CgoBMVYwVAIBADADAgEB
MUowSAYJKwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxv
Y2FsDA5DT1JQXHBraS5hZG1pbgwHTU1DLkVYRTCCA+WgggPhAgEBMIID2jCCAsIC
AQAwADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKfkFmk+e54eoeTg
pe0gIrxUQ646BQE1q20B7Ws6ptBTLVpX6sTQiJToL9clCsx2/f8zLXU6NgfZQNTX
/HkLyjX74jnssl/ZK2zEzuYSf9sQ0kaviQlmiJdqLhjJ16iGpGPwdjMRer4pF5gH
I3f4jkRjAjnX6TgbcWMGIdHhnpHnmJDV137OZBXi2p1l3J+51Rq7TXauaDVzrlwx
S4zbCYAJRQcgK4+LCliuOtdzFRv7bwtOuZGyCwc/MELvnbIBt1oez+1dyDrQHHU5
0bqpGooDWzL+W0NX6vRAweC/QYrO6K4ZbWTi5q+ilxg5YersA+Wrr8A9XRv3msBF
MJAQ5CkCAwEAAaCCAZMwHAYKKwYBBAGCNw0CAzEOFgwxMC4wLjIwMzQ4LjIwSAYJ
KwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxvY2FsDA5D
T1JQXHBraS5hZG1pbgwHTU1DLkVYRTB0BgorBgEEAYI3DQICMWYwZAIBAh5cAE0A
aQBjAHIAbwBzAG8AZgB0ACAARQBuAGgAYQBuAGMAZQBkACAAQwByAHkAcAB0AG8A
ZwByAGEAcABoAGkAYwAgAFAAcgBvAHYAaQBkAGUAcgAgAHYAMQAuADADAQAwgbIG
CSqGSIb3DQEJDjGBpDCBoTA+BgkrBgEEAYI3FQcEMTAvBicrBgEEAYI3FQiHydJI
goaqPISljyiHissog+KoU4Epg+GzQYfXj1gCAWQCAQMwEwYDVR0lBAwwCgYIKwYB
BQUHAwMwDgYDVR0PAQH/BAQDAgeAMBsGCSsGAQQBgjcVCgQOMAwwCgYIKwYBBQUH
AwMwHQYDVR0OBBYEFORnPdbmPsg6GlzxK82O667rqw0BMA0GCSqGSIb3DQEBBQUA
A4IBAQBleTBt1QDcM39DGznc7s/bTSxwXGimXBjQp9tyTwzMt8j2dj0CxaOZZmiE
78q05+zyHl249TJaoarIgGm7D8e3DcyXZDQB8Oj4k2k2F6wQlhCRMXJeIZwB4wBl
Cb9jatbSkNmhjuCSKc6D1rBvlQzvF8W2IkvbPXrJfh2yMGUvT12eRZiAMqGE4dvC
H3Zo1NT4VFARfl1JIYjYaOuaOl7VWMGPWvPZyf6jSFUeOZ1cfA0S90WLkEep3rKQ
kKjSFMb01g6r1zQXL1dFLuRuPP4TP9aTFGrJfYBWHyjmSJf28wOLiO9/oFgw5s1f
FC7ccSyHY3/t0gYEu+tM2D9QAIZuMAAwADGCAXswggF3AgEDgBTkZz3W5j7IOhpc
8SvNjuuu66sNATAJBgUrDgMCGgUAoD4wFwYJKoZIhvcNAQkDMQoGCCsGAQUFBwwC
MCMGCSqGSIb3DQEJBDEWBBSnll4YGPiBt7IQ3wiHrzoBiqzT4jANBgkqhkiG9w0B
AQEFAASCAQAiwmI0XTlU7h2kpfpPCA5vwjcSI5sLkbZrTn2wZqE2XbvdoMpPGMzB
6Ou+1WVYYxSJh8KF4BNVW37tjanetJMqdET5mi1kpcOSdycAdxtZ48W2+6QmELBM
z0jtQ8SstJUzeIiApJ4q+D8yDyYZMQ0uroGdXQAqm7JuEbPvNbYHIATfdpwNR35w
BE4WTAN/o+igEbz1vRl4M/WtjeX6/rt9I786sGSotxWYKc1y9yYPgA1hE7j8/ggo
z0HM0e/5NloR1W+YN3zbZU2/vuAahO73kN9iLheJMDNXPanKxl3N3ayb5+3/UYT1
7kqZ+z2ap+B29nS1Hh7oeGakjU5tjeKo
-----END NEW CERTIFICATE REQUEST-----

  Archived Key: EMPTY
  Key Recovery Agent Hashes: EMPTY
  Old Certificate: EMPTY
  Request Attributes: "
cdc:DC01.corp.cvilab.local
rmd:PKI-SRV01.corp.cvilab.local

ccm:PKI-SRV01.corp.cvilab.local"
  Request Type: 0x40400 (263168) -- CMC, Full Response
  Request Flags: 0x4 -- Force UTF-8
  Request Status Code: 0x0 (WIN32: 0) -- The operation completed successfully.
  Request Disposition: 0x15 (21) -- Revoked
  Request Disposition Message: "Revoked by CORP\pki.admin"
  Request Submission Date: 5/24/2026 1:03 PM
  Request Resolution Date: 5/24/2026 1:03 PM
  Revocation Date: 5/30/2026 11:10 AM
  Effective Revocation Date: 5/30/2026 11:10 AM
  Revocation Reason: 0x5 -- Reason: Cessation of Operation
  Requester Name: "CORP\pki.admin"
  Caller Name: "CORP\pki.admin"
  Signer Policies: EMPTY
  Signer Application Policies: EMPTY
  Officer: EMPTY
  Request Distinguished Name: EMPTY
  Request Binary Name:
0000	30 00                                              0.

  Request Country/Region: EMPTY
  Request Organization: EMPTY
  Request Organization Unit: EMPTY
  Request Common Name: EMPTY
  Request City: EMPTY
  Request State: EMPTY
  Request Title: EMPTY
  Request First Name: EMPTY
  Request Initials: EMPTY
  Request Last Name: EMPTY
  Request Domain Component: EMPTY
  Request Email Address: EMPTY
  Request Street Address: EMPTY
  Request Unstructured Name: EMPTY
  Request Unstructured Address: EMPTY
  Request Device Serial Number: EMPTY
  Attestation Challenge: EMPTY
  Endorsement Key Hash: EMPTY
  Endorsement Certificate Hash: EMPTY
  Binary Precertificate: EMPTY
  Issued Request ID: 0x5
  Binary Certificate:
-----BEGIN CERTIFICATE-----
MIIFtzCCBJ+gAwIBAgITRAAAAAW0kiwuJOEA6wAAAAAABTANBgkqhkiG9w0BAQsF
ADBgMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFjAUBgoJkiaJk/IsZAEZFgZjdmls
YWIxFDASBgoJkiaJk/IsZAEZFgRjb3JwMRkwFwYDVQQDExBDVkkgSXNzdWluZyBD
QSAxMB4XDTI2MDUyNDE5NTM1OFoXDTI3MDQyNjAyMzY1OFowFDESMBAGA1UEAxMJ
UEtJIEFkbWluMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAp+QWaT57
nh6h5OCl7SAivFRDrjoFATWrbQHtazqm0FMtWlfqxNCIlOgv1yUKzHb9/zMtdTo2
B9lA1Nf8eQvKNfviOeyyX9krbMTO5hJ/2xDSRq+JCWaIl2ouGMnXqIakY/B2MxF6
vikXmAcjd/iORGMCOdfpOBtxYwYh0eGekeeYkNXXfs5kFeLanWXcn7nVGrtNdq5o
NXOuXDFLjNsJgAlFByArj4sKWK4613MVG/tvC065kbILBz8wQu+dsgG3Wh7P7V3I
OtAcdTnRuqkaigNbMv5bQ1fq9EDB4L9Bis7orhltZOLmr6KXGDlh6uwD5auvwD1d
G/eawEUwkBDkKQIDAQABo4ICtDCCArAwPgYJKwYBBAGCNxUHBDEwLwYnKwYBBAGC
NxUIh8nSSIKGqjyEpY8oh4rLKIPiqFOBKYPhs0GH149YAgFkAgEDMBMGA1UdJQQM
MAoGCCsGAQUFBwMDMA4GA1UdDwEB/wQEAwIHgDAbBgkrBgEEAYI3FQoEDjAMMAoG
CCsGAQUFBwMDMB0GA1UdDgQWBBTkZz3W5j7IOhpc8SvNjuuu66sNATAfBgNVHSME
GDAWgBQRTV7b8y3BVCAHuM0/GS0Eu+hf0TCB3wYDVR0fBIHXMIHUMIHRoIHOoIHL
hoHIbGRhcDovLy9DTj1DVkklMjBJc3N1aW5nJTIwQ0ElMjAxLENOPVBLSS1TUlYw
MSxDTj1DRFAsQ049UHVibGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMs
Q049Q29uZmlndXJhdGlvbixEQz1jb3JwLERDPWN2aWxhYixEQz1sb2NhbD9jZXJ0
aWZpY2F0ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJp
YnV0aW9uUG9pbnQwgdEGCCsGAQUFBwEBBIHEMIHBMIG+BggrBgEFBQcwAoaBsWxk
YXA6Ly8vQ049Q1ZJJTIwSXNzdWluZyUyMENBJTIwMSxDTj1BSUEsQ049UHVibGlj
JTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixE
Qz1jb3JwLERDPWN2aWxhYixEQz1sb2NhbD9jQUNlcnRpZmljYXRlP2Jhc2U/b2Jq
ZWN0Q2xhc3M9Y2VydGlmaWNhdGlvbkF1dGhvcml0eTA2BgNVHREELzAtoCsGCisG
AQQBgjcUAgOgHQwbcGtpLmFkbWluQGNvcnAuY3ZpbGFiLmxvY2FsMA0GCSqGSIb3
DQEBCwUAA4IBAQA3XwBUv6txCxY0gPVXTvY+dMbTGCcYNL5mmvGtzM42CIKa2hkk
xVPfayGFZ1ntd8Ycq63zZTsQHO/MpFCYy3ciBqdASOL4xpGr/L5f97fuAybg7ads
kKPDFnsZfKOvG9oRnJBFV871y7969SaswY1vlIX3ERUxdhf2XE4hOa3hVRp/FKXn
X8Iy+AG56dJ9mviED3q4WFIS0IDZO5xD1FYSCIo6I13vGJPTDKVpMCYlU/mM37Kg
28Co/mid3C5pfq9lkkX7/vqUZW/vADYyyhxrI3AglGhbB1n+bRbQrAI2OfYId6xa
t/MVlYXgTAHBTaGLKSxV7nQHLRsNAO74yIJM
-----END CERTIFICATE-----

  Certificate Hash: "7f a1 7d be 13 f3 c0 f3 ff 3e 60 af 85 28 c4 ed b4 8d 96 ce"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.7887297.16107480" CVI Code Signing
  Template Enrollment Flags: 0x20 (32)
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
  Template General Flags: 0x20220 (131616)
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
    CT_FLAG_ADD_TEMPLATE_NAME -- 200 (512)
    CT_FLAG_IS_MODIFIED -- 20000 (131072)
  Template Private Key Flags: 0x5050100 (84214016)
    CTPRIVATEKEY_FLAG_USE_LEGACY_PROVIDER -- 100 (256)
    CTPRIVATEKEY_FLAG_ATTEST_NONE -- 0
    TEMPLATE_SERVER_VER_WINBLUE<<CTPRIVATEKEY_FLAG_SERVERVERSION_SHIFT -- 50000 (327680)
    TEMPLATE_CLIENT_VER_WINBLUE<<CTPRIVATEKEY_FLAG_CLIENTVERSION_SHIFT -- 5000000 (83886080)
  Serial Number: "4400000005b4922c2e24e100eb000000000005"
  Issuer Name ID: 0x0 CA Version 0.0
  Certificate Effective Date: 5/24/2026 12:53 PM
  Certificate Expiration Date: 4/25/2027 7:36 PM
  Issued Subject Key Identifier: "e4 67 3d d6 e6 3e c8 3a 1a 5c f1 2b cd 8e eb ae eb ab 0d 01"
  Binary Public Key:
0000	30 82 01 0a 02 82 01 01  00 a7 e4 16 69 3e 7b 9e
0010	1e a1 e4 e0 a5 ed 20 22  bc 54 43 ae 3a 05 01 35
0020	ab 6d 01 ed 6b 3a a6 d0  53 2d 5a 57 ea c4 d0 88
0030	94 e8 2f d7 25 0a cc 76  fd ff 33 2d 75 3a 36 07
0040	d9 40 d4 d7 fc 79 0b ca  35 fb e2 39 ec b2 5f d9
0050	2b 6c c4 ce e6 12 7f db  10 d2 46 af 89 09 66 88
0060	97 6a 2e 18 c9 d7 a8 86  a4 63 f0 76 33 11 7a be
0070	29 17 98 07 23 77 f8 8e  44 63 02 39 d7 e9 38 1b
0080	71 63 06 21 d1 e1 9e 91  e7 98 90 d5 d7 7e ce 64
0090	15 e2 da 9d 65 dc 9f b9  d5 1a bb 4d 76 ae 68 35
00a0	73 ae 5c 31 4b 8c db 09  80 09 45 07 20 2b 8f 8b
00b0	0a 58 ae 3a d7 73 15 1b  fb 6f 0b 4e b9 91 b2 0b
00c0	07 3f 30 42 ef 9d b2 01  b7 5a 1e cf ed 5d c8 3a
00d0	d0 1c 75 39 d1 ba a9 1a  8a 03 5b 32 fe 5b 43 57
00e0	ea f4 40 c1 e0 bf 41 8a  ce e8 ae 19 6d 64 e2 e6
00f0	af a2 97 18 39 61 ea ec  03 e5 ab af c0 3d 5d 1b
0100	f7 9a c0 45 30 90 10 e4  29 02 03 01 00 01

  Public Key Length: 0x800 (2048)
  Public Key Algorithm: "1.2.840.113549.1.1.1" RSA (RSA_SIGN)
  Public Key Algorithm Parameters:
0000	05 00                                              ..

  Publish Expired Certificate in CRL: 0x1
  User Principal Name: "pki.admin@corp.cvilab.local"
  Issued Distinguished Name: "CN=PKI Admin"
  Issued Binary Name:
0000	30 14 31 12 30 10 06 03  55 04 03 13 09 50 4b 49   0.1.0...U....PKI
0010	20 41 64 6d 69 6e                                   Admin

  Issued Country/Region: EMPTY
  Issued Organization: EMPTY
  Issued Organization Unit: EMPTY
  Issued Common Name: "PKI Admin"
  Issued City: EMPTY
  Issued State: EMPTY
  Issued Title: EMPTY
  Issued First Name: EMPTY
  Issued Initials: EMPTY
  Issued Last Name: EMPTY
  Issued Domain Component: EMPTY
  Issued Email Address: EMPTY
  Issued Street Address: EMPTY
  Issued Unstructured Name: EMPTY
  Issued Unstructured Address: EMPTY
  Issued Device Serial Number: EMPTY

  Request Attributes:
    RequestOSVersion: "10.0.20348.2"
    RequestCSPProvider: "Microsoft Enhanced Cryptographic Provider v1.0"
    cdc: "DC01.corp.cvilab.local"
    rmd: "PKI-SRV01.corp.cvilab.local"
    ccm: "PKI-SRV01.corp.cvilab.local"

  Certificate Extensions:
    1.3.6.1.4.1.311.21.7: Flags = 20000(Origin=Policy), Length = 31
    Certificate Template Information
        Template=CVI Code Signing(1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.7887297.16107480)
        Major Version Number=100
        Minor Version Number=3

    2.5.29.37: Flags = 20000(Origin=Policy), Length = c
    Enhanced Key Usage
        Code Signing (1.3.6.1.5.5.7.3.3)

    2.5.29.15: Flags = 20001(Critical, Origin=Policy), Length = 4
    Key Usage
        Digital Signature (80)

    1.3.6.1.4.1.311.21.10: Flags = 20000(Origin=Policy), Length = e
    Application Policies
        [1]Application Certificate Policy:
             Policy Identifier=Code Signing

    2.5.29.14: Flags = 10000(Origin=Request), Length = 16
    Subject Key Identifier
        e4673dd6e63ec83a1a5cf12bcd8eebaeebab0d01

    2.5.29.35: Flags = 20000(Origin=Policy), Length = 18
    Authority Key Identifier
        KeyID=114d5edbf32dc1542007b8cd3f192d04bbe85fd1

    2.5.29.31: Flags = 40000(Origin=Server), Length = d7
    CRL Distribution Points
        [1]CRL Distribution Point
             Distribution Point Name:
                  Full Name:
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevoc
ationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PKI-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=
cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=cer
tificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?object
Class=certificationAuthority)

    2.5.29.19: Flags = 20003(Critical, Disabled, Origin=Policy), Length = 2
    Basic Constraints
        Subject Type=End Entity
        Path Length Constraint=None

    2.5.29.17: Flags = 20000(Origin=Policy), Length = 2f
    Subject Alternative Name
        Other Name:
             Principal Name=pki.admin@corp.cvilab.local


Row 3:
  Request ID: 0x6
  Binary Request:
-----BEGIN NEW CERTIFICATE REQUEST-----
MIIGWgYJKoZIhvcNAQcCoIIGSzCCBkcCAQMxCzAJBgUrDgMCGgUAMIIEtAYIKwYB
BQUHDAKgggSmBIIEojCCBJ4waTBnAgECBgorBgEEAYI3CgoBMVYwVAIBADADAgEB
MUowSAYJKwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxv
Y2FsDA5DT1JQXHBraS5hZG1pbgwHTU1DLkVYRTCCBCugggQnAgEBMIIEIDCCAwgC
AQAwADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMWrnzfCoFAmzidR
WFLoDTgkj7ffQJXUvMub59qifc383Na4xKRM/Hp5eIcTvHwsEzl4C+r7hHYiKSwV
y4e34IczCIDAE6txAhlo+caQ3+KY/NMusDiCBYH87kzEG9N6+mUxvHdsrZ+/ScaB
/BYso0SEqycpZPOw9MMqQ7ZL7/6ac+bccbrSTE0Npe4t+UCMo47dqnFgShKn3R0J
ri3Gbj82n8cn+I9uz/lZaBmc0dM3gXo8+1ajQcaRpG4sXB1E+MQRHtaCTWRw5Kek
erIXQtRnkmw7yjzb3HA/u5AzGUaGkiVhm7cdBFOc121XEpy+geQ++w9sHc8nYZv3
N9ZpJK0CAwEAAaCCAdkwHAYKKwYBBAGCNw0CAzEOFgwxMC4wLjIwMzQ4LjIwSAYJ
KwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxvY2FsDA5D
T1JQXHBraS5hZG1pbgwHTU1DLkVYRTB0BgorBgEEAYI3DQICMWYwZAIBAR5cAE0A
aQBjAHIAbwBzAG8AZgB0ACAARQBuAGgAYQBuAGMAZQBkACAAQwByAHkAcAB0AG8A
ZwByAGEAcABoAGkAYwAgAFAAcgBvAHYAaQBkAGUAcgAgAHYAMQAuADADAQAwgfgG
CSqGSIb3DQEJDjGB6jCB5zA+BgkrBgEEAYI3FQcEMTAvBicrBgEEAYI3FQiHydJI
goaqPISljyiHissog+KoU4EpgrzrYIaT4yECAWQCAQUwEwYDVR0lBAwwCgYIKwYB
BQUHAwIwDgYDVR0PAQH/BAQDAgWgMBsGCSsGAQQBgjcVCgQOMAwwCgYIKwYBBQUH
AwIwRAYJKoZIhvcNAQkPBDcwNTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQC
AgCAMAcGBSsOAwIHMAoGCCqGSIb3DQMHMB0GA1UdDgQWBBQuqmo72WuaG7RM2D0h
VtClCWrpmTANBgkqhkiG9w0BAQUFAAOCAQEAxaK9SBkpw1sjxMqkPZQhmTrpC2r0
P/86Yftn52aeVk0zm92rGubouCuO/7wxoBJAalyWjoSX5AWf6UaJq3dg2IrXS5+T
i+kjbfExNXcBzKFdmEcDK89YdvNpwbh4vLe2WL1Rq2+8DplJgx9WvM/bzekm6mB6
CiOaVNV4rvdalVTskmFsFfSEYjNtVAINPobZhMz4k6CcTQEr6gRoD3MsCy1Jo+KY
DhH79xNAUdS6YmZFk7Ek1pAMAJMMM++SEgbBYignoxv4JBRbsgCKW3ULzSpCw/qq
cEuMruuWKpFM9Pzn0anYu+u8TxvfeFk3WXDSTFRumPzHWDuZaXOdM+r2STAAMAAx
ggF7MIIBdwIBA4AULqpqO9lrmhu0TNg9IVbQpQlq6ZkwCQYFKw4DAhoFAKA+MBcG
CSqGSIb3DQEJAzEKBggrBgEFBQcMAjAjBgkqhkiG9w0BCQQxFgQUB6hMqUrSYQtb
gjumIoKCtq6+0mkwDQYJKoZIhvcNAQEBBQAEggEAufz0woc80R0/cFYSp3BAQhw5
4sIi0rCg91SyQFSWQo3UwC62dwWvMP7OH4WVWVcHZWEa6a/lM6RpYFIXcA+2ZsvY
mS/Phqu6XUKhXf46ew4DTMTWxZuUcLLijLDHuO+FxhB+W8V6XnzRDpcsx/r/2Ni7
t+3Q1sZ+Zo1vfsMPy0sYfPpHNOEsVPac2EQdY/9P5XF2+ZaR3xghjI3xBGcpo0eR
V4N6L/ulXMNQrNsFWK6IweRsTYKd9r5yWWyy8kXUioWxmFvxrmIDVPDDOu7JnwPk
g1ZoYsxaNbsu66axxct9OOkQvuuDLWgi2WcSF4YOps22d3YE/HR4UKOo6aJI5w==
-----END NEW CERTIFICATE REQUEST-----

  Archived Key: EMPTY
  Key Recovery Agent Hashes: EMPTY
  Old Certificate: EMPTY
  Request Attributes: "
cdc:DC01.corp.cvilab.local
rmd:PKI-SRV01.corp.cvilab.local

ccm:PKI-SRV01.corp.cvilab.local"
  Request Type: 0x40400 (263168) -- CMC, Full Response
  Request Flags: 0x4 -- Force UTF-8
  Request Status Code: 0x0 (WIN32: 0) -- The operation completed successfully.
  Request Disposition: 0x15 (21) -- Revoked
  Request Disposition Message: "Revoked by CORP\pki.admin"
  Request Submission Date: 5/26/2026 3:20 PM
  Request Resolution Date: 5/26/2026 3:20 PM
  Revocation Date: 5/28/2026 3:34 PM
  Effective Revocation Date: 5/28/2026 3:33 PM
  Revocation Reason: 0x5 -- Reason: Cessation of Operation
  Requester Name: "CORP\pki.admin"
  Caller Name: "CORP\pki.admin"
  Signer Policies: EMPTY
  Signer Application Policies: EMPTY
  Officer: EMPTY
  Request Distinguished Name: EMPTY
  Request Binary Name:
0000	30 00                                              0.

  Request Country/Region: EMPTY
  Request Organization: EMPTY
  Request Organization Unit: EMPTY
  Request Common Name: EMPTY
  Request City: EMPTY
  Request State: EMPTY
  Request Title: EMPTY
  Request First Name: EMPTY
  Request Initials: EMPTY
  Request Last Name: EMPTY
  Request Domain Component: EMPTY
  Request Email Address: EMPTY
  Request Street Address: EMPTY
  Request Unstructured Name: EMPTY
  Request Unstructured Address: EMPTY
  Request Device Serial Number: EMPTY
  Attestation Challenge: EMPTY
  Endorsement Key Hash: EMPTY
  Endorsement Certificate Hash: EMPTY
  Binary Precertificate: EMPTY
  Issued Request ID: 0x6
  Binary Certificate:
-----BEGIN CERTIFICATE-----
MIIF/TCCBOWgAwIBAgITRAAAAAbviCJiNAX1egAAAAAABjANBgkqhkiG9w0BAQsF
ADBgMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFjAUBgoJkiaJk/IsZAEZFgZjdmls
YWIxFDASBgoJkiaJk/IsZAEZFgRjb3JwMRkwFwYDVQQDExBDVkkgSXNzdWluZyBD
QSAxMB4XDTI2MDUyNjIyMTAxOVoXDTI3MDQyNjAyMzY1OFowFDESMBAGA1UEAxMJ
UEtJIEFkbWluMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxaufN8Kg
UCbOJ1FYUugNOCSPt99AldS8y5vn2qJ9zfzc1rjEpEz8enl4hxO8fCwTOXgL6vuE
diIpLBXLh7fghzMIgMATq3ECGWj5xpDf4pj80y6wOIIFgfzuTMQb03r6ZTG8d2yt
n79JxoH8FiyjRISrJylk87D0wypDtkvv/ppz5txxutJMTQ2l7i35QIyjjt2qcWBK
EqfdHQmuLcZuPzafxyf4j27P+VloGZzR0zeBejz7VqNBxpGkbixcHUT4xBEe1oJN
ZHDkp6R6shdC1GeSbDvKPNvccD+7kDMZRoaSJWGbtx0EU5zXbVcSnL6B5D77D2wd
zydhm/c31mkkrQIDAQABo4IC+jCCAvYwPgYJKwYBBAGCNxUHBDEwLwYnKwYBBAGC
NxUIh8nSSIKGqjyEpY8oh4rLKIPiqFOBKYK862CGk+MhAgFkAgEFMBMGA1UdJQQM
MAoGCCsGAQUFBwMCMA4GA1UdDwEB/wQEAwIFoDAbBgkrBgEEAYI3FQoEDjAMMAoG
CCsGAQUFBwMCMEQGCSqGSIb3DQEJDwQ3MDUwDgYIKoZIhvcNAwICAgCAMA4GCCqG
SIb3DQMEAgIAgDAHBgUrDgMCBzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQULqpqO9lr
mhu0TNg9IVbQpQlq6ZkwHwYDVR0jBBgwFoAUEU1e2/MtwVQgB7jNPxktBLvoX9Ew
gd8GA1UdHwSB1zCB1DCB0aCBzqCBy4aByGxkYXA6Ly8vQ049Q1ZJJTIwSXNzdWlu
ZyUyMENBJTIwMSxDTj1QS0ktU1JWMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUy
MFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y29ycCxE
Qz1jdmlsYWIsREM9bG9jYWw/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNl
P29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHRBggrBgEFBQcBAQSB
xDCBwTCBvgYIKwYBBQUHMAKGgbFsZGFwOi8vL0NOPUNWSSUyMElzc3VpbmclMjBD
QSUyMDEsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZp
Y2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y29ycCxEQz1jdmlsYWIsREM9bG9jYWw/
Y0FDZXJ0aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRo
b3JpdHkwNgYDVR0RBC8wLaArBgorBgEEAYI3FAIDoB0MG3BraS5hZG1pbkBjb3Jw
LmN2aWxhYi5sb2NhbDANBgkqhkiG9w0BAQsFAAOCAQEANSg5YvEYSXYmAftPJqM5
jAepkG5BLew29SKEA/sZ04iMH+j4FdrNAev0r+xk8/7pGQVyEESRQLukCNjSI9KU
jCsNpUFzFM64dlJ+hSpvpT4jgXz7Lg7Qs5GOU/gA0wBWybPomRK9dwksjL+2r9bK
TWckNj0tJvBuxhYztPPrhVNEAD1JC32ODCIBAfvoQzr4XEBr/Gl9EBdKC7ageqDp
4ndtyvUqHDFyGWs2i6VE1OPjRoI5at0pTQwch9aMJuZjTNgGy2249nG5vLdYflRN
Tf6u+b2lC1ZSd4zqNSmA+wCSqT7rDS0KWOJ3hz6D8RxrVn9JVFv9GhPssQWtbNU2
hA==
-----END CERTIFICATE-----

  Certificate Hash: "9b c0 13 68 c2 62 75 26 bb a4 9f 21 69 50 95 98 07 5a f7 6a"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913"
  Template Enrollment Flags: 0x29 (41)
    CT_FLAG_INCLUDE_SYMMETRIC_ALGORITHMS -- 1
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
  Template General Flags: 0x2023a (131642)
    CT_FLAG_ADD_EMAIL -- 2
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_EXPORTABLE_KEY -- 10 (16)
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
    CT_FLAG_ADD_TEMPLATE_NAME -- 200 (512)
    CT_FLAG_IS_MODIFIED -- 20000 (131072)
  Template Private Key Flags: 0x5030010 (84082704)
    CTPRIVATEKEY_FLAG_EXPORTABLE_KEY -- 10 (16)
    CTPRIVATEKEY_FLAG_ATTEST_NONE -- 0
    TEMPLATE_SERVER_VER_2008R2<<CTPRIVATEKEY_FLAG_SERVERVERSION_SHIFT -- 30000 (196608)
    TEMPLATE_CLIENT_VER_WINBLUE<<CTPRIVATEKEY_FLAG_CLIENTVERSION_SHIFT -- 5000000 (83886080)
  Serial Number: "4400000006ef8822623405f57a000000000006"
  Issuer Name ID: 0x0 CA Version 0.0
  Certificate Effective Date: 5/26/2026 3:10 PM
  Certificate Expiration Date: 4/25/2027 7:36 PM
  Issued Subject Key Identifier: "2e aa 6a 3b d9 6b 9a 1b b4 4c d8 3d 21 56 d0 a5 09 6a e9 99"
  Binary Public Key:
0000	30 82 01 0a 02 82 01 01  00 c5 ab 9f 37 c2 a0 50
0010	26 ce 27 51 58 52 e8 0d  38 24 8f b7 df 40 95 d4
0020	bc cb 9b e7 da a2 7d cd  fc dc d6 b8 c4 a4 4c fc
0030	7a 79 78 87 13 bc 7c 2c  13 39 78 0b ea fb 84 76
0040	22 29 2c 15 cb 87 b7 e0  87 33 08 80 c0 13 ab 71
0050	02 19 68 f9 c6 90 df e2  98 fc d3 2e b0 38 82 05
0060	81 fc ee 4c c4 1b d3 7a  fa 65 31 bc 77 6c ad 9f
0070	bf 49 c6 81 fc 16 2c a3  44 84 ab 27 29 64 f3 b0
0080	f4 c3 2a 43 b6 4b ef fe  9a 73 e6 dc 71 ba d2 4c
0090	4d 0d a5 ee 2d f9 40 8c  a3 8e dd aa 71 60 4a 12
00a0	a7 dd 1d 09 ae 2d c6 6e  3f 36 9f c7 27 f8 8f 6e
00b0	cf f9 59 68 19 9c d1 d3  37 81 7a 3c fb 56 a3 41
00c0	c6 91 a4 6e 2c 5c 1d 44  f8 c4 11 1e d6 82 4d 64
00d0	70 e4 a7 a4 7a b2 17 42  d4 67 92 6c 3b ca 3c db
00e0	dc 70 3f bb 90 33 19 46  86 92 25 61 9b b7 1d 04
00f0	53 9c d7 6d 57 12 9c be  81 e4 3e fb 0f 6c 1d cf
0100	27 61 9b f7 37 d6 69 24  ad 02 03 01 00 01

  Public Key Length: 0x800 (2048)
  Public Key Algorithm: "1.2.840.113549.1.1.1" RSA (RSA_SIGN)
  Public Key Algorithm Parameters:
0000	05 00                                              ..

  Publish Expired Certificate in CRL: 0x0
  User Principal Name: "pki.admin@corp.cvilab.local"
  Issued Distinguished Name: "CN=PKI Admin"
  Issued Binary Name:
0000	30 14 31 12 30 10 06 03  55 04 03 13 09 50 4b 49   0.1.0...U....PKI
0010	20 41 64 6d 69 6e                                   Admin

  Issued Country/Region: EMPTY
  Issued Organization: EMPTY
  Issued Organization Unit: EMPTY
  Issued Common Name: "PKI Admin"
  Issued City: EMPTY
  Issued State: EMPTY
  Issued Title: EMPTY
  Issued First Name: EMPTY
  Issued Initials: EMPTY
  Issued Last Name: EMPTY
  Issued Domain Component: EMPTY
  Issued Email Address: EMPTY
  Issued Street Address: EMPTY
  Issued Unstructured Name: EMPTY
  Issued Unstructured Address: EMPTY
  Issued Device Serial Number: EMPTY

  Request Attributes:
    RequestOSVersion: "10.0.20348.2"
    RequestCSPProvider: "Microsoft Enhanced Cryptographic Provider v1.0"
    cdc: "DC01.corp.cvilab.local"
    rmd: "PKI-SRV01.corp.cvilab.local"
    ccm: "PKI-SRV01.corp.cvilab.local"

  Certificate Extensions:
    1.3.6.1.4.1.311.21.7: Flags = 20000(Origin=Policy), Length = 31
    Certificate Template Information
        Template=1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913
        Major Version Number=100
        Minor Version Number=5

    2.5.29.37: Flags = 20000(Origin=Policy), Length = c
    Enhanced Key Usage
        Client Authentication (1.3.6.1.5.5.7.3.2)

    2.5.29.15: Flags = 20001(Critical, Origin=Policy), Length = 4
    Key Usage
        Digital Signature, Key Encipherment (a0)

    1.3.6.1.4.1.311.21.10: Flags = 20000(Origin=Policy), Length = e
    Application Policies
        [1]Application Certificate Policy:
             Policy Identifier=Client Authentication

    1.2.840.113549.1.9.15: Flags = 20000(Origin=Policy), Length = 37
    SMIME Capabilities
        [1]SMIME Capability
             Object ID=1.2.840.113549.3.2
             Parameters=02 02 00 80
        [2]SMIME Capability
             Object ID=1.2.840.113549.3.4
             Parameters=02 02 00 80
        [3]SMIME Capability
             Object ID=1.3.14.3.2.7
        [4]SMIME Capability
             Object ID=1.2.840.113549.3.7

    2.5.29.14: Flags = 10000(Origin=Request), Length = 16
    Subject Key Identifier
        2eaa6a3bd96b9a1bb44cd83d2156d0a5096ae999

    2.5.29.35: Flags = 20000(Origin=Policy), Length = 18
    Authority Key Identifier
        KeyID=114d5edbf32dc1542007b8cd3f192d04bbe85fd1

    2.5.29.31: Flags = 40000(Origin=Server), Length = d7
    CRL Distribution Points
        [1]CRL Distribution Point
             Distribution Point Name:
                  Full Name:
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevoc
ationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PKI-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=
cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=cer
tificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?object
Class=certificationAuthority)

    2.5.29.19: Flags = 20003(Critical, Disabled, Origin=Policy), Length = 2
    Basic Constraints
        Subject Type=End Entity
        Path Length Constraint=None

    2.5.29.17: Flags = 20000(Origin=Policy), Length = 2f
    Subject Alternative Name
        Other Name:
             Principal Name=pki.admin@corp.cvilab.local


Row 4:
  Request ID: 0x7
  Binary Request:
-----BEGIN NEW CERTIFICATE REQUEST-----
MIIGWgYJKoZIhvcNAQcCoIIGSzCCBkcCAQMxCzAJBgUrDgMCGgUAMIIEtAYIKwYB
BQUHDAKgggSmBIIEojCCBJ4waTBnAgECBgorBgEEAYI3CgoBMVYwVAIBADADAgEB
MUowSAYJKwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxv
Y2FsDA5DT1JQXHBraS5hZG1pbgwHTU1DLkVYRTCCBCugggQnAgEBMIIEIDCCAwgC
AQAwADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKEjc1kPFiHvpB3j
nbchkyIITRvkMkHN48RJa90cxPVZMC1/RDl7oL7PeClFNDZsrggMSBUpWTve1d1R
ovmCDdlCUMPhWWjWHdsNjUrWIGbPM024IKk3Dv3oSdHI28awEoO+uK0yDq3nybQG
Sp1aaGRP4NhwFNQIe4oaWwbImh+3g9gTt/rjEJIlBtUGCGCyNilPZfa2XVHnbvuc
ywJ7lX9/L2dLiJcfYABqaQj/sWgD27A4AhpFyBPn8xgjJp2aP/hRGXPhqLq9p0/1
+xu6TfTk+D/UrJuFwNh1s44eq0vj0naoBsohWxcqwUK98Va7D5FIksAf8T0wnD63
jw8bu+UCAwEAAaCCAdkwHAYKKwYBBAGCNw0CAzEOFgwxMC4wLjIwMzQ4LjIwSAYJ
KwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxvY2FsDA5D
T1JQXHBraS5hZG1pbgwHTU1DLkVYRTB0BgorBgEEAYI3DQICMWYwZAIBAR5cAE0A
aQBjAHIAbwBzAG8AZgB0ACAARQBuAGgAYQBuAGMAZQBkACAAQwByAHkAcAB0AG8A
ZwByAGEAcABoAGkAYwAgAFAAcgBvAHYAaQBkAGUAcgAgAHYAMQAuADADAQAwgfgG
CSqGSIb3DQEJDjGB6jCB5zA+BgkrBgEEAYI3FQcEMTAvBicrBgEEAYI3FQiHydJI
goaqPISljyiHissog+KoU4EpgrzrYIaT4yECAWQCAQkwEwYDVR0lBAwwCgYIKwYB
BQUHAwIwDgYDVR0PAQH/BAQDAgWgMBsGCSsGAQQBgjcVCgQOMAwwCgYIKwYBBQUH
AwIwRAYJKoZIhvcNAQkPBDcwNTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQC
AgCAMAcGBSsOAwIHMAoGCCqGSIb3DQMHMB0GA1UdDgQWBBTShU2aX2LtMI78A+Qm
SkYU/L72KDANBgkqhkiG9w0BAQUFAAOCAQEAKsvbsBvMaVKnAKZ0Ix6x8xvoouTi
WR8v7k39jBj3O+j+IKxm6khQndLSHEpc3XwZ7NBtY9W3kOzEduwWb1gdZnaEgQGC
zC2Yw2L05bKdxriK8oQzFlMhbKuvEfXJqdpT0X8y/UcQElfiYVgZVZq3lc1p6Y52
gXTmlCoQHPMGlSEuPSaCaX+E2PTEnCzSorrcvLPhRdodFf+qSHRplippIXITJPYV
Ikva6mzOUUYsm58Dm6qz3JaPSTR5/VxwaKCUuOjZyUHj9638sTfv/i/Y3nkRoGX3
dpamQqp6AfhuzrfvreyVWiaZFla6Q4nkSh+fwh0gV5T0pH84StozGdcpYzAAMAAx
ggF7MIIBdwIBA4AU0oVNml9i7TCO/APkJkpGFPy+9igwCQYFKw4DAhoFAKA+MBcG
CSqGSIb3DQEJAzEKBggrBgEFBQcMAjAjBgkqhkiG9w0BCQQxFgQU3V+T6DeQCQ8Z
NM/Tx7BgQssQUQcwDQYJKoZIhvcNAQEBBQAEggEAUBiHFnA2+Xuh9XmBeaOxGx0f
PdpV2j3y6ZBnJHOsmAsRhHlEQ3Cp9G8ptm1Bb+J/6mP63JlheGL7/tOHCDVdcxbk
ad2XQIkupZRi7zny4csM09xSB+MZbTtlLuVaPb5f0HTsuTlkjdPr8OKjpr0RHDhk
wxdromFQmKkmFHJgbSyb8rntAH+XZF9oWLMv79PHVXoYUJD9X4sPri0vpFUbvzlm
7vsuv+7K7LtGVzTD6x0xo13qB2FYE9zYP2vtcFki+ZK3fwmo1yLV8wXxZ5/XwWC2
K0MFSxG4c9HtbeJ09IYXUazx9t5au+vBMZIpCvMx4o7iNlzz6SHDSLLi6HCP8Q==
-----END NEW CERTIFICATE REQUEST-----

  Archived Key: EMPTY
  Key Recovery Agent Hashes: EMPTY
  Old Certificate: EMPTY
  Request Attributes: "
cdc:DC01.corp.cvilab.local
rmd:PKI-SRV01.corp.cvilab.local

ccm:PKI-SRV01.corp.cvilab.local"
  Request Type: 0x40400 (263168) -- CMC, Full Response
  Request Flags: 0x4 -- Force UTF-8
  Request Status Code: 0x0 (WIN32: 0) -- The operation completed successfully.
  Request Disposition: 0x15 (21) -- Revoked
  Request Disposition Message: "Revoked by CORP\pki.admin"
  Request Submission Date: 5/26/2026 3:48 PM
  Request Resolution Date: 5/26/2026 3:48 PM
  Revocation Date: 5/28/2026 3:33 PM
  Effective Revocation Date: 5/28/2026 3:33 PM
  Revocation Reason: 0x5 -- Reason: Cessation of Operation
  Requester Name: "CORP\pki.admin"
  Caller Name: "CORP\pki.admin"
  Signer Policies: EMPTY
  Signer Application Policies: EMPTY
  Officer: EMPTY
  Request Distinguished Name: EMPTY
  Request Binary Name:
0000	30 00                                              0.

  Request Country/Region: EMPTY
  Request Organization: EMPTY
  Request Organization Unit: EMPTY
  Request Common Name: EMPTY
  Request City: EMPTY
  Request State: EMPTY
  Request Title: EMPTY
  Request First Name: EMPTY
  Request Initials: EMPTY
  Request Last Name: EMPTY
  Request Domain Component: EMPTY
  Request Email Address: EMPTY
  Request Street Address: EMPTY
  Request Unstructured Name: EMPTY
  Request Unstructured Address: EMPTY
  Request Device Serial Number: EMPTY
  Attestation Challenge: EMPTY
  Endorsement Key Hash: EMPTY
  Endorsement Certificate Hash: EMPTY
  Binary Precertificate: EMPTY
  Issued Request ID: 0x7
  Binary Certificate:
-----BEGIN CERTIFICATE-----
MIIF/TCCBOWgAwIBAgITRAAAAAd7mShJ8dLyDgAAAAAABzANBgkqhkiG9w0BAQsF
ADBgMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFjAUBgoJkiaJk/IsZAEZFgZjdmls
YWIxFDASBgoJkiaJk/IsZAEZFgRjb3JwMRkwFwYDVQQDExBDVkkgSXNzdWluZyBD
QSAxMB4XDTI2MDUyNjIyMzgzM1oXDTI3MDQyNjAyMzY1OFowFDESMBAGA1UEAxMJ
UEtJIEFkbWluMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAoSNzWQ8W
Ie+kHeOdtyGTIghNG+QyQc3jxElr3RzE9VkwLX9EOXugvs94KUU0NmyuCAxIFSlZ
O97V3VGi+YIN2UJQw+FZaNYd2w2NStYgZs8zTbggqTcO/ehJ0cjbxrASg764rTIO
refJtAZKnVpoZE/g2HAU1Ah7ihpbBsiaH7eD2BO3+uMQkiUG1QYIYLI2KU9l9rZd
Uedu+5zLAnuVf38vZ0uIlx9gAGppCP+xaAPbsDgCGkXIE+fzGCMmnZo/+FEZc+Go
ur2nT/X7G7pN9OT4P9Ssm4XA2HWzjh6rS+PSdqgGyiFbFyrBQr3xVrsPkUiSwB/x
PTCcPrePDxu75QIDAQABo4IC+jCCAvYwPgYJKwYBBAGCNxUHBDEwLwYnKwYBBAGC
NxUIh8nSSIKGqjyEpY8oh4rLKIPiqFOBKYK862CGk+MhAgFkAgEJMBMGA1UdJQQM
MAoGCCsGAQUFBwMCMA4GA1UdDwEB/wQEAwIFoDAbBgkrBgEEAYI3FQoEDjAMMAoG
CCsGAQUFBwMCMEQGCSqGSIb3DQEJDwQ3MDUwDgYIKoZIhvcNAwICAgCAMA4GCCqG
SIb3DQMEAgIAgDAHBgUrDgMCBzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQU0oVNml9i
7TCO/APkJkpGFPy+9igwHwYDVR0jBBgwFoAUEU1e2/MtwVQgB7jNPxktBLvoX9Ew
gd8GA1UdHwSB1zCB1DCB0aCBzqCBy4aByGxkYXA6Ly8vQ049Q1ZJJTIwSXNzdWlu
ZyUyMENBJTIwMSxDTj1QS0ktU1JWMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUy
MFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y29ycCxE
Qz1jdmlsYWIsREM9bG9jYWw/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNl
P29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHRBggrBgEFBQcBAQSB
xDCBwTCBvgYIKwYBBQUHMAKGgbFsZGFwOi8vL0NOPUNWSSUyMElzc3VpbmclMjBD
QSUyMDEsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZp
Y2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y29ycCxEQz1jdmlsYWIsREM9bG9jYWw/
Y0FDZXJ0aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRo
b3JpdHkwNgYDVR0RBC8wLaArBgorBgEEAYI3FAIDoB0MG3BraS5hZG1pbkBjb3Jw
LmN2aWxhYi5sb2NhbDANBgkqhkiG9w0BAQsFAAOCAQEAUy5+4iqU+dt1KDUilkA/
qwxd48H7ndLmoX1mWqGvEmU77bj9efPjtL8N5Kli6IUGwRO+G0Jfl6Qk/5RYZ5VR
viBv3pqTcgGOiFMXgCBXgkv++onwUZ3za5A3RS/FaGCk8RiLASwUqvPo/ZmDtKUW
kkxtAbikze8QjdeE4qPChJWxZmhP7z+bZ39cu6Qpn9EUM7aLeGGti9B9CJ1gzOzv
0Vny+x+oK5wl1WGqTQ2ZwvWou4mRhKNowTM0+PiTh5G+XHyrTPLp/AxGhqL9FXJV
Ig6QxoeEy/YzkAigujn8NwgBRtzkKm2eJ1yydrTAZFxmPjWx+aen4dg3zuu3c/LH
Xg==
-----END CERTIFICATE-----

  Certificate Hash: "25 d0 15 94 a2 bc 5b c3 ce d2 d1 25 99 23 a4 20 ee a7 6e 4a"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913"
  Template Enrollment Flags: 0x29 (41)
    CT_FLAG_INCLUDE_SYMMETRIC_ALGORITHMS -- 1
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
  Template General Flags: 0x2023a (131642)
    CT_FLAG_ADD_EMAIL -- 2
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_EXPORTABLE_KEY -- 10 (16)
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
    CT_FLAG_ADD_TEMPLATE_NAME -- 200 (512)
    CT_FLAG_IS_MODIFIED -- 20000 (131072)
  Template Private Key Flags: 0x5030010 (84082704)
    CTPRIVATEKEY_FLAG_EXPORTABLE_KEY -- 10 (16)
    CTPRIVATEKEY_FLAG_ATTEST_NONE -- 0
    TEMPLATE_SERVER_VER_2008R2<<CTPRIVATEKEY_FLAG_SERVERVERSION_SHIFT -- 30000 (196608)
    TEMPLATE_CLIENT_VER_WINBLUE<<CTPRIVATEKEY_FLAG_CLIENTVERSION_SHIFT -- 5000000 (83886080)
  Serial Number: "44000000077b992849f1d2f20e000000000007"
  Issuer Name ID: 0x0 CA Version 0.0
  Certificate Effective Date: 5/26/2026 3:38 PM
  Certificate Expiration Date: 4/25/2027 7:36 PM
  Issued Subject Key Identifier: "d2 85 4d 9a 5f 62 ed 30 8e fc 03 e4 26 4a 46 14 fc be f6 28"
  Binary Public Key:
0000	30 82 01 0a 02 82 01 01  00 a1 23 73 59 0f 16 21
0010	ef a4 1d e3 9d b7 21 93  22 08 4d 1b e4 32 41 cd
0020	e3 c4 49 6b dd 1c c4 f5  59 30 2d 7f 44 39 7b a0
0030	be cf 78 29 45 34 36 6c  ae 08 0c 48 15 29 59 3b
0040	de d5 dd 51 a2 f9 82 0d  d9 42 50 c3 e1 59 68 d6
0050	1d db 0d 8d 4a d6 20 66  cf 33 4d b8 20 a9 37 0e
0060	fd e8 49 d1 c8 db c6 b0  12 83 be b8 ad 32 0e ad
0070	e7 c9 b4 06 4a 9d 5a 68  64 4f e0 d8 70 14 d4 08
0080	7b 8a 1a 5b 06 c8 9a 1f  b7 83 d8 13 b7 fa e3 10
0090	92 25 06 d5 06 08 60 b2  36 29 4f 65 f6 b6 5d 51
00a0	e7 6e fb 9c cb 02 7b 95  7f 7f 2f 67 4b 88 97 1f
00b0	60 00 6a 69 08 ff b1 68  03 db b0 38 02 1a 45 c8
00c0	13 e7 f3 18 23 26 9d 9a  3f f8 51 19 73 e1 a8 ba
00d0	bd a7 4f f5 fb 1b ba 4d  f4 e4 f8 3f d4 ac 9b 85
00e0	c0 d8 75 b3 8e 1e ab 4b  e3 d2 76 a8 06 ca 21 5b
00f0	17 2a c1 42 bd f1 56 bb  0f 91 48 92 c0 1f f1 3d
0100	30 9c 3e b7 8f 0f 1b bb  e5 02 03 01 00 01

  Public Key Length: 0x800 (2048)
  Public Key Algorithm: "1.2.840.113549.1.1.1" RSA (RSA_SIGN)
  Public Key Algorithm Parameters:
0000	05 00                                              ..

  Publish Expired Certificate in CRL: 0x0
  User Principal Name: "pki.admin@corp.cvilab.local"
  Issued Distinguished Name: "CN=PKI Admin"
  Issued Binary Name:
0000	30 14 31 12 30 10 06 03  55 04 03 13 09 50 4b 49   0.1.0...U....PKI
0010	20 41 64 6d 69 6e                                   Admin

  Issued Country/Region: EMPTY
  Issued Organization: EMPTY
  Issued Organization Unit: EMPTY
  Issued Common Name: "PKI Admin"
  Issued City: EMPTY
  Issued State: EMPTY
  Issued Title: EMPTY
  Issued First Name: EMPTY
  Issued Initials: EMPTY
  Issued Last Name: EMPTY
  Issued Domain Component: EMPTY
  Issued Email Address: EMPTY
  Issued Street Address: EMPTY
  Issued Unstructured Name: EMPTY
  Issued Unstructured Address: EMPTY
  Issued Device Serial Number: EMPTY

  Request Attributes:
    RequestOSVersion: "10.0.20348.2"
    RequestCSPProvider: "Microsoft Enhanced Cryptographic Provider v1.0"
    cdc: "DC01.corp.cvilab.local"
    rmd: "PKI-SRV01.corp.cvilab.local"
    ccm: "PKI-SRV01.corp.cvilab.local"

  Certificate Extensions:
    1.3.6.1.4.1.311.21.7: Flags = 20000(Origin=Policy), Length = 31
    Certificate Template Information
        Template=1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913
        Major Version Number=100
        Minor Version Number=9

    2.5.29.37: Flags = 20000(Origin=Policy), Length = c
    Enhanced Key Usage
        Client Authentication (1.3.6.1.5.5.7.3.2)

    2.5.29.15: Flags = 20001(Critical, Origin=Policy), Length = 4
    Key Usage
        Digital Signature, Key Encipherment (a0)

    1.3.6.1.4.1.311.21.10: Flags = 20000(Origin=Policy), Length = e
    Application Policies
        [1]Application Certificate Policy:
             Policy Identifier=Client Authentication

    1.2.840.113549.1.9.15: Flags = 20000(Origin=Policy), Length = 37
    SMIME Capabilities
        [1]SMIME Capability
             Object ID=1.2.840.113549.3.2
             Parameters=02 02 00 80
        [2]SMIME Capability
             Object ID=1.2.840.113549.3.4
             Parameters=02 02 00 80
        [3]SMIME Capability
             Object ID=1.3.14.3.2.7
        [4]SMIME Capability
             Object ID=1.2.840.113549.3.7

    2.5.29.14: Flags = 10000(Origin=Request), Length = 16
    Subject Key Identifier
        d2854d9a5f62ed308efc03e4264a4614fcbef628

    2.5.29.35: Flags = 20000(Origin=Policy), Length = 18
    Authority Key Identifier
        KeyID=114d5edbf32dc1542007b8cd3f192d04bbe85fd1

    2.5.29.31: Flags = 40000(Origin=Server), Length = d7
    CRL Distribution Points
        [1]CRL Distribution Point
             Distribution Point Name:
                  Full Name:
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevoc
ationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PKI-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=
cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=cer
tificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?object
Class=certificationAuthority)

    2.5.29.19: Flags = 20003(Critical, Disabled, Origin=Policy), Length = 2
    Basic Constraints
        Subject Type=End Entity
        Path Length Constraint=None

    2.5.29.17: Flags = 20000(Origin=Policy), Length = 2f
    Subject Alternative Name
        Other Name:
             Principal Name=pki.admin@corp.cvilab.local


Maximum Row Index: 4

4 Rows
 148 Row Properties, Total Size = 17728, Max Size = 1630, Ave Size = 119
  20 Request Attributes, Total Size = 1496, Max Size = 132, Ave Size = 74
  43 Certificate Extensions, Total Size = 3943, Max Size = 241, Ave Size = 91
 211 Total Fields, Total Size = 23167, Max Size = 1630, Ave Size = 109
CertUtil: -view command completed successfully.
````

</details>


**Total number of revoked certificates found:**
```
4
```

### Step 4 — Find Certificates From Your Prior Labs

Use the requester filter to find certificates associated with the accounts used in your Week 10 and 11 labs.

```powershell
certutil -view -restrict "RequesterName=CORP\pki.admin"
```

```
PS C:\Windows\system32> certutil -view -restrict "RequesterName=CORP\pki.admin"
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  Request.RequestID             Request ID                    Long    4 -- Indexed
  Request.RawRequest            Binary Request                Binary  65536
  Request.RawArchivedKey        Archived Key                  Binary  65536
  Request.KeyRecoveryHashes     Key Recovery Agent Hashes     String  8192
  Request.RawOldCertificate     Old Certificate               Binary  16384
  Request.RequestAttributes     Request Attributes            String  32768
  Request.RequestType           Request Type                  Long    4
  Request.RequestFlags          Request Flags                 Long    4
  Request.StatusCode            Request Status Code           Long    4
  Request.Disposition           Request Disposition           Long    4 -- Indexed
  Request.DispositionMessage    Request Disposition Message   String  8192
  Request.SubmittedWhen         Request Submission Date       Date    8 -- Indexed
  Request.ResolvedWhen          Request Resolution Date       Date    8 -- Indexed
  Request.RevokedWhen           Revocation Date               Date    8
  Request.RevokedEffectiveWhen  Effective Revocation Date     Date    8 -- Indexed
  Request.RevokedReason         Revocation Reason             Long    4
  Request.RequesterName         Requester Name                String  2048 -- Indexed
  Request.CallerName            Caller Name                   String  2048 -- Indexed
  Request.SignerPolicies        Signer Policies               String  8192
  Request.SignerApplicationPolicies  Signer Application Policies   String  8192
  Request.Officer               Officer                       Long    4
  Request.DistinguishedName     Request Distinguished Name    String  8192
  Request.RawName               Request Binary Name           Binary  4096
  Request.Country               Request Country/Region        String  8192
  Request.Organization          Request Organization          String  8192
  Request.OrgUnit               Request Organization Unit     String  8192
  Request.CommonName            Request Common Name           String  8192
  Request.Locality              Request City                  String  8192
  Request.State                 Request State                 String  8192
  Request.Title                 Request Title                 String  8192
  Request.GivenName             Request First Name            String  8192
  Request.Initials              Request Initials              String  8192
  Request.SurName               Request Last Name             String  8192
  Request.DomainComponent       Request Domain Component      String  8192
  Request.EMail                 Request Email Address         String  8192
  Request.StreetAddress         Request Street Address        String  8192
  Request.UnstructuredName      Request Unstructured Name     String  8192
  Request.UnstructuredAddress   Request Unstructured Address  String  8192
  Request.DeviceSerialNumber    Request Device Serial Number  String  8192
  Request.AttestationChallenge  Attestation Challenge         Binary  4096
  Request.EndorsementKeyHash    Endorsement Key Hash          String  144 -- Indexed
  Request.EndorsementCertificateHash  Endorsement Certificate Hash  String  144 -- Indexed
  Request.RawPrecertificate     Binary Precertificate         Binary  16384
  RequestID                     Issued Request ID             Long    4 -- Indexed
  RawCertificate                Binary Certificate            Binary  16384
  CertificateHash               Certificate Hash              String  128 -- Indexed
  CertificateTemplate           Certificate Template          String  254 -- Indexed
  EnrollmentFlags               Template Enrollment Flags     Long    4
  GeneralFlags                  Template General Flags        Long    4
  PrivatekeyFlags               Template Private Key Flags    Long    4
  SerialNumber                  Serial Number                 String  128 -- Indexed
  IssuerNameID                  Issuer Name ID                Long    4
  NotBefore                     Certificate Effective Date    Date    8
  NotAfter                      Certificate Expiration Date   Date    8 -- Indexed
  SubjectKeyIdentifier          Issued Subject Key Identifier  String  128 -- Indexed
  RawPublicKey                  Binary Public Key             Binary  4096
  PublicKeyLength               Public Key Length             Long    4
  PublicKeyAlgorithm            Public Key Algorithm          String  254
  RawPublicKeyAlgorithmParameters  Public Key Algorithm Parameters  Binary  4096
  PublishExpiredCertInCRL       Publish Expired Certificate in CRL  Long    4
  UPN                           User Principal Name           String  2048 -- Indexed
  DistinguishedName             Issued Distinguished Name     String  8192
  RawName                       Issued Binary Name            Binary  4096
  Country                       Issued Country/Region         String  8192
  Organization                  Issued Organization           String  8192
  OrgUnit                       Issued Organization Unit      String  8192
  CommonName                    Issued Common Name            String  8192 -- Indexed
  Locality                      Issued City                   String  8192
  State                         Issued State                  String  8192
  Title                         Issued Title                  String  8192
  GivenName                     Issued First Name             String  8192
  Initials                      Issued Initials               String  8192
  SurName                       Issued Last Name              String  8192
  DomainComponent               Issued Domain Component       String  8192
  EMail                         Issued Email Address          String  8192
  StreetAddress                 Issued Street Address         String  8192
  UnstructuredName              Issued Unstructured Name      String  8192
  UnstructuredAddress           Issued Unstructured Address   String  8192
  DeviceSerialNumber            Issued Device Serial Number   String  8192

Row 1:
  Request ID: 0x4
  Binary Request:
-----BEGIN NEW CERTIFICATE REQUEST-----
MIIGWgYJKoZIhvcNAQcCoIIGSzCCBkcCAQMxCzAJBgUrDgMCGgUAMIIEtAYIKwYB
BQUHDAKgggSmBIIEojCCBJ4waTBnAgECBgorBgEEAYI3CgoBMVYwVAIBADADAgEB
MUowSAYJKwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxv
Y2FsDA5DT1JQXHBraS5hZG1pbgwHTU1DLkVYRTCCBCugggQnAgEBMIIEIDCCAwgC
AQAwADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANKBLS+0hBgzLcO5
i42pR/b/4Hzc10zIaMLAeskG+4rkHdDma/hX8Qyctc6+X8mjkBsxZBjwerHGrI+J
5DG34eHYsfNbA+ojP/Ft99J5e2TaCj12MmH3gyl+5iTf6O6T5R3vsErXgEuMFdWS
wHPzuoqZLT6sBUJHwlIvSfAgH/uCPRMrCS1qwocYopOkgJBpKAzeTlRn3n0iRVgw
iZxmTEYwGgJmdXgmPypDJ6yZKSVxnY6IqTOZiR1oGG7eWbVwrELUQcmuK0PdFDBP
TaGA1+JbcGZeo9HRRD43agU4+wCpIIS/GyidjwIWj66x7MaRr/RgDV1svLQJT7tw
Nb1gmlkCAwEAAaCCAdkwHAYKKwYBBAGCNw0CAzEOFgwxMC4wLjIwMzQ4LjIwSAYJ
KwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxvY2FsDA5D
T1JQXHBraS5hZG1pbgwHTU1DLkVYRTB0BgorBgEEAYI3DQICMWYwZAIBAR5cAE0A
aQBjAHIAbwBzAG8AZgB0ACAARQBuAGgAYQBuAGMAZQBkACAAQwByAHkAcAB0AG8A
ZwByAGEAcABoAGkAYwAgAFAAcgBvAHYAaQBkAGUAcgAgAHYAMQAuADADAQAwgfgG
CSqGSIb3DQEJDjGB6jCB5zA+BgkrBgEEAYI3FQcEMTAvBicrBgEEAYI3FQiHydJI
goaqPISljyiHissog+KoU4EpgrzrYIaT4yECAWQCAQUwEwYDVR0lBAwwCgYIKwYB
BQUHAwIwDgYDVR0PAQH/BAQDAgWgMBsGCSsGAQQBgjcVCgQOMAwwCgYIKwYBBQUH
AwIwRAYJKoZIhvcNAQkPBDcwNTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQC
AgCAMAcGBSsOAwIHMAoGCCqGSIb3DQMHMB0GA1UdDgQWBBRwHAG/cY/gAXNxOTsS
5H8z2zLSVTANBgkqhkiG9w0BAQUFAAOCAQEAGm25SvExJmKvVkZMA19ksMehBgSg
I43N1QMkyOJAZwYAt7EvdnB5F5fkfRiq9ZC52x8FbT/FcgwtCoDnTDXL3CndxNxL
c2y3rMEi5OvG3CHKKBHrYhCRsAQZx7IpZGffRsq/yGq4ZUggipaKZu2ZCSII24ah
LYLu1JVa3uUbDlwnXv04EjSDoL6NAqFRXR+20ODwxX2uxElSOjsN0Wfw+i0nl95k
pnplOV081DhBjzKstcZwqkGHDXm9Jh8ZhGEq8i5zVo9IqGHVitolq19Q8Eki6oqh
EyUC2loeisPp6JX5Ggl2VVKB/p0UqBqo7HV6VxjA9AGEBr8Ty577Zv/rmDAAMAAx
ggF7MIIBdwIBA4AUcBwBv3GP4AFzcTk7EuR/M9sy0lUwCQYFKw4DAhoFAKA+MBcG
CSqGSIb3DQEJAzEKBggrBgEFBQcMAjAjBgkqhkiG9w0BCQQxFgQUgUcMwJzvv3uj
evALu326lykKA0IwDQYJKoZIhvcNAQEBBQAEggEAqlKLBkRvmZHcGgEG5VZrNcio
Mi//IL7Y3FDpITJwHzkmZ3NOXHz8SfkM2CQQJshGXVdjuU2t4RzaXSJkCM66rLgY
k3kfbtG13iP2MmkymlZejq0oJL1Ek3GXeG2HIqnh0oHxmZlfTm03BlUByJ0K5bct
0c0fb+h8kfF1YSS/bz/f+5/SCfd5DJ384mTp3zF4jt9HqQcqKsb90HWCi4x9yoYB
hbs3FxMxBjM8t6egzTw6abTqhs8uuAOhCXugfDysgg0IOioflV3bqjnGebyqSZN0
GbgsvucKBWHGIqTXYmRRmDhmds5w4tsxDmxG+xEQxAo6ppH8VsDj3K8m7OpS7Q==
-----END NEW CERTIFICATE REQUEST-----

  Archived Key: EMPTY
  Key Recovery Agent Hashes: EMPTY
  Old Certificate: EMPTY
  Request Attributes: "
cdc:DC01.corp.cvilab.local
rmd:PKI-SRV01.corp.cvilab.local

ccm:PKI-SRV01.corp.cvilab.local"
  Request Type: 0x40400 (263168) -- CMC, Full Response
  Request Flags: 0x4 -- Force UTF-8
  Request Status Code: 0x0 (WIN32: 0) -- The operation completed successfully.
  Request Disposition: 0x15 (21) -- Revoked
  Request Disposition Message: "Revoked by CORP\pki.admin"
  Request Submission Date: 5/23/2026 5:39 PM
  Request Resolution Date: 5/23/2026 5:39 PM
  Revocation Date: 5/28/2026 3:34 PM
  Effective Revocation Date: 5/28/2026 3:34 PM
  Revocation Reason: 0x5 -- Reason: Cessation of Operation
  Requester Name: "CORP\pki.admin"
  Caller Name: "CORP\pki.admin"
  Signer Policies: EMPTY
  Signer Application Policies: EMPTY
  Officer: EMPTY
  Request Distinguished Name: EMPTY
  Request Binary Name:
0000	30 00                                              0.

  Request Country/Region: EMPTY
  Request Organization: EMPTY
  Request Organization Unit: EMPTY
  Request Common Name: EMPTY
  Request City: EMPTY
  Request State: EMPTY
  Request Title: EMPTY
  Request First Name: EMPTY
  Request Initials: EMPTY
  Request Last Name: EMPTY
  Request Domain Component: EMPTY
  Request Email Address: EMPTY
  Request Street Address: EMPTY
  Request Unstructured Name: EMPTY
  Request Unstructured Address: EMPTY
  Request Device Serial Number: EMPTY
  Attestation Challenge: EMPTY
  Endorsement Key Hash: EMPTY
  Endorsement Certificate Hash: EMPTY
  Binary Precertificate: EMPTY
  Issued Request ID: 0x4
  Binary Certificate:
-----BEGIN CERTIFICATE-----
MIIF/TCCBOWgAwIBAgITRAAAAAS3QrXauj1N6wAAAAAABDANBgkqhkiG9w0BAQsF
ADBgMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFjAUBgoJkiaJk/IsZAEZFgZjdmls
YWIxFDASBgoJkiaJk/IsZAEZFgRjb3JwMRkwFwYDVQQDExBDVkkgSXNzdWluZyBD
QSAxMB4XDTI2MDUyNDAwMjkxM1oXDTI3MDQyNjAyMzY1OFowFDESMBAGA1UEAxMJ
UEtJIEFkbWluMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0oEtL7SE
GDMtw7mLjalH9v/gfNzXTMhowsB6yQb7iuQd0OZr+FfxDJy1zr5fyaOQGzFkGPB6
scasj4nkMbfh4dix81sD6iM/8W330nl7ZNoKPXYyYfeDKX7mJN/o7pPlHe+wSteA
S4wV1ZLAc/O6ipktPqwFQkfCUi9J8CAf+4I9EysJLWrChxiik6SAkGkoDN5OVGfe
fSJFWDCJnGZMRjAaAmZ1eCY/KkMnrJkpJXGdjoipM5mJHWgYbt5ZtXCsQtRBya4r
Q90UME9NoYDX4ltwZl6j0dFEPjdqBTj7AKkghL8bKJ2PAhaPrrHsxpGv9GANXWy8
tAlPu3A1vWCaWQIDAQABo4IC+jCCAvYwPgYJKwYBBAGCNxUHBDEwLwYnKwYBBAGC
NxUIh8nSSIKGqjyEpY8oh4rLKIPiqFOBKYK862CGk+MhAgFkAgEFMBMGA1UdJQQM
MAoGCCsGAQUFBwMCMA4GA1UdDwEB/wQEAwIFoDAbBgkrBgEEAYI3FQoEDjAMMAoG
CCsGAQUFBwMCMEQGCSqGSIb3DQEJDwQ3MDUwDgYIKoZIhvcNAwICAgCAMA4GCCqG
SIb3DQMEAgIAgDAHBgUrDgMCBzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQUcBwBv3GP
4AFzcTk7EuR/M9sy0lUwHwYDVR0jBBgwFoAUEU1e2/MtwVQgB7jNPxktBLvoX9Ew
gd8GA1UdHwSB1zCB1DCB0aCBzqCBy4aByGxkYXA6Ly8vQ049Q1ZJJTIwSXNzdWlu
ZyUyMENBJTIwMSxDTj1QS0ktU1JWMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUy
MFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y29ycCxE
Qz1jdmlsYWIsREM9bG9jYWw/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNl
P29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHRBggrBgEFBQcBAQSB
xDCBwTCBvgYIKwYBBQUHMAKGgbFsZGFwOi8vL0NOPUNWSSUyMElzc3VpbmclMjBD
QSUyMDEsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZp
Y2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y29ycCxEQz1jdmlsYWIsREM9bG9jYWw/
Y0FDZXJ0aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRo
b3JpdHkwNgYDVR0RBC8wLaArBgorBgEEAYI3FAIDoB0MG3BraS5hZG1pbkBjb3Jw
LmN2aWxhYi5sb2NhbDANBgkqhkiG9w0BAQsFAAOCAQEAMUqgmlFn5HXcyj4w0fmF
vX9ddE9TrspSyg8B+5qG0GKboE1q4OQcH+u7kv1VPRdpr0ZVEC00WxT9AYMO4kC3
EO7S8VRGDqfv1TBCnttYRjc/s0n23T+2C6er/IvwG9uleL66hFW6fbL/mjHpikud
eFrhwBP9MCZZRqnjM+1QtFldx09YpK7Txz9QtzajV8e2xhfXrWdTvA5jLd7hlgu6
wFPoiLuyxsAAye5HCD/b11OaiUSCPNgzj9lxjOk4dAMpkXuxSxke2JwZCrltL1zJ
7lY2OCibCFq1UwZwkfTiweelqs/yRk57ivQd1pvL/eIk/JPNqaUHcbUGGp01sWby
2w==
-----END CERTIFICATE-----

  Certificate Hash: "4f 2a a3 df 00 7c 21 93 e7 19 a7 bf e7 5c f9 85 23 63 6d 54"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913"
  Template Enrollment Flags: 0x29 (41)
    CT_FLAG_INCLUDE_SYMMETRIC_ALGORITHMS -- 1
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
  Template General Flags: 0x2023a (131642)
    CT_FLAG_ADD_EMAIL -- 2
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_EXPORTABLE_KEY -- 10 (16)
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
    CT_FLAG_ADD_TEMPLATE_NAME -- 200 (512)
    CT_FLAG_IS_MODIFIED -- 20000 (131072)
  Template Private Key Flags: 0x5030010 (84082704)
    CTPRIVATEKEY_FLAG_EXPORTABLE_KEY -- 10 (16)
    CTPRIVATEKEY_FLAG_ATTEST_NONE -- 0
    TEMPLATE_SERVER_VER_2008R2<<CTPRIVATEKEY_FLAG_SERVERVERSION_SHIFT -- 30000 (196608)
    TEMPLATE_CLIENT_VER_WINBLUE<<CTPRIVATEKEY_FLAG_CLIENTVERSION_SHIFT -- 5000000 (83886080)
  Serial Number: "4400000004b742b5daba3d4deb000000000004"
  Issuer Name ID: 0x0 CA Version 0.0
  Certificate Effective Date: 5/23/2026 5:29 PM
  Certificate Expiration Date: 4/25/2027 7:36 PM
  Issued Subject Key Identifier: "70 1c 01 bf 71 8f e0 01 73 71 39 3b 12 e4 7f 33 db 32 d2 55"
  Binary Public Key:
0000	30 82 01 0a 02 82 01 01  00 d2 81 2d 2f b4 84 18
0010	33 2d c3 b9 8b 8d a9 47  f6 ff e0 7c dc d7 4c c8
0020	68 c2 c0 7a c9 06 fb 8a  e4 1d d0 e6 6b f8 57 f1
0030	0c 9c b5 ce be 5f c9 a3  90 1b 31 64 18 f0 7a b1
0040	c6 ac 8f 89 e4 31 b7 e1  e1 d8 b1 f3 5b 03 ea 23
0050	3f f1 6d f7 d2 79 7b 64  da 0a 3d 76 32 61 f7 83
0060	29 7e e6 24 df e8 ee 93  e5 1d ef b0 4a d7 80 4b
0070	8c 15 d5 92 c0 73 f3 ba  8a 99 2d 3e ac 05 42 47
0080	c2 52 2f 49 f0 20 1f fb  82 3d 13 2b 09 2d 6a c2
0090	87 18 a2 93 a4 80 90 69  28 0c de 4e 54 67 de 7d
00a0	22 45 58 30 89 9c 66 4c  46 30 1a 02 66 75 78 26
00b0	3f 2a 43 27 ac 99 29 25  71 9d 8e 88 a9 33 99 89
00c0	1d 68 18 6e de 59 b5 70  ac 42 d4 41 c9 ae 2b 43
00d0	dd 14 30 4f 4d a1 80 d7  e2 5b 70 66 5e a3 d1 d1
00e0	44 3e 37 6a 05 38 fb 00  a9 20 84 bf 1b 28 9d 8f
00f0	02 16 8f ae b1 ec c6 91  af f4 60 0d 5d 6c bc b4
0100	09 4f bb 70 35 bd 60 9a  59 02 03 01 00 01

  Public Key Length: 0x800 (2048)
  Public Key Algorithm: "1.2.840.113549.1.1.1" RSA (RSA_SIGN)
  Public Key Algorithm Parameters:
0000	05 00                                              ..

  Publish Expired Certificate in CRL: 0x0
  User Principal Name: "pki.admin@corp.cvilab.local"
  Issued Distinguished Name: "CN=PKI Admin"
  Issued Binary Name:
0000	30 14 31 12 30 10 06 03  55 04 03 13 09 50 4b 49   0.1.0...U....PKI
0010	20 41 64 6d 69 6e                                   Admin

  Issued Country/Region: EMPTY
  Issued Organization: EMPTY
  Issued Organization Unit: EMPTY
  Issued Common Name: "PKI Admin"
  Issued City: EMPTY
  Issued State: EMPTY
  Issued Title: EMPTY
  Issued First Name: EMPTY
  Issued Initials: EMPTY
  Issued Last Name: EMPTY
  Issued Domain Component: EMPTY
  Issued Email Address: EMPTY
  Issued Street Address: EMPTY
  Issued Unstructured Name: EMPTY
  Issued Unstructured Address: EMPTY
  Issued Device Serial Number: EMPTY

  Request Attributes:
    RequestOSVersion: "10.0.20348.2"
    RequestCSPProvider: "Microsoft Enhanced Cryptographic Provider v1.0"
    cdc: "DC01.corp.cvilab.local"
    rmd: "PKI-SRV01.corp.cvilab.local"
    ccm: "PKI-SRV01.corp.cvilab.local"

  Certificate Extensions:
    1.3.6.1.4.1.311.21.7: Flags = 20000(Origin=Policy), Length = 31
    Certificate Template Information
        Template=1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913
        Major Version Number=100
        Minor Version Number=5

    2.5.29.37: Flags = 20000(Origin=Policy), Length = c
    Enhanced Key Usage
        Client Authentication (1.3.6.1.5.5.7.3.2)

    2.5.29.15: Flags = 20001(Critical, Origin=Policy), Length = 4
    Key Usage
        Digital Signature, Key Encipherment (a0)

    1.3.6.1.4.1.311.21.10: Flags = 20000(Origin=Policy), Length = e
    Application Policies
        [1]Application Certificate Policy:
             Policy Identifier=Client Authentication

    1.2.840.113549.1.9.15: Flags = 20000(Origin=Policy), Length = 37
    SMIME Capabilities
        [1]SMIME Capability
             Object ID=1.2.840.113549.3.2
             Parameters=02 02 00 80
        [2]SMIME Capability
             Object ID=1.2.840.113549.3.4
             Parameters=02 02 00 80
        [3]SMIME Capability
             Object ID=1.3.14.3.2.7
        [4]SMIME Capability
             Object ID=1.2.840.113549.3.7

    2.5.29.14: Flags = 10000(Origin=Request), Length = 16
    Subject Key Identifier
        701c01bf718fe0017371393b12e47f33db32d255

    2.5.29.35: Flags = 20000(Origin=Policy), Length = 18
    Authority Key Identifier
        KeyID=114d5edbf32dc1542007b8cd3f192d04bbe85fd1

    2.5.29.31: Flags = 40000(Origin=Server), Length = d7
    CRL Distribution Points
        [1]CRL Distribution Point
             Distribution Point Name:
                  Full Name:
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevoc
ationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PKI-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=
cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=cer
tificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?object
Class=certificationAuthority)

    2.5.29.19: Flags = 20003(Critical, Disabled, Origin=Policy), Length = 2
    Basic Constraints
        Subject Type=End Entity
        Path Length Constraint=None

    2.5.29.17: Flags = 20000(Origin=Policy), Length = 2f
    Subject Alternative Name
        Other Name:
             Principal Name=pki.admin@corp.cvilab.local


Row 2:
  Request ID: 0x5
  Binary Request:
-----BEGIN NEW CERTIFICATE REQUEST-----
MIIGFAYJKoZIhvcNAQcCoIIGBTCCBgECAQMxCzAJBgUrDgMCGgUAMIIEbgYIKwYB
BQUHDAKgggRgBIIEXDCCBFgwaTBnAgECBgorBgEEAYI3CgoBMVYwVAIBADADAgEB
MUowSAYJKwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxv
Y2FsDA5DT1JQXHBraS5hZG1pbgwHTU1DLkVYRTCCA+WgggPhAgEBMIID2jCCAsIC
AQAwADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKfkFmk+e54eoeTg
pe0gIrxUQ646BQE1q20B7Ws6ptBTLVpX6sTQiJToL9clCsx2/f8zLXU6NgfZQNTX
/HkLyjX74jnssl/ZK2zEzuYSf9sQ0kaviQlmiJdqLhjJ16iGpGPwdjMRer4pF5gH
I3f4jkRjAjnX6TgbcWMGIdHhnpHnmJDV137OZBXi2p1l3J+51Rq7TXauaDVzrlwx
S4zbCYAJRQcgK4+LCliuOtdzFRv7bwtOuZGyCwc/MELvnbIBt1oez+1dyDrQHHU5
0bqpGooDWzL+W0NX6vRAweC/QYrO6K4ZbWTi5q+ilxg5YersA+Wrr8A9XRv3msBF
MJAQ5CkCAwEAAaCCAZMwHAYKKwYBBAGCNw0CAzEOFgwxMC4wLjIwMzQ4LjIwSAYJ
KwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxvY2FsDA5D
T1JQXHBraS5hZG1pbgwHTU1DLkVYRTB0BgorBgEEAYI3DQICMWYwZAIBAh5cAE0A
aQBjAHIAbwBzAG8AZgB0ACAARQBuAGgAYQBuAGMAZQBkACAAQwByAHkAcAB0AG8A
ZwByAGEAcABoAGkAYwAgAFAAcgBvAHYAaQBkAGUAcgAgAHYAMQAuADADAQAwgbIG
CSqGSIb3DQEJDjGBpDCBoTA+BgkrBgEEAYI3FQcEMTAvBicrBgEEAYI3FQiHydJI
goaqPISljyiHissog+KoU4Epg+GzQYfXj1gCAWQCAQMwEwYDVR0lBAwwCgYIKwYB
BQUHAwMwDgYDVR0PAQH/BAQDAgeAMBsGCSsGAQQBgjcVCgQOMAwwCgYIKwYBBQUH
AwMwHQYDVR0OBBYEFORnPdbmPsg6GlzxK82O667rqw0BMA0GCSqGSIb3DQEBBQUA
A4IBAQBleTBt1QDcM39DGznc7s/bTSxwXGimXBjQp9tyTwzMt8j2dj0CxaOZZmiE
78q05+zyHl249TJaoarIgGm7D8e3DcyXZDQB8Oj4k2k2F6wQlhCRMXJeIZwB4wBl
Cb9jatbSkNmhjuCSKc6D1rBvlQzvF8W2IkvbPXrJfh2yMGUvT12eRZiAMqGE4dvC
H3Zo1NT4VFARfl1JIYjYaOuaOl7VWMGPWvPZyf6jSFUeOZ1cfA0S90WLkEep3rKQ
kKjSFMb01g6r1zQXL1dFLuRuPP4TP9aTFGrJfYBWHyjmSJf28wOLiO9/oFgw5s1f
FC7ccSyHY3/t0gYEu+tM2D9QAIZuMAAwADGCAXswggF3AgEDgBTkZz3W5j7IOhpc
8SvNjuuu66sNATAJBgUrDgMCGgUAoD4wFwYJKoZIhvcNAQkDMQoGCCsGAQUFBwwC
MCMGCSqGSIb3DQEJBDEWBBSnll4YGPiBt7IQ3wiHrzoBiqzT4jANBgkqhkiG9w0B
AQEFAASCAQAiwmI0XTlU7h2kpfpPCA5vwjcSI5sLkbZrTn2wZqE2XbvdoMpPGMzB
6Ou+1WVYYxSJh8KF4BNVW37tjanetJMqdET5mi1kpcOSdycAdxtZ48W2+6QmELBM
z0jtQ8SstJUzeIiApJ4q+D8yDyYZMQ0uroGdXQAqm7JuEbPvNbYHIATfdpwNR35w
BE4WTAN/o+igEbz1vRl4M/WtjeX6/rt9I786sGSotxWYKc1y9yYPgA1hE7j8/ggo
z0HM0e/5NloR1W+YN3zbZU2/vuAahO73kN9iLheJMDNXPanKxl3N3ayb5+3/UYT1
7kqZ+z2ap+B29nS1Hh7oeGakjU5tjeKo
-----END NEW CERTIFICATE REQUEST-----

  Archived Key: EMPTY
  Key Recovery Agent Hashes: EMPTY
  Old Certificate: EMPTY
  Request Attributes: "
cdc:DC01.corp.cvilab.local
rmd:PKI-SRV01.corp.cvilab.local

ccm:PKI-SRV01.corp.cvilab.local"
  Request Type: 0x40400 (263168) -- CMC, Full Response
  Request Flags: 0x4 -- Force UTF-8
  Request Status Code: 0x0 (WIN32: 0) -- The operation completed successfully.
  Request Disposition: 0x15 (21) -- Revoked
  Request Disposition Message: "Revoked by CORP\pki.admin"
  Request Submission Date: 5/24/2026 1:03 PM
  Request Resolution Date: 5/24/2026 1:03 PM
  Revocation Date: 5/30/2026 11:10 AM
  Effective Revocation Date: 5/30/2026 11:10 AM
  Revocation Reason: 0x5 -- Reason: Cessation of Operation
  Requester Name: "CORP\pki.admin"
  Caller Name: "CORP\pki.admin"
  Signer Policies: EMPTY
  Signer Application Policies: EMPTY
  Officer: EMPTY
  Request Distinguished Name: EMPTY
  Request Binary Name:
0000	30 00                                              0.

  Request Country/Region: EMPTY
  Request Organization: EMPTY
  Request Organization Unit: EMPTY
  Request Common Name: EMPTY
  Request City: EMPTY
  Request State: EMPTY
  Request Title: EMPTY
  Request First Name: EMPTY
  Request Initials: EMPTY
  Request Last Name: EMPTY
  Request Domain Component: EMPTY
  Request Email Address: EMPTY
  Request Street Address: EMPTY
  Request Unstructured Name: EMPTY
  Request Unstructured Address: EMPTY
  Request Device Serial Number: EMPTY
  Attestation Challenge: EMPTY
  Endorsement Key Hash: EMPTY
  Endorsement Certificate Hash: EMPTY
  Binary Precertificate: EMPTY
  Issued Request ID: 0x5
  Binary Certificate:
-----BEGIN CERTIFICATE-----
MIIFtzCCBJ+gAwIBAgITRAAAAAW0kiwuJOEA6wAAAAAABTANBgkqhkiG9w0BAQsF
ADBgMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFjAUBgoJkiaJk/IsZAEZFgZjdmls
YWIxFDASBgoJkiaJk/IsZAEZFgRjb3JwMRkwFwYDVQQDExBDVkkgSXNzdWluZyBD
QSAxMB4XDTI2MDUyNDE5NTM1OFoXDTI3MDQyNjAyMzY1OFowFDESMBAGA1UEAxMJ
UEtJIEFkbWluMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAp+QWaT57
nh6h5OCl7SAivFRDrjoFATWrbQHtazqm0FMtWlfqxNCIlOgv1yUKzHb9/zMtdTo2
B9lA1Nf8eQvKNfviOeyyX9krbMTO5hJ/2xDSRq+JCWaIl2ouGMnXqIakY/B2MxF6
vikXmAcjd/iORGMCOdfpOBtxYwYh0eGekeeYkNXXfs5kFeLanWXcn7nVGrtNdq5o
NXOuXDFLjNsJgAlFByArj4sKWK4613MVG/tvC065kbILBz8wQu+dsgG3Wh7P7V3I
OtAcdTnRuqkaigNbMv5bQ1fq9EDB4L9Bis7orhltZOLmr6KXGDlh6uwD5auvwD1d
G/eawEUwkBDkKQIDAQABo4ICtDCCArAwPgYJKwYBBAGCNxUHBDEwLwYnKwYBBAGC
NxUIh8nSSIKGqjyEpY8oh4rLKIPiqFOBKYPhs0GH149YAgFkAgEDMBMGA1UdJQQM
MAoGCCsGAQUFBwMDMA4GA1UdDwEB/wQEAwIHgDAbBgkrBgEEAYI3FQoEDjAMMAoG
CCsGAQUFBwMDMB0GA1UdDgQWBBTkZz3W5j7IOhpc8SvNjuuu66sNATAfBgNVHSME
GDAWgBQRTV7b8y3BVCAHuM0/GS0Eu+hf0TCB3wYDVR0fBIHXMIHUMIHRoIHOoIHL
hoHIbGRhcDovLy9DTj1DVkklMjBJc3N1aW5nJTIwQ0ElMjAxLENOPVBLSS1TUlYw
MSxDTj1DRFAsQ049UHVibGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMs
Q049Q29uZmlndXJhdGlvbixEQz1jb3JwLERDPWN2aWxhYixEQz1sb2NhbD9jZXJ0
aWZpY2F0ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJp
YnV0aW9uUG9pbnQwgdEGCCsGAQUFBwEBBIHEMIHBMIG+BggrBgEFBQcwAoaBsWxk
YXA6Ly8vQ049Q1ZJJTIwSXNzdWluZyUyMENBJTIwMSxDTj1BSUEsQ049UHVibGlj
JTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixE
Qz1jb3JwLERDPWN2aWxhYixEQz1sb2NhbD9jQUNlcnRpZmljYXRlP2Jhc2U/b2Jq
ZWN0Q2xhc3M9Y2VydGlmaWNhdGlvbkF1dGhvcml0eTA2BgNVHREELzAtoCsGCisG
AQQBgjcUAgOgHQwbcGtpLmFkbWluQGNvcnAuY3ZpbGFiLmxvY2FsMA0GCSqGSIb3
DQEBCwUAA4IBAQA3XwBUv6txCxY0gPVXTvY+dMbTGCcYNL5mmvGtzM42CIKa2hkk
xVPfayGFZ1ntd8Ycq63zZTsQHO/MpFCYy3ciBqdASOL4xpGr/L5f97fuAybg7ads
kKPDFnsZfKOvG9oRnJBFV871y7969SaswY1vlIX3ERUxdhf2XE4hOa3hVRp/FKXn
X8Iy+AG56dJ9mviED3q4WFIS0IDZO5xD1FYSCIo6I13vGJPTDKVpMCYlU/mM37Kg
28Co/mid3C5pfq9lkkX7/vqUZW/vADYyyhxrI3AglGhbB1n+bRbQrAI2OfYId6xa
t/MVlYXgTAHBTaGLKSxV7nQHLRsNAO74yIJM
-----END CERTIFICATE-----

  Certificate Hash: "7f a1 7d be 13 f3 c0 f3 ff 3e 60 af 85 28 c4 ed b4 8d 96 ce"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.7887297.16107480" CVI Code Signing
  Template Enrollment Flags: 0x20 (32)
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
  Template General Flags: 0x20220 (131616)
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
    CT_FLAG_ADD_TEMPLATE_NAME -- 200 (512)
    CT_FLAG_IS_MODIFIED -- 20000 (131072)
  Template Private Key Flags: 0x5050100 (84214016)
    CTPRIVATEKEY_FLAG_USE_LEGACY_PROVIDER -- 100 (256)
    CTPRIVATEKEY_FLAG_ATTEST_NONE -- 0
    TEMPLATE_SERVER_VER_WINBLUE<<CTPRIVATEKEY_FLAG_SERVERVERSION_SHIFT -- 50000 (327680)
    TEMPLATE_CLIENT_VER_WINBLUE<<CTPRIVATEKEY_FLAG_CLIENTVERSION_SHIFT -- 5000000 (83886080)
  Serial Number: "4400000005b4922c2e24e100eb000000000005"
  Issuer Name ID: 0x0 CA Version 0.0
  Certificate Effective Date: 5/24/2026 12:53 PM
  Certificate Expiration Date: 4/25/2027 7:36 PM
  Issued Subject Key Identifier: "e4 67 3d d6 e6 3e c8 3a 1a 5c f1 2b cd 8e eb ae eb ab 0d 01"
  Binary Public Key:
0000	30 82 01 0a 02 82 01 01  00 a7 e4 16 69 3e 7b 9e
0010	1e a1 e4 e0 a5 ed 20 22  bc 54 43 ae 3a 05 01 35
0020	ab 6d 01 ed 6b 3a a6 d0  53 2d 5a 57 ea c4 d0 88
0030	94 e8 2f d7 25 0a cc 76  fd ff 33 2d 75 3a 36 07
0040	d9 40 d4 d7 fc 79 0b ca  35 fb e2 39 ec b2 5f d9
0050	2b 6c c4 ce e6 12 7f db  10 d2 46 af 89 09 66 88
0060	97 6a 2e 18 c9 d7 a8 86  a4 63 f0 76 33 11 7a be
0070	29 17 98 07 23 77 f8 8e  44 63 02 39 d7 e9 38 1b
0080	71 63 06 21 d1 e1 9e 91  e7 98 90 d5 d7 7e ce 64
0090	15 e2 da 9d 65 dc 9f b9  d5 1a bb 4d 76 ae 68 35
00a0	73 ae 5c 31 4b 8c db 09  80 09 45 07 20 2b 8f 8b
00b0	0a 58 ae 3a d7 73 15 1b  fb 6f 0b 4e b9 91 b2 0b
00c0	07 3f 30 42 ef 9d b2 01  b7 5a 1e cf ed 5d c8 3a
00d0	d0 1c 75 39 d1 ba a9 1a  8a 03 5b 32 fe 5b 43 57
00e0	ea f4 40 c1 e0 bf 41 8a  ce e8 ae 19 6d 64 e2 e6
00f0	af a2 97 18 39 61 ea ec  03 e5 ab af c0 3d 5d 1b
0100	f7 9a c0 45 30 90 10 e4  29 02 03 01 00 01

  Public Key Length: 0x800 (2048)
  Public Key Algorithm: "1.2.840.113549.1.1.1" RSA (RSA_SIGN)
  Public Key Algorithm Parameters:
0000	05 00                                              ..

  Publish Expired Certificate in CRL: 0x1
  User Principal Name: "pki.admin@corp.cvilab.local"
  Issued Distinguished Name: "CN=PKI Admin"
  Issued Binary Name:
0000	30 14 31 12 30 10 06 03  55 04 03 13 09 50 4b 49   0.1.0...U....PKI
0010	20 41 64 6d 69 6e                                   Admin

  Issued Country/Region: EMPTY
  Issued Organization: EMPTY
  Issued Organization Unit: EMPTY
  Issued Common Name: "PKI Admin"
  Issued City: EMPTY
  Issued State: EMPTY
  Issued Title: EMPTY
  Issued First Name: EMPTY
  Issued Initials: EMPTY
  Issued Last Name: EMPTY
  Issued Domain Component: EMPTY
  Issued Email Address: EMPTY
  Issued Street Address: EMPTY
  Issued Unstructured Name: EMPTY
  Issued Unstructured Address: EMPTY
  Issued Device Serial Number: EMPTY

  Request Attributes:
    RequestOSVersion: "10.0.20348.2"
    RequestCSPProvider: "Microsoft Enhanced Cryptographic Provider v1.0"
    cdc: "DC01.corp.cvilab.local"
    rmd: "PKI-SRV01.corp.cvilab.local"
    ccm: "PKI-SRV01.corp.cvilab.local"

  Certificate Extensions:
    1.3.6.1.4.1.311.21.7: Flags = 20000(Origin=Policy), Length = 31
    Certificate Template Information
        Template=CVI Code Signing(1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.7887297.16107480)
        Major Version Number=100
        Minor Version Number=3

    2.5.29.37: Flags = 20000(Origin=Policy), Length = c
    Enhanced Key Usage
        Code Signing (1.3.6.1.5.5.7.3.3)

    2.5.29.15: Flags = 20001(Critical, Origin=Policy), Length = 4
    Key Usage
        Digital Signature (80)

    1.3.6.1.4.1.311.21.10: Flags = 20000(Origin=Policy), Length = e
    Application Policies
        [1]Application Certificate Policy:
             Policy Identifier=Code Signing

    2.5.29.14: Flags = 10000(Origin=Request), Length = 16
    Subject Key Identifier
        e4673dd6e63ec83a1a5cf12bcd8eebaeebab0d01

    2.5.29.35: Flags = 20000(Origin=Policy), Length = 18
    Authority Key Identifier
        KeyID=114d5edbf32dc1542007b8cd3f192d04bbe85fd1

    2.5.29.31: Flags = 40000(Origin=Server), Length = d7
    CRL Distribution Points
        [1]CRL Distribution Point
             Distribution Point Name:
                  Full Name:
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevoc
ationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PKI-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=
cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=cer
tificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?object
Class=certificationAuthority)

    2.5.29.19: Flags = 20003(Critical, Disabled, Origin=Policy), Length = 2
    Basic Constraints
        Subject Type=End Entity
        Path Length Constraint=None

    2.5.29.17: Flags = 20000(Origin=Policy), Length = 2f
    Subject Alternative Name
        Other Name:
             Principal Name=pki.admin@corp.cvilab.local


Row 3:
  Request ID: 0x6
  Binary Request:
-----BEGIN NEW CERTIFICATE REQUEST-----
MIIGWgYJKoZIhvcNAQcCoIIGSzCCBkcCAQMxCzAJBgUrDgMCGgUAMIIEtAYIKwYB
BQUHDAKgggSmBIIEojCCBJ4waTBnAgECBgorBgEEAYI3CgoBMVYwVAIBADADAgEB
MUowSAYJKwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxv
Y2FsDA5DT1JQXHBraS5hZG1pbgwHTU1DLkVYRTCCBCugggQnAgEBMIIEIDCCAwgC
AQAwADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMWrnzfCoFAmzidR
WFLoDTgkj7ffQJXUvMub59qifc383Na4xKRM/Hp5eIcTvHwsEzl4C+r7hHYiKSwV
y4e34IczCIDAE6txAhlo+caQ3+KY/NMusDiCBYH87kzEG9N6+mUxvHdsrZ+/ScaB
/BYso0SEqycpZPOw9MMqQ7ZL7/6ac+bccbrSTE0Npe4t+UCMo47dqnFgShKn3R0J
ri3Gbj82n8cn+I9uz/lZaBmc0dM3gXo8+1ajQcaRpG4sXB1E+MQRHtaCTWRw5Kek
erIXQtRnkmw7yjzb3HA/u5AzGUaGkiVhm7cdBFOc121XEpy+geQ++w9sHc8nYZv3
N9ZpJK0CAwEAAaCCAdkwHAYKKwYBBAGCNw0CAzEOFgwxMC4wLjIwMzQ4LjIwSAYJ
KwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxvY2FsDA5D
T1JQXHBraS5hZG1pbgwHTU1DLkVYRTB0BgorBgEEAYI3DQICMWYwZAIBAR5cAE0A
aQBjAHIAbwBzAG8AZgB0ACAARQBuAGgAYQBuAGMAZQBkACAAQwByAHkAcAB0AG8A
ZwByAGEAcABoAGkAYwAgAFAAcgBvAHYAaQBkAGUAcgAgAHYAMQAuADADAQAwgfgG
CSqGSIb3DQEJDjGB6jCB5zA+BgkrBgEEAYI3FQcEMTAvBicrBgEEAYI3FQiHydJI
goaqPISljyiHissog+KoU4EpgrzrYIaT4yECAWQCAQUwEwYDVR0lBAwwCgYIKwYB
BQUHAwIwDgYDVR0PAQH/BAQDAgWgMBsGCSsGAQQBgjcVCgQOMAwwCgYIKwYBBQUH
AwIwRAYJKoZIhvcNAQkPBDcwNTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQC
AgCAMAcGBSsOAwIHMAoGCCqGSIb3DQMHMB0GA1UdDgQWBBQuqmo72WuaG7RM2D0h
VtClCWrpmTANBgkqhkiG9w0BAQUFAAOCAQEAxaK9SBkpw1sjxMqkPZQhmTrpC2r0
P/86Yftn52aeVk0zm92rGubouCuO/7wxoBJAalyWjoSX5AWf6UaJq3dg2IrXS5+T
i+kjbfExNXcBzKFdmEcDK89YdvNpwbh4vLe2WL1Rq2+8DplJgx9WvM/bzekm6mB6
CiOaVNV4rvdalVTskmFsFfSEYjNtVAINPobZhMz4k6CcTQEr6gRoD3MsCy1Jo+KY
DhH79xNAUdS6YmZFk7Ek1pAMAJMMM++SEgbBYignoxv4JBRbsgCKW3ULzSpCw/qq
cEuMruuWKpFM9Pzn0anYu+u8TxvfeFk3WXDSTFRumPzHWDuZaXOdM+r2STAAMAAx
ggF7MIIBdwIBA4AULqpqO9lrmhu0TNg9IVbQpQlq6ZkwCQYFKw4DAhoFAKA+MBcG
CSqGSIb3DQEJAzEKBggrBgEFBQcMAjAjBgkqhkiG9w0BCQQxFgQUB6hMqUrSYQtb
gjumIoKCtq6+0mkwDQYJKoZIhvcNAQEBBQAEggEAufz0woc80R0/cFYSp3BAQhw5
4sIi0rCg91SyQFSWQo3UwC62dwWvMP7OH4WVWVcHZWEa6a/lM6RpYFIXcA+2ZsvY
mS/Phqu6XUKhXf46ew4DTMTWxZuUcLLijLDHuO+FxhB+W8V6XnzRDpcsx/r/2Ni7
t+3Q1sZ+Zo1vfsMPy0sYfPpHNOEsVPac2EQdY/9P5XF2+ZaR3xghjI3xBGcpo0eR
V4N6L/ulXMNQrNsFWK6IweRsTYKd9r5yWWyy8kXUioWxmFvxrmIDVPDDOu7JnwPk
g1ZoYsxaNbsu66axxct9OOkQvuuDLWgi2WcSF4YOps22d3YE/HR4UKOo6aJI5w==
-----END NEW CERTIFICATE REQUEST-----

  Archived Key: EMPTY
  Key Recovery Agent Hashes: EMPTY
  Old Certificate: EMPTY
  Request Attributes: "
cdc:DC01.corp.cvilab.local
rmd:PKI-SRV01.corp.cvilab.local

ccm:PKI-SRV01.corp.cvilab.local"
  Request Type: 0x40400 (263168) -- CMC, Full Response
  Request Flags: 0x4 -- Force UTF-8
  Request Status Code: 0x0 (WIN32: 0) -- The operation completed successfully.
  Request Disposition: 0x15 (21) -- Revoked
  Request Disposition Message: "Revoked by CORP\pki.admin"
  Request Submission Date: 5/26/2026 3:20 PM
  Request Resolution Date: 5/26/2026 3:20 PM
  Revocation Date: 5/28/2026 3:34 PM
  Effective Revocation Date: 5/28/2026 3:33 PM
  Revocation Reason: 0x5 -- Reason: Cessation of Operation
  Requester Name: "CORP\pki.admin"
  Caller Name: "CORP\pki.admin"
  Signer Policies: EMPTY
  Signer Application Policies: EMPTY
  Officer: EMPTY
  Request Distinguished Name: EMPTY
  Request Binary Name:
0000	30 00                                              0.

  Request Country/Region: EMPTY
  Request Organization: EMPTY
  Request Organization Unit: EMPTY
  Request Common Name: EMPTY
  Request City: EMPTY
  Request State: EMPTY
  Request Title: EMPTY
  Request First Name: EMPTY
  Request Initials: EMPTY
  Request Last Name: EMPTY
  Request Domain Component: EMPTY
  Request Email Address: EMPTY
  Request Street Address: EMPTY
  Request Unstructured Name: EMPTY
  Request Unstructured Address: EMPTY
  Request Device Serial Number: EMPTY
  Attestation Challenge: EMPTY
  Endorsement Key Hash: EMPTY
  Endorsement Certificate Hash: EMPTY
  Binary Precertificate: EMPTY
  Issued Request ID: 0x6
  Binary Certificate:
-----BEGIN CERTIFICATE-----
MIIF/TCCBOWgAwIBAgITRAAAAAbviCJiNAX1egAAAAAABjANBgkqhkiG9w0BAQsF
ADBgMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFjAUBgoJkiaJk/IsZAEZFgZjdmls
YWIxFDASBgoJkiaJk/IsZAEZFgRjb3JwMRkwFwYDVQQDExBDVkkgSXNzdWluZyBD
QSAxMB4XDTI2MDUyNjIyMTAxOVoXDTI3MDQyNjAyMzY1OFowFDESMBAGA1UEAxMJ
UEtJIEFkbWluMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxaufN8Kg
UCbOJ1FYUugNOCSPt99AldS8y5vn2qJ9zfzc1rjEpEz8enl4hxO8fCwTOXgL6vuE
diIpLBXLh7fghzMIgMATq3ECGWj5xpDf4pj80y6wOIIFgfzuTMQb03r6ZTG8d2yt
n79JxoH8FiyjRISrJylk87D0wypDtkvv/ppz5txxutJMTQ2l7i35QIyjjt2qcWBK
EqfdHQmuLcZuPzafxyf4j27P+VloGZzR0zeBejz7VqNBxpGkbixcHUT4xBEe1oJN
ZHDkp6R6shdC1GeSbDvKPNvccD+7kDMZRoaSJWGbtx0EU5zXbVcSnL6B5D77D2wd
zydhm/c31mkkrQIDAQABo4IC+jCCAvYwPgYJKwYBBAGCNxUHBDEwLwYnKwYBBAGC
NxUIh8nSSIKGqjyEpY8oh4rLKIPiqFOBKYK862CGk+MhAgFkAgEFMBMGA1UdJQQM
MAoGCCsGAQUFBwMCMA4GA1UdDwEB/wQEAwIFoDAbBgkrBgEEAYI3FQoEDjAMMAoG
CCsGAQUFBwMCMEQGCSqGSIb3DQEJDwQ3MDUwDgYIKoZIhvcNAwICAgCAMA4GCCqG
SIb3DQMEAgIAgDAHBgUrDgMCBzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQULqpqO9lr
mhu0TNg9IVbQpQlq6ZkwHwYDVR0jBBgwFoAUEU1e2/MtwVQgB7jNPxktBLvoX9Ew
gd8GA1UdHwSB1zCB1DCB0aCBzqCBy4aByGxkYXA6Ly8vQ049Q1ZJJTIwSXNzdWlu
ZyUyMENBJTIwMSxDTj1QS0ktU1JWMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUy
MFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y29ycCxE
Qz1jdmlsYWIsREM9bG9jYWw/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNl
P29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHRBggrBgEFBQcBAQSB
xDCBwTCBvgYIKwYBBQUHMAKGgbFsZGFwOi8vL0NOPUNWSSUyMElzc3VpbmclMjBD
QSUyMDEsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZp
Y2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y29ycCxEQz1jdmlsYWIsREM9bG9jYWw/
Y0FDZXJ0aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRo
b3JpdHkwNgYDVR0RBC8wLaArBgorBgEEAYI3FAIDoB0MG3BraS5hZG1pbkBjb3Jw
LmN2aWxhYi5sb2NhbDANBgkqhkiG9w0BAQsFAAOCAQEANSg5YvEYSXYmAftPJqM5
jAepkG5BLew29SKEA/sZ04iMH+j4FdrNAev0r+xk8/7pGQVyEESRQLukCNjSI9KU
jCsNpUFzFM64dlJ+hSpvpT4jgXz7Lg7Qs5GOU/gA0wBWybPomRK9dwksjL+2r9bK
TWckNj0tJvBuxhYztPPrhVNEAD1JC32ODCIBAfvoQzr4XEBr/Gl9EBdKC7ageqDp
4ndtyvUqHDFyGWs2i6VE1OPjRoI5at0pTQwch9aMJuZjTNgGy2249nG5vLdYflRN
Tf6u+b2lC1ZSd4zqNSmA+wCSqT7rDS0KWOJ3hz6D8RxrVn9JVFv9GhPssQWtbNU2
hA==
-----END CERTIFICATE-----

  Certificate Hash: "9b c0 13 68 c2 62 75 26 bb a4 9f 21 69 50 95 98 07 5a f7 6a"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913"
  Template Enrollment Flags: 0x29 (41)
    CT_FLAG_INCLUDE_SYMMETRIC_ALGORITHMS -- 1
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
  Template General Flags: 0x2023a (131642)
    CT_FLAG_ADD_EMAIL -- 2
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_EXPORTABLE_KEY -- 10 (16)
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
    CT_FLAG_ADD_TEMPLATE_NAME -- 200 (512)
    CT_FLAG_IS_MODIFIED -- 20000 (131072)
  Template Private Key Flags: 0x5030010 (84082704)
    CTPRIVATEKEY_FLAG_EXPORTABLE_KEY -- 10 (16)
    CTPRIVATEKEY_FLAG_ATTEST_NONE -- 0
    TEMPLATE_SERVER_VER_2008R2<<CTPRIVATEKEY_FLAG_SERVERVERSION_SHIFT -- 30000 (196608)
    TEMPLATE_CLIENT_VER_WINBLUE<<CTPRIVATEKEY_FLAG_CLIENTVERSION_SHIFT -- 5000000 (83886080)
  Serial Number: "4400000006ef8822623405f57a000000000006"
  Issuer Name ID: 0x0 CA Version 0.0
  Certificate Effective Date: 5/26/2026 3:10 PM
  Certificate Expiration Date: 4/25/2027 7:36 PM
  Issued Subject Key Identifier: "2e aa 6a 3b d9 6b 9a 1b b4 4c d8 3d 21 56 d0 a5 09 6a e9 99"
  Binary Public Key:
0000	30 82 01 0a 02 82 01 01  00 c5 ab 9f 37 c2 a0 50
0010	26 ce 27 51 58 52 e8 0d  38 24 8f b7 df 40 95 d4
0020	bc cb 9b e7 da a2 7d cd  fc dc d6 b8 c4 a4 4c fc
0030	7a 79 78 87 13 bc 7c 2c  13 39 78 0b ea fb 84 76
0040	22 29 2c 15 cb 87 b7 e0  87 33 08 80 c0 13 ab 71
0050	02 19 68 f9 c6 90 df e2  98 fc d3 2e b0 38 82 05
0060	81 fc ee 4c c4 1b d3 7a  fa 65 31 bc 77 6c ad 9f
0070	bf 49 c6 81 fc 16 2c a3  44 84 ab 27 29 64 f3 b0
0080	f4 c3 2a 43 b6 4b ef fe  9a 73 e6 dc 71 ba d2 4c
0090	4d 0d a5 ee 2d f9 40 8c  a3 8e dd aa 71 60 4a 12
00a0	a7 dd 1d 09 ae 2d c6 6e  3f 36 9f c7 27 f8 8f 6e
00b0	cf f9 59 68 19 9c d1 d3  37 81 7a 3c fb 56 a3 41
00c0	c6 91 a4 6e 2c 5c 1d 44  f8 c4 11 1e d6 82 4d 64
00d0	70 e4 a7 a4 7a b2 17 42  d4 67 92 6c 3b ca 3c db
00e0	dc 70 3f bb 90 33 19 46  86 92 25 61 9b b7 1d 04
00f0	53 9c d7 6d 57 12 9c be  81 e4 3e fb 0f 6c 1d cf
0100	27 61 9b f7 37 d6 69 24  ad 02 03 01 00 01

  Public Key Length: 0x800 (2048)
  Public Key Algorithm: "1.2.840.113549.1.1.1" RSA (RSA_SIGN)
  Public Key Algorithm Parameters:
0000	05 00                                              ..

  Publish Expired Certificate in CRL: 0x0
  User Principal Name: "pki.admin@corp.cvilab.local"
  Issued Distinguished Name: "CN=PKI Admin"
  Issued Binary Name:
0000	30 14 31 12 30 10 06 03  55 04 03 13 09 50 4b 49   0.1.0...U....PKI
0010	20 41 64 6d 69 6e                                   Admin

  Issued Country/Region: EMPTY
  Issued Organization: EMPTY
  Issued Organization Unit: EMPTY
  Issued Common Name: "PKI Admin"
  Issued City: EMPTY
  Issued State: EMPTY
  Issued Title: EMPTY
  Issued First Name: EMPTY
  Issued Initials: EMPTY
  Issued Last Name: EMPTY
  Issued Domain Component: EMPTY
  Issued Email Address: EMPTY
  Issued Street Address: EMPTY
  Issued Unstructured Name: EMPTY
  Issued Unstructured Address: EMPTY
  Issued Device Serial Number: EMPTY

  Request Attributes:
    RequestOSVersion: "10.0.20348.2"
    RequestCSPProvider: "Microsoft Enhanced Cryptographic Provider v1.0"
    cdc: "DC01.corp.cvilab.local"
    rmd: "PKI-SRV01.corp.cvilab.local"
    ccm: "PKI-SRV01.corp.cvilab.local"

  Certificate Extensions:
    1.3.6.1.4.1.311.21.7: Flags = 20000(Origin=Policy), Length = 31
    Certificate Template Information
        Template=1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913
        Major Version Number=100
        Minor Version Number=5

    2.5.29.37: Flags = 20000(Origin=Policy), Length = c
    Enhanced Key Usage
        Client Authentication (1.3.6.1.5.5.7.3.2)

    2.5.29.15: Flags = 20001(Critical, Origin=Policy), Length = 4
    Key Usage
        Digital Signature, Key Encipherment (a0)

    1.3.6.1.4.1.311.21.10: Flags = 20000(Origin=Policy), Length = e
    Application Policies
        [1]Application Certificate Policy:
             Policy Identifier=Client Authentication

    1.2.840.113549.1.9.15: Flags = 20000(Origin=Policy), Length = 37
    SMIME Capabilities
        [1]SMIME Capability
             Object ID=1.2.840.113549.3.2
             Parameters=02 02 00 80
        [2]SMIME Capability
             Object ID=1.2.840.113549.3.4
             Parameters=02 02 00 80
        [3]SMIME Capability
             Object ID=1.3.14.3.2.7
        [4]SMIME Capability
             Object ID=1.2.840.113549.3.7

    2.5.29.14: Flags = 10000(Origin=Request), Length = 16
    Subject Key Identifier
        2eaa6a3bd96b9a1bb44cd83d2156d0a5096ae999

    2.5.29.35: Flags = 20000(Origin=Policy), Length = 18
    Authority Key Identifier
        KeyID=114d5edbf32dc1542007b8cd3f192d04bbe85fd1

    2.5.29.31: Flags = 40000(Origin=Server), Length = d7
    CRL Distribution Points
        [1]CRL Distribution Point
             Distribution Point Name:
                  Full Name:
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevoc
ationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PKI-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=
cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=cer
tificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?object
Class=certificationAuthority)

    2.5.29.19: Flags = 20003(Critical, Disabled, Origin=Policy), Length = 2
    Basic Constraints
        Subject Type=End Entity
        Path Length Constraint=None

    2.5.29.17: Flags = 20000(Origin=Policy), Length = 2f
    Subject Alternative Name
        Other Name:
             Principal Name=pki.admin@corp.cvilab.local


Row 4:
  Request ID: 0x7
  Binary Request:
-----BEGIN NEW CERTIFICATE REQUEST-----
MIIGWgYJKoZIhvcNAQcCoIIGSzCCBkcCAQMxCzAJBgUrDgMCGgUAMIIEtAYIKwYB
BQUHDAKgggSmBIIEojCCBJ4waTBnAgECBgorBgEEAYI3CgoBMVYwVAIBADADAgEB
MUowSAYJKwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxv
Y2FsDA5DT1JQXHBraS5hZG1pbgwHTU1DLkVYRTCCBCugggQnAgEBMIIEIDCCAwgC
AQAwADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKEjc1kPFiHvpB3j
nbchkyIITRvkMkHN48RJa90cxPVZMC1/RDl7oL7PeClFNDZsrggMSBUpWTve1d1R
ovmCDdlCUMPhWWjWHdsNjUrWIGbPM024IKk3Dv3oSdHI28awEoO+uK0yDq3nybQG
Sp1aaGRP4NhwFNQIe4oaWwbImh+3g9gTt/rjEJIlBtUGCGCyNilPZfa2XVHnbvuc
ywJ7lX9/L2dLiJcfYABqaQj/sWgD27A4AhpFyBPn8xgjJp2aP/hRGXPhqLq9p0/1
+xu6TfTk+D/UrJuFwNh1s44eq0vj0naoBsohWxcqwUK98Va7D5FIksAf8T0wnD63
jw8bu+UCAwEAAaCCAdkwHAYKKwYBBAGCNw0CAzEOFgwxMC4wLjIwMzQ4LjIwSAYJ
KwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxvY2FsDA5D
T1JQXHBraS5hZG1pbgwHTU1DLkVYRTB0BgorBgEEAYI3DQICMWYwZAIBAR5cAE0A
aQBjAHIAbwBzAG8AZgB0ACAARQBuAGgAYQBuAGMAZQBkACAAQwByAHkAcAB0AG8A
ZwByAGEAcABoAGkAYwAgAFAAcgBvAHYAaQBkAGUAcgAgAHYAMQAuADADAQAwgfgG
CSqGSIb3DQEJDjGB6jCB5zA+BgkrBgEEAYI3FQcEMTAvBicrBgEEAYI3FQiHydJI
goaqPISljyiHissog+KoU4EpgrzrYIaT4yECAWQCAQkwEwYDVR0lBAwwCgYIKwYB
BQUHAwIwDgYDVR0PAQH/BAQDAgWgMBsGCSsGAQQBgjcVCgQOMAwwCgYIKwYBBQUH
AwIwRAYJKoZIhvcNAQkPBDcwNTAOBggqhkiG9w0DAgICAIAwDgYIKoZIhvcNAwQC
AgCAMAcGBSsOAwIHMAoGCCqGSIb3DQMHMB0GA1UdDgQWBBTShU2aX2LtMI78A+Qm
SkYU/L72KDANBgkqhkiG9w0BAQUFAAOCAQEAKsvbsBvMaVKnAKZ0Ix6x8xvoouTi
WR8v7k39jBj3O+j+IKxm6khQndLSHEpc3XwZ7NBtY9W3kOzEduwWb1gdZnaEgQGC
zC2Yw2L05bKdxriK8oQzFlMhbKuvEfXJqdpT0X8y/UcQElfiYVgZVZq3lc1p6Y52
gXTmlCoQHPMGlSEuPSaCaX+E2PTEnCzSorrcvLPhRdodFf+qSHRplippIXITJPYV
Ikva6mzOUUYsm58Dm6qz3JaPSTR5/VxwaKCUuOjZyUHj9638sTfv/i/Y3nkRoGX3
dpamQqp6AfhuzrfvreyVWiaZFla6Q4nkSh+fwh0gV5T0pH84StozGdcpYzAAMAAx
ggF7MIIBdwIBA4AU0oVNml9i7TCO/APkJkpGFPy+9igwCQYFKw4DAhoFAKA+MBcG
CSqGSIb3DQEJAzEKBggrBgEFBQcMAjAjBgkqhkiG9w0BCQQxFgQU3V+T6DeQCQ8Z
NM/Tx7BgQssQUQcwDQYJKoZIhvcNAQEBBQAEggEAUBiHFnA2+Xuh9XmBeaOxGx0f
PdpV2j3y6ZBnJHOsmAsRhHlEQ3Cp9G8ptm1Bb+J/6mP63JlheGL7/tOHCDVdcxbk
ad2XQIkupZRi7zny4csM09xSB+MZbTtlLuVaPb5f0HTsuTlkjdPr8OKjpr0RHDhk
wxdromFQmKkmFHJgbSyb8rntAH+XZF9oWLMv79PHVXoYUJD9X4sPri0vpFUbvzlm
7vsuv+7K7LtGVzTD6x0xo13qB2FYE9zYP2vtcFki+ZK3fwmo1yLV8wXxZ5/XwWC2
K0MFSxG4c9HtbeJ09IYXUazx9t5au+vBMZIpCvMx4o7iNlzz6SHDSLLi6HCP8Q==
-----END NEW CERTIFICATE REQUEST-----

  Archived Key: EMPTY
  Key Recovery Agent Hashes: EMPTY
  Old Certificate: EMPTY
  Request Attributes: "
cdc:DC01.corp.cvilab.local
rmd:PKI-SRV01.corp.cvilab.local

ccm:PKI-SRV01.corp.cvilab.local"
  Request Type: 0x40400 (263168) -- CMC, Full Response
  Request Flags: 0x4 -- Force UTF-8
  Request Status Code: 0x0 (WIN32: 0) -- The operation completed successfully.
  Request Disposition: 0x15 (21) -- Revoked
  Request Disposition Message: "Revoked by CORP\pki.admin"
  Request Submission Date: 5/26/2026 3:48 PM
  Request Resolution Date: 5/26/2026 3:48 PM
  Revocation Date: 5/28/2026 3:33 PM
  Effective Revocation Date: 5/28/2026 3:33 PM
  Revocation Reason: 0x5 -- Reason: Cessation of Operation
  Requester Name: "CORP\pki.admin"
  Caller Name: "CORP\pki.admin"
  Signer Policies: EMPTY
  Signer Application Policies: EMPTY
  Officer: EMPTY
  Request Distinguished Name: EMPTY
  Request Binary Name:
0000	30 00                                              0.

  Request Country/Region: EMPTY
  Request Organization: EMPTY
  Request Organization Unit: EMPTY
  Request Common Name: EMPTY
  Request City: EMPTY
  Request State: EMPTY
  Request Title: EMPTY
  Request First Name: EMPTY
  Request Initials: EMPTY
  Request Last Name: EMPTY
  Request Domain Component: EMPTY
  Request Email Address: EMPTY
  Request Street Address: EMPTY
  Request Unstructured Name: EMPTY
  Request Unstructured Address: EMPTY
  Request Device Serial Number: EMPTY
  Attestation Challenge: EMPTY
  Endorsement Key Hash: EMPTY
  Endorsement Certificate Hash: EMPTY
  Binary Precertificate: EMPTY
  Issued Request ID: 0x7
  Binary Certificate:
-----BEGIN CERTIFICATE-----
MIIF/TCCBOWgAwIBAgITRAAAAAd7mShJ8dLyDgAAAAAABzANBgkqhkiG9w0BAQsF
ADBgMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFjAUBgoJkiaJk/IsZAEZFgZjdmls
YWIxFDASBgoJkiaJk/IsZAEZFgRjb3JwMRkwFwYDVQQDExBDVkkgSXNzdWluZyBD
QSAxMB4XDTI2MDUyNjIyMzgzM1oXDTI3MDQyNjAyMzY1OFowFDESMBAGA1UEAxMJ
UEtJIEFkbWluMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAoSNzWQ8W
Ie+kHeOdtyGTIghNG+QyQc3jxElr3RzE9VkwLX9EOXugvs94KUU0NmyuCAxIFSlZ
O97V3VGi+YIN2UJQw+FZaNYd2w2NStYgZs8zTbggqTcO/ehJ0cjbxrASg764rTIO
refJtAZKnVpoZE/g2HAU1Ah7ihpbBsiaH7eD2BO3+uMQkiUG1QYIYLI2KU9l9rZd
Uedu+5zLAnuVf38vZ0uIlx9gAGppCP+xaAPbsDgCGkXIE+fzGCMmnZo/+FEZc+Go
ur2nT/X7G7pN9OT4P9Ssm4XA2HWzjh6rS+PSdqgGyiFbFyrBQr3xVrsPkUiSwB/x
PTCcPrePDxu75QIDAQABo4IC+jCCAvYwPgYJKwYBBAGCNxUHBDEwLwYnKwYBBAGC
NxUIh8nSSIKGqjyEpY8oh4rLKIPiqFOBKYK862CGk+MhAgFkAgEJMBMGA1UdJQQM
MAoGCCsGAQUFBwMCMA4GA1UdDwEB/wQEAwIFoDAbBgkrBgEEAYI3FQoEDjAMMAoG
CCsGAQUFBwMCMEQGCSqGSIb3DQEJDwQ3MDUwDgYIKoZIhvcNAwICAgCAMA4GCCqG
SIb3DQMEAgIAgDAHBgUrDgMCBzAKBggqhkiG9w0DBzAdBgNVHQ4EFgQU0oVNml9i
7TCO/APkJkpGFPy+9igwHwYDVR0jBBgwFoAUEU1e2/MtwVQgB7jNPxktBLvoX9Ew
gd8GA1UdHwSB1zCB1DCB0aCBzqCBy4aByGxkYXA6Ly8vQ049Q1ZJJTIwSXNzdWlu
ZyUyMENBJTIwMSxDTj1QS0ktU1JWMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUy
MFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y29ycCxE
Qz1jdmlsYWIsREM9bG9jYWw/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNl
P29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHRBggrBgEFBQcBAQSB
xDCBwTCBvgYIKwYBBQUHMAKGgbFsZGFwOi8vL0NOPUNWSSUyMElzc3VpbmclMjBD
QSUyMDEsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZp
Y2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y29ycCxEQz1jdmlsYWIsREM9bG9jYWw/
Y0FDZXJ0aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRo
b3JpdHkwNgYDVR0RBC8wLaArBgorBgEEAYI3FAIDoB0MG3BraS5hZG1pbkBjb3Jw
LmN2aWxhYi5sb2NhbDANBgkqhkiG9w0BAQsFAAOCAQEAUy5+4iqU+dt1KDUilkA/
qwxd48H7ndLmoX1mWqGvEmU77bj9efPjtL8N5Kli6IUGwRO+G0Jfl6Qk/5RYZ5VR
viBv3pqTcgGOiFMXgCBXgkv++onwUZ3za5A3RS/FaGCk8RiLASwUqvPo/ZmDtKUW
kkxtAbikze8QjdeE4qPChJWxZmhP7z+bZ39cu6Qpn9EUM7aLeGGti9B9CJ1gzOzv
0Vny+x+oK5wl1WGqTQ2ZwvWou4mRhKNowTM0+PiTh5G+XHyrTPLp/AxGhqL9FXJV
Ig6QxoeEy/YzkAigujn8NwgBRtzkKm2eJ1yydrTAZFxmPjWx+aen4dg3zuu3c/LH
Xg==
-----END CERTIFICATE-----

  Certificate Hash: "25 d0 15 94 a2 bc 5b c3 ce d2 d1 25 99 23 a4 20 ee a7 6e 4a"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913"
  Template Enrollment Flags: 0x29 (41)
    CT_FLAG_INCLUDE_SYMMETRIC_ALGORITHMS -- 1
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
  Template General Flags: 0x2023a (131642)
    CT_FLAG_ADD_EMAIL -- 2
    CT_FLAG_PUBLISH_TO_DS -- 8
    CT_FLAG_EXPORTABLE_KEY -- 10 (16)
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
    CT_FLAG_ADD_TEMPLATE_NAME -- 200 (512)
    CT_FLAG_IS_MODIFIED -- 20000 (131072)
  Template Private Key Flags: 0x5030010 (84082704)
    CTPRIVATEKEY_FLAG_EXPORTABLE_KEY -- 10 (16)
    CTPRIVATEKEY_FLAG_ATTEST_NONE -- 0
    TEMPLATE_SERVER_VER_2008R2<<CTPRIVATEKEY_FLAG_SERVERVERSION_SHIFT -- 30000 (196608)
    TEMPLATE_CLIENT_VER_WINBLUE<<CTPRIVATEKEY_FLAG_CLIENTVERSION_SHIFT -- 5000000 (83886080)
  Serial Number: "44000000077b992849f1d2f20e000000000007"
  Issuer Name ID: 0x0 CA Version 0.0
  Certificate Effective Date: 5/26/2026 3:38 PM
  Certificate Expiration Date: 4/25/2027 7:36 PM
  Issued Subject Key Identifier: "d2 85 4d 9a 5f 62 ed 30 8e fc 03 e4 26 4a 46 14 fc be f6 28"
  Binary Public Key:
0000	30 82 01 0a 02 82 01 01  00 a1 23 73 59 0f 16 21
0010	ef a4 1d e3 9d b7 21 93  22 08 4d 1b e4 32 41 cd
0020	e3 c4 49 6b dd 1c c4 f5  59 30 2d 7f 44 39 7b a0
0030	be cf 78 29 45 34 36 6c  ae 08 0c 48 15 29 59 3b
0040	de d5 dd 51 a2 f9 82 0d  d9 42 50 c3 e1 59 68 d6
0050	1d db 0d 8d 4a d6 20 66  cf 33 4d b8 20 a9 37 0e
0060	fd e8 49 d1 c8 db c6 b0  12 83 be b8 ad 32 0e ad
0070	e7 c9 b4 06 4a 9d 5a 68  64 4f e0 d8 70 14 d4 08
0080	7b 8a 1a 5b 06 c8 9a 1f  b7 83 d8 13 b7 fa e3 10
0090	92 25 06 d5 06 08 60 b2  36 29 4f 65 f6 b6 5d 51
00a0	e7 6e fb 9c cb 02 7b 95  7f 7f 2f 67 4b 88 97 1f
00b0	60 00 6a 69 08 ff b1 68  03 db b0 38 02 1a 45 c8
00c0	13 e7 f3 18 23 26 9d 9a  3f f8 51 19 73 e1 a8 ba
00d0	bd a7 4f f5 fb 1b ba 4d  f4 e4 f8 3f d4 ac 9b 85
00e0	c0 d8 75 b3 8e 1e ab 4b  e3 d2 76 a8 06 ca 21 5b
00f0	17 2a c1 42 bd f1 56 bb  0f 91 48 92 c0 1f f1 3d
0100	30 9c 3e b7 8f 0f 1b bb  e5 02 03 01 00 01

  Public Key Length: 0x800 (2048)
  Public Key Algorithm: "1.2.840.113549.1.1.1" RSA (RSA_SIGN)
  Public Key Algorithm Parameters:
0000	05 00                                              ..

  Publish Expired Certificate in CRL: 0x0
  User Principal Name: "pki.admin@corp.cvilab.local"
  Issued Distinguished Name: "CN=PKI Admin"
  Issued Binary Name:
0000	30 14 31 12 30 10 06 03  55 04 03 13 09 50 4b 49   0.1.0...U....PKI
0010	20 41 64 6d 69 6e                                   Admin

  Issued Country/Region: EMPTY
  Issued Organization: EMPTY
  Issued Organization Unit: EMPTY
  Issued Common Name: "PKI Admin"
  Issued City: EMPTY
  Issued State: EMPTY
  Issued Title: EMPTY
  Issued First Name: EMPTY
  Issued Initials: EMPTY
  Issued Last Name: EMPTY
  Issued Domain Component: EMPTY
  Issued Email Address: EMPTY
  Issued Street Address: EMPTY
  Issued Unstructured Name: EMPTY
  Issued Unstructured Address: EMPTY
  Issued Device Serial Number: EMPTY

  Request Attributes:
    RequestOSVersion: "10.0.20348.2"
    RequestCSPProvider: "Microsoft Enhanced Cryptographic Provider v1.0"
    cdc: "DC01.corp.cvilab.local"
    rmd: "PKI-SRV01.corp.cvilab.local"
    ccm: "PKI-SRV01.corp.cvilab.local"

  Certificate Extensions:
    1.3.6.1.4.1.311.21.7: Flags = 20000(Origin=Policy), Length = 31
    Certificate Template Information
        Template=1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913
        Major Version Number=100
        Minor Version Number=9

    2.5.29.37: Flags = 20000(Origin=Policy), Length = c
    Enhanced Key Usage
        Client Authentication (1.3.6.1.5.5.7.3.2)

    2.5.29.15: Flags = 20001(Critical, Origin=Policy), Length = 4
    Key Usage
        Digital Signature, Key Encipherment (a0)

    1.3.6.1.4.1.311.21.10: Flags = 20000(Origin=Policy), Length = e
    Application Policies
        [1]Application Certificate Policy:
             Policy Identifier=Client Authentication

    1.2.840.113549.1.9.15: Flags = 20000(Origin=Policy), Length = 37
    SMIME Capabilities
        [1]SMIME Capability
             Object ID=1.2.840.113549.3.2
             Parameters=02 02 00 80
        [2]SMIME Capability
             Object ID=1.2.840.113549.3.4
             Parameters=02 02 00 80
        [3]SMIME Capability
             Object ID=1.3.14.3.2.7
        [4]SMIME Capability
             Object ID=1.2.840.113549.3.7

    2.5.29.14: Flags = 10000(Origin=Request), Length = 16
    Subject Key Identifier
        d2854d9a5f62ed308efc03e4264a4614fcbef628

    2.5.29.35: Flags = 20000(Origin=Policy), Length = 18
    Authority Key Identifier
        KeyID=114d5edbf32dc1542007b8cd3f192d04bbe85fd1

    2.5.29.31: Flags = 40000(Origin=Server), Length = d7
    CRL Distribution Points
        [1]CRL Distribution Point
             Distribution Point Name:
                  Full Name:
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevoc
ationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PKI-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=
cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=cer
tificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?object
Class=certificationAuthority)

    2.5.29.19: Flags = 20003(Critical, Disabled, Origin=Policy), Length = 2
    Basic Constraints
        Subject Type=End Entity
        Path Length Constraint=None

    2.5.29.17: Flags = 20000(Origin=Policy), Length = 2f
    Subject Alternative Name
        Other Name:
             Principal Name=pki.admin@corp.cvilab.local


Maximum Row Index: 4

4 Rows
 148 Row Properties, Total Size = 17728, Max Size = 1630, Ave Size = 119
  20 Request Attributes, Total Size = 1496, Max Size = 132, Ave Size = 74
  43 Certificate Extensions, Total Size = 3943, Max Size = 241, Ave Size = 91
 211 Total Fields, Total Size = 23167, Max Size = 1630, Ave Size = 109
CertUtil: -view command completed successfully.
```

If you also used CORP\cert.manager:
```powershell
certutil -view -restrict "RequesterName=CORP\cert.manager"
```

```

PS C:\Windows\system32> certutil -view -restrict "RequesterName=CORP\cert.manager"
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  Request.RequestID             Request ID                    Long    4 -- Indexed
  Request.RawRequest            Binary Request                Binary  65536
  Request.RawArchivedKey        Archived Key                  Binary  65536
  Request.KeyRecoveryHashes     Key Recovery Agent Hashes     String  8192
  Request.RawOldCertificate     Old Certificate               Binary  16384
  Request.RequestAttributes     Request Attributes            String  32768
  Request.RequestType           Request Type                  Long    4
  Request.RequestFlags          Request Flags                 Long    4
  Request.StatusCode            Request Status Code           Long    4
  Request.Disposition           Request Disposition           Long    4 -- Indexed
  Request.DispositionMessage    Request Disposition Message   String  8192
  Request.SubmittedWhen         Request Submission Date       Date    8 -- Indexed
  Request.ResolvedWhen          Request Resolution Date       Date    8 -- Indexed
  Request.RevokedWhen           Revocation Date               Date    8
  Request.RevokedEffectiveWhen  Effective Revocation Date     Date    8 -- Indexed
  Request.RevokedReason         Revocation Reason             Long    4
  Request.RequesterName         Requester Name                String  2048 -- Indexed
  Request.CallerName            Caller Name                   String  2048 -- Indexed
  Request.SignerPolicies        Signer Policies               String  8192
  Request.SignerApplicationPolicies  Signer Application Policies   String  8192
  Request.Officer               Officer                       Long    4
  Request.DistinguishedName     Request Distinguished Name    String  8192
  Request.RawName               Request Binary Name           Binary  4096
  Request.Country               Request Country/Region        String  8192
  Request.Organization          Request Organization          String  8192
  Request.OrgUnit               Request Organization Unit     String  8192
  Request.CommonName            Request Common Name           String  8192
  Request.Locality              Request City                  String  8192
  Request.State                 Request State                 String  8192
  Request.Title                 Request Title                 String  8192
  Request.GivenName             Request First Name            String  8192
  Request.Initials              Request Initials              String  8192
  Request.SurName               Request Last Name             String  8192
  Request.DomainComponent       Request Domain Component      String  8192
  Request.EMail                 Request Email Address         String  8192
  Request.StreetAddress         Request Street Address        String  8192
  Request.UnstructuredName      Request Unstructured Name     String  8192
  Request.UnstructuredAddress   Request Unstructured Address  String  8192
  Request.DeviceSerialNumber    Request Device Serial Number  String  8192
  Request.AttestationChallenge  Attestation Challenge         Binary  4096
  Request.EndorsementKeyHash    Endorsement Key Hash          String  144 -- Indexed
  Request.EndorsementCertificateHash  Endorsement Certificate Hash  String  144 -- Indexed
  Request.RawPrecertificate     Binary Precertificate         Binary  16384
  RequestID                     Issued Request ID             Long    4 -- Indexed
  RawCertificate                Binary Certificate            Binary  16384
  CertificateHash               Certificate Hash              String  128 -- Indexed
  CertificateTemplate           Certificate Template          String  254 -- Indexed
  EnrollmentFlags               Template Enrollment Flags     Long    4
  GeneralFlags                  Template General Flags        Long    4
  PrivatekeyFlags               Template Private Key Flags    Long    4
  SerialNumber                  Serial Number                 String  128 -- Indexed
  IssuerNameID                  Issuer Name ID                Long    4
  NotBefore                     Certificate Effective Date    Date    8
  NotAfter                      Certificate Expiration Date   Date    8 -- Indexed
  SubjectKeyIdentifier          Issued Subject Key Identifier  String  128 -- Indexed
  RawPublicKey                  Binary Public Key             Binary  4096
  PublicKeyLength               Public Key Length             Long    4
  PublicKeyAlgorithm            Public Key Algorithm          String  254
  RawPublicKeyAlgorithmParameters  Public Key Algorithm Parameters  Binary  4096
  PublishExpiredCertInCRL       Publish Expired Certificate in CRL  Long    4
  UPN                           User Principal Name           String  2048 -- Indexed
  DistinguishedName             Issued Distinguished Name     String  8192
  RawName                       Issued Binary Name            Binary  4096
  Country                       Issued Country/Region         String  8192
  Organization                  Issued Organization           String  8192
  OrgUnit                       Issued Organization Unit      String  8192
  CommonName                    Issued Common Name            String  8192 -- Indexed
  Locality                      Issued City                   String  8192
  State                         Issued State                  String  8192
  Title                         Issued Title                  String  8192
  GivenName                     Issued First Name             String  8192
  Initials                      Issued Initials               String  8192
  SurName                       Issued Last Name              String  8192
  DomainComponent               Issued Domain Component       String  8192
  EMail                         Issued Email Address          String  8192
  StreetAddress                 Issued Street Address         String  8192
  UnstructuredName              Issued Unstructured Name      String  8192
  UnstructuredAddress           Issued Unstructured Address   String  8192
  DeviceSerialNumber            Issued Device Serial Number   String  8192

Maximum Row Index: 0

0 Rows
   0 Row Properties, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Request Attributes, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Certificate Extensions, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Total Fields, Total Size = 0, Max Size = 0, Ave Size = 0
CertUtil: -view command completed successfully.
```

### Step 5 — Document Two Specific Certificate Records

Identify at least **two specific certificates** from your prior lab work (Weeks 10 or 11). For each, record the full certificate record fields.

---

**Certificate Record 1**

Request ID:
```
16
```

Requester Name:
```
CORP\PKI-SRV01$
```

Certificate Template:
```
CVI-WebServer
```

Issued Common Name:
```
CN = PKI-SRV01.corp.cvilab.local
```

Not Before:
```
‎Sunday, ‎June ‎14, ‎2026 12:06:00 PM
```

Not After:
```
‎Sunday, ‎April ‎25, ‎2027 7:36:58 PM
```

Serial Number:
```
4400000010bf15486a229bae24000000000010
```

Disposition:
```
20
```

Revocation Date (if revoked):
```
N/A
```

Revocation Reason (if revoked):
```
N/A
```

**Which lab did this certificate come from?**
```
Week 10
```

---

**Certificate Record 2**

Request ID:
```
5
```

Requester Name:
```
CORP\pki.admin
```

Certificate Template:
```
CVI Code Signing
```

Issued Common Name:
```
CN = PKI Admin
```

Not Before:
```
‎‎Sunday, ‎May ‎24, ‎2026 12:53:58 PM
```

Not After:
```
‎‎Sunday, ‎April ‎25, ‎2027 7:36:58 PM
```

Serial Number:
```
440000000d5ff28d7e92a9e01f00000000000d
```

Disposition:
```
21
```

Revocation Date (if revoked):
```
‎Saturday, ‎May ‎30, ‎2026 11:10:00 AM
```

Revocation Reason (if revoked):
```
Cease of Operation
```

**Which lab did this certificate come from?**
```
eek 11 (certificate created/issued) and revoked in Week 12.
```

---

### Step 6 — Filter by Template

Run the following to see how certificates are distributed across templates:

```powershell
# List all issued certificates showing only template and CN
certutil -view -restrict "Disposition=20" -out "CertificateTemplate,CommonName,NotAfter"
```

```
PS C:\Windows\system32> certutil -view -restrict "Disposition=20" -out "CertificateTemplate,CommonName,NotAfter"
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  CertificateTemplate           Certificate Template          String  254 -- Indexed
  CommonName                    Issued Common Name            String  8192 -- Indexed
  NotAfter                      Certificate Expiration Date   Date    8 -- Indexed

Row 1:
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.13096150.8214139" CVI-WebServer
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 2:
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.16756659.8111121" CVI Service Account
  Issued Common Name: "Svc Autoenroll"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 3:
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.1.32" OCSP Response Signing
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/13/2026 12:25 PM

Row 4:
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.1.32" OCSP Response Signing
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/13/2026 12:26 PM

Row 5:
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.1.32" OCSP Response Signing
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/13/2026 12:39 PM

Row 6:
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.13096150.8214139" CVI-WebServer
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 7:
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.13096150.8214139" CVI-WebServer
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 8:
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.13096150.8214139" CVI-WebServer
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Maximum Row Index: 8

8 Rows
  24 Row Properties, Total Size = 1726, Max Size = 166, Ave Size = 71
   0 Request Attributes, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Certificate Extensions, Total Size = 0, Max Size = 0, Ave Size = 0
  24 Total Fields, Total Size = 1726, Max Size = 166, Ave Size = 71
CertUtil: -view command completed successfully.
```

**Templates represented in your CA database:**
```
OCSP Response Signing
CVI-WebServer
CVI Service Account
```

---

## Part C — Analysis

Answer the following questions using the event log data from Part A and the certificate database data from Part B.

### Analysis Question 1

Select **one event log entry** from Part A (any event type). In 3–5 sentences, explain what this event tells you about the CA's operational state at the time it was generated. Be specific about what action caused the event and what the event confirms about the CA.

```
The CA was not operational at the time because it could not initialize its database after the required database files in the CertLog folder were missing. Because the database and log files were missing, Active Directory Certificate Services failed to start when the service was started. This event confirms that the CA depends on its database and transaction log files to initialize correctly and become operational.
```

### Analysis Question 2

Select **one certificate record** from Part B. In 3–5 sentences, explain what this record tells you about the lifecycle of that specific certificate. Include the template used, who requested it, its current status, and what you would need to do next if the certificate were approaching expiry.

```
This certificate was requested by CORP\pki.admin and was issued using the CVI Code Signing template. Its current status is revoked, which means it can no longer be used. If this certificate had not been revoked and was approaching its expiration date, it would need to be renewed before it expired to avoid interrupting business operations that depend on it.
```

### Analysis Question 3

In 4–6 sentences, explain how the Application event log and the CA certificate database complement each other as operational data sources. Give a specific example scenario where you would need to consult **both** sources to fully understand what happened — and explain why one source alone would be insufficient.

```
The CA certificate database records certificate information such as who requested the certificate, the template used, and whether the certificate was issued or revoked. The Application event log records what the CA service was doing, such as service start and stop events, database errors, and Active Directory communication issues. If someone reported that their certificate was not working, I would first check the CA database to verify the certificate's status and validity. Then I would check the Application event log to see if the CA experienced any errors or other issues that could explain what happened, since the database alone would not show operational events.

```

---

## Lab Report Questions

Answer each question in complete sentences.

**1. What event type did you find most frequently in the Application event log? What does the frequency of that event type tell you about the CA's most common operation in your lab environment?**

```
The CA service start and stop events appeared most frequently in the Application event log. The high number of these events shows that the CA service was frequently started and stopped during the lab exercises for testing, troubleshooting, backups, and recovery tasks.
```

**2. The certutil -view command queries the CA database, not the event log. What is the difference between what these two sources record — and what would you lose operationally if you had access to the event log but not the CA database?**

```
The difference is that the Application event log records what the CA service was doing, while the CA database records certificate information such as whether a certificate is issued, revoked, or pending. If you only had access to the Application event log, you would lose access to certificate information stored in the CA database, such as the requester, certificate template, validity period, serial number, and current certificate status.
```

**3. In the Disposition field, you saw codes 20 (Issued) and 21 (Revoked). If a certificate shows Disposition 21, what additional fields should you check to understand the full context of the revocation, and why?**

```
If a certificate shows Disposition 21 (Revoked), you should check the Revocation Reason and Revocation Date. These fields show why the certificate was revoked and when the revocation occurred, which helps provide the full context of the certificate's status.
```

**4. The Application event log records CA events by default. The Security event log does not record CA events without additional configuration (covered in Lesson 3). Based on what you found in Part A, what operational information is present in the Application log — and what is absent that would be important in a production environment?**

```
The Application event log contains information about what the CA service was doing, such as service starts and stops, errors, and Active Directory communication issues. However, it does not provide information about which user or administrator performed the actions. In a production environment, this information would be important for accountability, auditing, and investigating security-related events.
```

---

## Submission Checklist

- [X] Logged in as CORP\pki.admin — whoami output included
- [X] CA running and responding — Get-Service and certutil -ping output included
- [X] Application event log filtered by Source = CertificationAuthority
- [X] At least three distinct event types documented with Event ID, timestamp, and description
- [X] Event type summary table completed
- [X] certutil -view -restrict "Disposition=20" output included (all issued certs)
- [X] certutil -view -restrict "Disposition=21" output included (all revoked certs)
- [X] certutil -view requester filter output included
- [X] At least two specific certificate records documented in full
- [X] Template distribution output included
- [X] All three Part C analysis questions answered (minimum sentence counts met)
- [X] All four lab report questions answered in complete sentences
- [X] File committed to `labs/week-14/lab-01-event-log-navigation.md`
