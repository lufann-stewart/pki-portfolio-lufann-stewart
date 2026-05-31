# Lab 01: Revoke a Certificate and Observe CRL Propagation

Lufann Stewart  
May 30, 2026    
**Phase:** 2 | **Week:** 12  
**Submission Path:** `labs/week-12/lab-01-revoke-and-crl-propagation.md`

---

## Pre-Lab Verification

If you can log into PKI-SRV01 as CORP\pki.admin, you are communicating with DC01 and the environment is ready. Proceed to Part A.

---

## Part A — Choose Your Certificate

You will revoke one certificate issued during Weeks 10 or 11. The choice of certificate matters — one is reserved for Week 15 and must not be revoked.

### Step 1 — Open the CA Console

1. Press **Win + R**, type `certsrv.msc`, and press **Enter**
2. The Certification Authority console opens
3. In the left pane, expand **CVI Issuing CA 1** → click **Issued Certificates**

### Step 2 — Identify a Certificate to Revoke

1. Browse the list to locate certificates issued in Weeks 10 or 11
2. Right-click the certificate you plan to revoke → **Properties**
3. Note the Common Name, Serial Number, and Certificate Template name
4. Close Properties — do not revoke yet

> **⚠️ Critical — Do NOT select svc.autoenroll:** The `svc.autoenroll` service account certificate is required for Week 15 autoenrollment testing. Revoking it now will break that lab environment. If you accidentally revoke it and the reason code was NOT Key Compromise, you can unrevoce it: go to **Revoked Certificates** → right-click the certificate → **Unrevoke**.

**Recommended revocation candidates (in order of preference):**

| Candidate | Template | Why It's a Good Choice |
|---|---|---|
| Code signing cert issued to CORP\pki.admin | CVI-CodeSigning | Best choice — no forward dependency in any remaining week |
| TLS certificate from Week 10 | CVI-WebServer | Acceptable |
| Any other issued cert that is not svc.autoenroll | Any | Acceptable |

### Step 3 — Record the Certificate Details

1. From an elevated PowerShell prompt, retrieve the serial number of your chosen certificate:

```powershell
certutil -view -restrict "CommonName=<cn-of-your-cert>" -out "RequestID,SerialNumber,CommonName,NotAfter"
```

**Certificate selected for revocation:**

```
Common Name / Subject: CN = PKI Admin
```

**Template name:**

```
CVICodeSigning
```

**Serial number:**

```
4400000005b4922c2e24e100eb000000000005
```

**Why you selected this certificate (and confirmed it is not svc.autoenroll):**

```
I selected the Code Signing certificate because it can be revoked without affecting future labs. I confirmed that it was not the svc.autoenroll certificate, which is required for later testing. I also chose it because a Code Signing certificate is a very important certificate that could be misused to sign software or scripts if it were ever compromised.

```

---

## Part B — Revoke the Certificate

### Step 1 — Initiate the Revocation

1. In certsrv.msc → **Issued Certificates**, locate your chosen certificate in the list
2. Right-click the individual certificate row → **All Tasks → Revoke Certificate**

> **If the option is greyed out:** You right-clicked the Issued Certificates folder rather than an individual certificate row. Click the folder to open the list, then right-click a specific certificate row. Also confirm you are logged in as CORP\pki.admin, not a local account.

### Step 2 — Choose a Revocation Reason Code

1. The Revoke Certificate dialog opens with a dropdown for the reason code
2. Review the reason codes and choose the most appropriate one for your scenario:

| Reason Code | When to Use | Urgency | Reversible? |
|---|---|---|---|
| Key Compromise | Private key was exposed or stolen | Immediate — publish CRL now | No |
| CA Compromise | CA private key was exposed | Immediate + CA rebuild required | No |
| Affiliation Changed | Employee left or changed roles | Same-day | Yes |
| Superseded | Replacing with a new certificate | Next scheduled CRL cycle | Yes |
| Cessation of Operation | Service or system decommissioned | Next scheduled CRL cycle | Yes |

> **Why reason codes matter:** Key Compromise is the only code that requires immediate CRL publication regardless of the normal schedule. It also cannot be reversed — once a certificate is revoked for Key Compromise, it cannot be unrevoked. All other reason codes can be reversed from the Revoked Certificates view if revoked by mistake (except CA Compromise).

3. Select your reason code from the dropdown

### Step 3 — Confirm the Revocation

1. Click **Yes** to confirm
2. The certificate immediately moves from Issued Certificates to **Revoked Certificates**

### Step 4 — Record the Revocation Details

1. Click **Revoked Certificates** in the left pane
2. Locate your certificate and note the timestamp shown in the **Revocation Date** column

