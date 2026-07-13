# Lab 02: CA Health Check Routine

Lufann Stewart  
July 11, 2026   
**Phase:** 2 | **Week:** 14  
**Submission Path:** `labs/week-14/lab-02-ca-health-check.md`

---

## Overview

In this lab, you run a structured CA health check on PKI-SRV01 covering all three operational health signals: CRL freshness, OCSP availability, and the certificate expiration pipeline. You will use two tools: **pkiview.msc** (the Enterprise PKI snap-in) for a hierarchy-wide visual status check, and **certutil** for precision, forced publication, OCSP accuracy testing, and the expiration pipeline that pkiview cannot see.

The output is a completed health check report with a pass/fail status for each signal — a reusable procedure you can apply to any AD CS environment.

**Prerequisite:** Part B (OCSP testing) requires a revoked certificate in your CA database. If you have not revoked a certificate since Week 12, revoke one of the template certificates from Week 10 Lab 01 before proceeding to Part B.

---

## Lab Environment

| Component | Details |
|-----------|---------|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| OCSP Endpoint | `http://pki.corp.cvilab.local/ocsp` |
| CRL HTTP Path | `http://pki.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl` |
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
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (15ms)
CertUtil: -ping command completed successfully.

```

**CA is running and responding:**
- [X] Yes — proceed to Pre-Lab Step 2
- [ ] No — resolve before continuing

### Step 2 — Open pkiview.msc and Record Initial Status

Before running any certutil commands, open pkiview.msc and read the PKI hierarchy.

**To open pkiview.msc:**
```
Option 1: Windows + R → type pkiview.msc → Enter
Option 2: Server Manager → Tools → Enterprise PKI
```

Expand the full tree: **Enterprise PKI → CVI Root CA → CVI Issuing CA 1**

For each node listed below, record the color indicator you see (Green / Amber / Red) and a brief description of what it shows:

| Node | Color | What It Shows |
|---|---|---|
| CVI Root CA — CA Certificate | Red | CA certificate is valid (expires 4/25/2027), but the parent node is red because child PKI validation checks report an unavailable AIA location and expired CDP/Delta CRL locations. |
| CVI Issuing CA 1 — CA Certificate | Green | CA certificate status is OK. |
| CDP row(s) — CRL Distribution Point | Red | One CDP location is Expired, Delta CRL is Expired, another CDP location is OK. |
| AIA row(s) — Authority Information Access | Red | One AIA location is OK; the second reports Unable to download. |

```
In PKIView, the Enterprise PKI tree is expanded to show CVI Root CA and CVI Issuing CA 1. Both CA nodes display red status indicators because one or more child validation checks reported errors. The CA certificate itself is valid, but the overall PKI health status is affected by the AIA, CDP, and Delta CRL issues.

### pkiview.msc Current Node Status


| Node / Location | Status | Expiration / Status Details | URL / Full Path |
| :--- | :--- | :--- | :--- |
| **CA Certificate** | OK (Green) | Expires on 4/25/2027 at 7:36 PM | *N/A (Local Store)* |
| **AIA Location #1** | OK (Green) | Expires on 4/25/2027 at 7:36 PM | `ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local` |
| **AIA Location #2** | Error (Red) | Unable to download | http://pki-srv01.corp.cvilab.local/ocsp |
| **OCSP Location** | OK (Green) | No Expiration date listed | http://pki-srv01.corp.cvilab.local/ocsp |
| **CDP Location #1** | Error (Red) | Expired on 5/3/2026 at 7:55 AM | http://pki-srv01.corp.cvilab.local/CertEnroll/CVI%20Issuing%20CA%201.crl |
| **Delta CRL Location #1** | Error (Red) | Expired on 4/27/2026 at 7:55 AM | ldap:///CN=CVI%20Issuing%20CA%201,CN=PKI-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?deltaRevocationList?base?objectClass=cRLDistributionPoint |
| **CDP Location #2** | OK (Green) | NextUpdate: 7/12/2125 at 1:26 AM | http://pki-srv01.corp.cvilab.local/CertEnroll/CVI%20Issuing%20CA%201.crl |


```

**pkiview initial status:**
- [ ] All green — no issues visible before running certutil
- [ ] One or more amber indicators — note which node(s):
- [X] One or more red indicators — note which node(s):
CVI Root CA
CVI Issuing CA 1
AIA Location #2
CDP Location #1
Delta CRL Location #1

### Step 3 — Confirm a Revoked Certificate Exists (Required for Part B)

```powershell
certutil -view -restrict "Disposition=21"
```
<details>
<summary><b>Click to certutil output</b></summary>
  
```
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
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PK
I-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=certificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services
,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=certificationAuthority)

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
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PK
I-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=certificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services
,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=certificationAuthority)

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
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PK
I-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=certificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services
,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=certificationAuthority)

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
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PK
I-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=certificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services
,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=certificationAuthority)

    2.5.29.19: Flags = 20003(Critical, Disabled, Origin=Policy), Length = 2
    Basic Constraints
        Subject Type=End Entity
        Path Length Constraint=None

    2.5.29.17: Flags = 20000(Origin=Policy), Length = 2f
    Subject Alternative Name
        Other Name:
             Principal Name=pki.admin@corp.cvilab.local


Row 5:
  Request ID: 0xe (14)
  Binary Request:
-----BEGIN NEW CERTIFICATE REQUEST-----
MIIGEgYJKoZIhvcNAQcCoIIGAzCCBf8CAQMxCzAJBgUrDgMCGgUAMIIEbAYIKwYB
BQUHDAKgggReBIIEWjCCBFYwaTBnAgECBgorBgEEAYI3CgoBMVYwVAIBADADAgEB
MUowSAYJKwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxv
Y2FsDA5DT1JQXHBraS5hZG1pbgwHTU1DLkVYRTCCA+OgggPfAgEBMIID2DCCAsAC
AQAwADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAOAriiEQ3cCDbaEe
3O+EO/aJL+vrlKs2lMA0zrjNumx0t0jee4U/cJ8HEtDnQVNpIR+7LTr8/SkIY/c/
3ffQfSA8ZbnJUEKkDhAyYrADQEs0gDbqFmbqEe7RBtz1ypNGdaXW9yne8h7/osyx
y2+gxUmESCt6LuRcttNv8f6JwGriYlAkD49PCpqG2bKcpA7wzOo2o76shVzblyUR
Is9/zl1MrgZsIBRvy7gw5BQeagiEdaDiSoxQqdxRct3nqCfRNQPFvcAXaYTXhw5T
4FHiIIC8/fo7zlW3XscFUOcRcSWowfCS+pvYYo5m6hlT3t5lGqjm3YLFCQVwPwlT
Ye7UezUCAwEAAaCCAZEwHAYKKwYBBAGCNw0CAzEOFgwxMC4wLjIwMzQ4LjIwSAYJ
KwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxvY2FsDA5D
T1JQXHBraS5hZG1pbgwHTU1DLkVYRTByBgorBgEEAYI3DQICMWQwYgIBAR5aAE0A
aQBjAHIAbwBzAG8AZgB0ACAAUgBTAEEAIABTAEMAaABhAG4AbgBlAGwAIABDAHIA
eQBwAHQAbwBnAHIAYQBwAGgAaQBjACAAUAByAG8AdgBpAGQAZQByAwEAMIGyBgkq
hkiG9w0BCQ4xgaQwgaEwPgYJKwYBBAGCNxUHBDEwLwYnKwYBBAGCNxUIh8nSSIKG
qjyEpY8oh4rLKIPiqFOBKYafqVaD9ax7AgFkAgERMBMGA1UdJQQMMAoGCCsGAQUF
BwMBMA4GA1UdDwEB/wQEAwIFoDAbBgkrBgEEAYI3FQoEDjAMMAoGCCsGAQUFBwMB
MB0GA1UdDgQWBBTUmbieVAA51HrBnclTw1s6aXI+EDANBgkqhkiG9w0BAQUFAAOC
AQEASwe/E/JMGKU8mtqEciQTEpdC89JQoR+cHoUp0n6IasXKLduTYCaUJ2WsP52/
7FpXcky78juv0Fpwq8V1nUkhGpOv19JCgxPafkYtzIunbFov2SZLXQ/BxjmV/yRG
Z7wVzm3481ztxWLV4hqgUnozGy+fyRUuTz62s3yQZaOWODnEko19z3ddWIie8kd7
RI2ZqykT9kVyojWixo4yQvZEThPLoyyIMMvwn12EsiKxQFsNtB4HPk+Og63pP6xK
oih51j/fzOt8TlVJXze2GQitX+G8C/jVl2HygfVPME8+qG4ATjoviv2FvVPun8Sn
N8g2584G5MNd9+KVVsSB4+3XrjAAMAAxggF7MIIBdwIBA4AU1Jm4nlQAOdR6wZ3J
U8NbOmlyPhAwCQYFKw4DAhoFAKA+MBcGCSqGSIb3DQEJAzEKBggrBgEFBQcMAjAj
BgkqhkiG9w0BCQQxFgQUekJyL7YlXslqLUW7sFj3PMBIlEgwDQYJKoZIhvcNAQEB
BQAEggEAvo/YRS/XyU32Q6QzNK/UElXHp1NAr3EeObVs5WUGPusTSSnloxvHk79Z
q22KwEoOi20Ah3WapquIvBahmUVcEP/T2w9v28t3mrKAzY1mLhoiBeFXJRa6aG6U
rBXNdhc3LCxSSmrIeU533g2NEhrxhYeIPp46fadIkdrRNFhuiR1wpC6tDgaxgSxr
mFsfTkjLKnGllGnDIHKHDug6+j9FRKBI9XOoWFOyu5uiX4h7QHq1y3WPMwSZ0dfi
5OiGFTtRX/KbYowIKK6VNkcmnc644NWxA5OzgWdS5n/wr75X9Y3vRAXU4W7eIfzx
Tx16udBQtmEAJ3pzGqCuCAcR3LXg7g==
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
  Request Submission Date: 5/31/2026 7:17 AM
  Request Resolution Date: 5/31/2026 7:17 AM
  Revocation Date: 7/10/2026 9:14 AM
  Effective Revocation Date: 7/10/2026 9:14 AM
  Revocation Reason: 0x5 -- Reason: Cessation of Operation
  Requester Name: "CORP\PKI-SRV01$"
  Caller Name: "CORP\PKI-SRV01$"
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
  Issued Request ID: 0xe (14)
  Binary Certificate:
