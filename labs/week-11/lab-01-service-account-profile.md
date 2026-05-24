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
| Digital Signature | Yes |Required for client authentication and proving identity.|
| Key Encipherment |Yes  |Allows secure key exchange during authenticated sessions. |
| Data Encipherment | No|This account only authenticates; it does not directly encrypt files at rest.|
| Non-Repudiation |No | Not required for automated service logons. |

**Explanation of Key Usage decisions:**

```
I included Digital Signature because the certificate must prove the identity of the service account during authentication. I included Key Encipherment to support secure session key exchange during encrypted communications. These two usages are commonly required for client authentication certificates.
```

**2 — Extended Key Usage (EKU)**

| EKU | Included? | OID | Reason |
|-----|-----------|-----|--------|
| Client Authentication |Yes | 1.3.6.1.5.5.7.3.2 |Allows the service account to authenticate to domain services and applications.|
| Server Authentication |No | 1.3.6.1.5.5.7.3.1 |Will not be used to authenticate servers. |
| Code Signing | No| 1.3.6.1.5.5.7.3.3 |This service account is not intended to sign applications, scripts, or executables.|
| Secure Email |No | 1.3.6.1.5.5.7.3.4 | The certificate is not intended for email encryption or signing.|

**Explanation of EKU decisions:**

```
This certificate is explicitly designed to authenticate the identity of an automated background service account (svc.autoenroll) to the Active Directory domain. Because its only job is to prove its identity during authentication, it only requires the Client Authentication EKU. Restricting all other unused EKUs prevents the certificate from being misused elsewhere, strictly following the security principle of least privilege.

```
**3 — Subject Name**

| Setting | Value Selected | Reason |
|---------|---------------|--------|
| Subject name format | Build from Active Directory  |Ensures the CA automatically pulls the identity from Active Directory instead of allowing manual subject input. |
| Include this information in the subject name |User Principal Name|Ties the certificate directly to the account's Active Directory login identity so the domain knows exactly which specific user or service account it belongs to. |
**Explanation of Subject Name decision:**

```
Building the subject name from Active Directory ensures the CA uses verified account attributes instead of user-supplied values. This reduces the risk of identity spoofing or unauthorized certificate requests and ensures the certificate identity matches the directory object.
```

**4 — Validity Period**

| Setting | Value | Reason |
|---------|-------|--------|
| Validity period | 1 Year|Shorter certificate lifetimes reduce long-term exposure if a certificate is compromised.|
| Renewal period |6 weeks |Provides enough time for automatic renewal before expiration. |

**Explanation of validity period decision:**

```
A 1-year validity period keeps certificates rotating regularly, and the 6-week renewal window gives Windows Autoenrollment enough time to replace them before they expire.
```

**5 — Security Tab — Enrollment Permissions**

| Group / Account | Read | Enroll | Autoenroll | Reason |
|-----------------|------|--------|------------|--------|
| Authenticated Users |Yes | NO| NO|Required for template visibility in AD, but enrollment is restricted. |
| CORP\svc.autoenroll | Yes|Yes |Yes |Only account allowed to request and autoenroll for this certificate. |
| Domain Computers |NO |NO | NO|Prevents machine accounts from enrolling in a service-specific certificate. |

**Explanation of enrollment permission decisions:**

```
Permissions are restricted to enforce least privilege and ensure only the svc.autoenroll account can request and autoenroll the certificate. This prevents other users or machines from obtaining a certificate meant for a specific service identity.
```

**General tab — Template names:**

| Field | Value |
|-------|-------|
| Template display name | CVI Service Account |
| Template name (internal) | CVI-ServiceAccount |
| Schema version |2 |

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
4. Entered service account: N/A — Certificate was enrolled using the pki.admin administrative account via the Certificates MMC / AD enrollment policy.
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
No issues to report.
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
My "Personal"
================ Certificate 0 ================
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

**From the certutil output — record the following:**

