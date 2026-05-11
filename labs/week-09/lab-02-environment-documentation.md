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
| Revoked Certificates  |      Folder is empty                   |
| Issued Certificates   |       Folder is empty                   |
| Pending Requests      |       Folder is empty                   |
| Failed Requests       |     Folder is empty                     |
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
- Templates visible in certtmpl.msc: 
  >Administrator, Authenticated Session, Basic EFS, CA Exchange, CEP Encryption, Code Signing, Computer, Cross Certification Authority, Directory Email Replication, Domain Controller, Domain Controller Authentication, EFS Recovery, Enrollment Agency, Enrollment Agency (Computer), Exchange Enrollment (Offline Request), Exchange Signature Only, Exchange User, IPSec, IPSec (Offline Request), Kerberos Authentication, Key Recovery Agent, OCSP Response Signing, RAS and IAS Server, Root Certification Authority, Router (Offline request), Smartcard Logon, Smartcard User, Subordinate Certification Authority, Trust List Signing, User, User Signature Only, Web Server, Workstation Authentication

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
  >The Subject field confirmed that the certificate belongs to CVI Issuing CA 1. The Issuer field revealed that it was signed by the CVI Root CA, demonstrating the hierarchical trust relationship between the offline Root CA and the Issuing CA. The SHA1 certificate hash functioned as the certificate thumbprint, uniquely identifying the certificate within the PKI environment.

---

## Part C — Active Directory Documentation

- PKI Admins OU location:
  >corp.cvilab.local/PKI Admins
- pki.admin group memberships:
  >Domain Admins, Domain Users, PKI Admins
- cert.manager group memberships:
  >Domain Users, PKI Admins
- Domain-joined computers visible:
  >DC01 (located in the Domain Controllers OU)      
   PKI-SRV01 (located in the Computers container)
- certtmpl.msc on DC01 — same templates as PKI-SRV01:
  >Yes

---

## Part D — Environment Summary 
The environment is made up of three virtual machines that support an Active Directory and PKI setup. DC01 functions as the domain controller and provides Active Directory Domain Services and DNS for the corp.cvilab.local domain. PKI-SRV01 is the enterprise issuing certificate authority and is responsible for issuing certificates to users, computers, and services in the domain. The Root CA is kept offline and is used only to sign the issuing CA certificate, which helps maintain the security of the trust root and reduces exposure to compromise.   

The certificate authority hierarchy starts with the Root CA, which acts as the trust anchor for the entire PKI environment and remains offline for security purposes. The Root CA signs the issuing CA certificate for CVI Issuing CA 1, which establishes a trusted chain of authority. Below the issuing CA are end entities such as users, computers, and services that receive certificates but cannot issue certificates themselves. This structure ensures that certificate issuance is controlled and that the most sensitive authority remains isolated.   

Certificate templates are stored centrally in the Active Directory Configuration partition and are available across the entire domain. They define the types of certificates that can be issued, such as user, machine, web server, and domain controller certificates. These templates exist in Active Directory as directory objects and represent the central definition of certificate types available in the environment. Certificate enrollment is controlled through these templates and enforced by the issuing CA based on policy and permissions.   

The Active Directory structure includes organizational units, user accounts, and security groups that support certificate management. DC01 appears under the Domain Controllers container, while PKI-SRV01 appears under the Computers container as a domain-joined machine. The Root CA is not listed because it is not joined to the domain and remains offline by design. The PKI Admins OU is located at `corp.cvilab.local/PKI Admins` and contains two user accounts, PKI Admin and Cert Manager, which are used to separate responsibilities within the PKI environment, as well as a security group called PKI Admins that controls administrative access to certificate authority management functions.   

One thing I found interesting and unexpected was how certificate templates exist in Active Directory but must be accessed differently depending on the system and tools available. On DC01, I was not able to use the `certtmpl.msc` console as part of my workflow, so I instead queried Active Directory directly using PowerShell to retrieve the certificate template objects:   

`Get-ADObject -LDAPFilter "(objectClass=pKICertificateTemplate)" -SearchBase "CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local"`   

This returned the full set of certificate template objects stored in Active Directory. On PKI-SRV01, I attempted to use the same PowerShell approach, but it failed because the Active Directory PowerShell module (RSAT tools) was not available on that system. As a result, the command was not recognized and returned an error. To continue the comparison from the certificate authority side, I used `certutil -ADTemplate`. This displayed the same underlying certificate templates but also included additional information such as enrollment and Auto-Enroll status. Some entries showed “Access is denied,” which reflects permission restrictions rather than missing templates. This comparison showed that Active Directory is the central source of certificate template data, while different systems and tools expose that data in different ways depending on installed components and permissions.