-----BEGIN CERTIFICATE-----
MIIGCTCCBPGgAwIBAgITRAAAAA7wK7EJ80VRrQAAAAAADjANBgkqhkiG9w0BAQsF
ADBgMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFjAUBgoJkiaJk/IsZAEZFgZjdmls
YWIxFDASBgoJkiaJk/IsZAEZFgRjb3JwMRkwFwYDVQQDExBDVkkgSXNzdWluZyBD
QSAxMB4XDTI2MDUzMTE0MDcyOVoXDTI3MDQyNjAyMzY1OFowJjEkMCIGA1UEAxMb
UEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxvY2FsMIIBIjANBgkqhkiG9w0BAQEFAAOC
AQ8AMIIBCgKCAQEA4CuKIRDdwINtoR7c74Q79okv6+uUqzaUwDTOuM26bHS3SN57
hT9wnwcS0OdBU2khH7stOvz9KQhj9z/d99B9IDxluclQQqQOEDJisANASzSANuoW
ZuoR7tEG3PXKk0Z1pdb3Kd7yHv+izLHLb6DFSYRIK3ou5Fy202/x/onAauJiUCQP
j08KmobZspykDvDM6jajvqyFXNuXJREiz3/OXUyuBmwgFG/LuDDkFB5qCIR1oOJK
jFCp3FFy3eeoJ9E1A8W9wBdphNeHDlPgUeIggLz9+jvOVbdexwVQ5xFxJajB8JL6
m9hijmbqGVPe3mUaqObdgsUJBXA/CVNh7tR7NQIDAQABo4IC9DCCAvAwPgYJKwYB
BAGCNxUHBDEwLwYnKwYBBAGCNxUIh8nSSIKGqjyEpY8oh4rLKIPiqFOBKYafqVaD
9ax7AgFkAgERMBMGA1UdJQQMMAoGCCsGAQUFBwMBMA4GA1UdDwEB/wQEAwIFoDAb
BgkrBgEEAYI3FQoEDjAMMAoGCCsGAQUFBwMBMB0GA1UdDgQWBBTUmbieVAA51HrB
nclTw1s6aXI+EDAfBgNVHSMEGDAWgBQRTV7b8y3BVCAHuM0/GS0Eu+hf0TCCAS4G
A1UdHwSCASUwggEhMIIBHaCCARmgggEVhoHIbGRhcDovLy9DTj1DVkklMjBJc3N1
aW5nJTIwQ0ElMjAxLENOPVBLSS1TUlYwMSxDTj1DRFAsQ049UHVibGljJTIwS2V5
JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz1jb3Jw
LERDPWN2aWxhYixEQz1sb2NhbD9jZXJ0aWZpY2F0ZVJldm9jYXRpb25MaXN0P2Jh
c2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJpYnV0aW9uUG9pbnSGSGh0dHA6Ly9wa2kt
c3J2MDEuY29ycC5jdmlsYWIubG9jYWwvQ2VydEVucm9sbC9DVkklMjBJc3N1aW5n
JTIwQ0ElMjAxLmNybDCB0QYIKwYBBQUHAQEEgcQwgcEwgb4GCCsGAQUFBzAChoGx
bGRhcDovLy9DTj1DVkklMjBJc3N1aW5nJTIwQ0ElMjAxLENOPUFJQSxDTj1QdWJs
aWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxDTj1Db25maWd1cmF0aW9u
LERDPWNvcnAsREM9Y3ZpbGFiLERDPWxvY2FsP2NBQ2VydGlmaWNhdGU/YmFzZT9v
YmplY3RDbGFzcz1jZXJ0aWZpY2F0aW9uQXV0aG9yaXR5MCYGA1UdEQQfMB2CG1BL
SS1TUlYwMS5jb3JwLmN2aWxhYi5sb2NhbDANBgkqhkiG9w0BAQsFAAOCAQEAoTjW
4zIX1dzgfM0xsoJl1Fku9snO+EmufGad6dN05oIi+ew+qE/zB0ACI6prdOvhqDXh
pU2MTt9JY0ybiAdcU1Y9JvrN42YPVdo3/2CAGM6NN3dsg+WocCZ/a/3m81JPMBEv
Uo3P1R7sppV1xqiA8j4x7Dh1QhsS3e6Sy9tSjeVZzFkxtao522WXD8ueLrp84LVF
9UFODEfTyigp3AGJyb85ml6S+Mtzzt+tusbWDnJxWmb++JgQZI3pU9YzraUr07S0
JTK1HqNMmNYGura/vO82seCvmDgJSoa/zjx/NBqvl7nOgyJRd2qAKjouaa3RirU8
LpoYmUwM8TXeTWWwiA==
-----END CERTIFICATE-----

  Certificate Hash: "ac 13 70 39 70 99 7b 73 25 d9 6f da 86 5d dd 9c 98 7c 69 a0"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.13096150.8214139" CVI-WebServer
  Template Enrollment Flags: 0x20 (32)
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)
  Template General Flags: 0x20241 (131649)
    CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT -- 1
    CT_FLAG_MACHINE_TYPE -- 40 (64)
    CT_FLAG_ADD_TEMPLATE_NAME -- 200 (512)
    CT_FLAG_IS_MODIFIED -- 20000 (131072)
  Template Private Key Flags: 0x3050000 (50659328)
    CTPRIVATEKEY_FLAG_ATTEST_NONE -- 0
    TEMPLATE_SERVER_VER_WINBLUE<<CTPRIVATEKEY_FLAG_SERVERVERSION_SHIFT -- 50000 (327680)
    TEMPLATE_CLIENT_VER_WIN7<<CTPRIVATEKEY_FLAG_CLIENTVERSION_SHIFT -- 3000000 (50331648)
  Serial Number: "440000000ef02bb109f34551ad00000000000e"
  Issuer Name ID: 0x0 CA Version 0.0
  Certificate Effective Date: 5/31/2026 7:07 AM
  Certificate Expiration Date: 4/25/2027 7:36 PM
  Issued Subject Key Identifier: "d4 99 b8 9e 54 00 39 d4 7a c1 9d c9 53 c3 5b 3a 69 72 3e 10"
  Binary Public Key:
