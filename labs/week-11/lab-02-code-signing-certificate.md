# Lab 02: Issue a Code Signing Certificate

Lufann Stewart  
May 24, 2026   
**Phase:** 2 | **Week:** 11  
**Submission Path:** `labs/week-11/lab-02-code-signing-certificate.md`

---

## Pre-Lab Verification

Run on PKI-SRV01 before starting.

```powershell
Get-Service -Name CertSvc
certutil -ping
```

**CertSvc status:** Running
**CA responding:** Yes

---

## Part A — Design the CVI-CodeSigning Template

Open the Certificate Templates console: **Run → certtmpl.msc**

### Template Duplication

**Source template duplicated:** Code Signing template

**Compatibility settings selected:**
- Certification Authority: Windows Server 2012 R2
- Certificate Recipient: Windows 8.1 / Windows Server 2012 R2

### Design Decisions

**1 — Key Usage**

| Key Usage | Included? | Reason |
|-----------|-----------|--------|
| Digital Signature |Yes |Used to digitally sign code so systems can verify the authenticity and integrity of the signed file. |
| Key Encipherment | No  |Not related to code signing as this is used in TLS to protect sesssion keys |
| Non-Repudiation |No |For a code signing certificate, the Digital Signature key usage is sufficient because it enables the cryptographic signing operation required for Authenticode. The Code Signing EKU already restricts the certificate’s intended purpose to code signing, so Non-Repudiation is not strictly necessary. |

**Explanation of Key Usage decision:**

```
A code signing certificate primarily requires the Digital Signature key usage because it is used to digitally sign code and allow systems to verify the integrity and authenticity of the signed file. Other key usages such as Key Encipherment are unnecessary because the certificate is not being used for TLS encryption, client authentication, or email protection.
```

**2 — EKU**

| EKU | Included? | Reason |
|-----|-----------|--------|
| Code Signing (1.3.6.1.5.5.7.3.3) | Yes|Required to authorize the certificate for software and script signing operations. |
| Client Authentication |No|Not required because the certificate is not intended for user or device authentication.|
| Other |No | The Code Signing EKU restricts the certificate’s intended purpose to software and script signing operations. No additional EKUs were added because the certificate is not intended for client authentication, server authentication, email encryption, or other PKI functions. Limiting the EKUs reduces unnecessary trust exposure and follows the principle of least privilege.|

**Explanation of EKU decision:**

```
The Code Signing EKU restricts the certificate’s intended usage to software and script signing operations. No additional EKUs were added because the certificate is not intended for client authentication, server authentication, email protection, or other PKI purposes. Restricting the EKUs reduces unnecessary trust exposure and follows the principle of least privilege.
```

**3 — Subject Name**

| Setting | Value | Reason |
|---------|-------|--------|
| Subject name format |Common Name (CN) |Pulls the requester's display name from Active Directory to tie the signature to a specific identity |
| Subject built from |Built form Active Directory | AD provides the identity attributes used to build the subject.|

**4 — Validity and Enrollment Permissions**

| Setting | Value | Reason |
|---------|-------|--------|
| Validity period |1 year |Limits the exposure window if the certificate or private key is compromised and ensures periodic revalidation of trust and enrollment policies.|
| Enroll — account(s) granted |Pki.Admin | This is the account I am using and need to be able to enroll to create and publish the certificates|
| Autoenroll |No |Code signing certificates are high-trust certificates and should require deliberate enrollment rather than automatic issuance. |

**Template names:**

| Field | Value |
|-------|-------|
| Template display name | CVI Code Signing |
| Template name (internal) | CVI-CodeSigning |

**Template saved:**
- [X] Yes — visible in certtmpl.msc

---

## Part B — Publish and Issue the Certificate

### Publish the Template

**Steps performed:**

1. certsrv.msc → CVI Issuing CA 1 → Certificate Templates → New → Certificate Template to Issue
2. Selected **CVI-CodeSigning**

**CVI-CodeSigning visible in Certificate Templates node:**
- [X] Yes

### Request the Certificate (as pki.admin)

**Steps performed:**

1. mmc.exe → Certificates snap-in → **My user account**
2. Personal → All Tasks → Request New Certificate
3. Enrollment policy: Active Directory Enrollment Policy
4. Template selected: **CVI-CodeSigning**
5. Enrolled

**Certificate issued:**
- [X] Yes — immediately
- [ ] Pending — describe:

```
The CVI-CodeSigning certificate was successfully enrolled through Active Directory Certificate Services and is visible in the MMC Certificates console with its expiration date, intended purposes, and associated certificate details.
```

