# Lab 02: CA Health Check Routine

Lufann Stewart  
**Date Completed:**  
**Phase:** 2 | **Week:** 14  
**Submission Path:** `labs/week-14/lab-02-ca-health-check.md`

---

## Overview

In this lab, you run a structured CA health check on PKI-SRV01 covering all three operational health signals: CRL freshness, OCSP availability, and the certificate expiration pipeline. You will use two tools: **pkiview.msc** (the Enterprise PKI snap-in) for a hierarchy-wide visual status check, and **certutil** for precision, forced publication, OCSP accuracy testing, and the expiration pipeline that pkiview cannot see.

The output is a completed health check report with a pass/fail status for each signal — a reusable procedure you can apply to any AD CS environment.

**Prerequisite:** Part B (OCSP testing) requires a revoked certificate in your CA database. If you have not revoked a certificate since Week 12, revoke one of the template certificates from Week 10 Lab 01 before proceeding to Part B.

---

## Lab Environment

| Component | Details |
|-----------|---------|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| OCSP Endpoint | `http://pki.corp.cvilab.local/ocsp` |
| CRL HTTP Path | `http://pki.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl` |
| Login Account | CORP\pki.admin |

---

## Pre-Lab Check

### Step 1 — Confirm Login and CA Status

```powershell
whoami
Get-Service CertSvc
certutil -ping
```

```
(paste all three outputs here)
```

**CA is running and responding:**
- [ ] Yes — proceed to Pre-Lab Step 2
- [ ] No — resolve before continuing

### Step 2 — Open pkiview.msc and Record Initial Status

Before running any certutil commands, open pkiview.msc and read the PKI hierarchy.

**To open pkiview.msc:**
```
Option 1: Windows + R → type pkiview.msc → Enter
Option 2: Server Manager → Tools → Enterprise PKI
```

Expand the full tree: **Enterprise PKI → CVI Root CA → CVI Issuing CA 1**

For each node listed below, record the color indicator you see (Green / Amber / Red) and a brief description of what it shows:

| Node | Color | What It Shows |
|---|---|---|
| CVI Root CA — CA Certificate | | |
| CVI Issuing CA 1 — CA Certificate | | |
| CDP row(s) — CRL Distribution Point | | |
| AIA row(s) — Authority Information Access | | |

```
(describe what you see in pkiview — which nodes are expanded, any amber or red indicators, URLs shown)
```

**pkiview initial status:**
- [ ] All green — no issues visible before running certutil
- [ ] One or more amber indicators — note which node(s):
- [ ] One or more red indicators — note which node(s):

### Step 3 — Confirm a Revoked Certificate Exists (Required for Part B)

```powershell
certutil -view -restrict "Disposition=21"
```

```
(paste output here)
```

**At least one revoked certificate exists in the database:**
- [ ] Yes — serial number to use in Part B: `___________________`
- [ ] No — revoke a certificate now before proceeding:
  ```powershell
  # Find an issued certificate's serial number first
  certutil -view -restrict "Disposition=20" -out "SerialNumber,CommonName"
  # Then revoke it:
  certutil -revoke <serial_number> 5
  ```
  Revoked serial number: `___________________`

---

## Part A — CRL Freshness Check

### Step 1 — Read pkiview CDP Status Before Publishing

You already recorded the CDP color in the Pre-Lab. Before running certutil -CRL, note what pkiview shows:

CDP row color before certutil -CRL: **___________________**

URL shown in the CDP row (hover or right-click → Properties to see the URL):
```
(record the CDP URL pkiview is testing)
```

### Step 2 — Publish a Fresh CRL

Run from an elevated PowerShell prompt on PKI-SRV01:

```powershell
certutil -CRL
```

**Expected output:**
```
CertUtil: -CRL command completed successfully.
```

```
(paste certutil -CRL output here)
```

**certutil -CRL completed without errors:**
- [ ] Yes
- [ ] No — error message:

**After certutil -CRL, return to pkiview.msc.** You may need to right-click the Issuing CA node and select Refresh.

CDP row color after certutil -CRL: **___________________**

