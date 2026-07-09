# Lab 01: AD CS Event Log Navigation

Lufann Stewart    
July 9, 2026  
**Phase:** 2 | **Week:** 14  
**Submission Path:** `labs/week-14/lab-01-event-log-navigation.md`

---

## Overview

In this lab, you navigate the two data sources that record what your CA has been doing: the Windows Application event log and the CA certificate database. Both contain a record of the operations you performed across Weeks 10 through 13 — certificate issuance, revocation, CRL publication, and CA service lifecycle events.

This lab has three parts. Part A navigates the Application event log on PKI-SRV01. Part B queries the CA certificate database using certutil -view. Part C asks you to connect both sources to reconstruct an operational picture of the CA.

**No new CA operations are required.** The data you need already exists from prior lab work.

---

## Lab Environment

| Component | Details |
|-----------|---------|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| Login Account | CORP\pki.admin |
| Domain | corp.cvilab.local |

---

## Pre-Lab Check

### Step 1 — Confirm Login

```powershell
whoami
```

**Expected:** `corp\pki.admin`

```
corp\pki.admin
```

### Step 2 — Confirm CA Is Running

```powershell
Get-Service CertSvc
certutil -ping
```

```
Status   Name               DisplayName                           
------   ----               -----------                           
Running  CertSvc            Active Directory Certificate Services 
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (0ms)
CertUtil: -ping command completed successfully.
```

**CA is running and responding:**
- [X] Yes — proceed to Part A
- [ ] No — describe issue:

---

## Part A — Application Event Log Navigation

The Windows Application event log records CA operational events by default — no configuration required. You will create a filtered view and document the CA event history on PKI-SRV01.

### Step 1 — Open Event Viewer

Log into PKI-SRV01. Open **Event Viewer**:
- Start → search "Event Viewer" → Open
- Or run: `eventvwr.msc`

### Step 2 — Navigate to the Application Log

In the left panel: **Windows Logs → Application**

### Step 3 — Create a Filtered View

Right-click **Application** → **Filter Current Log**

In the filter dialog:
- **Event sources:** Type `CertificationAuthority` and select it
- Leave all other fields at default
- Click **OK**

> **To save this filter as a Custom View (recommended):**
> Right-click **Custom Views** in the left panel → **Create Custom View** → same filter settings → Name it "CA Events" → OK

### Step 4 — Document Your Findings

Review the filtered event list. Identify at least **three distinct event types** present in your log. For each event, record the information below.

---

**Event 1**

Event ID:
```
17
```

Timestamp:
```
7/1/2026 2:16:19 PM
```

Source:
```
CertificationAuthority
```

Event Description (copy from the General tab of the event):
```
Active Directory Certificate Services did not start: Unable to initialize the database connection for CVI Issuing CA 1.  File not found 0xc8000713 (ESE: -1811 JET_errFileNotFound).
```

What this event represents (in your own words):
```
The files in the certlog folder were not found therefore the database could not initazlize when AD CS was attempting to be started.
```

---

**Event 2**

Event ID:
```
38
```

Timestamp:
```
6/30/2026 12:04:11 PM
```

Event Description:
```
Active Directory Certificate Services for CVI Issuing CA 1 was stopped.
```

What this event represents:
```
AD CS services were stopped using command more than likely.
```

---

**Event 3**

Event ID:
```
94
```

Timestamp:
```
5/31/2026 3:03:17 PM
```

Event Description:
```
Active Directory Certificate Services CVI Issuing CA 1 can not open the certificate store at CN=NTAuthCertificates,CN=Public Key Services,CN=Services in the Active Directory's configuration container.
```

What this event represents:
```
Certificate Store was unable to be opened maybe due to an error of some kind.
```

---

**Additional events (optional — document as many as you find useful):**

```
Event ID: 44  
Logged: 5/29/2026 4:08:27 PM
Description:  The "Windows default" Policy Module "Initialize" method returned an error. The specified domain either does not exist or could not be contacted. The returned status code is 0x8007054b (1355).  The Active Directory containing the Certification Authority could not be contacted.
................
```

### Step 5 — Event Type Summary

Based on your filtered view, complete this table:

| Event Type | Present? | Approximate Count |
|---|---|---|
| CRL publication events | No | 0 |
| CA service start/stop events | Yes | 68 |
| Certificate template update events | No | 0 |
| CA configuration change events | Yes | 1 |
| Error events (Level = Error) | Yes / No | 39 |

---

## Part B — CA Certificate Database Query

The CA certificate database contains every certificate request, issuance, and revocation since the CA was stood up. You will query it using certutil -view from an elevated PowerShell prompt on PKI-SRV01.

### Step 1 — Open an Elevated PowerShell Prompt

Right-click **Windows PowerShell** → **Run as Administrator**

Confirm: `whoami` returns `corp\pki.admin`

### Step 2 — Query All Issued Certificates

```powershell
certutil -view -restrict "Disposition=20"
```

This returns all certificates with Disposition = 20 (Issued and active).

```
(paste certutil -view output here — all issued certificates)
```

