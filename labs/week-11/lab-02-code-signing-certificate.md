# Lab 02: Issue a Code Signing Certificate

Lufann Stewart  
May 24, 2026    
**Phase:** 2 | **Week:** 11  
**Submission Path:** `labs/week-11/lab-02-code-signing-certificate.md`

---

## Pre-Lab Verification

If you can log into PKI-SRV01 as **CORP\pki.admin**, you are communicating with DC01 and the environment is ready. Proceed to Part A.

---

## Part A — Design the CVI-CodeSigning Template

### Step 1 — Open the Certificate Templates Console

1. Press **Win + R**, type `certtmpl.msc`, and press **Enter**
2. The Certificate Templates console opens showing all templates installed on this CA

### Step 2 — Duplicate the Code Signing Template

1. Scroll through the list to find the built-in **Code Signing** template
2. Right-click **Code Signing** → select **Duplicate Template**
3. A new template Properties window opens — this is your working copy

> **Why Code Signing and not User?** The built-in Code Signing template already has the correct Key Usage (Digital Signature only) and EKU (Code Signing only) pre-configured. Starting from User would require removing multiple EKUs and introduces the risk of leaving incorrect settings in place.

**Source template duplicated:** Code Signing

### Step 3 — Set Compatibility Settings

1. Click the **Compatibility** tab
2. Set **Certification Authority** to: `Windows Server 2012 R2`
3. Set **Certificate Recipient** to: `Windows 8.1 / Windows Server 2012 R2`
4. Click **OK** on any informational dialog that appears

**Compatibility settings:**
- Certification Authority: Windows Server 2012 R2
- Certificate Recipient: Windows 8.1 / Windows Server 2012 R2

### Step 4 — Set the Template Name (General Tab)

1. Click the **General** tab
2. Change **Template display name** to: `CVI Code Signing`
3. Confirm the **Template name** (internal) auto-fills as `CVI-CodeSigning`

**Template names:**

| Field | Value |
|-------|-------|
| Template display name | CVI Code Signing |
| Template name (internal) | CVI-CodeSigning |

### Step 5 — Configure Key Usage

1. Click the **Extensions** tab
2. Select **Key Usage** in the list → click **Edit**
3. Confirm only **Digital Signature** is checked. Uncheck anything else if present
4. Check **Make this extension critical**
5. Click **OK**

| Key Usage | Included? | Reason |
|-----------|-----------|--------|
| Digital Signature |Yes |Used to digitally sign code so systems can verify the authenticity and integrity of the signed file. |
| Key Encipherment | No  |Not related to code signing as this is used in TLS to protect sesssion keys |
| Non-Repudiation |No |For a code signing certificate, the Digital Signature key usage is sufficient because it enables the cryptographic signing operation required for Authenticode. The Code Signing EKU already restricts the certificate’s intended purpose to code signing, so Non-Repudiation is not strictly necessary. |
**Explanation of Key Usage decision:**

```
Digital Signature is the only Key Usage needed because code signing is just about creating and verifying a cryptographic signature on the file. That signature proves the code hasn’t been changed and confirms who published it. The other Key Usages deal with things like encryption or secure communication, which aren’t part of the code signing process at all.
```

### Step 6 — Configure Extended Key Usage (Application Policies)

1. Still on the **Extensions** tab, select **Application Policies** → click **Edit**
2. Confirm only **Code Signing (1.3.6.1.5.5.7.3.3)** is listed
3. If any other EKUs are present, select them and click **Remove**
4. Click **OK**

| EKU | Included? | Reason |
|-----|-----------|--------|
| Code Signing (1.3.6.1.5.5.7.3.3) | Yes|Required to authorize the certificate for software and script signing operations. |
| Client Authentication |No|Not required because the certificate is not intended for user or device authentication.|
| Other |No | The Code Signing EKU restricts the certificate’s intended purpose to software and script signing operations. No additional EKUs were added because the certificate is not intended for client authentication, server authentication, email encryption, or other PKI functions. Limiting the EKUs reduces unnecessary trust exposure and follows the principle of least privilege.|

