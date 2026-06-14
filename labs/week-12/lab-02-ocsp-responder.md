# Lab 02: Configure and Test the OCSP Responder

Lufann Stewart  
May 30, 2026     
**Phase:** 2 | **Week:** 12  
**Submission Path:** `labs/week-12/lab-02-ocsp-responder.md`

---

## Prerequisites

Complete Lab 01 before starting Lab 02. You need:
- A **revoked certificate** from Lab 01 (for OCSP Revoked status testing)
- A **valid, non-revoked certificate** (your TLS cert from Week 10 or another cert from Week 11 that you did not revoke)

---

## Pre-Lab Verification

If you can log into PKI-SRV01 as CORP\pki.admin, you are communicating with DC01 and the environment is ready. Proceed to Part A.

---

## Part A — Install the Online Responder Role

### Step 1 — Open Add Roles and Features

1. On PKI-SRV01, open **Server Manager**
2. Click **Manage → Add Roles and Features**

### Step 2 — Select Installation Type

1. On the Installation Type page, select **Role-based or feature-based installation**
2. Click **Next**

### Step 3 — Confirm Server Selection

1. On the Server Selection page, confirm **PKI-SRV01** is selected
2. Click **Next**

### Step 4 — Add the Online Responder Role

1. On the Server Roles page, expand **Active Directory Certificate Services**
2. Check **Online Responder**
3. When prompted to add required features, click **Add Features**
4. Click **Next**

### Step 5 — Complete the Installation

1. Continue through the remaining pages without changes
2. Click **Install**
3. Wait for the installation to complete before continuing — do not close Server Manager or proceed to Part B until the progress bar finishes

### Step 6 — Verify the Online Responder Service

1. In your elevated PowerShell prompt, confirm the service installed and is running:

```powershell
Get-Service -Name OCSPSvc
```

Expected output:
```
Status   Name               DisplayName                           
------   ----               -----------                           
Running  OCSPSvc            Online Responder Service     
```

> **If OCSPSvc does not appear:** The role installation may not have completed. Check Server Manager → Local Server for any pending configuration tasks. A server restart may be required on some configurations.

**Installation completed successfully:**
- [X] Yes
- [ ] No — describe the error:

```
No issues to report.
```

---

## Part B — Configure the Revocation Configuration

### Step 1 — Open the Online Responder Snap-in

1. Navigate to:
```
Start → Administrative Tools → Online Responder
```

If Online Responder is not visible in Administrative Tools:
1. Press **Win + R**, type `mmc.exe`, press **Enter**
2. Go to **File → Add/Remove Snap-ins**
3. Select **Online Responder** → click **Add** → click **OK**

### Step 2 — Start the Add Revocation Configuration Wizard

1. In the Online Responder snap-in, right-click **Revocation Configuration**
2. Select **Add Revocation Configuration**

### Step 3 — Work Through the Wizard

1. In the wizard, configure each page as follows:

| Wizard Page | Setting | Value |
|---|---|---|
| Name | Descriptive name for this configuration | `CVI Issuing CA 1 Revocation` (or similar) |
| CA Certificate Source | How to find the CA certificate | Select a certificate for an Existing enterprise CA |
| Browse | Select the CA | CVI Issuing CA 1 |
| Signing Certificate | How to obtain the signing cert | Automatically select a signing certificate (leave default) |

2. Click **Next** through each page, then click **Finish**

> **Why "Automatically select a signing certificate":** The Online Responder will automatically request an OCSP Response Signing certificate from the CA using the built-in OCSP Response Signing template. This is the correct behavior — do not select a certificate manually unless instructed.

### Step 4 — Read the Status Indicator

1. After the wizard completes, the Online Responder snap-in shows a status indicator for your revocation configuration
2. Wait up to 60 seconds, then press **F5** to refresh

| Status Indicator | Meaning | What to Do |
|---|---|---|
| Green | Operational — OCSP signing certificate has been issued and the responder is active | No action needed |
| Yellow | Configuration pending — signing cert may not have been issued yet | Wait 30–60 seconds and press F5 to refresh. If still yellow after 2 minutes, check Part C. |
| Red | Configuration failed — signing cert missing or CA unreachable | Confirm CertSvc is running. Check certsrv.msc → Certificate Templates and confirm the OCSP Response Signing template is published. |

**Revocation configuration name entered:**

```
CVI Issuing CA 1 Revocation
```

**CA certificate found and selected:**
- [X] Yes
- [ ] No — describe:

**Status indicator after configuration:**
- [X] Green (operational)
- [ ] Yellow (warning — describe below)
- [ ] Red (error — describe below)

```
No issues to report.
```

---

## Part C — Verify the OCSP Signing Certificate

When you created the revocation configuration, the Online Responder automatically requested an OCSP signing certificate from CVI Issuing CA 1. This certificate is what the Online Responder uses to sign every OCSP response it issues. You need to confirm it has the correct properties.

### Step 1 — List the Personal Store Certificates

1. In an elevated PowerShell prompt, list all certificates in the machine Personal store:

```powershell
certutil -store My
```

2. Locate the certificate with `OCSP Signing` in its Enhanced Key Usage section

```
================ Certificate 4 ================
Serial Number: 440000000ba5579effd72b412500000000000b
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
 NotBefore: 5/30/2026 12:25 PM
 NotAfter: 6/13/2026 12:25 PM
Subject: CN=PKI-SRV01.corp.cvilab.local
Non-root Certificate
Template: OCSPResponseSigning, OCSP Response Signing
Cert Hash(sha1): 23f6cb9ad02185a3c33f3a7b1b2ef71622150fcb
  Key Container = te-OCSPResponseSigning-151f19a3-0da7-46cb-b941-68950767d92e
  Unique container name: afc01196953219ab9274d1ddfb19d695_f0a99c17-76d3-498a-97de-2992c06105fd
  Provider = Microsoft Software Key Storage Provider
Private key is NOT exportable
Signature test passed
CertUtil: -store command completed successfully.


```

### Step 2 — Inspect the OCSP Signing Certificate

1. Run a targeted inspection using the certificate's subject name or thumbprint from Step 1:

```powershell
certutil -store My "<OCSP signing cert subject or thumbprint>"
```

### Step 3 — Verify the Required Properties

1. Confirm the following properties are present in the certutil output:

| Property | Expected Value | Observed Value |
|---|---|---|
| Extended Key Usage | OCSP Signing — OID 1.3.6.1.5.5.7.3.9 |OCSP Signing (1.3.6.1.5.5.7.3.9) |
| id-pkix-ocsp-nocheck extension | Present — OID 1.3.6.1.5.5.7.48.1.5 |Present — OID 1.3.6.1.5.5.7.48.1.5 |
| Key Usage | Digital Signature |Digital Signature |
| Issuer | CVI Issuing CA 1 | CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local|
| Validity (NotAfter) | In the future |6/13/2026 12:25 PM |

**Why these properties are required:**

| Property | Why Required |
|---|---|
| OCSP Signing EKU (1.3.6.1.5.5.7.3.9) | Authorizes this key specifically for signing OCSP responses. Without it, relying parties reject the response signature. |
| id-pkix-ocsp-nocheck (1.3.6.1.5.5.7.48.1.5) | Tells relying parties NOT to check the revocation status of the OCSP signing cert itself. Without it, checking an OCSP response would require another OCSP query — creating an infinite circular dependency. |
| Short validity (auto-renewed) | The Online Responder auto-renews the signing cert before expiry. Short validity limits exposure if the signing key is ever compromised. |
| Issued by CVI Issuing CA 1 | The signing cert must chain to the same CA as the certificates it is responding about. Clients validate this chain. |

> **If the OCSP Signing EKU is missing:** The wrong template was used during revocation configuration setup. Delete the revocation configuration in the snap-in and recreate it — the wizard should auto-select the correct OCSP Response Signing template when the Online Responder role is properly configured.

**OCSP signing certificate subject:**

```
CN=PKI-SRV01.corp.cvilab.local
```

**All expected properties present:**
- [X] Yes
- [ ] No — describe what is missing:

---

## Part D — Test the OCSP Responder

### Step 1 — Find the OCSP URL in the AIA Extension

1. Export a certificate to a `.cer` file if needed (from certsrv.msc → Issued or Revoked Certificates → right-click → Export), then run:

```powershell
certutil -dump <certificate.cer> | findstr /i "OCSP Authority"
```

The OCSP URL format in your environment is typically:
```
http://pki-srv01.corp.cvilab.local/ocsp
```

**OCSP URL found in the AIA extension:**

```
URL=http://pki-srv01.corp.cvilab.local/ocsp
```

### Step 2 — Test a Valid Certificate

1. Use a certificate that was NOT revoked in Lab 01 — your TLS certificate from Week 10, or any other valid certificate
2. Run:

```powershell
certutil -URL <valid-certificate.cer>
```

3. In the URL Retrieval Tool window:
   - Select **OCSP** from the dropdown
   - Click **Retrieve**

Expected result: **OK (0) — Certificate is Good**

**OCSP response for the VALID certificate:**
- [X] Good
- [ ] Revoked (unexpected — investigate)
- [ ] Unknown
- [ ] Connection timeout / error

```
Exclude leaf cert:
  Chain: 2d1462d1d3b25d2c89279021e2e2a6525874c27e
Full chain:
  Chain: efe52d9275b99a20c5f2a58a0fcb2b547ea378de
------------------------------------
Verified Issuance Policies: None
Verified Application Policies:
    1.3.6.1.5.5.7.3.1 Server Authentication
Leaf certificate revocation check passed
CertUtil: -verify command completed successfully.
```

### Step 3 — Test the Revoked Certificate from Lab 01

1. Run:

```powershell
certutil -URL <revoked-certificate.cer>
```

2. Select **OCSP** → click **Retrieve**

Expected result: **Certificate is Revoked**

> **If you get "Good" for the revoked certificate:** The OCSP responder is reading a stale CRL that does not yet include the Lab 01 revocation. Run `certutil -CRL` to force a fresh CRL publication, wait 60 seconds, and retest.

**OCSP response for the REVOKED certificate:**
- [X] Revoked
- [ ] Good (unexpected — investigate using the note above)
- [ ] Unknown
- [ ] Connection timeout / error

```
Exclude leaf cert:
  Chain: 1a58a976ff11d81b8fbec4a8625e0d76b14ef84e
Full chain:
  Chain: e423bafb0f1b21c7e511fa65521545daf0598dc7
  Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
  NotBefore: 5/24/2026 12:53 PM
  NotAfter: 4/25/2027 7:36 PM
  Subject: CN=PKI Admin
  Serial: 4400000005b4922c2e24e100eb000000000005
  SubjectAltName: Other Name:Principal Name=pki.admin@corp.cvilab.local
  Template: CVI Code Signing
  Cert: 7fa17dbe13f3c0f3ff3e60af8528c4edb48d96ce
The certificate is revoked. 0x80092010 (-2146885616 CRYPT_E_REVOKED)
------------------------------------
Certificate is REVOKED
Leaf certificate is REVOKED (Reason=5)
CertUtil: -verify command completed successfully.
```

### Step 4 — Run Full Certificate Verification on Both Certificates

1. Run certutil -verify on both certificates and record the output:

```powershell
# Valid certificate — should complete without a revocation error
certutil -verify <valid-certificate.cer>

# Revoked certificate — should fail with revocation error
certutil -verify <revoked-certificate.cer>
```

**certutil -verify output for the VALID certificate:**

