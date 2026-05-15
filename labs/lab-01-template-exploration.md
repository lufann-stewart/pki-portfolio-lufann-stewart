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
user template name and internal name are the same there is also an option greyed out an checked for publish in Active Diretory then uneerthat an option thats greyed out to do not auto reenroll if a duplicate certificate exists in active directity the validity and renewal periods are greyed out you cant change them.
```

**Request Handling tab — Purpose:**

```
Signature and Encryption
```

**Subject Name tab — Subject name format:**

```
(Built from AD? Supplied in request? What fields?) Need to answer what fields
Built from information in Active Directory it says include aemail name box and its checked greyed out also and the type of subject says user checked an greyed out
```

**Extensions tab — Key Usage:**

```
Signature requirements: 
Digital signature 
Allow key exchange only with key encryption
Crtitical extension.

```

**Extensions tab — Application Policies (EKU):**

```
Encrypting File System 
Secure Email 
Client Sthentication

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
(Built from AD? Supplied in request?) 
Built from information in Active Directory
```

**Extensions tab — Key Usage:**

```
Signature requirements: 
Digital signature 
Allow key exchange only with key encryption
Crtitical extension.

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
Crtitical extension.

```

**Extensions tab — Application Policies (EKU):**

```
Server Authentication
```

---

### Template Comparison

In your own words — what is the most significant difference between the User, Computer, and Web Server templates?

```
(your answer here — 3–5 sentences)
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
| Validity period |2 year |6 weeks |
| Renewal period |1 year |6 weeks |

**Rationale for validity period chosen:**

```
(why did you choose this period?) IDK becaue it has to be checked and sausidited for accuracy to make asure everything is working especalluy casue its internet facing?
```

**Subject Name tab — changes made:**

| Setting | Change Made | Rationale |
|---------|-------------|-----------|
| | | |

**Security tab — permissions confirmed:**

| Group / Account | Enroll | Autoenroll |
|-----------------|--------|------------|
| Domain Computers |X | |
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
(paste output here)
```

**From the certutil output — record the following:**

| Field | Value from certutil Output |
|-------|---------------------------|
| Template Name |CVI-WebServer |
| Template OID |CVI-WebServer |
| Schema Version | |
| Key Usage | |
| Enhanced Key Usage (EKU) | |
| Validity Period | |
| Subject Name flags | |

---

## Reflection

**Why does AD CS require you to duplicate a built-in template rather than modifying it directly?**

```
Microsoft locks the template to prevent accidental breakage duplicating snures no accidents while allowing new templates to be created and modified . I got that from this Built-in templates cannot be modified directly — Microsoft locks them to prevent accidental breakage. Duplicating creates a new, editable copy.
```

**One setting in the template you found unexpected or would want to explore further:**
Maybe the Server tab that has the options: DO not store certificates and requests in the CA database 

```
(your observation here)
```

---

## Submission Checklist

- [ ] Pre-lab verification completed and outputs recorded
- [ ] Part A: All three templates explored with tab-level observations
- [ ] Part A: Comparison question answered in own words
- [ ] Part B: Duplicate template created as CVI-WebServer
- [ ] Part B: All changes documented with rationale
- [ ] Part C: certutil -template output pasted and key fields extracted
- [ ] Reflection section completed
- [ ] File saved as `lab-01-template-exploration.md`
- [ ] File committed to portfolio repo under `labs/week-10/`