### Step 3 — Dump the CRL and Read the Validity Timestamps

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl"
```

> **If the file name is different:** Check the CertEnroll folder for the .crl file name.
> ```powershell
> dir "C:\Windows\System32\CertSrv\CertEnroll\" | Where-Object {$_.Name -like "*.crl"}
> ```

```
(paste certutil -dump output here — first 40 lines sufficient)
```

**Record the key timestamps from the output:**

ThisUpdate (when this CRL was published):
```
(enter timestamp)
```

NextUpdate (when this CRL expires):
```
(enter timestamp)
```

**Calculate time remaining until CRL expiry:**

Current date/time (run `Get-Date`):
```
(enter current date/time)
```

Hours until NextUpdate:
```
(calculate: NextUpdate minus current time — show your calculation)
```

**CRL freshness status:**
- [ ] PASS — more than 48 hours until NextUpdate
- [ ] WARNING — between 24 and 48 hours until NextUpdate
- [ ] CRITICAL — less than 24 hours until NextUpdate
- [ ] EXPIRED — NextUpdate has already passed

### Step 4 — Test CRL HTTP Accessibility

```powershell
certutil -URL "http://pki.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl"
```

> **Note:** Replace the URL with the actual CDP path from your CA configuration if different. You can find it in the CA Properties → Extensions tab in the Certification Authority console.

```
(paste certutil -URL output here)
```

**CRL HTTP accessibility result:**
- [ ] VERIFIED — CRL is accessible at the HTTP CDP
- [ ] FAILED — error message:

**Compare certutil -URL result with pkiview CDP color — do they agree?**
- [ ] Yes — both show healthy / both show problem
- [ ] No — describe the difference:

**Part A Summary:**

| Check | Tool Used | Result | Status |
|---|---|---|---|
| CDP color before publishing | pkiview.msc | Green / Amber / Red | — |
| certutil -CRL published without error | certutil | | PASS / FAIL |
| CDP color after publishing | pkiview.msc | Green / Amber / Red | — |
| CRL NextUpdate timestamp | certutil -dump | | — |
| Hours until CRL expiry | certutil -dump | | PASS / WARN / CRITICAL |
| HTTP CDP accessibility | certutil -URL | VERIFIED / FAILED | PASS / FAIL |

---

## Part B — OCSP Availability Check

### Step 1 — Read pkiview AIA Status

Before running certutil, record what pkiview shows for the AIA row:

AIA row color in pkiview: **___________________**

URL shown in the AIA row:
```
(record the AIA/OCSP URL pkiview is testing)
```

**pkiview AIA status interpretation:**
- [ ] Green — OCSP endpoint appears reachable per pkiview
- [ ] Amber or Red — pkiview flagged an issue; note:

> **Important:** A green AIA row in pkiview confirms the endpoint is reachable. It does **not** confirm the OCSP responder is returning accurate revocation data. Steps 2 and 3 test accuracy — pkiview cannot do this.

### Step 2 — Test OCSP With a Valid Certificate

Find a currently issued (not revoked) certificate from your database and test its OCSP status.

```powershell
# Get serial numbers of issued certificates
certutil -view -restrict "Disposition=20" -out "SerialNumber,CommonName"
```

```
(paste output here — use to identify a valid certificate serial number)
```

Serial number of valid certificate to test: `___________________`
Certificate CN: `___________________`

Export the certificate from the Certification Authority MMC (right-click issued cert → Open → Details → Copy to File → save as .cer), then test:

```powershell
certutil -URL <path-to-valid-cert.cer>
```

```
(paste certutil -URL output for valid certificate here)
```

**OCSP response for valid certificate:**
- [ ] GOOD — certificate status returned as good/valid
- [ ] Unexpected response — describe:

### Step 3 — Test OCSP With a Revoked Certificate

Use the serial number of the revoked certificate identified in the pre-lab check.

Export the revoked certificate from the Certification Authority MMC (Revoked Certificates folder → right-click → Open → Details → Copy to File → save as .cer), then test:

```powershell
certutil -URL <path-to-revoked-cert.cer>
```

```
(paste certutil -URL output for revoked certificate here)
```

**OCSP response for revoked certificate:**
- [ ] REVOKED — certificate correctly identified as revoked
- [ ] Unexpected response — describe:

> **If OCSP returns GOOD for the revoked certificate:** The OCSP responder is reading stale CRL data. Run certutil -CRL to republish, wait 30 seconds, and retest. pkiview showed the AIA row as green even while this was happening — this is the limit of what pkiview can tell you about OCSP health.

### Step 4 — Compare pkiview and certutil for OCSP

**pkiview AIA row status:** ___________________

**certutil OCSP result for valid cert:** ___________________

**certutil OCSP result for revoked cert:** ___________________

**Did pkiview's AIA status match what certutil confirmed about OCSP accuracy?**
- [ ] Yes — pkiview green and certutil confirmed accurate responses
- [ ] Partially — pkiview green but certutil revealed an accuracy issue
- [ ] No — describe:

**Part B Summary:**

| Check | Tool Used | Result | Status |
|---|---|---|---|
| AIA/OCSP endpoint color | pkiview.msc | Green / Amber / Red | — |
| Valid cert returns GOOD status | certutil -URL | | PASS / FAIL |
| Revoked cert returns REVOKED status | certutil -URL | | PASS / FAIL |

> **If OCSP fails for the revoked certificate:** Confirm the CRL was published after the revocation (certutil -CRL), and confirm the OCSP responder is configured to read the current CRL. The OCSP responder uses the CRL as its revocation data source.

---

## Part C — Certificate Expiration Pipeline

> **Note:** pkiview does not show individual issued certificate expiry. This entire part uses certutil only.

### Step 1 — Calculate Target Dates

```powershell
# Get current date and calculate expiry windows
$today  = Get-Date
$date30 = $today.AddDays(30).ToString("M/d/yyyy")
$date60 = $today.AddDays(60).ToString("M/d/yyyy")
$date90 = $today.AddDays(90).ToString("M/d/yyyy")