**Explanation of EKU decision:**

```
The Code Signing EKU restricts the certificate’s intended usage to software and script signing operations. No additional EKUs were added because the certificate is not intended for client authentication, server authentication, email protection, or other PKI purposes. Restricting the EKUs reduces unnecessary trust exposure and follows the principle of least privilege.
```

### Step 7 — Configure Subject Name

1. Click the **Subject Name** tab
2. Select **Build from this Active Directory information**
3. Under Subject name format, select **User principal name (UPN)**

| Setting | Value | Reason |
|---------|-------|--------|
| Subject name format |User Principal Name (UPN) |Used to build subject identity from AD; in this lab the resulting subject resolves to CN=PKI Admin based on enrollment behavior.|
| Subject built from |Built from Active Directory | AD provides the identity attributes used to build the subject.|

### Step 8 — Set Validity Period and Enrollment Permissions

**Validity:**
1. Click the **General** tab
2. Set **Validity period** to `1` year (or 2 — document your reasoning)
3. Leave **Renewal period** at the default

**Security / Enrollment Permissions:**
1. Click the **Security** tab
2. Select **Authenticated Users** — confirm Read is checked, Enroll is NOT checked
3. Select **Domain Admins** — confirm Read and Enroll are checked
4. pki.admin should inherit Enroll through Domain Admins. If it does not, click **Add** → type `pki.admin` → Check Names → OK → check **Read** and **Enroll**
5. Do NOT enable Autoenroll — code signing certificates should require a deliberate enrollment action
6. Click **Apply**

| Setting | Value | Reason |
|---------|-------|--------|
| Validity period |1 year |Limits the exposure window if the certificate or private key is compromised and ensures periodic revalidation of trust and enrollment policies.|
| Enroll — account(s) granted |pki.Admin and Domain Admins | Limits certificate enrollment to approved administrative accounts to reduce unnecessary certificate issuance and ensure only trusted users can request code-signing certificates.
| Autoenroll |No |Code signing certificates are high-trust certificates and should require deliberate enrollment rather than automatic issuance. |
### Step 9 — Save the Template

1. Click **OK** to close the Properties window
2. Verify **CVI-CodeSigning** now appears in the certtmpl.msc list

**Template saved:**
- [X] Yes — visible in certtmpl.msc

---

## Part B — Publish and Issue the Certificate

### Step 1 — Publish the Template to the CA

1. Press **Win + R**, type `certsrv.msc`, and press **Enter**
2. Expand **CVI Issuing CA 1** in the left pane
3. Right-click **Certificate Templates** → **New** → **Certificate Template to Issue**
4. Scroll to find **CVI-CodeSigning** → select it → click **OK**
5. The template should appear in the Certificate Templates node within 30 seconds. If it doesn't, right-click the node → **Refresh**

**CVI-CodeSigning visible in Certificate Templates node:**
- [X] Yes

### Step 2 — Request the Certificate (as pki.admin)

1. Press **Win + R**, type `mmc.exe`, and press **Enter**
2. Go to **File → Add/Remove Snap-in**
3. Select **Certificates** → click **Add**
4. Choose **My user account** → click **Finish** → click **OK**
5. In the left pane, expand **Certificates (Current User)** → expand **Personal**

> **Note:** If no certificates have been issued yet, you will not see a **Certificates** folder under Personal — this is expected. Right-click **Personal** directly → **All Tasks** → **Request New Certificate**. Once the certificate is issued, the Certificates folder will appear automatically.

6. Right-click **Personal** → **All Tasks** → **Request New Certificate**
7. Click **Next** through the enrollment wizard
8. Select **Active Directory Enrollment Policy** → click **Next**
9. The **CVI-CodeSigning** template should appear in the list. Check the box next to it
10. Click **Enroll**
11. Enrollment should complete immediately. Click **Finish**

**Certificate issued:**
- [X] Yes — immediately
- [ ] Pending — describe:

```
No issues to report. Template enrollement through the wizard was successful.
```

### Step 3 — Record the Request ID

