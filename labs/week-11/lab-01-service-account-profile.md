# Lab 01: Build a Certificate Profile for a Service Account

Lufann Stewart  
May 23, 2026   
**Phase:** 2 | **Week:** 11  
**Submission Path:** `labs/week-11/lab-01-service-account-profile.md`

---

## Pre-Lab Verification

If you can log into PKI-SRV01 as **CORP\pki.admin**, you are communicating with DC01 and the environment is ready. Proceed to Part A.

---

## Part A — Design the CVI-ServiceAccount Template

### Step 1 — Open the Certificate Templates Console

1. Press **Win + R**, type `certtmpl.msc`, and press **Enter**
2. The Certificate Templates console opens, showing all templates installed on this CA
3. Scroll through the list to get familiar with what already exists

### Step 2 — Duplicate the User Template

1. Scroll down to find the **User** template in the list
2. Right-click **User** → select **Duplicate Template**
3. A new template Properties window opens — this is your working copy

> **Why User?** The svc.autoenroll account is an AD user-type object, not a machine account. The User template is the correct baseline. The Computer template is for machine accounts and includes Server Authentication EKU, which is not appropriate for a service account.

**Source template duplicated:** User

**Reason for choosing this source template:**

```
The certificate is being issued to a service account, which is an AD user object (svc.autoenroll). The User template gives you the right starting point for subject name sourcing from AD user attributes. The Computer template is for machine identity (DNS-based subject names).
```

### Step 3 — Set Compatibility Settings

1. Click the **Compatibility** tab
2. Set **Certification Authority** to: `Windows Server 2012 R2`
3. Set **Certificate Recipient** to: `Windows 8.1 / Windows Server 2012 R2`
4. Click **OK** on any informational dialog that appears

**Compatibility settings selected:**
- Certification Authority: Windows Server 2012 R2
- Certificate Recipient: Windows 8.1 / Windows Server 2012 R2

### Step 4 — Set the Template Name (General Tab)

1. Click the **General** tab
2. Change **Template display name** to: `CVI Service Account`
3. The **Template name** (internal name, no spaces) will auto-fill as `CVI-ServiceAccount` — confirm this
4. Note the **Schema version** shown at the bottom

**General tab — Template names:**

| Field | Value |
|-------|-------|
| Template display name | CVI Service Account |
| Template name (internal) | CVI-ServiceAccount |
| Schema version | 4 |
### Step 5 — Configure Key Usage

1. Click the **Extensions** tab
2. In the Extensions list, select **Key Usage** → click **Edit**
3. In the Key Usage dialog:
   - Check **Digital Signature**
   - Uncheck **Key Encipherment** (if checked)
   - Uncheck **Data Encipherment** (if checked)
   - Uncheck **Non-Repudiation** (if checked)
   - Check **Make this extension critical**
4. Click **OK**

Document what you set and why:

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

### Step 6 — Configure Extended Key Usage (Application Policies)

1. Still on the **Extensions** tab, select **Application Policies** → click **Edit**
2. The User template comes with several EKUs pre-populated (Client Authentication, Encrypting File System, Secure Email). You need to remove all but one:
   - Select **Encrypting File System** → click **Remove**
   - Select **Secure Email** → click **Remove**
   - Leave **Client Authentication** (1.3.6.1.5.5.7.3.2) — this is the only EKU needed
3. Click **OK**

Document your EKU decisions:

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

### Step 7 — Configure Subject Name

1. Click the **Subject Name** tab
2. Select **Build from this Active Directory information**
3. Under **Subject name format**, select **User principal name (UPN)** from the dropdown
4. Uncheck any other options under "Include this information in alternate subject name" that are not needed

Document your settings:

| Setting | Value Selected | Reason |
|---------|---------------|--------|
| Subject name format | Build from Active Directory  |Ensures the CA automatically pulls the identity from Active Directory instead of allowing manual subject input. |
| Include this information in the subject name |User Principal Name|Ties the certificate directly to the account's Active Directory login identity so the domain knows exactly which specific user or service account it belongs to. |

**Explanation of Subject Name decision:**

```
Building the subject name from Active Directory ensures the CA uses verified account attributes instead of user-supplied values. This reduces the risk of identity spoofing or unauthorized certificate requests and ensures the certificate identity matches the directory object.
```

### Step 8 — Set Validity Period

1. Click the **General** tab (you may already be on it)
2. Find the **Validity period** and **Renewal period** fields
3. Set **Validity period** to `1` year (or 2 years — document your reasoning)
4. Leave **Renewal period** at the default (6 weeks)

> **Note:** The CA itself has a maximum validity setting. If you set 5 years here but certs issue for only 2, the CA's max is overriding your template. You can check with: `certutil -getreg ca\ValidityPeriod`

| Setting | Value | Reason |
|---------|-------|--------|
| Validity period | 1 Year|Shorter certificate lifetimes reduce long-term exposure if a certificate is compromised.|
| Renewal period |6 weeks |Provides enough time for automatic renewal before expiration. |

