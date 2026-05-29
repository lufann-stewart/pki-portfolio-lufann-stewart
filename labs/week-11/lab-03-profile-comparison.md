# Lab 03: Compare Certificate Profiles Side by Side

Lufann Stewart  
May 29, 2026  
**Phase:** 2 | **Week:** 11  
**Submission Path:** `labs/week-11/lab-03-profile-comparison.md`

---

## Overview

Lab 03 is a synthesis exercise. You have issued three certificates across Weeks 10 and 11:

1. **TLS certificate** — issued from CVI-WebServer in Week 10, Lab 02
2. **Service account certificate** — issued from CVI-ServiceAccount in Week 11, Lab 01
3. **Code signing certificate** — issued from CVI-CodeSigning in Week 11, Lab 02

In this lab, you inspect all three certificates using certutil, document their settings in a structured comparison table, and write an analytical explanation of why the differences exist.

> **Prerequisite:** All three certificates must be present before you begin. If the Week 10 TLS certificate is no longer available, contact your instructor before proceeding.

---

## Pre-Lab — Locate All Three Certificates

If you can log into PKI-SRV01 as **CORP\pki.admin**, you are communicating with DC01 and the environment is ready.

### Step 1 — List All Certificates in the Personal Store

Open **PowerShell** on PKI-SRV01 as CORP\pki.admin and run:

```powershell
certutil -store My
```

This lists all certificates in the current user's personal store. You should see at least the code signing certificate from Lab 02 and the TLS certificate from Week 10. Scroll through the output and identify each certificate by its Template or Subject field.

**Week 10 TLS certificate present:**
- [X] Yes — Thumbprint: e0046ea8c9d051f976c47ff60246cf3f488ad4f8
- [ ] No — contact instructor before proceeding

### Step 2 — Locate the Service Account Certificate

The service account certificate was enrolled via the runas session in Lab 01. If it does not appear in the output above, run:

```powershell
certutil -store -service svc.autoenroll My
```

This checks the svc.autoenroll service account's certificate store separately.

### Step 3 — Record All Three Thumbprints

From the certutil outputs above, find and record the thumbprint for each certificate. The thumbprint is a 40-character hex string shown near the bottom of each certificate entry.

| Certificate | Template | Thumbprint |
|-------------|----------|------------|
| TLS (Week 10, Lab 02) | CVI-WebServer |e0046ea8c9d051f976c47ff60246cf3f488ad4f8 |
| Service Account (Lab 01) | CVI-ServiceAccount |7bff83b595815dd3ec80e4e658b4b73cb67a7e1e |
| Code Signing (Lab 02) | CVI-CodeSigning |7fa17dbe13f3c0f3ff3e60af8528c4edb48d96ce|

---

## Part A — Inspect All Three Certificates

### Step 1 — Inspect Each Certificate Using Its Thumbprint

Run the following command for each certificate, replacing the placeholder with the actual thumbprint. Remove any spaces from the thumbprint before running.

**Certificate 1 — TLS (CVI-WebServer)**

```powershell
certutil -store My "<TLS-thumbprint>"
```

**Full certutil output:**

```
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
CertUtil: -store command completed successfully.

```

---

**Certificate 2 — Service Account (CVI-ServiceAccount)**

```powershell
certutil -store My "<ServiceAccount-thumbprint>"
```

If the service account certificate is in the service account store rather than the personal store, use:

```powershell
certutil -store -service svc.autoenroll My "<ServiceAccount-thumbprint>"
```

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

---

**Certificate 3 — Code Signing (CVI-CodeSigning)**

```powershell
certutil -store My "<CodeSigning-thumbprint>"
```

**Full certutil output:**

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
CertUtil: -store command completed successfully.

