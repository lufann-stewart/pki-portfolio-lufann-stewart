# Lab 01: Build a Certificate Profile for a Service Account

Lufann Stewart  
May 23, 2026  
**Phase:** 2 | **Week:** 11  
**Submission Path:** `labs/week-11/lab-01-service-account-profile.md`

---

## Pre-Lab Verification

Run the following on PKI-SRV01 before starting. Do not proceed until all checks pass.

```powershell
# Check 1 — CA service running
Get-Service -Name CertSvc

# Check 2 — CA responding
certutil -ping

# Check 3 — Confirm svc.autoenroll account exists in AD
Get-ADUser -Identity svc.autoenroll -Properties UserPrincipalName | Select-Object Name, UserPrincipalName
```

**All checks passed:**
- [X] Yes
- [ ] No — describe the issue and how you resolved it:

```
No issues to report.
```

---

## Part A — Design the CVI-ServiceAccount Template

Open the Certificate Templates console: **Run → certtmpl.msc**

### Template Duplication

**Source template duplicated:** 
  > User

**Reason for choosing this source template:**

```
Use the User template — not Computer. Here's why: the certificate is being issued to a service account, which is an AD user object (svc.autoenroll). The User template gives you the right starting point for subject name sourcing from AD user attributes. The Computer template is for machine identity (DNS-based subject names).
```

**Compatibility settings selected:**
- Certification Authority: Windows Server 2012 R2
- Certificate Recipient: Windows 8.1 / Windows Server 2012 R2

### Design Decisions

Document every setting with a reason. This is the design record for the template.

**1 — Key Usage**

| Key Usage | Included? | Reason |
|-----------|-----------|--------|
| Digital Signature | Yes |Allows the service account to sign authentication challenges. |
| Key Encipherment |Yes  |Allows the server to securely wrap and exchange keys during the TLS handshake. |
| Data Encipherment | No|This account only authenticates; it does not directly encrypt files at rest.|
| Non-Repudiation |No | Not required for automated service logons. |

**Explanation of Key Usage decisions:**

```
I inculduded Digital Sngature and Key enchphirement becuase they work together this s clent authentication and needs the the singature to verify and use the key enchypirement to securley wrap and echagne keys during tls handsjake.
```

**2 — Extended Key Usage (EKU)**

| EKU | Included? | OID | Reason |
|-----|-----------|-----|--------|
| Client Authentication |Yes | 1.3.6.1.5.5.7.3.2 |Needs to be authenticated |
| Server Authentication |No | 1.3.6.1.5.5.7.3.1 |WIll not be authenticating servers |
| Code Signing | No| 1.3.6.1.5.5.7.3.3 |a service account will not be code signing noe necesary |
| Secure Email |No | 1.3.6.1.5.5.7.3.4 | this is not a email service|

**Explanation of EKU decisions:**

```
This certificate is explicitly designed to authenticate the identity of an automated background service account (svc.autoenroll) to the Active Directory domain. Because its only job is to prove who it is during login, it only requires the Client Authentication EKU. Restricting all other unused EKUs prevents the certificate from being misused elsewhere, strictly following the security principle of least privilege.

```
**3 — Subject Name**

| Setting | Value Selected | Reason |
|---------|---------------|--------|
| Subject name format | Build from AD  |We're not creating a request we dont need to |
| Include this information in the subject name |User Principal Name|Links the certificate directly to its login identifier. |
**Explanation of Subject Name decision:**

```
Building from AD forces the CA to look up the account requesting the certificate directly in Active Directory and pull its official attributes. This prevents a user or a compromised automated script from manually typing in a fake identity (like Administrator), completely blocking privilege escalation attacks.
```

**4 — Validity Period**

| Setting | Value | Reason |
|---------|-------|--------|
| Validity period | 1 Year| year validity period to enrure a shorter renewal period |
| Renewal period |6 weeks | |

**Explanation of validity period decision:**