**Explanation of validity period decision:**

```
A 1-year validity period keeps certificates rotating regularly, and the 6-week renewal window gives Windows Autoenrollment enough time to replace them before they expire.
```

### Step 9 — Set Enrollment Permissions (Security Tab)

1. Click the **Security** tab
2. You will see **Authenticated Users** and **Domain Admins** already listed
3. Add svc.autoenroll manually:
   - Click **Add**
   - In the object picker, type `svc.autoenroll` and click **Check Names**
   - Confirm the account resolves to `CORP\svc.autoenroll`, then click **OK**
   - With **svc.autoenroll** selected in the list, check **Read**, **Enroll**, and **Autoenroll**
4. Click **Apply**
5. Review the permissions on all three entries and document them below

| Group / Account | Read | Enroll | Autoenroll | Reason |
|-----------------|------|--------|------------|--------|
| Authenticated Users | Yes | No | No |Required for template visibility in AD, but enrollment is restricted. |
| CORP\svc.autoenroll | Yes| Yes | Yes |Only account allowed to request and autoenroll for this certificate. |
| Domain Admins |Yes | Yes | No |Domain Admins were given Read and Enroll permissions so administrators can access and manually request certificates if needed without automatically receiving them through autoenrollment.
 |

**Explanation of enrollment permission decisions:**

```
Permissions are restricted to enforce least privilege and ensure only the svc.autoenroll account can request and autoenroll the certificate. This prevents other users or machines from obtaining a certificate meant for a specific service identity.
```

### Step 10 — Save the Template

1. Click **OK** to close the Properties window and save the template
2. Verify **CVI-ServiceAccount** now appears in the certtmpl.msc list

**Template saved and visible in certtmpl.msc:**
- [X] Yes

---

## Part B — Publish the Template and Issue the Certificate

### Step 1 — Publish the Template to the CA

1. Press **Win + R**, type `certsrv.msc`, and press **Enter**
2. Expand **CVI Issuing CA 1** in the left pane
3. Right-click **Certificate Templates** → **New** → **Certificate Template to Issue**
4. In the Enable Certificate Templates dialog, scroll to find **CVI-ServiceAccount**
5. Select it → click **OK**
6. The template should appear in the Certificate Templates node within 30 seconds. If it doesn't, right-click the node → **Refresh**

**CVI-ServiceAccount visible in Certificate Templates node:**
- [X] Yes
- [ ] No — describe what happened:

```
No issues to report.
```

---

### Step 2 — Request the Certificate for svc.autoenroll

> **Important:** svc.autoenroll is an AD user account, not a Windows service. The Certificates snap-in "Service account" option lists Windows system services only — svc.autoenroll will not appear there. Instead, use `runas` to open MMC running as the svc.autoenroll account.

1. Open **Command Prompt** or **PowerShell** as Administrator
2. Run the following command:
   ```
   runas /user:CORP\svc.autoenroll mmc.exe
   ```
3. Enter the **svc.autoenroll password** when prompted
4. A new MMC window opens — this window is running as svc.autoenroll
5. In the new MMC window: **File → Add/Remove Snap-in**
6. Select **Certificates** → click **Add**
7. Choose **My user account** → click **Finish** → click **OK**
8. Expand **Certificates (Current User)** → **Personal**

> **Note:** If no certificates have been issued yet for this account, you will not see a **Certificates** folder under Personal — this is expected. Right-click **Personal** directly → **All Tasks** → **Request New Certificate**. Once the certificate is issued, the Certificates folder will appear automatically.

9. Right-click **Personal** → **All Tasks** → **Request New Certificate**
10. Click **Next** through the enrollment wizard
11. Select **Active Directory Enrollment Policy** → click **Next**
12. The CVI-ServiceAccount template should appear in the list. Check the box next to it
13. Click **Enroll**
14. Enrollment should complete immediately (Status: Succeeded). Click **Finish**

**Enrollment wizard — enrollment policy selected:**

```
Active Directory Enrollment Policy
```

**Templates visible in the wizard:**

```
Basic EFS
CVI Service Account
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

### Step 1 — Inspect the Certificate with certutil

Open **PowerShell** on PKI-SRV01 as CORP\pki.admin and run:

```powershell
certutil -store My
```

> If the certificate was enrolled via the runas MMC session, it will be in the svc.autoenroll user's personal store, not the pki.admin store. To view it, you can either re-open the runas MMC window, or check the CA's Issued Certificates node (Step 2 below) to confirm issuance.

**Full certutil output:**

```
My "Personal"
================ Certificate 0 ================
Serial Number: 440000000911304cdb8a83f133000000000009
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
 NotBefore: 5/28/2026 3:14 PM
 NotAfter: 4/25/2027 7:36 PM