**Total number of issued certificates found:**
```
(enter count)
```

### Step 3 — Query All Revoked Certificates

```powershell
certutil -view -restrict "Disposition=21"
```

```
(paste certutil -view output here — revoked certificates)
```

**Total number of revoked certificates found:**
```
(enter count)
```

### Step 4 — Find Certificates From Your Prior Labs

Use the requester filter to find certificates associated with the accounts used in your Week 10 and 11 labs.

```powershell
certutil -view -restrict "RequesterName=CORP\pki.admin"
```

```
(paste output here)
```

If you also used CORP\cert.manager:
```powershell
certutil -view -restrict "RequesterName=CORP\cert.manager"
```

```
(paste output here)
```

### Step 5 — Document Two Specific Certificate Records

Identify at least **two specific certificates** from your prior lab work (Weeks 10 or 11). For each, record the full certificate record fields.

---

**Certificate Record 1**

Request ID:
```
(enter Request ID)
```

Requester Name:
```
(enter CORP\account)
```

Certificate Template:
```
(enter template name)
```

Issued Common Name:
```
(enter CN)
```

Not Before:
```
(enter validity start)
```

Not After:
```
(enter expiry date)
```

Serial Number:
```
(enter hex serial)
```

Disposition:
```
20 (Issued) — or — 21 (Revoked)
```

Revocation Date (if revoked):
```
(enter if applicable)
```

Revocation Reason (if revoked):
```
(enter if applicable)
```

**Which lab did this certificate come from?**
```
(e.g., Week 10 Lab 01, Week 11 Lab 02)
```

---

**Certificate Record 2**

Request ID:
```
(enter Request ID)
```

Requester Name:
```
(enter CORP\account)
```

Certificate Template:
```
(enter template name)
```

Issued Common Name:
```
(enter CN)
```

Not Before:
```
(enter validity start)
```

Not After:
```
(enter expiry date)
```

Serial Number:
```
(enter hex serial)
```

Disposition:
```
20 (Issued) — or — 21 (Revoked)
```

**Which lab did this certificate come from?**
```
(e.g., Week 10 Lab 01, Week 11 Lab 02)
```

---

### Step 6 — Filter by Template

Run the following to see how certificates are distributed across templates:

```powershell
# List all issued certificates showing only template and CN
certutil -view -restrict "Disposition=20" -out "CertificateTemplate,CommonName,NotAfter"
```

```
(paste output here)
```

**Templates represented in your CA database:**
```
(list the template names you see)
```

---

## Part C — Analysis

Answer the following questions using the event log data from Part A and the certificate database data from Part B.

### Analysis Question 1

Select **one event log entry** from Part A (any event type). In 3–5 sentences, explain what this event tells you about the CA's operational state at the time it was generated. Be specific about what action caused the event and what the event confirms about the CA.

```
(your answer here — minimum 3 sentences)
```

### Analysis Question 2

Select **one certificate record** from Part B. In 3–5 sentences, explain what this record tells you about the lifecycle of that specific certificate. Include the template used, who requested it, its current status, and what you would need to do next if the certificate were approaching expiry.

```
(your answer here — minimum 3 sentences)
```

### Analysis Question 3

In 4–6 sentences, explain how the Application event log and the CA certificate database complement each other as operational data sources. Give a specific example scenario where you would need to consult **both** sources to fully understand what happened — and explain why one source alone would be insufficient.

```
(your answer here — minimum 4 sentences)
```

---

## Lab Report Questions

Answer each question in complete sentences.

**1. What event type did you find most frequently in the Application event log? What does the frequency of that event type tell you about the CA's most common operation in your lab environment?**

```
(your answer here)
```

**2. The certutil -view command queries the CA database, not the event log. What is the difference between what these two sources record — and what would you lose operationally if you had access to the event log but not the CA database?**

```
(your answer here)
```

**3. In the Disposition field, you saw codes 20 (Issued) and 21 (Revoked). If a certificate shows Disposition 21, what additional fields should you check to understand the full context of the revocation, and why?**

```
(your answer here)
```

**4. The Application event log records CA events by default. The Security event log does not record CA events without additional configuration (covered in Lesson 3). Based on what you found in Part A, what operational information is present in the Application log — and what is absent that would be important in a production environment?**

```
(your answer here)
```

---

## Submission Checklist

- [ ] Logged in as CORP\pki.admin — whoami output included
- [ ] CA running and responding — Get-Service and certutil -ping output included
- [ ] Application event log filtered by Source = CertificationAuthority
- [ ] At least three distinct event types documented with Event ID, timestamp, and description
- [ ] Event type summary table completed
- [ ] certutil -view -restrict "Disposition=20" output included (all issued certs)
- [ ] certutil -view -restrict "Disposition=21" output included (all revoked certs)
- [ ] certutil -view requester filter output included
- [ ] At least two specific certificate records documented in full
- [ ] Template distribution output included
- [ ] All three Part C analysis questions answered (minimum sentence counts met)
- [ ] All four lab report questions answered in complete sentences
- [ ] File committed to `labs/week-14/lab-01-event-log-navigation.md`