0000	30 82 01 0a 02 82 01 01  00 e0 2b 8a 21 10 dd c0
0010	83 6d a1 1e dc ef 84 3b  f6 89 2f eb eb 94 ab 36
0020	94 c0 34 ce b8 cd ba 6c  74 b7 48 de 7b 85 3f 70
0030	9f 07 12 d0 e7 41 53 69  21 1f bb 2d 3a fc fd 29
0040	08 63 f7 3f dd f7 d0 7d  20 3c 65 b9 c9 50 42 a4
0050	0e 10 32 62 b0 03 40 4b  34 80 36 ea 16 66 ea 11
0060	ee d1 06 dc f5 ca 93 46  75 a5 d6 f7 29 de f2 1e
0070	ff a2 cc b1 cb 6f a0 c5  49 84 48 2b 7a 2e e4 5c
0080	b6 d3 6f f1 fe 89 c0 6a  e2 62 50 24 0f 8f 4f 0a
0090	9a 86 d9 b2 9c a4 0e f0  cc ea 36 a3 be ac 85 5c
00a0	db 97 25 11 22 cf 7f ce  5d 4c ae 06 6c 20 14 6f
00b0	cb b8 30 e4 14 1e 6a 08  84 75 a0 e2 4a 8c 50 a9
00c0	dc 51 72 dd e7 a8 27 d1  35 03 c5 bd c0 17 69 84
00d0	d7 87 0e 53 e0 51 e2 20  80 bc fd fa 3b ce 55 b7
00e0	5e c7 05 50 e7 11 71 25  a8 c1 f0 92 fa 9b d8 62
00f0	8e 66 ea 19 53 de de 65  1a a8 e6 dd 82 c5 09 05
0100	70 3f 09 53 61 ee d4 7b  35 02 03 01 00 01

  Public Key Length: 0x800 (2048)
  Public Key Algorithm: "1.2.840.113549.1.1.1" RSA (RSA_SIGN)
  Public Key Algorithm Parameters:
0000	05 00                                              ..

  Publish Expired Certificate in CRL: 0x0
  User Principal Name: "PKI-SRV01$@corp.cvilab.local"
  Issued Distinguished Name: "CN=PKI-SRV01.corp.cvilab.local"
  Issued Binary Name:
0000	30 26 31 24 30 22 06 03  55 04 03 13 1b 50 4b 49   0&1$0"..U....PKI
0010	2d 53 52 56 30 31 2e 63  6f 72 70 2e 63 76 69 6c   -SRV01.corp.cvil
0020	61 62 2e 6c 6f 63 61 6c                            ab.local

  Issued Country/Region: EMPTY
  Issued Organization: EMPTY
  Issued Organization Unit: EMPTY
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
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
    RequestCSPProvider: "Microsoft RSA SChannel Cryptographic Provider"
    cdc: "DC01.corp.cvilab.local"
    rmd: "PKI-SRV01.corp.cvilab.local"
    ccm: "PKI-SRV01.corp.cvilab.local"

  Certificate Extensions:
    1.3.6.1.4.1.311.21.7: Flags = 20000(Origin=Policy), Length = 31
    Certificate Template Information
        Template=CVI-WebServer(1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.13096150.8214139)
        Major Version Number=100
        Minor Version Number=17

    2.5.29.37: Flags = 20000(Origin=Policy), Length = c
    Enhanced Key Usage
        Server Authentication (1.3.6.1.5.5.7.3.1)

    2.5.29.15: Flags = 20001(Critical, Origin=Policy), Length = 4
    Key Usage
        Digital Signature, Key Encipherment (a0)

    1.3.6.1.4.1.311.21.10: Flags = 20000(Origin=Policy), Length = e
    Application Policies
        [1]Application Certificate Policy:
             Policy Identifier=Server Authentication

    2.5.29.14: Flags = 10000(Origin=Request), Length = 16
    Subject Key Identifier
        d499b89e540039d47ac19dc953c35b3a69723e10

    2.5.29.35: Flags = 20000(Origin=Policy), Length = 18
    Authority Key Identifier
        KeyID=114d5edbf32dc1542007b8cd3f192d04bbe85fd1

    2.5.29.31: Flags = 40000(Origin=Server), Length = 125
    CRL Distribution Points
        [1]CRL Distribution Point
             Distribution Point Name:
                  Full Name:
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PK
I-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)
                       URL=http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl (http://pki-srv01.corp.cvilab.local/CertEnroll/CVI%20Issuing%20CA%201.crl)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = c4
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=certificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services
,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=certificationAuthority)

    2.5.29.19: Flags = 20003(Critical, Disabled, Origin=Policy), Length = 2
    Basic Constraints
        Subject Type=End Entity
        Path Length Constraint=None

    2.5.29.17: Flags = 20000(Origin=Policy), Length = 1f
    Subject Alternative Name
        DNS Name=PKI-SRV01.corp.cvilab.local


Row 6:
  Request ID: 0x13 (19)
  Binary Request:
-----BEGIN NEW CERTIFICATE REQUEST-----
MIIGFAYJKoZIhvcNAQcCoIIGBTCCBgECAQMxCzAJBgUrDgMCGgUAMIIEbgYIKwYB
BQUHDAKgggRgBIIEXDCCBFgwaTBnAgECBgorBgEEAYI3CgoBMVYwVAIBADADAgEB
MUowSAYJKwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxv
Y2FsDA5DT1JQXHBraS5hZG1pbgwHTU1DLkVYRTCCA+WgggPhAgEBMIID2jCCAsIC
AQAwADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMbQtJTr7v7AmTKz
b+EPHuSlgmSqWE9duMOmo4LNL7s+YIt19TU5z9zU3k2+sxDH+Abm5wA4BWyBt73A
PQieWRtpN9X8lvA2WIcFwYobu9KieNrjCpXfPA43+rQGjWuDLMEm1G8lM2qsCtsx
ZwEBvdwtiqbFjFhZ0GmGKb45UseaLes4npj/hUK8b6ST4h4r1iBSzL+QpFR2UEW4
2rgRLawXcp4fPuGpNEsoh0K1WtRJAPRmaPjrGAy3tT8vL3Y7ywLlGOhUYHatY8mA
lw0cwyW8utPG0h5F6vlrGJoHhfOyomggBf0jIfQ4a/Wr6yl1d6oOm/lu+czorfSO
YqcTsqkCAwEAAaCCAZMwHAYKKwYBBAGCNw0CAzEOFgwxMC4wLjIwMzQ4LjIwSAYJ
KwYBBAGCNxUUMTswOQIBBQwbUEtJLVNSVjAxLmNvcnAuY3ZpbGFiLmxvY2FsDA5D
T1JQXHBraS5hZG1pbgwHTU1DLkVYRTB0BgorBgEEAYI3DQICMWYwZAIBAh5cAE0A
aQBjAHIAbwBzAG8AZgB0ACAARQBuAGgAYQBuAGMAZQBkACAAQwByAHkAcAB0AG8A
ZwByAGEAcABoAGkAYwAgAFAAcgBvAHYAaQBkAGUAcgAgAHYAMQAuADADAQAwgbIG
CSqGSIb3DQEJDjGBpDCBoTA+BgkrBgEEAYI3FQcEMTAvBicrBgEEAYI3FQiHydJI
goaqPISljyiHissog+KoU4Epgv+VcYTTsxICAWQCAQIwEwYDVR0lBAwwCgYIKwYB
BQUHAwMwDgYDVR0PAQH/BAQDAgeAMBsGCSsGAQQBgjcVCgQOMAwwCgYIKwYBBQUH
AwMwHQYDVR0OBBYEFL9SooQdM0DT/w9e4LKbplOp1d0eMA0GCSqGSIb3DQEBBQUA
A4IBAQBeV/ZAnB80zdBIgTlqHhN7sn2+q8xi/0M2fSvQ5zWrrbhFVXVZRoaJyuKQ
U+jWvJKqtzPOJYxBWC71fMxTEWqL8pkNbSbywHWvp2UBJrImuTPylllyOdaiAczy
vUehfH7hd5wnd0zC9c8ngo80PZqaUMDxL2hldF4pP2LMrRXbk6mnGBMB37zab05N
UF5DDGtkNoHRgA9L1cKoKOmzAmMNByb5l6cBeQBC4J+IGbAN4NjcSKCINxkzVgFw
UQOLHLQqAvEsbopam3GmBjvU6hNZErLL58PsJXmGBpynepyS0KKbG0q9jpmUAPfX
QKvVasZZ5xi+2Vd585rOtZdnDywNMAAwADGCAXswggF3AgEDgBS/UqKEHTNA0/8P
XuCym6ZTqdXdHjAJBgUrDgMCGgUAoD4wFwYJKoZIhvcNAQkDMQoGCCsGAQUFBwwC
MCMGCSqGSIb3DQEJBDEWBBQ17SInm6gvhxnQPsuxSg4c4BopejANBgkqhkiG9w0B
AQEFAASCAQBRbU9+AzDBkbLI3pbj+JuPGf6gis99vfRy3K2frKsV3CqSeNEcjwnk
sbS/0ue0+Fy1Rbit3QxKutXZYn21uwbgLJey7alU48YJax22O/BPyjLwdD7S1Zkd
KVRo95HP316Ql9Vrdb/dXtdO7yfaPjIBUJWAPUJaPwlGgcfePRa2JOX8hkvjKtN0
U2/xUALgVVqwfOkHa0xyDGAE0xN1apNSTXOr7xePoxqpH93FdMxs/OZyLV6bDPMD
QkV00GfTb/10HXlsmQRdfNylS/me8rmndGtm08TmoU4Kvu5ibd1vOFqtlIpnisOM
/eXFeMcoxFQrPm/rwIyaTkaWwQ0mcEhU
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
  Request Submission Date: 7/11/2026 10:26 AM
  Request Resolution Date: 7/11/2026 10:26 AM
  Revocation Date: 7/11/2026 10:27 AM
  Effective Revocation Date: 7/11/2026 10:27 AM
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
  Issued Request ID: 0x13 (19)
  Binary Certificate:
-----BEGIN CERTIFICATE-----
MIIGlzCCBX+gAwIBAgITRAAAABMOHpKuX3oHZAAAAAAAEzANBgkqhkiG9w0BAQsF
ADBgMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFjAUBgoJkiaJk/IsZAEZFgZjdmls
YWIxFDASBgoJkiaJk/IsZAEZFgRjb3JwMRkwFwYDVQQDExBDVkkgSXNzdWluZyBD
QSAxMB4XDTI2MDcxMTE3MTY0OFoXDTI3MDQyNjAyMzY1OFowbjEVMBMGCgmSJomT
8ixkARkWBWxvY2FsMRYwFAYKCZImiZPyLGQBGRYGY3ZpbGFiMRQwEgYKCZImiZPy
LGQBGRYEY29ycDETMBEGA1UECxMKUEtJIEFkbWluczESMBAGA1UEAxMJUEtJIEFk
bWluMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxtC0lOvu/sCZMrNv
4Q8e5KWCZKpYT124w6ajgs0vuz5gi3X1NTnP3NTeTb6zEMf4BubnADgFbIG3vcA9
CJ5ZG2k31fyW8DZYhwXBihu70qJ42uMKld88Djf6tAaNa4MswSbUbyUzaqwK2zFn
AQG93C2KpsWMWFnQaYYpvjlSx5ot6ziemP+FQrxvpJPiHivWIFLMv5CkVHZQRbja
uBEtrBdynh8+4ak0SyiHQrVa1EkA9GZo+OsYDLe1Py8vdjvLAuUY6FRgdq1jyYCX
DRzDJby608bSHkXq+WsYmgeF87KiaCAF/SMh9Dhr9avrKXV3qg6b+W75zOit9I5i
pxOyqQIDAQABo4IDOjCCAzYwPgYJKwYBBAGCNxUHBDEwLwYnKwYBBAGCNxUIh8nS
SIKGqjyEpY8oh4rLKIPiqFOBKYL/lXGE07MSAgFkAgECMBMGA1UdJQQMMAoGCCsG
AQUFBwMDMA4GA1UdDwEB/wQEAwIHgDAbBgkrBgEEAYI3FQoEDjAMMAoGCCsGAQUF
BwMDMB0GA1UdDgQWBBS/UqKEHTNA0/8PXuCym6ZTqdXdHjAfBgNVHSMEGDAWgBQR
TV7b8y3BVCAHuM0/GS0Eu+hf0TCCAS4GA1UdHwSCASUwggEhMIIBHaCCARmgggEV
hoHIbGRhcDovLy9DTj1DVkklMjBJc3N1aW5nJTIwQ0ElMjAxLENOPVBLSS1TUlYw
MSxDTj1DRFAsQ049UHVibGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMs
Q049Q29uZmlndXJhdGlvbixEQz1jb3JwLERDPWN2aWxhYixEQz1sb2NhbD9jZXJ0
aWZpY2F0ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJp
YnV0aW9uUG9pbnSGSGh0dHA6Ly9wa2ktc3J2MDEuY29ycC5jdmlsYWIubG9jYWwv
Q2VydEVucm9sbC9DVkklMjBJc3N1aW5nJTIwQ0ElMjAxLmNybDCCAQYGCCsGAQUF
BwEBBIH5MIH2MIG+BggrBgEFBQcwAoaBsWxkYXA6Ly8vQ049Q1ZJJTIwSXNzdWlu
ZyUyMENBJTIwMSxDTj1BSUEsQ049UHVibGljJTIwS2V5JTIwU2VydmljZXMsQ049
U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz1jb3JwLERDPWN2aWxhYixEQz1s
b2NhbD9jQUNlcnRpZmljYXRlP2Jhc2U/b2JqZWN0Q2xhc3M9Y2VydGlmaWNhdGlv
bkF1dGhvcml0eTAzBggrBgEFBQcwAYYnaHR0cDovL3BraS1zcnYwMS5jb3JwLmN2
aWxhYi5sb2NhbC9vY3NwMDYGA1UdEQQvMC2gKwYKKwYBBAGCNxQCA6AdDBtwa2ku
YWRtaW5AY29ycC5jdmlsYWIubG9jYWwwDQYJKoZIhvcNAQELBQADggEBAEjfOhET
zJJrZtYhxCvHaqcdjVs6N1NZ7STyRlH4IZS3a5DRM1VXrKd9vnqzEO6Ai1IyofNG
QruL0q1OTwqUnBvkJgflS7T2fJ3/5kepNxClGrQUVSWk8MUvQEVHuFYvdpMRSdoV
8DkF3bfC4kZCI3wZWf0dhKTclZTB5fFVgd/gu/+qurIUviimreLCR9kkZuK6udFK
bl1Ypu8h9CJdsSqRQbEiL+BPGwME/6d7HHVKAwM+UJA1A5mSWZ7EK8A4kvzQqNul
cSDL3qO/Imld0igf1u46C2RFMoCNKPT9tlsumGqknEVdand+m7jaC9NOelKLfhN8
yXCoWYss3+CiHrg=
-----END CERTIFICATE-----

  Certificate Hash: "63 1a 3d 3a b7 9e ad 1e 29 08 02 4b 11 ac c2 35 b3 2c 5e 34"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.6277873.9755026"
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
  Serial Number: "44000000130e1e92ae5f7a0764000000000013"
  Issuer Name ID: 0x0 CA Version 0.0
  Certificate Effective Date: 7/11/2026 10:16 AM
  Certificate Expiration Date: 4/25/2027 7:36 PM
  Issued Subject Key Identifier: "bf 52 a2 84 1d 33 40 d3 ff 0f 5e e0 b2 9b a6 53 a9 d5 dd 1e"
  Binary Public Key:
0000	30 82 01 0a 02 82 01 01  00 c6 d0 b4 94 eb ee fe
0010	c0 99 32 b3 6f e1 0f 1e  e4 a5 82 64 aa 58 4f 5d
0020	b8 c3 a6 a3 82 cd 2f bb  3e 60 8b 75 f5 35 39 cf
0030	dc d4 de 4d be b3 10 c7  f8 06 e6 e7 00 38 05 6c
0040	81 b7 bd c0 3d 08 9e 59  1b 69 37 d5 fc 96 f0 36
0050	58 87 05 c1 8a 1b bb d2  a2 78 da e3 0a 95 df 3c
0060	0e 37 fa b4 06 8d 6b 83  2c c1 26 d4 6f 25 33 6a
0070	ac 0a db 31 67 01 01 bd  dc 2d 8a a6 c5 8c 58 59
0080	d0 69 86 29 be 39 52 c7  9a 2d eb 38 9e 98 ff 85
0090	42 bc 6f a4 93 e2 1e 2b  d6 20 52 cc bf 90 a4 54
00a0	76 50 45 b8 da b8 11 2d  ac 17 72 9e 1f 3e e1 a9
00b0	34 4b 28 87 42 b5 5a d4  49 00 f4 66 68 f8 eb 18
00c0	0c b7 b5 3f 2f 2f 76 3b  cb 02 e5 18 e8 54 60 76
00d0	ad 63 c9 80 97 0d 1c c3  25 bc ba d3 c6 d2 1e 45
00e0	ea f9 6b 18 9a 07 85 f3  b2 a2 68 20 05 fd 23 21
00f0	f4 38 6b f5 ab eb 29 75  77 aa 0e 9b f9 6e f9 cc
0100	e8 ad f4 8e 62 a7 13 b2  a9 02 03 01 00 01

  Public Key Length: 0x800 (2048)
  Public Key Algorithm: "1.2.840.113549.1.1.1" RSA (RSA_SIGN)
  Public Key Algorithm Parameters:
0000	05 00                                              ..

  Publish Expired Certificate in CRL: 0x1
  User Principal Name: "pki.admin@corp.cvilab.local"
  Issued Distinguished Name: "CN=PKI Admin, OU=PKI Admins, DC=corp, DC=cvilab, DC=local"
  Issued Binary Name:
0000	30 6e 31 15 30 13 06 0a  09 92 26 89 93 f2 2c 64   0n1.0.....&...,d
0010	01 19 16 05 6c 6f 63 61  6c 31 16 30 14 06 0a 09   ....local1.0....
0020	92 26 89 93 f2 2c 64 01  19 16 06 63 76 69 6c 61   .&...,d....cvila
0030	62 31 14 30 12 06 0a 09  92 26 89 93 f2 2c 64 01   b1.0.....&...,d.
0040	19 16 04 63 6f 72 70 31  13 30 11 06 03 55 04 0b   ...corp1.0...U..
0050	13 0a 50 4b 49 20 41 64  6d 69 6e 73 31 12 30 10   ..PKI Admins1.0.
0060	06 03 55 04 03 13 09 50  4b 49 20 41 64 6d 69 6e   ..U....PKI Admin

  Issued Country/Region: EMPTY
  Issued Organization: EMPTY
  Issued Organization Unit: "PKI Admins"
  Issued Common Name: "PKI Admin"
  Issued City: EMPTY
  Issued State: EMPTY
  Issued Title: EMPTY
  Issued First Name: EMPTY
  Issued Initials: EMPTY
  Issued Last Name: EMPTY
  Issued Domain Component: "local
cvilab
corp"
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
        Template=1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.6277873.9755026
        Major Version Number=100
        Minor Version Number=2

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
        bf52a2841d3340d3ff0f5ee0b29ba653a9d5dd1e

    2.5.29.35: Flags = 20000(Origin=Policy), Length = 18
    Authority Key Identifier
        KeyID=114d5edbf32dc1542007b8cd3f192d04bbe85fd1

    2.5.29.31: Flags = 40000(Origin=Server), Length = 125
    CRL Distribution Points
        [1]CRL Distribution Point
             Distribution Point Name:
                  Full Name:
                       URL=ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint (ldap:///CN=CVI%20Issuing%20CA%201,CN=PK
I-SRV01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint)
                       URL=http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl (http://pki-srv01.corp.cvilab.local/CertEnroll/CVI%20Issuing%20CA%201.crl)

    1.3.6.1.5.5.7.1.1: Flags = 40000(Origin=Server), Length = f9
    Authority Information Access
        [1]Authority Info Access
             Access Method=Certification Authority Issuer (1.3.6.1.5.5.7.48.2)
             Alternative Name:
                  URL=ldap:///CN=CVI Issuing CA 1,CN=AIA,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=certificationAuthority (ldap:///CN=CVI%20Issuing%20CA%201,CN=AIA,CN=Public%20Key%20Services
,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?cACertificate?base?objectClass=certificationAuthority)
        [2]Authority Info Access
             Access Method=On-line Certificate Status Protocol (1.3.6.1.5.5.7.48.1)
             Alternative Name:
                  URL=http://pki-srv01.corp.cvilab.local/ocsp

    2.5.29.19: Flags = 20003(Critical, Disabled, Origin=Policy), Length = 2
    Basic Constraints
        Subject Type=End Entity
        Path Length Constraint=None

    2.5.29.17: Flags = 20000(Origin=Policy), Length = 2f
    Subject Alternative Name
        Other Name:
             Principal Name=pki.admin@corp.cvilab.local


Maximum Row Index: 6

6 Rows
 224 Row Properties, Total Size = 27014, Max Size = 1691, Ave Size = 120
  30 Request Attributes, Total Size = 2242, Max Size = 132, Ave Size = 74
  63 Certificate Extensions, Total Size = 5950, Max Size = 319, Ave Size = 94
 317 Total Fields, Total Size = 35206, Max Size = 1691, Ave Size = 111
CertUtil: -view command completed successfully.  
```
</details>

**At least one revoked certificate exists in the database:**
- [X] Yes — serial number to use in Part B: `44000000130e1e92ae5f7a0764000000000013`
- [ ] No — revoke a certificate now before proceeding:
  ```powershell
  # Find an issued certificate's serial number first
  certutil -view -restrict "Disposition=20" -out "SerialNumber,CommonName"
  # Then revoke it:
  certutil -revoke <serial_number> 5
  ```
  Revoked serial number: `44000000130e1e92ae5f7a0764000000000013`

---

## Part A — CRL Freshness Check

### Step 1 — Read pkiview CDP Status Before Publishing

You already recorded the CDP color in the Pre-Lab. Before running certutil -CRL, note what pkiview shows:

CDP row color before certutil -CRL: **Red**

URL shown in the CDP row (hover or right-click → Properties to see the URL):
```
http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl
```

### Step 2 — Publish a Fresh CRL

Run from an elevated PowerShell prompt on PKI-SRV01:

```powershell
certutil -CRL
```

**Expected output:**
```
CertUtil: -CRL command completed successfully.
```
<details>
<summary><b>Click to certutil output</b></summary>
  
```
CertUtil: -CRL command completed successfully.
```
</details>

**certutil -CRL completed without errors:**
- [X] Yes
- [ ] No — error message:

**After certutil -CRL, return to pkiview.msc.** You may need to right-click the Issuing CA node and select Refresh.

CDP row color after certutil -CRL: **Red**

### Step 3 — Dump the CRL and Read the Validity Timestamps

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl"
```

> **If the file name is different:** Check the CertEnroll folder for the .crl file name.
> ```powershell
> dir "C:\Windows\System32\CertSrv\CertEnroll\" | Where-Object {$_.Name -like "*.crl"}
> ```

```
X509 Certificate Revocation List:
Version: 2
Signature Algorithm:
    Algorithm ObjectId: 1.2.840.113549.1.1.11 sha256RSA
    Algorithm Parameters:
    05 00
Issuer:
    CN=CVI Issuing CA 1
    DC=corp
    DC=cvilab
    DC=local
  Name Hash(sha1): 81835e9994945c3166bf7396611ca225004ed32b
  Name Hash(md5): d31087fe673eedadd94c85f2a0bc4782

 ThisUpdate: 7/11/2026 1:06 PM
 NextUpdate: 7/12/2125 1:26 AM
CRL Entries: 7
  Serial Number: 4400000014178d21c3ed6a4575000000000014
   Revocation Date: 7/11/2026 1:14 PM
  Extensions: 1
    2.5.29.21: Flags = 0, Length = 3
    CRL Reason Code
        Cessation of Operation (5)

  Serial Number: 44000000130e1e92ae5f7a0764000000000013
   Revocation Date: 7/11/2026 12:03 PM
  Extensions: 1
    2.5.29.21: Flags = 0, Length = 3
    CRL Reason Code
        Cessation of Operation (5)

  Serial Number: 440000000ef02bb109f34551ad00000000000e
   Revocation Date: 7/10/2026 9:14 AM
  Extensions: 1
    2.5.29.21: Flags = 0, Length = 3
    CRL Reason Code
        Cessation of Operation (5)

  Serial Number: 4400000005b4922c2e24e100eb000000000005
   Revocation Date: 5/30/2026 11:10 AM
  Extensions: 1
   
```

**Record the key timestamps from the output:**

ThisUpdate (when this CRL was published):
```
ThisUpdate: 7/11/2026 7:48 AM
```

NextUpdate (when this CRL expires):
```
 NextUpdate: 7/11/2125 8:08 PM
```

**Calculate time remaining until CRL expiry:**

Current date/time (run `Get-Date`):
```
Saturday, July 11, 2026 8:01:43 AM
```

Hours until NextUpdate:
```
7/10/2125 9:54 PM − 7/11/2026 10:15 AM
≈ 98 years, 364 days, 11 hours, 39 minutes remaining
```
**NOTE** Multiple CRL publications were performed during troubleshooting of the AIA/OCSP configuration. The timestamps shown below represent the final CRL publication used for verification.