```

---

### Step 2 — Complete the Comparison Table

Using the certutil outputs above, fill in every cell of the table below. Look for each field in the certutil output — Key Usage and EKU are shown in the Extensions section. No cells should be left blank.

> **Finding the EKU OID:** In the certutil output, look for the line starting with `1.3.6.1.5.5.7.3.` under Enhanced Key Usage — this is the OID. If you can't find it, run: `certutil -store My "<thumbprint>" | findstr OID`

| Field | TLS Certificate | Service Account Certificate | Code Signing Certificate |
|-------|----------------|----------------------------|--------------------------|
| Template Name | CVI-WebServer | CVI-ServiceAccount | CVI-CodeSigning |
| Subject |CN=PKI-SRV01.corp.cvilab.local |CN=Svc Autoenroll, OU=Service Accounts, DC=corp, DC=cvilab, DC=local|CN=PKI Admin |
| Subject Source | Supplied in request | Build from AD | Build from AD |
| Issuer | CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local| CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local|CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local |
| Key Usage |Digital Signature, Key Encipherment |Digital Signature, Key Encipherment |Digital Signature |
| EKU |Server Authentication |Client Authentication |Code Signing |
| EKU OID(s) |1.3.6.1.5.5.7.3.1 |1.3.6.1.5.5.7.3.2 |1.3.6.1.5.5.7.3.3 |
| Validity Period |5/22/2026 - 4/25/2027 |5/28/2026 - 4/25/2027 |5/24/2026 - 4/25/2027 |
| Serial Number |44000000030d20ca71500ba5c4000000000003|440000000911304cdb8a83f133000000000009 |4400000005b4922c2e24e100eb000000000005|
| Thumbprint |e0046ea8c9d051f976c47ff60246cf3f488ad4f8 |7bff83b595815dd3ec80e4e658b4b73cb67a7e1e |7fa17dbe13f3c0f3ff3e60af8528c4edb48d96ce |
| Request ID |3 | 9|5 |

---

## Part B — Written Analysis

This section is the substance of Lab 03. Write in **prose paragraphs — not bullet points**. Each answer should explain the *why*, not just describe what you observed.

**1 — Key Usage**

Each of the three certificates has different Key Usage settings. Explain *why* each certificate has the Key Usage it has. For each, trace the setting back to the cryptographic operation the use case requires.

```
The TLS Web Server certificate includes Digital Signature and Key Encipherment because it must prove the identity of the server during the TLS handshake and support secure key exchange when establishing encrypted sessions. The service account certificate also includes Digital Signature and Key Encipherment because it is used for certificate-based authentication and secure communications with other systems and services.

The code signing certificate only includes Digital Signature because its purpose is to sign software and scripts. It does not need to encrypt communications or exchange session keys. The digital signature allows systems to verify the integrity and authenticity of the code and confirm that it has not been modified since it was signed.
```

---

**2 — Extended Key Usage (EKU)**

Each certificate has a different EKU. Explain why each certificate has the EKU it has. For each, identify the relying party application that enforces that EKU and what it would do if the EKU were absent or wrong.

```
Each certificate has a different EKU because each one is intended for a different purpose. The TLS certificate uses the Server Authentication EKU, which is enforced by web browsers and TLS clients when validating the identity of a server. If this EKU were missing or incorrect, clients could reject the certificate during the TLS handshake.

The service account certificate uses the Client Authentication EKU. This is enforced by systems that perform certificate-based logon or authentication. If the Client Authentication EKU were absent, the certificate could not be used to authenticate the service account.

The code signing certificate uses the Code Signing EKU. This is enforced by Windows, PowerShell, and other software verification mechanisms when validating signed code. If the Code Signing EKU were missing or incorrect, signed applications or scripts could fail validation or be treated as untrusted.
```

---

**3 — Subject Name Source**

The TLS certificate uses "Supplied in request" for its subject. The service account and code signing certificates use "Build from Active Directory." Explain why the subject name source is different for the TLS certificate — what is it about the TLS use case that makes AD-supplied subject names impractical?

```
The TLS certificate uses “Supplied in request” because server identity in TLS is based on network and DNS naming rather than Active Directory objects. Unlike user or service account certificates, a web server may represent a hostname, virtual machine, or service endpoint that does not directly map to a single AD user or computer attribute in a consistent way.

