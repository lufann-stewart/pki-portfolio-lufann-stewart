# Lab 02 — AD CS Console Exploration & CA Hierarchy Documentation

Lufann Stewart 
May 8, 2026
**Phase:** 2 | **Week:** 9  
**Submission Path:** labs/week-09/lab-02-environment-documentation.md

---

## Part A — AD CS Console Exploration (PKI-SRV01)

certsrv.msc console nodes table:

| Node                  | Contents / Observations |
|-----------------------|:-------------------------|
| Revoked Certificates  |      Folder is Empty                   |
| Issued Certificates   |       Folder is Empty                   |
| Pending Requests      |       Folder is Empty                   |
| Failed Requests       |     Folder is Empty                     |
| Certificate Templates |     Contains various default templates including: Directory Email Replication, Domain Controller Authentication, Kerberos Authentication, EFS Recovery Agent, Basic EFS, Domain Controller, Web Server, Computer, User, Subordinate Certification Authority, and Administrator. These define the intended purpose and technical attributes for certificates the CA can issue.                     |

- CA Properties — CDP path:  
  Local File Path:
  ```
  C:\Windows\System32\CertSrv\CertEnroll\<CaName><CRLNameSuffix><DeltaCRLAllowed>.crl
  ```
  LDAP Path:  
  ```
  ldap:///CN=<CATruncatedName><CRLNameSuffix>,CN=<ServerShortName>,CN=CDP,CN=Public Key Services,CN=Services,<ConfigurationContainer><CDPObjectClass>
  ```
- CA Properties — AIA path:
  Local File Path:
  ```
  C:\Windows\System32\CertSrv\CertEnroll\<ServerDNSName>_<CaName><CertificateName>.crt
  ```
  LDAP Path:  
  ```
  ldap:///CN=<CATruncatedName>,CN=AIA,CN=Public Key Services,CN=Services,<ConfigurationContainer><CAObjectClass>
  ```
- CA Properties — Database path:
  ```
  C:\Windows\system32\CertLog
  ```
- Templates visible in certtmpl.msc: (list here)
  Administrator, Aiuthenciated Session, Basic EFS, CA Exchange, CEP Encryption, Code SIgning, COmputer, Cross Ceretification Authroity, Directory Email Replication, Domain Controller, Domain Controller Authentication, EFS Reovery, Enrollment Agency, Enrollement Agency (computer), Exchange Enrollemnt A

---

## Part B — CA Hierarchy Verification

Command: `certutil -store -enterprise Root`

Output:
```
Root "Trusted Root Certification Authorities"
================ Certificate 0 ================
Serial Number: 26373e51a6ab669340c47caef2232ce1
Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
 NotBefore: 4/25/2026 6:15 PM
 NotAfter: 4/25/2046 6:25 PM
Subject: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
CA Version: V0.0
Signature matches Public Key
Root Certificate: Subject matches Issuer
Cert Hash(sha1): b805e6ab548f6e7c57d3989f61de7fe6a51031d1
No key provider information
Cannot find the certificate and private key for decryption.
CertUtil: -store command completed successfully.
```

Command: `certutil -store -enterprise CA`

Output:
```
CA "Intermediate Certification Authorities"
================ Certificate 0 ================
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
No key provider information
Missing stored keyset
CertUtil: -store command completed successfully.
```

Observations — what Subject, Issuer, and Thumbprint confirm:
(write here)

---

## Part C — Active Directory Documentation

- PKI Admins OU location: (describe here)
- pki.admin group memberships: (list here)
- cert.manager group memberships: (list here)
- Domain-joined computers visible: (list here)
- certtmpl.msc on DC01 — same templates as PKI-SRV01: Yes / No

---

## Part D — Environment Summary

Write in prose paragraphs covering:

1. **Environment Topology**
2. **CA Hierarchy**
3. **Certificate Templates**
4. **Active Directory Structure**
5. **One thing you found interesting or unexpected**

*This section is referenced in the Week 13 backup and recovery lab — write it as a runbook entry.*