1. Open **certsrv.msc**
2. Expand **CVI Issuing CA 1** → click **Issued Certificates**
3. Find the pki.admin code signing certificate
4. Record the Request ID below

**Request ID from certsrv.msc Issued Certificates node:** 5

> **Save this Request ID.** It is used in Week 12 revocation and in Lab 03.

### Step 4 — Verify the Certificate

Open **PowerShell** on PKI-SRV01 and run:

```powershell
certutil -user -store My
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
Template: 1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.5191136.12906913
Cert Hash(sha1): 4f2aa3df007c2193e719a7bfe75cf98523636d54
  Key Container = 0e3dbca0a5f48b9e90025e99f66cf711_f0a99c17-76d3-498a-97de-2992c06105fd
  Simple container name: te-CVI-ServiceAccount-1adfbcb8-b530-447b-a47b-e9e5e4e0517c
  Provider = Microsoft Enhanced Cryptographic Provider v1.0
Encryption test passed
CertUtil: -store command completed successfully.

```

Locate the CVI-CodeSigning certificate in the output and confirm the EKU field:

| Field | Value |
|-------|-------|
| Subject |CN=PKI Admin |
| EKU |Code Signing (1.3.6.1.5.5.7.3.3) |
| Validity |‎Sunday, ‎May ‎24, ‎2026 12:53:58 PM to ‎Sunday, ‎April ‎25, ‎2027 7:36:58 PM  |
| Thumbprint |7fa17dbe13f3c0f3ff3e60af8528c4edb48d96ce |

**EKU = 1.3.6.1.5.5.7.3.3 (Code Signing) confirmed:**
- [X] Yes
- [ ] No — describe discrepancy:

---

## Part C — Sign a PowerShell Script

### Step 1 — Create the Test Script

Open **PowerShell** on PKI-SRV01 as CORP\pki.admin and run the following block. Copy and paste it exactly:

```powershell
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

### Step 2 — Retrieve the Code Signing Certificate

Run the following to confirm the certificate is accessible and select it into a variable:

```powershell
$cert = Get-ChildItem -Path Cert:\CurrentUser\My -CodeSigningCert | Select-Object -First 1
$cert | Select-Object Subject, Thumbprint, NotAfter
```

**Output of certificate selection:**

```
Subject      Thumbprint                               NotAfter            
-------      ----------                               --------            
CN=PKI Admin 7FA17DBE13F3C0F3FF3E60AF8528C4EDB48D96CE 4/25/2027 7:36:58 PM
```

> If this returns nothing, the certificate was not issued with the Code Signing EKU. Go back to certtmpl.msc, check the Application Policies on CVI-CodeSigning, and re-enroll.

### Step 3 — Sign the Script

```powershell
$result = Set-AuthenticodeSignature -FilePath "C:\Scripts\Test-CVI.ps1" -Certificate $cert
$result
```

**Set-AuthenticodeSignature output:**

```
Directory: C:\Scripts


SignerCertificate                         Status                                                                                                      Path                                                                                                       
-----------------                         ------                                                                                                      ----                                                                                                       
7FA17DBE13F3C0F3FF3E60AF8528C4EDB48D96CE  Valid                                                                                                       Test-CVI.ps1   
```

Expected result: **Status = Valid**

### Step 4 — Verify the Signature

```powershell
Get-AuthenticodeSignature -FilePath "C:\Scripts\Test-CVI.ps1"
```

**Full Get-AuthenticodeSignature output:**

```
Directory: C:\Scripts


SignerCertificate                         Status                                                                                                      Path                                                                                                       
-----------------                         ------                                                                                                      ----                                                                                                       
7FA17DBE13F3C0F3FF3E60AF8528C4EDB48D96CE  Valid                                                                                                       Test-CVI.ps1                                                                                               

```

**Status:**
- [X] Valid
- [ ] Other — describe:

### Step 5 — Check for a Timestamp

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

### Step 6 — Hash Mismatch Test

Modify the script after signing to verify the signature breaks:

```powershell
Add-Content -Path "C:\Scripts\Test-CVI.ps1" -Value "# Modified after signing"