**Request ID from certsrv.msc Issued Certificates node:** 5

> **Save this Request ID.** It is used in Week 12 revocation and in Lab 03.

### Verify the Certificate

```powershell
certutil -store My
```

**Full certutil output for the code signing certificate:**

```
My "Personal"
================ Certificate 0 ================
Serial Number: 4400000005b4922c2e24e100eb000000000005
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
 NotBefore: 5/24/2026 12:53 PM
 NotAfter: 4/25/2027 7:36 PM
Subject: CN=PKI Admin
Non-root Certificate
Template: CVICodeSigning, CVI Code Signing
Cert Hash(sha1): 7fa17dbe13f3c0f3ff3e60af8528c4edb48d96ce
  Key Container = 203caffb941a0d6ca37a704567e68714_f0a99c17-76d3-498a-97de-2992c06105fd
  Simple container name: te-CVICodeSigning-c6a99d8c-20e6-4939-a873-bdddaf747842
  Provider = Microsoft Enhanced Cryptographic Provider v1.0
Private key is NOT exportable
Signature test passed

================ Certificate 1 ================
Serial Number: 4400000004b742b5daba3d4deb000000000004
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
 NotBefore: 5/23/2026 5:29 PM
 NotAfter: 4/25/2027 7:36 PM
Subject: CN=PKI Admin
Non-root Certificate
Template: CVI-ServiceAccount
Cert Hash(sha1): 4f2aa3df007c2193e719a7bfe75cf98523636d54
  Key Container = 0e3dbca0a5f48b9e90025e99f66cf711_f0a99c17-76d3-498a-97de-2992c06105fd
  Simple container name: te-CVI-ServiceAccount-1adfbcb8-b530-447b-a47b-e9e5e4e0517c
  Provider = Microsoft Enhanced Cryptographic Provider v1.0
Encryption test passed
CertUtil: -store command completed successfully.
```

**Key fields confirmed:**

| Field | Value |
|-------|-------|
| Subject | CN=PKI Admin |
| EKU | Code Signing (1.3.6.1.5.5.7.3.3)|
| Validity | 1 year|
| Thumbprint |7fa17dbe13f3c0f3ff3e60af8528c4edb48d96ce |

**EKU = 1.3.6.1.5.5.7.3.3 (Code Signing) confirmed:**
- [X] Yes
- [ ] No — describe discrepancy:

---

## Part C — Sign a PowerShell Script

### Create the Test Script

Create a simple PowerShell script on PKI-SRV01:

```powershell
# Create the test script
$scriptContent = @'
# CVI Phase 2 — Week 11 Code Signing Test
Write-Host "This script is signed with a CVI code signing certificate."
Write-Host "Issued to: pki.admin"
Write-Host "Date: $(Get-Date)"
'@

New-Item -Path "C:\Scripts" -ItemType Directory -Force
Set-Content -Path "C:\Scripts\Test-CVI.ps1" -Value $scriptContent
```

**Script created at C:\Scripts\Test-CVI.ps1:**
- [X] Yes

---

### Sign the Script

```powershell
# Get the code signing certificate
$cert = Get-ChildItem -Path Cert:\CurrentUser\My -CodeSigningCert | Select-Object -First 1

# Confirm this is the right certificate
$cert | Select-Object Subject, Thumbprint, NotAfter
```

**Output of certificate selection:**

```
Subject      Thumbprint                               NotAfter            
-------      ----------                               --------            
CN=PKI Admin 7FA17DBE13F3C0F3FF3E60AF8528C4EDB48D96CE 4/25/2027 7:36:58 PM
```

```powershell
# Sign the script
$result = Set-AuthenticodeSignature -FilePath "C:\Scripts\Test-CVI.ps1" -Certificate $cert
$result
```

**Set-AuthenticodeSignature output:**

```
Directory: C:\Scripts


SignerCertificate                         Status                                                          Path                                                           
-----------------                         ------                                                          ----                                                           
7FA17DBE13F3C0F3FF3E60AF8528C4EDB48D96CE  Valid                                                           Test-CVI.ps1  
```

---

### Verify the Signature

```powershell
Get-AuthenticodeSignature -FilePath "C:\Scripts\Test-CVI.ps1"
```

**Full Get-AuthenticodeSignature output:**

```
 Directory: C:\Scripts


SignerCertificate                         Status                                                          Path                                                           
-----------------                         ------                                                          ----                                                           
7FA17DBE13F3C0F3FF3E60AF8528C4EDB48D96CE  Valid                                                           Test-CVI.ps1                                                   

```

**Status:**
- [X] Valid
- [ ] Other — describe:

**Is a timestamp present?**