```
A 1-year validity period keeps certificates rotating regularly, and the 6-week renewal window gives Windows Autoenrollment enough time to replace them before they expire.
```

**5 — Security Tab — Enrollment Permissions**

| Group / Account | Read | Enroll | Autoenroll | Reason |
|-----------------|------|--------|------------|--------|
| Authenticated Users |Yes | NO| NO|Allows domain accounts to see that the template exists, but stops them from requesting it. |
| CORP\svc.autoenroll | Yes|Yes |Yes |This is the specific service account that requires the certificate to run its services. |
| Domain Computers |NO |NO | NO|Admins manage the template infrastructure but do not need to enroll for this specific service cert. |

**Explanation of enrollment permission decisions:**

```
Permissions are configured this way to enforce the principle of least privilege. By restricting "Enroll" and "Autoenroll" strictly to `CORP\svc.autoenroll`, we prevent unauthorized users, administrators, or standard domain computers from requesting this specialized certificate. This eliminates the risk of identity spoofing or privilege escalation within the environment.
```

**General tab — Template names:**

| Field | Value |
|-------|-------|
| Template display name | CVI Service Account |
| Template name (internal) | CVI-ServiceAccount |
| Schema version | |

**Template saved and visible in certtmpl.msc:**
- [X] Yes

---

## Part B — Publish the Template and Issue the Certificate

### Publish the Template

**Steps performed on PKI-SRV01 as CORP\pki.admin:**

1. Opened **certsrv.msc**
2. Expanded **CVI Issuing CA 1** → right-clicked **Certificate Templates** → **New → Certificate Template to Issue**
3. Selected **CVI-ServiceAccount** from the list
4. Clicked **OK**

**CVI-ServiceAccount visible in Certificate Templates node:**
- [X] Yes
- [ ] No — describe what happened:

```
No issues to report.
```

---

### Request the Certificate for svc.autoenroll

**Steps performed:**

1. Opened **mmc.exe** → **File → Add/Remove Snap-in**
2. Added **Certificates** snap-in
3. Selected: **Service account** → **Local computer**
4. Entered service account: ________________
5. Navigated to **Personal → Certificates**
6. Right-clicked → **All Tasks → Request New Certificate**

**Enrollment wizard — enrollment policy selected:**

```
Active Directory Enrollment Policy 
```

**Templates visible in the wizard:**

```
Administrator
Basic EFS
CVI-ServiceAccount
EFS REcovery Agent
User
```

**CVI-ServiceAccount visible in wizard:**
- [X] Yes
- [ ] No — troubleshooting steps taken:

**Certificate request submitted:**
- [X] Yes — issued immediately
- [ ] Yes — pending manager approval (describe resolution):
- [ ] No — error encountered:

```
No issue to report
```

---

## Part C — Verify the Issued Certificate

### Via certutil

Run the following to inspect the service account certificate store:

```powershell
certutil -store -service svc.autoenroll My
```

Or, if the certificate was issued to the personal store under the current user context:

```powershell
certutil -store My
```

**Full certutil output:**

```
svc.autoenroll
CertUtil: -store command FAILED: 0x80070002 (WIN32: 2 ERROR_FILE_NOT_FOUND)
CertUtil: The system cannot find the file specified.

PS C:\Windows\system32> certutil -store My
My "Personal"
================ Certificate 0 ================
Serial Number: 44000000030d20ca71500ba5c4000000000003
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
 NotBefore: 5/22/2026 5:45 PM
 NotAfter: 4/25/2027 7:36 PM
Subject: CN=PKI-SRV01.corp.cvilab.local
Non-root Certificate
Template: CVI-WebServer
Cert Hash(sha1): e0046ea8c9d051f976c47ff60246cf3f488ad4f8
  Key Container = a547eac941e3a6e7ae8e70257435eee5_f0a99c17-76d3-498a-97de-2992c06105fd
  Simple container name: te-CVI-WebServer-f00259a7-e599-4167-965d-0298e0b61c88
  Provider = Microsoft RSA SChannel Cryptographic Provider
Private key is NOT exportable
Encryption test passed

================ Certificate 1 ================
Serial Number: 5800000002f7714edc7f317c46000000000002
Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
 NotBefore: 4/25/2026 7:26 PM
 NotAfter: 4/25/2027 7:36 PM
Subject: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
CA Version: V0.0
Certificate Template Name (Certificate Type): SubCA
Non-root Certificate
Template: SubCA, Subordinate Certification Authority
Cert Hash(sha1): 5137a597de2c3085ec5816c7f11edc18cfcdbaf8
  Key Container = CVI Issuing CA 1
  Unique container name: b52f658bb3f263e6f529f3a0187c63bc_f0a99c17-76d3-498a-97de-2992c06105fd
  Provider = Microsoft Software Key Storage Provider
Signature test passed
CertUtil: -store command completed successfully.
```