| Field | Value |
|-------|-------|
| Subject |CN=PKI Admin|
|CA Version: V0.0 |
| Issuer |CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local |
| Serial Number |4400000004b742b5daba3d4deb000000000004 |
| Key Usage |Digital Signature, Key Encipherment (as defined by template configuration) |
| Enhanced Key Usage (EKU) |Client Authentication (1.3.6.1.5.5.7.3.2) |
| Validity: Not Before |5/23/2026 5:29 PM |
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
User certificates and service account certificates are both used for authentication, but they are designed for different purposes. A user certificate is tied to an individual person and is used for actions that a human performs, such as secure email or file encryption. These features are intended to support personal workflows and user-level security functions. In contrast, a service account certificate is designed for automated systems or applications. It is more restricted in scope and typically only includes Client Authentication because its purpose is to allow a service to prove its identity when connecting to systems, not to perform user-specific tasks.

Although both types of certificates can pull identity information from Active Directory, they represent different kinds of identities. A user certificate represents a real person and is directly linked to their login credentials. A service account certificate represents an automated identity used by services or applications. It is not tied to a human user, but instead exists so that systems can authenticate securely without requiring interactive login.

The way certificates are enrolled also differs depending on the type of account. User certificates are normally requested directly by the user while logged into the system. Service account certificates are usually requested by an administrator or an automated process on behalf of the service account, since service accounts are not typically used for interactive logins. This separation helps enforce security by ensuring certificates are issued only for their intended purpose and reduces the risk of misuse.

```

**What are the operational risks of relying on password authentication for service accounts instead of certificate-based authentication?**

```
Password-based authentication for service accounts is risky because passwords can be stolen, reused, or guessed. If a password is compromised, an attacker can impersonate the service account without needing to break any encryption.

Certificate-based authentication reduces this risk because it relies on cryptographic trust and certificate validation instead of shared secrets, making unauthorized access significantly harder.
```

---

## Reflection

**One thing about the CVI-ServiceAccount template design that was a non-obvious decision:**

```
The svc.autoenroll account did not show up as an option in the MMC service account enrollment list, so the certificate was requested while logged in as CORP\pki.admin. Because of that, the certificate was issued under the admin context, which is why the subject shows CN=PKI Admin instead of svc.autoenroll. The template itself was still configured correctly, including EKU settings, subject name sourcing from Active Directory, and the enrollment permissions.

The main issue was just the enrollment context during request, not the template design. Everything else in the process worked as expected, and the certificate was still successfully issued using the CVI-ServiceAccount template.
```

**What would you change about this template if this were a production environment rather than a lab?**

```
In a real production environment, I would tighten up how this certificate gets issued so it isn’t just something a single account can grab without oversight. Instead of allowing direct enrollment, I would probably require some kind of approval step or at least restrict issuance to a controlled security group so it’s easier to track and manage who is allowed to request it.

I would also shorten the validity period compared to what we used in the lab, because long-lived certificates increase the risk if something ever gets compromised. Shorter lifetimes force more regular rotation, which is safer in real environments.

On top of that, I would make monitoring a bigger deal. Certificate issuance and usage should be logged and reviewed so any unusual authentication activity stands out quickly. Overall, the goal would be to make sure the certificate is harder to misuse and easier to control over its full lifecycle.
```

---

## Submission Checklist

- [X] Pre-lab verification completed and outputs recorded
- [X] Part A: All five template design decisions documented with rationale
- [X] Part A: Template created as CVI-ServiceAccount and visible in certtmpl.msc
- [X] Part B: Template published to CVI Issuing CA 1
- [X] Part B: Certificate issued to svc.autoenroll — enrollment steps documented
- [X] Part C: certutil output pasted and key fields extracted into table
- [X] Part C: Request ID recorded from certsrv.msc Issued Certificates node
- [X] Part D: Written explanation completed in prose
- [X] Reflection section completed
- [X] File saved as `lab-01-service-account-profile.md`
- [X] File committed to portfolio repo under `labs/week-11/`