```powershell
(Get-AuthenticodeSignature "C:\Scripts\Test-CVI.ps1").TimeStamperCertificate
```

**TimeStamperCertificate output:**

```
$null
```

**Timestamp present:**
- [ ] Yes — note the timestamp authority:
- [X] No — note this in Part D

---

### Hash Mismatch Test (Destructive)

Modify the script after signing to verify the signature breaks:

```powershell
# Add content to the signed script
Add-Content -Path "C:\Scripts\Test-CVI.ps1" -Value "# Modified after signing"

# Re-verify
Get-AuthenticodeSignature -FilePath "C:\Scripts\Test-CVI.ps1"
```

**Get-AuthenticodeSignature output after modification:**

```
Directory: C:\Scripts


SignerCertificate                         Status                                                          Path                                                           
-----------------                         ------                                                          ----                                                           
                                          NotSigned                                                       Test-CVI.ps1                                                   
```

**Status after modification:**
- [ ] HashMismatch
- [X] Other — describe: After modifying the signed script, the digital signature was no longer valid. In this environment, the system reported the file as NotSigned rather than HashMismatch, indicating that the signature validation was broken by the change to the file contents or structure.
---

## Part D — Written Explanation

**What does the Code Signing EKU enforce, and at what layer?**

Cover: what application or OS component checks for the Code Signing EKU, what it does when the EKU is present vs. absent, and how this is different from the cryptographic validity check.

```
The Code Signing EKU is enforced at the application and operating system trust layer. Applications such as PowerShell, Windows Defender Application Control, and other software validation mechanisms check whether the certificate contains the Code Signing EKU before trusting the certificate for signing operations. If the EKU is missing, the certificate may still be cryptographically valid, but the system will reject it for code signing purposes because it is not authorized for that usage. This differs from cryptographic validation, which checks whether the digital signature itself is mathematically valid and whether the file contents still match the original signed hash.
```

**What did the hash mismatch test demonstrate about what the signature is protecting?**

Cover: what the signature covers (the code hash), what the mismatch status means, and why this matters for software integrity in a production environment.

```
The hash mismatch test showed that a digital signature protects the integrity of the file contents. After the script was modified following signing, the system detected that the file no longer matched the original signed state, and the signature was no longer valid. In this lab environment, the result appeared as "NotSigned" instead of a clear "HashMismatch," but the outcome was the same: the file had been altered after signing and could no longer be trusted. This is important in real systems because it prevents modified or tampered scripts from being treated as legitimate software.
```

**Should the CVI-CodeSigning template require CA certificate manager approval in a production environment? Why or why not?**

```
Yes. In a production environment, code signing certificates should require CA certificate manager approval. These are high-trust certificates, and if one is issued to the wrong person, it could be used to sign malicious software that users or systems may trust as legitimate. Requiring certificate manager approval adds an administrative oversight step that helps prevent accidental or unauthorized issuance before code signing capability is granted.
```

---

## Reflection

**Why is a timestamp operationally significant for a code signing certificate — particularly for software that will be distributed and executed over a long period?**

```
A timestamp allows the system to verify that the code was signed while the certificate was still valid. This means the signed software can remain trusted even after the certificate expires, as long as the timestamp confirms it was signed during the valid period.
```

**One thing about the code signing workflow you would want to understand better or configure differently:**

```
One thing I would want to understand better is how to configure and integrate a timestamp authority in a code signing environment. In particular, how timestamping services are set up so that signed code remains valid after a certificate expires, and how systems verify the timestamp during signature validation. I would also like to understand how timestamp authorities are secured and trusted within a production PKI infrastructure.
```

---

## Submission Checklist

- [X] Pre-lab verification completed
- [X] Part A: CVI-CodeSigning template designed with rationale for all settings
- [X] Part A: Template created and visible in certtmpl.msc
- [X] Part B: Template published to CVI Issuing CA 1
- [X] Part B: Certificate issued to pki.admin — Request ID recorded
- [X] Part B: certutil output pasted with EKU confirmed as Code Signing
- [X] Part C: Test script created
- [X] Part C: Script signed and Set-AuthenticodeSignature output pasted
- [X] Part C: Get-AuthenticodeSignature output = Valid, pasted
- [X] Part C: Timestamp check output pasted
- [X] Part C: Hash mismatch test performed and output pasted (Status = HashMismatch) (signature invalidated after modification)
- [X] Part D: Written explanation in prose — EKU enforcement and hash mismatch
- [X] Reflection completed
- [X] File saved as `lab-02-code-signing-certificate.md`
- [X] File committed to portfolio repo under `labs/week-11/`
