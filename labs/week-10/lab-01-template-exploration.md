# Lab 01: Explore and Duplicate a Certificate Template

Lufann Stewart 
May 15, 2026
**Phase:** 2 | **Week:** 10  
**Submission Path:** `labs/week-10/lab-01-template-exploration.md`

---

## Pre-Lab Verification

Run the following on PKI-SRV01 before starting. Do not proceed until all checks pass.

```powershell
# Check 1 — CA service running
Get-Service -Name CertSvc

# Check 2 — CA responding
certutil -ping

# Check 3 — Issuing CA cert in enterprise store
certutil -store -enterprise CA
```

**All checks passed:**
- [X] Yes
- [ ] No — describe the issue and how you resolved it:

```
No issues found
```

---

## Part A — Explore Three Built-in Templates

Open the Certificate Templates console: **Run → certtmpl.msc**

### Template 1: User

| Field | Value |
|-------|-------|
| Template Display Name | User |
| Template Name (internal) | User |
| Minimum Supported CA |Windows 2000  |
| Validity Period |1 year |
| Renewal Period |6 weeks |

**General tab — notes:**

```
The User template display name and internal template name are both “User.” The “Publish certificate in Active Directory” option is enabled but greyed out. There is also a greyed-out option that prevents auto reenrollment if a duplicate certificate already exists in Active Directory. The validity period and renewal period settings are also greyed out because built-in templates cannot be modified directly.
```

**Request Handling tab — Purpose:**

```
Signature and Encryption
```

**Subject Name tab — Subject name format:**

```
The subject name is built from information in Active Directory rather than being supplied in the request. The template uses the user object information from Active Directory, and the “Include e-mail name in subject name” option is enabled. The subject type is set to “User,” and these settings are greyed out because the built-in template cannot be modified directly.
```

**Extensions tab — Key Usage:**

```
Signature requirements: 
Digital signature 
Allow key exchange only with key encryption
Critical extension.

```

**Extensions tab — Application Policies (EKU):**

```
Encrypting File System 
Secure Email 
Client Authentication

```

---

### Template 2: Computer

| Field | Value |
|-------|-------|
| Template Display Name |Computer |
| Template Name (internal) |Machine |
| Validity Period |1 year |
| Renewal Period |6 weeks |

**Request Handling tab — Purpose:**

```
Signature and Encryption
```

**Subject Name tab:**

```
Built from information in Active Directory
```

**Extensions tab — Key Usage:**

```
Signature requirements: 
Digital signature 
Allow key exchange only with key encryption
Critical extension.

```

**Extensions tab — Application Policies (EKU):**

```
Client Authentication
Server Authentication

```

---

### Template 3: Web Server

| Field | Value |
|-------|-------|
| Template Display Name |Web Server |
| Template Name (internal) | WebServer|
| Validity Period |2 years |
| Renewal Period |6 weeks |

**Request Handling tab — Purpose:**

```
Signature and Encryption
```

**Subject Name tab:**

```
Supplied in the request.
```

**Extensions tab — Key Usage:**

```
Signature requirements: 
Digital signature 
Allow key exchange only with key encryption
Critical extension.

```

**Extensions tab — Application Policies (EKU):**

```
Server Authentication
```

---

### Template Comparison

In your own words — what is the most significant difference between the User, Computer, and Web Server templates?

```
The most significant difference between the User, Computer, and Web Server templates is their purpose and how the subject name is generated. The User and Computer templates automatically build the subject name from information stored in Active Directory, while the Web Server template requires the subject name to be supplied in the request. This is important because web servers may need certificates for DNS names or websites that are not directly tied to an Active Directory object. The templates also differ in their EKUs and intended use cases, such as secure email for users and server authentication for web servers. Additionally, the Web Server template has a longer validity period of two years compared to one year for the User and Computer templates.
```

Why does the Web Server template use "Supplied in the request" for the subject name rather than building it from Active Directory?

```
The Web Server template uses “Supplied in the request” because web servers often need certificates for specific DNS names or websites that may not match the computer name stored in Active Directory. This allows the requester to manually specify the exact subject name needed for the website or service. If Active Directory automatically generated the name, it could create certificate mismatches that would cause browser trust or HTTPS errors.
```

---

## Part B — Duplicate the Web Server Template

**Steps performed:**

1. Right-clicked the **Web Server** template → **Duplicate Template**
2. Selected compatibility settings:

   - Certification Authority: Windows Server 2012 R2
   - Certificate Recipient: Windows 7 / Server 2008 R2

3. Opened the properties of the new duplicate template.

**General tab — changes made:**