**From the certutil output — record the following:**

| Field | Value |
|-------|-------|
| Subject |PKI Admin
CA Version: V0.0 |
| Issuer |CN=CVI Root CA, DC=corp, DC=cvilab, DC=local |
| Serial Number |4400000004b742b5daba3d4deb000000000004 |
| Key Usage |Digital Signature, Key Encipherment |
| Enhanced Key Usage (EKU) |Client Authentication (1.3.6.1.5.5.7.3.2) |
| Validity: Not Before |5/22/2026 5:45 PM |
| Validity: Not After | 4/25/2027 7:36 PM |
| Thumbprint |4f2aa3df007c2193e719a7bfe75cf98523636d54 |

---

### In certsrv.msc — Issued Certificates Node

Navigate to **certsrv.msc → CVI Issuing CA 1 → Issued Certificates**.

**Certificate visible in Issued Certificates node:**
- [X] Yes

**Record from the Issued Certificates node:**

| Column | Value |
|--------|-------|
| Request ID | 4|
| Requester Name |CORP/pki.admin |
| Certificate Template |CVI-ServiceAccount |
| Issued Common Name |PKI-SRV01.corp.cvilab.local |
| Certificate Expiration Date |4/25/2027 7:36 PM |

> **Save this Request ID.** You will use it in Week 12 if you revoke this certificate, and in Lab 03 for the comparison exercise.

---

## Part D — Written Explanation

**What makes a service account certificate different from a user certificate? Address the following:**

1. Key difference in EKU — what does a user certificate include that the service account certificate should not, and why? 
2. Subject Name source — both use "Build from AD," but what is different about the identity being represented?
3. Enrollment — who requests a user certificate vs. who requests a service account certificate, and why does this matter? 

```
EFS and Secure Email Service Account is used for autpmated tasks this is not a user account that ould be sued for sendinf emails or accesing efs for an automated task service and for a user account a user cert
```

**What are the operational risks of relying on password authentication for service accounts instead of certificate-based authentication?**

```
Cant veryify who is is anyone can fge the password with certbased it has to check the cahin of trust make sure that ithe requests is who it says is is a
```

---

## Reflection

**One thing about the CVI-ServiceAccount template design that was a non-obvious decision:**

```
(your observation here)
```

**What would you change about this template if this were a production environment rather than a lab?**

```
(your answer here — think about approval workflow, validity period, or monitoring)
```

---

## Submission Checklist

- [ ] Pre-lab verification completed and outputs recorded
- [ ] Part A: All five template design decisions documented with rationale
- [ ] Part A: Template created as CVI-ServiceAccount and visible in certtmpl.msc
- [ ] Part B: Template published to CVI Issuing CA 1
- [ ] Part B: Certificate issued to svc.autoenroll — enrollment steps documented
- [ ] Part C: certutil output pasted and key fields extracted into table
- [ ] Part C: Request ID recorded from certsrv.msc Issued Certificates node
- [ ] Part D: Written explanation completed in prose
- [ ] Reflection section completed
- [ ] File saved as `lab-01-service-account-profile.md`
- [ ] File committed to portfolio repo under `labs/week-11/`