```
Issuer:
    CN=CVI Issuing CA 1
    DC=corp
    DC=cvilab
    DC=local
  Name Hash(sha1): 81835e9994945c3166bf7396611ca225004ed32b
  Name Hash(md5): d31087fe673eedadd94c85f2a0bc4782
Subject:
    CN=PKI-SRV01.corp.cvilab.local
  Name Hash(sha1): e769839e105732f62c8e69094e30fa7236489d8b
  Name Hash(md5): be8a0f3f4ca922c0b051445d60d44c8f
Cert Serial Number: 440000000ef02bb109f34551ad00000000000e

dwFlags = CA_VERIFY_FLAGS_CONSOLE_TRACE (0x20000000)
dwFlags = CA_VERIFY_FLAGS_DUMP_CHAIN (0x40000000)
ChainFlags = CERT_CHAIN_REVOCATION_CHECK_CHAIN_EXCLUDE_ROOT (0x40000000)
HCCE_LOCAL_MACHINE
CERT_CHAIN_POLICY_BASE
-------- CERT_CHAIN_CONTEXT --------
ChainContext.dwInfoStatus = CERT_TRUST_HAS_PREFERRED_ISSUER (0x100)
ChainContext.dwRevocationFreshnessTime: 5 Weeks, 12 Hours, 26 Minutes, 32 Seconds

SimpleChain.dwInfoStatus = CERT_TRUST_HAS_PREFERRED_ISSUER (0x100)
SimpleChain.dwRevocationFreshnessTime: 5 Weeks, 12 Hours, 26 Minutes, 32 Seconds

CertContext[0][0]: dwInfoStatus=102 dwErrorStatus=0
  Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
  NotBefore: 5/31/2026 7:07 AM
  NotAfter: 4/25/2027 7:36 PM
  Subject: CN=PKI-SRV01.corp.cvilab.local
  Serial: 440000000ef02bb109f34551ad00000000000e
  SubjectAltName: DNS Name=PKI-SRV01.corp.cvilab.local
  Template: CVI-WebServer
  Cert: ac13703970997b7325d96fda865ddd9c987c69a0
  Element.dwInfoStatus = CERT_TRUST_HAS_KEY_MATCH_ISSUER (0x2)
  Element.dwInfoStatus = CERT_TRUST_HAS_PREFERRED_ISSUER (0x100)
    CRL 15:
    Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
    ThisUpdate: 5/30/2026 12:28 PM
    NextUpdate: 6/7/2026 12:48 AM
    CRL: 6d6e77ff4cc1e1403be1af1adbd5025600cf95d6
  Application[0] = 1.3.6.1.5.5.7.3.1 Server Authentication

CertContext[0][1]: dwInfoStatus=102 dwErrorStatus=0
  Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
  NotBefore: 4/25/2026 7:26 PM
  NotAfter: 4/25/2027 7:36 PM
  Subject: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
  Serial: 5800000002f7714edc7f317c46000000000002
  Template: SubCA
  Cert: 5137a597de2c3085ec5816c7f11edc18cfcdbaf8
  Element.dwInfoStatus = CERT_TRUST_HAS_KEY_MATCH_ISSUER (0x2)
  Element.dwInfoStatus = CERT_TRUST_HAS_PREFERRED_ISSUER (0x100)
    CRL 03:
    Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
    ThisUpdate: 4/25/2026 7:29 PM
    NextUpdate: 10/25/2026 7:49 AM
    CRL: 4a2d0a2156f52deba457fa057b1cb81569838582

CertContext[0][2]: dwInfoStatus=10c dwErrorStatus=0
  Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
  NotBefore: 4/25/2026 6:15 PM
  NotAfter: 4/25/2046 6:25 PM
  Subject: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
  Serial: 26373e51a6ab669340c47caef2232ce1
  Cert: b805e6ab548f6e7c57d3989f61de7fe6a51031d1
  Element.dwInfoStatus = CERT_TRUST_HAS_NAME_MATCH_ISSUER (0x4)
  Element.dwInfoStatus = CERT_TRUST_IS_SELF_SIGNED (0x8)
  Element.dwInfoStatus = CERT_TRUST_HAS_PREFERRED_ISSUER (0x100)

Exclude leaf cert:
  Chain: 7559c6f12759ebe25335501ca7b416d42a935d85
Full chain:
  Chain: a55e339cd90060ad7b8a532e278395992f5a1bd0
------------------------------------
Verified Issuance Policies: None
Verified Application Policies:
    1.3.6.1.5.5.7.3.1 Server Authentication
Leaf certificate revocation check passed
CertUtil: -verify command completed successfully.
```

**certutil -verify output for the REVOKED certificate:**

```
Exclude leaf cert:
  Chain: 1a58a976ff11d81b8fbec4a8625e0d76b14ef84e
Full chain:
  Chain: e423bafb0f1b21c7e511fa65521545daf0598dc7
  Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
  NotBefore: 5/24/2026 12:53 PM
  NotAfter: 4/25/2027 7:36 PM
  Subject: CN=PKI Admin
  Serial: 4400000005b4922c2e24e100eb000000000005
  SubjectAltName: Other Name:Principal Name=pki.admin@corp.cvilab.local
  Template: CVI Code Signing
  Cert: 7fa17dbe13f3c0f3ff3e60af8528c4edb48d96ce
The certificate is revoked. 0x80092010 (-2146885616 CRYPT_E_REVOKED)
------------------------------------
Certificate is REVOKED
Leaf certificate is REVOKED (Reason=5)
CertUtil: -verify command completed successfully.
```

---

## Part E — Inspect the AIA Extension in Context

### Step 1 — Dump a Full Certificate

1. Run a full dump of one of your certificates:

```powershell
certutil -dump <certificate.cer>
```

### Step 2 — Locate the Authority Information Access Section

1. Find the **Authority Information Access** section in the output
2. It contains two distinct entries:

| AIA Entry | Typical URL Format | Purpose |
|---|---|---|
| CA Issuers (caIssuers) | `http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crt` | Tells relying parties where to download the issuing CA's certificate to build the trust chain |
| OCSP (id-ad-ocsp) | `http://pki-srv01.corp.cvilab.local/ocsp` | Used by a relying party to query real-time revocation status via OCSP