Subject: CN=Svc Autoenroll, OU=Service Accounts, DC=corp, DC=cvilab, DC=local
Non-root Certificate
Template: CVI-ServiceAccount, CVI Service Account
Cert Hash(sha1): 7bff83b595815dd3ec80e4e658b4b73cb67a7e1e
  Key Container = ce9bd3f51b49e9f605ed02f7c971f605_f0a99c17-76d3-498a-97de-2992c06105fd
  Simple container name: te-CVI-ServiceAccount-9f57fc06-547c-49c2-932d-7374d22c5bae
  Provider = Microsoft Enhanced Cryptographic Provider v1.0
Encryption test passed
CertUtil: -store command completed successfully.


```

**From the certutil output — record the following:**

| Field | Value |
|-------|-------|
| Subject | CN=Svc Autoenroll, OU=Service Accounts, DC=corp, DC=cvilab, DC=local |
| Issuer | CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local |
| Serial Number |440000000911304cdb8a83f133000000000009 |
| Key Usage | Digital Signature, Key Encipherment (a0) |
| Enhanced Key Usage (EKU) | Client Authentication (1.3.6.1.5.5.7.3.2) |
| Validity: Not Before | ‎Thursday, ‎May ‎28, ‎2026 3:14:09 PM|
| Validity: Not After | ‎Sunday, ‎April ‎25, ‎2027 7:36:58 PM |
| Thumbprint |7bff83b595815dd3ec80e4e658b4b73cb67a7e1e |

### Step 2 — Confirm in certsrv.msc and Record the Request ID

1. Open **certsrv.msc** (Press Win + R → type `certsrv.msc`)
2. Expand **CVI Issuing CA 1** → click **Issued Certificates**
3. Find the svc.autoenroll certificate in the list (sort by Request ID or Requester Name)
4. Double-click it to open and confirm it shows the CVI-ServiceAccount template
5. Record the values below

**Certificate visible in Issued Certificates node:**
- [X] Yes

**Record from the Issued Certificates node:**

| Column | Value |
|--------|-------|
| Request ID | 9 |
| Requester Name | CORP\svc.autoenroll |
| Certificate Template | "1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.16756659.8111121" CVI Service Account |
| Issued Common Name | Svc Autoenroll |
| Certificate Expiration Date | 4/25/2027 7:36 PM |

> **Save this Request ID.** You will use it in Week 12 to revoke this certificate, and in Lab 03 for the comparison exercise.

---

## Part D — Written Explanation

Answer the following questions in plain prose paragraphs — not bullet points. Aim for 2–3 paragraphs total across the two questions.

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
One non-obvious design consideration was understanding that certificate enrollment behavior depends heavily on the identity context used during the request. At first, I assumed enrollment could be done from the administrative MMC session without affecting the final certificate identity, but during implementation I validated that the enrollment context directly determines the subject information issued by the CA.

By correcting the process and enrolling the certificate using the svc.autoenroll account via runas, the certificate was issued with the correct subject identity tied to the service account rather than the administrative account. This reinforced the importance of matching the enrollment context to the intended identity when working with certificate templates.

The key takeaway is that even when a template is correctly configured with Active Directory-based subject naming, the final certificate identity still depends on the account used during enrollment, making proper execution of the workflow just as important as the template design itself.
```

**What would you change about this template if this were a production environment rather than a lab?**

```
In a real production environment, I would tighten up how this certificate gets issued so it isn’t just something a single account can grab without oversight. Instead of allowing direct enrollment, I would probably require some kind of approval step or at least restrict issuance to a controlled security group so it’s easier to track and manage who is allowed to request it.

I would also shorten the validity period compared to what we used in the lab, because long-lived certificates increase the risk if something ever gets compromised. Shorter lifetimes force more regular rotation, which is safer in real environments.

On top of that, I would make monitoring a bigger deal. Certificate issuance and usage should be logged and reviewed so any unusual authentication activity stands out quickly. Overall, the goal would be to make sure the certificate is harder to misuse and easier to control over its full lifecycle.
```

---

## Submission Checklist

- [X] Pre-lab verification completed
- [X] Part A: Template duplicated from the User template
- [X] Part A: All five design decisions (Key Usage, EKU, Subject Name, Validity, Security) documented with rationale
- [X] Part A: Template saved as CVI-ServiceAccount and visible in certtmpl.msc
- [X] Part B: Template published to CVI Issuing CA 1
- [X] Part B: Certificate requested via runas /user:CORP\svc.autoenroll mmc.exe
- [X] Part B: Certificate issued — enrollment outcome documented
- [X] Part C: certutil output pasted and key fields extracted into table
- [X] Part C: Request ID recorded from certsrv.msc Issued Certificates node
- [X] Part D: Written explanation completed in prose
- [X] Reflection section completed
- [X] File saved as `lab-01-service-account-profile.md`
- [X] File committed to portfolio repo under `labs/week-11/`