Get-AuthenticodeSignature -FilePath "C:\Scripts\Test-CVI.ps1"
```

**Get-AuthenticodeSignature output after modification:**
Test 1 — Append Modification (Breaks signature structure) 
```
Directory: C:\Scripts


SignerCertificate                         Status                                                                                                      Path                                                                                                       
-----------------                         ------                                                                                                      ----                                                                                                       
                                          NotSigned                                                                                                   Test-CVI.ps1
```
Test 2 — In-place Content Modification (Triggers hash mismatch)
```
Directory: C:\Scripts


SignerCertificate                         Status                                                                                                      Path                                                                                                       
-----------------                         ------                                                                                                      ----                                                                                                       
7FA17DBE13F3C0F3FF3E60AF8528C4EDB48D96CE  HashMismatch                                                                                                Test-CVI.ps1                                                                                               


```
**Status after modification:**
- [X] HashMismatch
- [ ] Other — describe:

---

## Part D — Written Explanation

Answer the following in plain prose paragraphs — not bullet points.

**What does the Code Signing EKU enforce, and at what layer?**

Cover: what application or OS component checks for the Code Signing EKU, what it does when the EKU is present vs. absent, and how this is different from the cryptographic validity check.

```
The Code Signing EKU is enforced at the application and operating system trust layer. Applications such as PowerShell, Windows Defender Application Control, and other software validation mechanisms check whether the certificate contains the Code Signing EKU before trusting the certificate for signing operations. If the EKU is missing, the certificate may still be cryptographically valid, but the system will reject it for code signing purposes because it is not authorized for that usage. This differs from cryptographic validation, which checks whether the digital signature itself is mathematically valid and whether the file contents still match the original signed hash.
```

**What did the hash mismatch test demonstrate about what the signature is protecting?**

Cover: what the signature covers (the code hash), what the mismatch status means, and why this matters for software integrity in a production environment.

```
The hash mismatch test showed that a digital signature protects the full integrity of a file’s contents, but the way Windows reports a failure depends on how the file gets changed.

When I used the Add-Content command from the lab, it added text to the very bottom of the script. Since PowerShell scripts store the Authenticode signature at the end of the file, appending anything after it effectively broke the signature structure completely. At that point, Windows couldn’t even interpret a valid signature anymore, so it showed the file as NotSigned instead of HashMismatch.

To see the actual mismatch behavior, I ran a second test where I changed the content inside the script itself without touching the signature block:

(Get-Content "C:\Scripts\Test-CVI.ps1") -replace 'pki.admin', 'tampered.user' | Set-Content "C:\Scripts\Test-CVI.ps1"

In this case, the signature was still readable, but the underlying file content no longer matched the original signed hash. That’s why Windows was able to detect the difference and return HashMismatch.

Both outcomes point to the same security idea: once a file is signed, any change afterward breaks trust. In real environments, that’s what prevents modified or tampered scripts from being executed as if they were legitimate.
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
- [X] Part A: Template duplicated from the built-in Code Signing template
- [X] Part A: All settings configured — Key Usage, EKU, Subject Name, Validity, Enrollment Permissions
- [X] Part A: Template saved as CVI-CodeSigning and visible in certtmpl.msc
- [X] Part B: Template published to CVI Issuing CA 1
- [X] Part B: Certificate issued to pki.admin — Request ID recorded
- [X] Part B: certutil output pasted with EKU confirmed as Code Signing (1.3.6.1.5.5.7.3.3)
- [X] Part C: Test script created at C:\Scripts\Test-CVI.ps1
- [X] Part C: Certificate selection output pasted
- [X] Part C: Set-AuthenticodeSignature output pasted (Status = Valid)
- [X] Part C: Get-AuthenticodeSignature output pasted (Status = Valid)
- [X] Part C: Timestamp check output pasted
- [X] Part C: Hash mismatch test output pasted (Status = HashMismatch)
- [X] Part D: Written explanation completed in prose
- [X] Reflection completed
- [X] File saved as `lab-02-code-signing-certificate.md`
- [X] File committed to portfolio repo under `labs/week-11/`