| Setting | Original Value | New Value |
|---------|---------------|-----------|
| Template display name | Web Server | CVI-WebServer |
| Template name | WebServer | CVI-WebServer |
| Validity period |2 years |1 year |
| Renewal period |6 weeks |6 weeks |

**Rationale for validity period chosen:**

```
A validity period of one year was chosen to improve security and reduce long-term risk exposure for the certificate. Shorter certificate lifetimes help ensure certificates are reviewed, renewed, and audited more frequently, especially for internet-facing web services. This also helps organizations respond more quickly to configuration changes or compromised certificates if issues occur.
```

**Subject Name tab — changes made:**

| Setting | Change Made | Rationale |
|---------|-------------|-----------|
|Subject name format   | No changes made  | The subject name is configured to be supplied in the request, meaning the requester must manually define the certificate subject (such as the DNS name). This is required for web server certificates because Active Directory does not automatically generate appropriate names for web-facing services.

**Security tab — permissions confirmed:**

| Group / Account | Enroll | Autoenroll |
|-----------------|--------|------------|
| Domain Computers |X| |
| Authenticated Users | | |

**Template saved:**
- [X] Yes — template visible in certtmpl.msc

---

## Part C — Inspect the Duplicate Template with certutil

Run the following command on PKI-SRV01:

```powershell
certutil -template CVI-WebServer
```

**Full output:**

```
Name: Active Directory Enrollment Policy
  Id: {41635678-B3E8-4BD7-8FE7-D49A1E336991}
  Url: ldap:
34 Templates:

  Template[8]:
  TemplatePropCommonName = CVI-WebServer
  TemplatePropFriendlyName = CVI-WebServer
  TemplatePropSecurityDescriptor = O:S-1-5-21-3975454498-3980183307-2685672490-1105G:S-1-5-21-3975454498-3980183307-2685672490-519D:PAI(OA;;RPWPCR;0e10c968-78fb-11d2-90d4-00c04f79dc55;;DA)(OA;;RPWPCR;0e10c968-78fb-11d2-90d4-00c04f79dc55;;S-1-5-21-3975454498-
3980183307-2685672490-519)(A;;CCDCLCSWRPWPDTLOSDRCWDWO;;;DA)(A;;CCDCLCSWRPWPDTLOSDRCWDWO;;;S-1-5-21-3975454498-3980183307-2685672490-519)(A;;CCDCLCSWRPWPDTLOSDRCWDWO;;;S-1-5-21-3975454498-3980183307-2685672490-1105)(A;;LCRPLORC;;;AU)

    Allow Enroll	CORP\Domain Admins
    Allow Enroll	CORP\Enterprise Admins
    Allow Full Control	CORP\Domain Admins
    Allow Full Control	CORP\Enterprise Admins
    Allow Full Control	CORP\pki.admin
    Allow Read	NT AUTHORITY\Authenticated Users

```

**From the certutil output — record the following:**

| Field | Value from certutil Output |
|-------|---------------------------|
| Template Name |CVI-WebServer |
| Template OID |1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.13096150.8214139 |
| Schema Version |2 |
| Key Usage | Digital Signature, Key Encipherment |
| Enhanced Key Usage (EKU) | Server Authentication|
| Validity Period |1 Year |
| Subject Name flags | CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT|

---

## Reflection

**Why does AD CS require you to duplicate a built-in template rather than modifying it directly?**

```
Active Directory Certificate Services requires administrators to duplicate built-in templates because the default templates are protected to prevent accidental changes that could disrupt certificate enrollment across the domain. Since these templates are used as trusted baseline configurations, modifying them directly could impact multiple systems and break existing certificate issuance. Duplicating a template allows administrators to safely customize settings for specific use cases without affecting the original default configuration.
```

**One setting in the template you found unexpected or would want to explore further:**
Maybe the Server tab that has the options: DO not store certificates and requests in the CA database 

```
One setting I found interesting is the option related to not storing certificates and requests in the CA database. I would want to explore this further because it affects whether issued certificates and requests are logged and tracked by the Certificate Authority. Disabling storage could reduce audit visibility and make it harder to troubleshoot or verify certificate issuance history, which is important in security environments.
```

---

## Submission Checklist

- [X] Pre-lab verification completed and outputs recorded
- [X] Part A: All three templates explored with tab-level observations
- [X] Part A: Comparison question answered in own words
- [X] Part B: Duplicate template created as CVI-WebServer
- [X] Part B: All changes documented with rationale
- [X] Part C: certutil -template output pasted and key fields extracted
- [X] Reflection section completed
- [X] File saved as `lab-01-template-exploration.md`
- [X] File committed to portfolio repo under `labs/week-10/`