**Revocation reason code selected:**

| Reason Code | Selected? |
|---|---|
| Key Compromise | No |
| CA Compromise | No |
| Affiliation Changed | No |
| Superseded | No |
| Cessation of Operation | Yes |

**Why you chose this reason code:**

```
The certificate is no longer needed and is being retired as 
part of lab cleanup. There is no suspected key compromise.
```

**Certificate appears in Revoked Certificates view:**
- [X] Yes
- [ ] No — describe:

**Timestamp of revocation (from Revoked Certificates view):**

```
5/30/2026 11:10 AM
```

---

## Part C — Publish the CRL

After marking the certificate revoked, the revocation is internal to the CA. Relying parties have no way to know about it until a new CRL is published and distributed. This step makes the revocation visible to the outside world.

### Step 1 — Publish a New CRL from the CA Console

1. In certsrv.msc, right-click **CVI Issuing CA 1** — the CA name itself in the left pane, not a subfolder
2. Select **All Tasks → Publish**
3. In the Publish CRL dialog, select **New CRL** → click **OK**

> **Why "New CRL" and not "Delta CRL":** A New CRL is a complete, standalone revocation list. A Delta CRL only contains changes since the last Base CRL, so relying parties must have the Base CRL to use it. For a fresh lab environment where you want the revocation to be immediately verifiable without dependencies, publish a New CRL.

### Step 2 — Confirm Publication from PowerShell

1. In your elevated PowerShell prompt, run:

```powershell
certutil -CRL
```

Expected output:
```
CRL published successfully.
CertUtil: -CRL command completed successfully.
```

> **If you see "Access is denied":** You are not running from an elevated prompt, or you are not logged in as CORP\pki.admin. Right-click PowerShell → **Run as Administrator**. Confirm the session is CORP\pki.admin (not a local account) with `whoami`.
/
```
CertUtil: -CRL command completed successfully.
```

**CRL published without errors:**
- [X] Yes
- [ ] No — describe the error:

---

## Part D — Locate and Inspect the CRL

### Step 1 — Export the Revoked Certificate

1. In certsrv.msc → **Revoked Certificates**, right-click your certificate → **Export**
2. Save it as `revoked.cer` — you will use this file in Parts D and E

### Step 2 — Find the CRL Distribution Point URL in the Certificate

1. Run the following to extract the CDP URL from the certificate:

```powershell
certutil -dump revoked.cer | findstr "CDP http ldap"
```

The CDP URL in your environment will typically be in this format:
```
http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl
```

There may also be an LDAP CDP entry for Active Directory-based distribution.

**CRL Distribution Point URL found in the certificate:**

```
ldap:///CN=CVI Issuing CA 1,CN=PKI-SRV01,CN=CDP,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=cvilab,DC=local?certificateRevocationList?base?objectClass=cRLDistributionPoint
```

### Step 3 — Inspect the CRL on Disk

1. Dump the CRL and examine the key fields:

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl"
```

> You can also download it via the HTTP CDP URL and inspect the downloaded file:
> ```powershell
> Invoke-WebRequest -Uri "<CDP-URL>" -OutFile downloaded.crl
> certutil -dump downloaded.crl
> ```

**Record the key fields from the CRL output:**

```
ThisUpdate: 5/30/2026 11:06 AM
```

```
NextUpdate: 6/6/2026 11:26 PM
```

> **ThisUpdate** is when this CRL was published — it should closely match the time you ran `certutil -CRL` in Part C. **NextUpdate** is when this CRL expires. Relying parties must download a fresh CRL before this time. With default settings, NextUpdate will be approximately one week from ThisUpdate.

### Step 4 — Confirm Your Revoked Certificate Appears in the CRL

1. Search the CRL for your certificate's serial number:

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl" | findstr /i "<your-serial-number>"
```

```
Serial Number: 4400000005b4922c2e24e100eb000000000005
```

> **If the serial number does not appear:** You most likely published the CRL before completing the revocation, or you are looking at the wrong CRL file. Confirm the revocation appears in the Revoked Certificates view in certsrv.msc, then republish with `certutil -CRL`. Use the CDP URL from the certificate itself to confirm you are inspecting the correct CRL file.

**Your revoked certificate's serial number appears in the CRL:**
- [X] Yes
- [ ] No — describe what you see and what you suspect:

---

## Part E — Verify Revocation Status

### Step 1 — Run Certificate Verification

1. Run a full certificate verification against the revoked certificate:

```powershell
certutil -verify revoked.cer
```

### Step 2 — Clear the CRL Cache If Needed

1. If verification still shows the certificate as valid, the local CRL cache may be serving a stale copy. Clear it and retry:

```powershell
certutil -urlcache CRL delete
certutil -verify revoked.cer
```

**Expected output when revocation is confirmed:**
```
ERROR: Verifying leaf certificate revocation status returned The certificate is revoked.
0x80092010 (-2146885616 CRYPT_E_REVOKED)
CertUtil: -verify command failed. 0x80092010 (-2146885616 CRYPT_E_REVOKED)
```

> **If you see CRYPT_E_REVOCATION_OFFLINE instead:** The CRL is not accessible at the CDP URL — check network connectivity within the lab environment and confirm the CRL file is in `C:\Windows\System32\CertSrv\CertEnroll\`. Either output is acceptable for this lab if you include a written explanation of what you observed.

```
------------------------------------
Certificate is REVOKED
Leaf certificate is REVOKED (Reason=5)
CertUtil: -verify command completed successfully.
```

**certutil -verify reports the certificate as revoked:**
- [X] Yes — paste the relevant error text:
- [ ] No — describe what you see and what you tried:

---

## Part F — Lab Report

Answer all questions in your own words. Write in complete sentences.

**1. Walk through the two-step revocation workflow you performed. What happens at each step — and why is the CRL publication step not optional even when the certificate has already been revoked in the CA?**

```
First, in certsrv.msc I went to Issued Certificates. I found the code signing certificate I wanted to revoke, right-clicked it, and selected “Revoke Certificate.” I chose “Cease of Operation” as the reason code and confirmed it. This updated the CA database and moved the certificate into the Revoked Certificates folder.

Next, still in certsrv.msc, I published a new CRL so the revocation would be visible to other systems. I right-clicked the Revoked Certificates folder, went to All Tasks → Publish, and selected “New CRL.” After that, I ran certutil -CRL in PowerShell to confirm the CRL was generated and published successfully.

That second step is required because revoking the certificate in certsrv.msc only updates the CA itself. Clients don’t check the CA directly — they check the CRL. Without publishing the updated CRL, the revocation wouldn’t be enforced outside of the CA.
```

**2. You chose a specific revocation reason code. What would have changed operationally if you had chosen Key Compromise instead — particularly regarding CRL publication timing and whether the revocation could be reversed?**

```
If Key Compromise had been selected, the revocation would indicate that the private key may have been exposed, which is a high-severity security event. Operationally, this means the certificate must be treated as immediately untrusted and should not be reversed. However, CRL publication still works the same way; the difference is urgency and security impact, not the CRL mechanism itself.
```

**3. What does the NextUpdate field in the CRL tell a relying party? If a relying party tries to verify your revoked certificate after NextUpdate has passed and no new CRL has been published, what happens?**

```
The NextUpdate field tells a relying party when the current CRL expires and when a new CRL must be retrieved. If the relying party checks revocation after NextUpdate has passed and no new CRL is available, the certificate may be treated as untrusted or revocation checks may fail depending on the system configuration, potentially blocking validation or triggering an offline revocation error.
```

**4. A relying party in a soft-fail environment had already cached the CRL before you published the updated one in Part C. Would they know the certificate is revoked? Why or why not — and what mechanism exists to reduce this propagation lag?**

```
No, the relying party would not know the certificate is revoked because it is using a cached CRL that does not contain the updated revocation entry. Soft-fail systems will continue trusting cached revocation data until it expires. The lag is reduced by using shorter CRL lifetimes, Delta CRLs, and OCSP, which provide more frequent or real-time revocation checking compared to full CRL downloads.
```

**5. What is one thing you would do differently or check additionally in a production environment that you did not need to do in this lab?**

```
In a production environment, I would verify that both CRL and OCSP are properly configured and reachable from all client systems. I would also confirm that HTTP CDP paths are available and that CRL expiration intervals are appropriate to avoid revocation checking gaps.
```

---

## Submission Checklist

- [X] Logged in as CORP\pki.admin (not a local account)
- [X] Certificate selected — NOT svc.autoenroll — serial number documented
- [X] Revocation reason code chosen with written rationale
- [X] Certificate confirmed in Revoked Certificates view with timestamp
- [X] CRL published — certutil -CRL output shows "CRL published successfully"
- [X] CRL Distribution Point URL identified from the certificate
- [X] CRL inspected: ThisUpdate and NextUpdate recorded
- [X] Revoked serial number confirmed present in the CRL
- [X] certutil -verify shows CRYPT_E_REVOKED (0x80092010) or CRYPT_E_REVOCATION_OFFLINE with explanation
- [X] All five lab report questions answered in complete sentences
- [X] Lab file committed to `labs/week-12/lab-01-revoke-and-crl-propagation.md`