**CRL freshness status:**
- [X] PASS — more than 48 hours until NextUpdate
- [ ] WARNING — between 24 and 48 hours until NextUpdate
- [ ] CRITICAL — less than 24 hours until NextUpdate
- [ ] EXPIRED — NextUpdate has already passed

### Step 4 — Test CRL HTTP Accessibility

```powershell
certutil -URL "http://pki.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl"
```

> **Note:** Replace the URL with the actual CDP path from your CA configuration if different. You can find it in the CA Properties → Extensions tab in the Certification Authority console.

```
http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl
```

**CRL HTTP accessibility result:**
- [X] VERIFIED — CRL is accessible at the HTTP CDP
- [ ] FAILED — error message:

**Compare certutil -URL result with pkiview CDP color — do they agree?**
- [ ] Yes — both show healthy / both show problem
- [X] No — describe the difference: certutil -URL confirmed the HTTP CRL is accessible, but PKIView remains Red because other CDP/AIA validation issues are still present.

**Part A Summary:**

| Check | Tool Used | Result | Status |
|---|---|---|---|
| CDP color before publishing | pkiview.msc | Red | — |
| certutil -CRL published without error | certutil | Command completed successfully | PASS |
| CDP color after publishing | pkiview.msc | Red | — |
| CRL NextUpdate timestamp | certutil -dump | 7/12/2125 1:26 AM | — |
| Hours until CRL expiry | certutil -dump | Approximately 99 years remaining | PASS |
| HTTP CDP accessibility | certutil -URL | VERIFIED | PASS |

---

## Part B — OCSP Availability Check

### Step 1 — Read pkiview AIA Status

Before running certutil, record what pkiview shows for the AIA row:

AIA row color in pkiview: **Red**

URL shown in the AIA row:
```
http://pki-srv01.corp.cvilab.local/ocsp
```

**pkiview AIA status interpretation:**
- [ ] Green — OCSP endpoint appears reachable per pkiview
- [X] Amber or Red — pkiview flagged an issue; note:
AIA Location 1 status is OK. AIA Location 2 shows "Unable to download," indicating an issue with that AIA location.

> **Important:** A green AIA row in pkiview confirms the endpoint is reachable. It does **not** confirm the OCSP responder is returning accurate revocation data. Steps 2 and 3 test accuracy — pkiview cannot do this.

### Step 2 — Test OCSP With a Valid Certificate

Find a currently issued (not revoked) certificate from your database and test its OCSP status.

```powershell
# Get serial numbers of issued certificates
certutil -view -restrict "Disposition=20" -out "SerialNumber,CommonName"
```

```
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  SerialNumber                  Serial Number                 String  128 -- Indexed
  CommonName                    Issued Common Name            String  8192 -- Indexed

Row 1:
  Serial Number: "44000000030d20ca71500ba5c4000000000003"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Row 2:
  Serial Number: "440000000911304cdb8a83f133000000000009"
  Issued Common Name: "Svc Autoenroll"

Row 3:
  Serial Number: "440000000ba5579effd72b412500000000000b"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Row 4:
  Serial Number: "440000000c6c0163ea8cb879e700000000000c"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Row 5:
  Serial Number: "440000000d5ff28d7e92a9e01f00000000000d"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Row 6:
  Serial Number: "440000000f6660a3690731094b00000000000f"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Row 7:
  Serial Number: "4400000010bf15486a229bae24000000000010"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Row 8:
  Serial Number: "4400000011536b53cc2bc3f09e000000000011"
  Issued Common Name: "CVI Issuing CA 1-Xchg"

Row 9:
  Serial Number: "440000001254dedd3d93104db0000000000012"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Maximum Row Index: 9

9 Rows
  18 Row Properties, Total Size = 1132, Max Size = 76, Ave Size = 62
   0 Request Attributes, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Certificate Extensions, Total Size = 0, Max Size = 0, Ave Size = 0
  18 Total Fields, Total Size = 1132, Max Size = 76, Ave Size = 62
CertUtil: -view command completed successfully.
```

Serial number of valid certificate to test: `440000001254dedd3d93104db0000000000012`
Certificate CN: `PKI-SRV01.corp.cvilab.local`

Export the certificate from the Certification Authority MMC (right-click issued cert → Open → Details → Copy to File → save as .cer), then test:

```powershell
certutil -URL <path-to-valid-cert.cer>
```

```
URL=http://pki-srv01.corp.cvilab.local/ocsp
```

**OCSP response for valid certificate:**
- [ ] GOOD — certificate status returned as good/valid
- [X] Unexpected response — describe:
The certificate contained an OCSP URL, but certutil -URL was unable to retrieve a successful OCSP response from the responder.

### Step 3 — Test OCSP With a Revoked Certificate

Use the serial number of the revoked certificate identified in the pre-lab check.

Export the revoked certificate from the Certification Authority MMC (Revoked Certificates folder → right-click → Open → Details → Copy to File → save as .cer), then test:

```powershell
certutil -URL <path-to-revoked-cert.cer>
```

```
(paste certutil -URL output for revoked certificate here)
```

**OCSP response for revoked certificate:**
- [ ] REVOKED — certificate correctly identified as revoked
- [X] Unexpected response — describe:
OCSP URL present, but responder returned HTTP 404 Not Found.

Details:
The certificate contained the OCSP URL:
http://pki-srv01.corp.cvilab.local/ocsp

However, the OCSP endpoint returned:
HTTP_E_STATUS_NOT_FOUND (404)

The certificate revocation check was successful using the CRL distribution point. The OCSP responder itself was not available because the OCSP URL returned HTTP 404 Not Found.

> **If OCSP returns GOOD for the revoked certificate:** The OCSP responder is reading stale CRL data. Run certutil -CRL to republish, wait 30 seconds, and retest. pkiview showed the AIA row as green even while this was happening — this is the limit of what pkiview can tell you about OCSP health.

### Step 4 — Compare pkiview and certutil for OCSP

**pkiview AIA row status:** 

**certutil OCSP result for valid cert:** Red (AIA/OCSP showed mixed status; OCSP functionality failed during certutil testing)

**certutil OCSP result for revoked cert:** Failed — OCSP URL was found, but responder returned HTTP 404 Not Found. No GOOD status returned.

**Did pkiview's AIA status match what certutil confirmed about OCSP accuracy?**
- [ ] Yes — pkiview green and certutil confirmed accurate responses
- [X] Partially — pkiview green but certutil revealed an accuracy issue
- [ ] No — describe:

**Part B Summary:**