Write-Host "Today:   $today"
Write-Host "30 days: $date30"
Write-Host "60 days: $date60"
Write-Host "90 days: $date90"
```

```
(paste output here — record your target dates)
```

Today's date: `___________________`
30-day threshold: `___________________`
60-day threshold: `___________________`
90-day threshold: `___________________`

### Step 2 — 30-Day Expiry Query (Action Required Window)

```powershell
certutil -view -restrict "Disposition=20","NotAfter<=$date30" -out "RequestID,CommonName,RequesterName,CertificateTemplate,NotAfter"
```

```
(paste output here — or "No results" if no certificates expire within 30 days)
```

**Certificates expiring within 30 days:**
- [ ] None found — PASS
- [ ] Found — count: ___ — list CNs:

### Step 3 — 60-Day Expiry Query (Outreach Window)

```powershell
certutil -view -restrict "Disposition=20","NotAfter<=$date60" -out "RequestID,CommonName,RequesterName,CertificateTemplate,NotAfter"
```

```
(paste output here)
```

**Certificates expiring within 60 days:**
- [ ] None found — PASS
- [ ] Found — count: ___ — list CNs:

### Step 4 — 90-Day Expiry Query (Awareness Window)

```powershell
certutil -view -restrict "Disposition=20","NotAfter<=$date90" -out "RequestID,CommonName,RequesterName,CertificateTemplate,NotAfter"
```

```
(paste output here)
```

**Certificates expiring within 90 days:**
- [ ] None found — PASS
- [ ] Found — count: ___ — list CNs:

### Step 5 — Check for Already-Expired Certificates

```powershell
certutil -view -restrict "Disposition=20","NotAfter<=$today" -out "RequestID,CommonName,NotAfter"
```

```
(paste output here)
```

**Already-expired certificates with Disposition=20 (should be zero in a healthy CA):**
- [ ] None — PASS
- [ ] Found — count: ___ (these certificates should be revoked if still showing as Issued)

**Part C Summary:**

| Window | Count | Status |
|---|---|---|
| Already expired (should be 0) | | PASS / CRITICAL |
| Expiring within 30 days | | PASS / ACTION |
| Expiring within 60 days | | PASS / WARN |
| Expiring within 90 days | | PASS / MONITOR |

---

## Part D — Health Check Summary Report

Complete the full health check summary table. This is the deliverable that makes the health check reusable.

**Health Check Report — PKI-SRV01 / CVI Issuing CA 1**

Date of health check: `___________________`
Conducted by: `___________________`
pkiview.msc initial status (describe what you saw before running certutil): `___________________`

| Signal | Check | Tool | Result | Status |
|---|---|---|---|---|
| **CRL Freshness** | CDP color before publishing | pkiview | Green / Amber / Red | — |
| | CRL published without error | certutil -CRL | | PASS / FAIL |
| | CDP color after publishing | pkiview | Green / Amber / Red | — |
| | HTTP CDP accessible (VERIFIED) | certutil -URL | | PASS / FAIL |
| | Hours until CRL NextUpdate | certutil -dump | | PASS / WARN / CRITICAL |
| **OCSP Availability** | AIA/OCSP endpoint color | pkiview | Green / Amber / Red | — |
| | Valid cert returns GOOD | certutil -URL | | PASS / FAIL |
| | Revoked cert returns REVOKED | certutil -URL | | PASS / FAIL |
| **Expiration Pipeline** | No certs already expired | certutil -view | | PASS / FAIL |
| | 30-day expiry count | certutil -view | | PASS / ACTION |
| | 60-day expiry count | certutil -view | | PASS / WARN |
| | 90-day expiry count | certutil -view | | PASS / MONITOR |

**Overall CA health status:**
- [ ] Healthy — all signals PASS
- [ ] Attention needed — one or more signals WARNING
- [ ] Action required — one or more signals CRITICAL or ACTION

**Summary of any findings requiring follow-up:**
```
(list any findings here, or "No follow-up required — all signals healthy")
```

---

## Health Check Procedure (Reusable)

Write the complete health check as a repeatable procedure — the steps another administrator could follow to run the same check on any AD CS issuing CA. Include both pkiview.msc and certutil steps in the correct order.

```
(document the procedure as a numbered list — cover all three signals, both tools, and note where pkiview is used vs. where certutil is required)
```

---

## Lab Report Questions

**1. You opened pkiview.msc before running any certutil commands. Describe one thing pkiview told you that you could not have known from certutil alone, and one thing certutil told you that pkiview cannot show. What does this tell you about the role of each tool in a CA health check?**

```
(your answer here)
```

**2. In Part B, you tested OCSP with both a valid and a revoked certificate. pkiview showed the AIA row as green — meaning the OCSP endpoint was reachable. Why is testing with a known-revoked certificate a required step that pkiview cannot replace? What would it mean operationally if the revoked certificate returned a GOOD status instead of REVOKED?**

```
(your answer here)
```

**3. In Part C, you checked for certificates expiring within 30, 60, and 90 days. pkiview does not show this data. AD CS does not generate any automatic alerts. Given this, what operational discipline is required to prevent a certificate expiry from becoming a service outage — and what would a mature weekly health check routine look like for a CA with 200 issued certificates?**

```
(your answer here)
```

**4. If you were setting up this health check to run automatically on a weekly schedule, which signal would you consider most urgent to monitor — CRL freshness, OCSP availability, or the expiration pipeline? Explain your reasoning, including what failure in that signal would look like within your first hour of not catching it.**

```
(your answer here)
```

**5. pkiview.msc has been a standard tool in Windows Server AD CS since 2003. Enterprise CLM platforms like Keyfactor provide richer dashboards that automate much of what you did manually in this lab. Based on what you observed in this lab, what does pkiview show that a newer platform dashboard would also need to show — and what would a platform need to add to go beyond what pkiview and certutil provide manually?**

```
(your answer here)
```

---

## Submission Checklist

- [ ] Logged in as CORP\pki.admin — whoami output included
- [ ] CA running and responding — Get-Service and certutil -ping output included
- [ ] pkiview.msc opened — initial status of all nodes recorded in Pre-Lab Step 2
- [ ] Revoked certificate confirmed or created — serial number recorded
- [ ] Part A: CDP color in pkiview recorded before and after certutil -CRL
- [ ] certutil -CRL output included — completed without errors
- [ ] certutil -dump on CRL included — ThisUpdate and NextUpdate recorded
- [ ] Hours until CRL expiry calculated and documented
- [ ] CRL HTTP accessibility tested — certutil -URL output included
- [ ] pkiview vs. certutil comparison completed for Part A
- [ ] Part A summary table completed
- [ ] Part B: AIA color in pkiview recorded and URL noted
- [ ] OCSP tested with valid certificate — certutil -URL output and GOOD response documented
- [ ] OCSP tested with revoked certificate — certutil -URL output and REVOKED response documented
- [ ] pkiview vs. certutil comparison completed for Part B
- [ ] Part B summary table completed
- [ ] Target dates (30/60/90) calculated and recorded
- [ ] All three expiry query outputs included
- [ ] Already-expired certificate check completed
- [ ] Part C summary table completed
- [ ] Part D health check summary table completed with pkiview initial status and overall status
- [ ] Reusable health check procedure written — includes both pkiview and certutil steps
- [ ] All five lab report questions answered in complete sentences
- [ ] File committed to `labs/week-14/lab-02-ca-health-check.md`