> **Why both entries matter:** The CA Issuers URL is used for chain building — if a relying party does not have the issuing CA certificate in its trust store, it downloads it from this URL to complete the chain. The OCSP URL is used for revocation checking. A certificate with a missing or unreachable AIA entry will fail validation in environments that require complete chain building or revocation checking.

**CA Issuers URL:**

```
http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crt
```

**OCSP URL:**

```
http://pki-srv01.corp.cvilab.local/ocsp
```

**Explain in one sentence what each URL is used for by a relying party:**

```
CA Issuers URL: Used by a relying party to download the intermediate CA certificate to build and verify a trusted certificate path.

OCSP URL: Used by a relying party to submit a real-time status request to check if a specific certificate is valid or revoked.
```

---

## Part F — Lab Report

Answer all questions in your own words. Write in complete sentences.

**1. What is the operational difference between a relying party downloading a CRL and sending an OCSP query? From an operational standpoint, what does each approach trade off?**

```
A CRL is a published list of revoked certificates that gets downloaded and cached on the client side. That cuts down on constant network requests, but it also means there’s always a delay between when a certificate is revoked and when that change actually gets enforced.

OCSP, on the other hand, checks revocation status one certificate at a time by querying an Online Responder. The responder answers using the latest CRL it has, so the result is more real-time. The tradeoff is you now depend on the responder being reachable, and the accuracy still ultimately depends on how up to date the CRL is.
```

**2. The OCSP signing certificate has two properties not found on standard end-entity certificates. Name them, give their OIDs, and explain why each is required.**

```
The OCSP Signing Extended Key Usage (1.3.6.1.5.5.7.3.9) allows the certificate to be used specifically for signing OCSP responses. Without this EKU, clients will reject the certificate for OCSP signing because it would not be authorized for that purpose.

The id-pkix-ocsp-nocheck extension (1.3.6.1.5.5.7.48.1.5) tells clients not to check the revocation status of the OCSP signing certificate itself. This prevents a circular dependency where checking the OCSP response would require another OCSP check, which could loop indefinitely.
```

**3. Your Online Responder reads the CA's CRL for its revocation data. What happens to the accuracy of OCSP responses if the CRL becomes stale — specifically, what response would a relying party receive for a certificate that was revoked after the last CRL was published?**

```
If the CRL becomes stale, the OCSP responder will continue using the old data and will incorrectly respond with a status of "Good" for a certificate that was revoked after the last successful CRL publication.
```

**4. If the OCSP responder on PKI-SRV01 became unreachable, and a relying party was configured for hard-fail revocation checking, what would happen to all certificate verifications in your environment — including valid certificates?**

```
If the responder becomes unreachable under a hard-fail configuration, all certificate verifications will fail immediately and block user access, meaning even completely valid, unrevoked certificates will be rejected by the environment.
```

**5. Compare the two test results from Part D: Good for the valid certificate and Revoked for the Lab 01 certificate. What did the OCSP response tell the relying party in real time that a stale CRL could not?**

```
A CRL provides a cached list of all certificates that were revoked as of the last CRL publication time.
An OCSP query checks the revocation status of a specific certificate at the time of the request, using data from the CA’s most recently published CRL.
This makes OCSP more current than relying on a downloaded CRL alone, but it is still limited by how recently the OCSP responder has refreshed its CRL data.
```

---

## Submission Checklist

- [X] Logged in as CORP\pki.admin (not a local account)
- [X] Online Responder role service installed — OCSPSvc status confirmed Running
- [X] Revocation configuration created for CVI Issuing CA 1
- [X] Online Responder snap-in shows green status
- [X] OCSP signing certificate located in the Personal store
- [X] OCSP Signing EKU (1.3.6.1.5.5.7.3.9) confirmed present
- [X] id-pkix-ocsp-nocheck extension (1.3.6.1.5.5.7.48.1.5) confirmed present
- [X] OCSP URL identified from AIA extension
- [X] certutil -URL for valid certificate: Good response documented
- [X] certutil -URL for revoked certificate (from Lab 01): Revoked response documented
- [X] certutil -verify output for both certificates included
- [X] AIA extension entries (CA Issuers URL + OCSP URL) identified and explained
- [X] All five lab report questions answered in complete sentences
- [X] Lab file committed to `labs/week-12/lab-02-ocsp-responder.md`