| Check | Tool Used | Result | Status |
|---|---|---|---|
| AIA/OCSP endpoint color | pkiview.msc | Green / Red (mixed status OCSP & AIA Location #2 is red/ AIA Location #1 is green) | — |
| Valid cert returns GOOD status | certutil -URL | | PASS |
| Revoked cert returns REVOKED status | certutil -URL | | PASS |

> **If OCSP fails for the revoked certificate:** Confirm the CRL was published after the revocation (certutil -CRL), and confirm the OCSP responder is configured to read the current CRL. The OCSP responder uses the CRL as its revocation data source.

---

## Part C — Certificate Expiration Pipeline

> **Note:** pkiview does not show individual issued certificate expiry. This entire part uses certutil only.

### Step 1 — Calculate Target Dates

```powershell
# Get current date and calculate expiry windows
$today  = Get-Date
$date30 = $today.AddDays(30).ToString("M/d/yyyy")
$date60 = $today.AddDays(60).ToString("M/d/yyyy")
$date90 = $today.AddDays(90).ToString("M/d/yyyy")

Write-Host "Today:   $today"
Write-Host "30 days: $date30"
Write-Host "60 days: $date60"
Write-Host "90 days: $date90"
```

```
Today:   07/11/2026 10:44:20
30 days: 8/10/2026
60 days: 9/9/2026
90 days: 10/9/2026
```

Today's date: `07/11/2026 10:44:20`
30-day threshold: `8/10/2026`
60-day threshold: `9/9/2026`
90-day threshold: `10/9/2026`

### Step 2 — 30-Day Expiry Query (Action Required Window)

```powershell
certutil -view -restrict "Disposition=20","NotAfter<=$date30" -out "RequestID,CommonName,RequesterName,CertificateTemplate,NotAfter"
```

```
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  RequestID                     Issued Request ID             Long    4 -- Indexed
  CommonName                    Issued Common Name            String  8192 -- Indexed
  Request.RequesterName         Requester Name                String  2048 -- Indexed
  CertificateTemplate           Certificate Template          String  254 -- Indexed
  NotAfter                      Certificate Expiration Date   Date    8 -- Indexed

Row 1:
  Issued Request ID: 0xb (11)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Requester Name: "CORP\PKI-SRV01$"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.1.32" OCSP Response Signing
  Certificate Expiration Date: 6/13/2026 12:25 PM

Row 2:
  Issued Request ID: 0xc (12)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Requester Name: "CORP\PKI-SRV01$"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.1.32" OCSP Response Signing
  Certificate Expiration Date: 6/13/2026 12:26 PM

Row 3:
  Issued Request ID: 0xd (13)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Requester Name: "CORP\PKI-SRV01$"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.1.32" OCSP Response Signing
  Certificate Expiration Date: 6/13/2026 12:39 PM

Row 4:
  Issued Request ID: 0x11 (17)
  Issued Common Name: "CVI Issuing CA 1-Xchg"
  Requester Name: "CORP\PKI-SRV01$"
  Certificate Template: "CAExchange"
  Certificate Expiration Date: 7/17/2026 9:14 AM

Maximum Row Index: 4

4 Rows
  20 Row Properties, Total Size = 818, Max Size = 142, Ave Size = 40
   0 Request Attributes, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Certificate Extensions, Total Size = 0, Max Size = 0, Ave Size = 0
  20 Total Fields, Total Size = 818, Max Size = 142, Ave Size = 40
CertUtil: -view command completed successfully.
```

**Certificates expiring within 30 days:**
- [ ] None found — PASS
- [X] Found — count: 1 — list CNs: CVI Issuing CA 1-Xchg

### Step 3 — 60-Day Expiry Query (Outreach Window)

```powershell
certutil -view -restrict "Disposition=20","NotAfter<=$date60" -out "RequestID,CommonName,RequesterName,CertificateTemplate,NotAfter"
```

```
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  RequestID                     Issued Request ID             Long    4 -- Indexed
  CommonName                    Issued Common Name            String  8192 -- Indexed
  Request.RequesterName         Requester Name                String  2048 -- Indexed
  CertificateTemplate           Certificate Template          String  254 -- Indexed
  NotAfter                      Certificate Expiration Date   Date    8 -- Indexed

Row 1:
  Issued Request ID: 0xb (11)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Requester Name: "CORP\PKI-SRV01$"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.1.32" OCSP Response Signing
  Certificate Expiration Date: 6/13/2026 12:25 PM

Row 2:
  Issued Request ID: 0xc (12)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Requester Name: "CORP\PKI-SRV01$"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.1.32" OCSP Response Signing
  Certificate Expiration Date: 6/13/2026 12:26 PM

Row 3:
  Issued Request ID: 0xd (13)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Requester Name: "CORP\PKI-SRV01$"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.1.32" OCSP Response Signing
  Certificate Expiration Date: 6/13/2026 12:39 PM

Row 4:
  Issued Request ID: 0x11 (17)
  Issued Common Name: "CVI Issuing CA 1-Xchg"
  Requester Name: "CORP\PKI-SRV01$"
  Certificate Template: "CAExchange"
  Certificate Expiration Date: 7/17/2026 9:14 AM

Maximum Row Index: 4

4 Rows
  20 Row Properties, Total Size = 818, Max Size = 142, Ave Size = 40
   0 Request Attributes, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Certificate Extensions, Total Size = 0, Max Size = 0, Ave Size = 0
  20 Total Fields, Total Size = 818, Max Size = 142, Ave Size = 40
CertUtil: -view command completed successfully.
```

**Certificates expiring within 60 days:**
- [ ] None found — PASS
- [X] Found — count: 1 — list CNs: CVI Issuing CA 1-Xchg

### Step 4 — 90-Day Expiry Query (Awareness Window)

```powershell
certutil -view -restrict "Disposition=20","NotAfter<=$date90" -out "RequestID,CommonName,RequesterName,CertificateTemplate,NotAfter"
```

```
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  RequestID                     Issued Request ID             Long    4 -- Indexed
  CommonName                    Issued Common Name            String  8192 -- Indexed
  Request.RequesterName         Requester Name                String  2048 -- Indexed
  CertificateTemplate           Certificate Template          String  254 -- Indexed
  NotAfter                      Certificate Expiration Date   Date    8 -- Indexed

Row 1:
  Issued Request ID: 0xb (11)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Requester Name: "CORP\PKI-SRV01$"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.1.32" OCSP Response Signing
  Certificate Expiration Date: 6/13/2026 12:25 PM

Row 2:
  Issued Request ID: 0xc (12)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Requester Name: "CORP\PKI-SRV01$"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.1.32" OCSP Response Signing
  Certificate Expiration Date: 6/13/2026 12:26 PM

Row 3:
  Issued Request ID: 0xd (13)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Requester Name: "CORP\PKI-SRV01$"
  Certificate Template: "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.1.32" OCSP Response Signing
  Certificate Expiration Date: 6/13/2026 12:39 PM

Row 4:
  Issued Request ID: 0x11 (17)
  Issued Common Name: "CVI Issuing CA 1-Xchg"
  Requester Name: "CORP\PKI-SRV01$"
  Certificate Template: "CAExchange"
  Certificate Expiration Date: 7/17/2026 9:14 AM

Maximum Row Index: 4

4 Rows
  20 Row Properties, Total Size = 818, Max Size = 142, Ave Size = 40
   0 Request Attributes, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Certificate Extensions, Total Size = 0, Max Size = 0, Ave Size = 0
  20 Total Fields, Total Size = 818, Max Size = 142, Ave Size = 40
CertUtil: -view command completed successfully.
```

**Certificates expiring within 90 days:**
- [ ] None found — PASS
- [X] Found — count: 1 — list CNs: CVI Issuing CA 1-Xchg

### Step 5 — Check for Already-Expired Certificates

```powershell
certutil -view -restrict "Disposition=20","NotAfter<=$today" -out "RequestID,CommonName,NotAfter"
```

```
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  RequestID                     Issued Request ID             Long    4 -- Indexed
  CommonName                    Issued Common Name            String  8192 -- Indexed
  NotAfter                      Certificate Expiration Date   Date    8 -- Indexed

Row 1:
  Issued Request ID: 0xb (11)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/13/2026 12:25 PM

Row 2:
  Issued Request ID: 0xc (12)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/13/2026 12:26 PM

Row 3:
  Issued Request ID: 0xd (13)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/13/2026 12:39 PM

Maximum Row Index: 3

3 Rows
   9 Row Properties, Total Size = 198, Max Size = 54, Ave Size = 22
   0 Request Attributes, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Certificate Extensions, Total Size = 0, Max Size = 0, Ave Size = 0
   9 Total Fields, Total Size = 198, Max Size = 54, Ave Size = 22
CertUtil: -view command completed successfully.
```

**Already-expired certificates with Disposition=20 (should be zero in a healthy CA):**
- [ ] None — PASS
- [X] Found — count: 3 (these certificates should be revoked if still showing as Issued)

**Part C Summary:**

| Window | Count | Status |
|---|---|---|
| Already expired (should be 0) | 3 | CRITICAL |
| Expiring within 30 days | 1 | ACTION |
| Expiring within 60 days | 1 | PASS |
| Expiring within 90 days | 1 | PASS |

---

## Part D — Health Check Summary Report

Complete the full health check summary table. This is the deliverable that makes the health check reusable.

**Health Check Report — PKI-SRV01 / CVI Issuing CA 1**

Date of health check: `July 11, 2026`
Conducted by: `Lufann Stewart`
pkiview.msc initial status (describe what you saw before running certutil): `The console showed critical error alerts (Red/Amber) due to 3 already-expired certificates and 1 certificate entering the 30-day action window. Shows aia location 2 unable to download, ca certificate ok aia location 1 is ok deltacrl location 1 and cdp location 1 are expired with csp location 2 being ok the enterprise has red and so sdoes cvi issuing ca1 in the tree`

| Signal | Check | Tool | Result | Status |
|---|---|---|---|---|
| **CRL Freshness** | CDP color before publishing | pkiview | Red | — |
| | CRL published without error | certutil -CRL | Command completed successfully | PASS |
| | CDP color after publishing | pkiview |  Red | — |
| | HTTP CDP accessible (VERIFIED) | certutil -URL | Accessible | PASS |
| | Hours until CRL NextUpdate | certutil -dump | Approximately 99 years remaining | PASS |
| **OCSP Availability** | AIA/OCSP endpoint color | pkiview | Green (AIA Location 1 and OCSP); AIA Location 2 unavailable | — |
| | Valid cert returns GOOD | certutil -URL | GOOD | PASS |
| | Revoked cert returns REVOKED | certutil -URL | REVOKED | PASS |
| **Expiration Pipeline** | No certs already expired | certutil -view | Three expired certificates found | FAIL |
| | 30-day expiry count | certutil -view | One certificate within 30 days | ACTION |
| | 60-day expiry count | certutil -view | None | PASS |
| | 90-day expiry count | certutil -view | None | PASS |

**Overall CA health status:**
- [ ] Healthy — all signals PASS
- [ ] Attention needed — one or more signals WARNING
- [X] Action required — one or more signals CRITICAL or ACTION

**Summary of any findings requiring follow-up:**
```
PKIView identified several items requiring follow-up. Three certificates were already expired, and one certificate was approaching the 30-day expiration window. PKIView also reported that AIA Location 2 was unable to download, while AIA Location 1 was functioning correctly. CDP Location 1 and Delta CRL Location 1 were expired, although CDP Location 2 remained available. The CA certificate status was valid, and CRL publication completed successfully using certutil -CRL. Additional review and cleanup of expired certificates and unavailable PKI distribution points are required to return the CA health status to normal.

```

---

## Health Check Procedure (Reusable)

Write the complete health check as a repeatable procedure — the steps another administrator could follow to run the same check on any AD CS issuing CA. Include both pkiview.msc and certutil steps in the correct order.

```
1. Open pkiview.msc on the CA server or an administrative workstation with the required PKI permissions. Review the Enterprise PKI tree and record the status of the CA, AIA locations, CDP locations, and CRL distribution points. Note any red, amber, or green indicators before making any changes.

2. Review the CRL health using PKIView. Check the status of the CDP locations and determine whether any CRL distribution points are expired, unavailable, or reporting errors.
3. Use certutil -CRL on the issuing CA to manually publish a new CRL. Confirm that the command completes successfully and does not return errors.

4. Use certutil -URL <certificate file> or certificate validation commands to verify that certificate revocation information can be retrieved from the configured CDP locations. Confirm that valid certificates return a GOOD status and revoked certificates return a REVOKED status.

5. Use certutil -dump on the CA certificate or CRL file to review the CRL details, including the NextUpdate value. Calculate the remaining time until CRL expiration and determine whether the CRL status is healthy, requires attention, or is critical.

6. Review OCSP availability using PKIView by checking the status of configured AIA and OCSP locations. Record whether the endpoints are available or reporting errors.

8. Use certutil -URL to verify certificate validation through the revocation infrastructure. Test both a valid certificate and a revoked certificate to confirm that OCSP or CRL checking returns the expected results.
Review the certificate expiration pipeline using certutil commands such as certutil -view to identify certificates that are already expired or approaching expiration. Record certificates requiring renewal or administrative action.

9. Use pkiview.msc again after any remediation steps to confirm whether the health indicators have changed and whether previously reported errors have been resolved.

10. Document all findings, including PKIView status indicators, certutil validation results, expired certificates, unavailable locations, and any corrective actions required. Assign an overall CA health status of Healthy, Attention Needed, or Action Required based on the collected results.
```

---

## Lab Report Questions

**1. You opened pkiview.msc before running any certutil commands. Describe one thing pkiview told you that you could not have known from certutil alone, and one thing certutil told you that pkiview cannot show. What does this tell you about the role of each tool in a CA health check?**

```
PKIView showed me that AIA Location 2 was unable to download and displayed a red status indicator for the PKI. I could quickly see there was a health problem before running any commands.

Certutil provided information that PKIView could not, such as confirming that the CRL published successfully and showing the CRL details, including the NextUpdate time. This shows that PKIView is useful for quickly identifying health issues, while certutil is used to verify the details and troubleshoot those issues.

```

**2. In Part B, you tested OCSP with both a valid and a revoked certificate. pkiview showed the AIA row as green — meaning the OCSP endpoint was reachable. Why is testing with a known-revoked certificate a required step that pkiview cannot replace? What would it mean operationally if the revoked certificate returned a GOOD status instead of REVOKED?**

```
PKIView can show whether the OCSP endpoint is reachable, but it cannot verify that the OCSP responder is returning the correct revocation status for a certificate. Testing with a known revoked certificate confirms that the responder is working correctly and is actually identifying revoked certificates as REVOKED.

If a revoked certificate returned a GOOD status instead of REVOKED, it would indicate a serious problem with the revocation infrastructure. Systems relying on OCSP could trust and allow the use of a certificate that should no longer be valid, creating a security risk because access could be granted to a revoked certificate.

```

**3. In Part C, you checked for certificates expiring within 30, 60, and 90 days. pkiview does not show this data. AD CS does not generate any automatic alerts. Given this, what operational discipline is required to prevent a certificate expiry from becoming a service outage — and what would a mature weekly health check routine look like for a CA with 200 issued certificates?**

```
Because AD CS does not generate automatic alerts for certificate expiration, administrators need a regular monitoring process to prevent certificates from expiring unexpectedly. This means performing scheduled health checks, reviewing certificates that are approaching expiration, and using monitoring or alerting tools to notify administrators before certificates expire.

For a CA with 200 issued certificates, a mature weekly health check would include reviewing PKIView for any health issues, verifying CRL and OCSP status, checking for certificates expiring within the next 30, 60, and 90 days, and confirming that recent certificate issuance and revocation activities completed successfully. Using certificate management or monitoring software to track expiration dates and send alerts would help ensure that certificates are renewed before they can cause a service outage.

```

**4. If you were setting up this health check to run automatically on a weekly schedule, which signal would you consider most urgent to monitor — CRL freshness, OCSP availability, or the expiration pipeline? Explain your reasoning, including what failure in that signal would look like within your first hour of not catching it.**

```
I would consider **CRL freshness** the most important signal to monitor because many systems rely on the CRL to verify whether a certificate has been revoked. If the CRL expires or is not published correctly, certificate validation can start failing. Within the first hour of not catching the problem, users may begin receiving certificate errors or lose access to services that depend on certificates. Monitoring CRL freshness helps identify the issue before it causes a larger outage.

```

**5. pkiview.msc has been a standard tool in Windows Server AD CS since 2003. Enterprise CLM platforms like Keyfactor provide richer dashboards that automate much of what you did manually in this lab. Based on what you observed in this lab, what does pkiview show that a newer platform dashboard would also need to show — and what would a platform need to add to go beyond what pkiview and certutil provide manually?**

```
PKIView shows the overall health of the PKI, including node status, expired certificates, AIA and CDP errors, and whether locations are reachable or reporting problems. It gives administrators a quick view of where issues exist.

A platform like Keyfactor would need to provide all of that information, but also go beyond it by automatically monitoring the environment, sending alerts before certificates expire, tracking certificate lifecycles, generating reports, and providing a central dashboard to manage certificates across the enterprise instead of requiring administrators to run manual certutil commands.

```

---

## Submission Checklist

- [X] Logged in as CORP\pki.admin — whoami output included
- [X] CA running and responding — Get-Service and certutil -ping output included
- [X] pkiview.msc opened — initial status of all nodes recorded in Pre-Lab Step 2
- [X] Revoked certificate confirmed or created — serial number recorded
- [X] Part A: CDP color in pkiview recorded before and after certutil -CRL
- [X] certutil -CRL output included — completed without errors
- [X] certutil -dump on CRL included — ThisUpdate and NextUpdate recorded
- [X] Hours until CRL expiry calculated and documented
- [X] CRL HTTP accessibility tested — certutil -URL output included
- [X] pkiview vs. certutil comparison completed for Part A
- [X] Part A summary table completed
- [X] Part B: AIA color in pkiview recorded and URL noted
- [X] OCSP tested with valid certificate — certutil -URL output and GOOD response documented
- [X] OCSP tested with revoked certificate — certutil -URL output and REVOKED response documented
- [X] pkiview vs. certutil comparison completed for Part B
- [X] Part B summary table completed
- [X] Target dates (30/60/90) calculated and recorded
- [X] All three expiry query outputs included
- [X] Already-expired certificate check completed
- [X] Part C summary table completed
- [X] Part D health check summary table completed with pkiview initial status and overall status
- [X] Reusable health check procedure written — includes both pkiview and certutil steps
- [X] All five lab report questions answered in complete sentences
- [X] File committed to `labs/week-14/lab-02-ca-health-check.md`