Using a supplied subject allows the administrator or request process to explicitly define the correct DNS name or common name for the server, ensuring the certificate matches how clients actually reach the service. This is critical in TLS because clients validate the certificate against the hostname they connect to, not an Active Directory identity.
```

---

**4 — Security Question**

Imagine a single certificate that combined all three EKUs: Server Authentication, Client Authentication, and Code Signing. What security risk would this create? Be specific — what could an attacker do with a compromised private key from this combined certificate that they could not do with any one of the three individual certificates?

```
If a single certificate combined Server Authentication, Client Authentication, and Code Signing EKUs, the compromise of its private key would create a critical security failure across multiple trust boundaries. An attacker could impersonate trusted servers during TLS sessions, allowing them to intercept or modify encrypted traffic through man-in-the-middle attacks. They could also authenticate as a legitimate user or service through Client Authentication, gaining unauthorized access to systems and services that rely on certificate-based identity.

Most importantly, the inclusion of Code Signing would allow the attacker to sign malicious software or scripts in a way that appears trusted by operating systems and security controls. This would enable malware or unauthorized code to execute as if it came from a legitimate source. Combining all three EKUs in one certificate collapses separate security roles into a single trust anchor, meaning one key compromise breaks confidentiality, integrity, and software trust simultaneously.
```

---

## Reflection

**Which of the three certificates would you consider most critical to revoke quickly if the private key were compromised — and why?**

```
The Code Signing certificate is the most critical one to revoke if the private key is compromised because it controls what software the system is willing to trust and run. If someone gets access to that key, they could sign malicious scripts or programs so they look legitimate and trusted by Windows and security tools.

That’s more dangerous than the TLS or client authentication certificates because those mainly affect network access or encrypted communication. A compromised code signing cert affects the actual software running on the system, which can lead to persistent malware or unauthorized code being treated as safe.
```

**What would you add to this comparison if you were presenting it to a security team evaluating the PKI environment?**

```
If I were presenting this to a security team, I would add how each certificate type fits into the overall risk model and what happens if each one is compromised. For example, I would highlight the blast radius of each certificate: TLS affects encrypted communication, client authentication affects access to systems and services, and code signing affects trust in software execution.

I would also include how each certificate is monitored and revoked in a real environment, including how quickly revocation would take effect and what systems rely on CRL or OCSP checking. Another important addition would be the lifecycle controls around each certificate, such as enrollment restrictions, autoenrollment policies, and whether approval workflows are required.

Finally, I would call out operational risks like misconfigured templates or overly broad enrollment permissions, because those are often more dangerous in real environments than the certificates themselves.
```

---

## Submission Checklist

- [X] Pre-lab: All three thumbprints recorded in the table
- [X] Pre-lab: Service account cert located (personal store or svc.autoenroll store)
- [X] Part A: certutil output for all three certificates pasted in full
- [X] Part A: Comparison table fully completed — no blank cells
- [X] Part B: Key Usage analysis written in prose — traces each setting to a cryptographic operation
- [X] Part B: EKU analysis written in prose — identifies the relying party for each
- [X] Part B: Subject Name source difference explained with reasoning
- [X] Part B: Security question answered — specific risk identified for a combined-EKU certificate
- [X] Reflection completed
- [X] File saved as `lab-03-profile-comparison.md`
- [X] All three Week 11 lab files committed together with a single meaningful commit message
- [X] Request IDs for all three certificates recorded — needed for Week 12
- [X] All three Week 11 labs committed with a single meaningful commit message
- [X] Request IDs for all three certificates recorded (TLS from Week 10, Service Account, Code Signing) — needed for Week 12
