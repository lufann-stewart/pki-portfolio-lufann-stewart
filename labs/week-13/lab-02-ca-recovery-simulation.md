# Lab 02: CA Recovery Simulation — Snapshot Restore

Lufann Stewart  
July 1, 2026    
**Phase:** 2 | **Week:** 13  
**Submission Path:** `labs/week-13/lab-02-ca-recovery-simulation.md`

---

## Overview

In this lab, you simulate a CA failure and recover from it using a VirtualBox snapshot. You will take a snapshot of PKI-SRV01 after confirming the Lab 01 backup is in place, perform a destructive operation that takes the CA offline, and then restore from the snapshot to bring it back to full operational state. This lab demonstrates Scenario 2a from Lesson 3: OS-level failure with a recent snapshot available.

**Lab 01 must be complete before starting this lab.** The Lab 01 backup files must be present in C:\CABackup on PKI-SRV01.

---

## Lab Environment

| Component | Details |
|---|---|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| Domain Account | CORP\pki.admin |
| Snapshot Tool | VirtualBox (on your host machine) |
| Backup Location | C:\CABackup (from Lab 01) |

---

## Pre-Lab Verification

### Step 1 — Confirm Lab 01 Backup Is Present

Log into PKI-SRV01 as CORP\pki.admin. From an elevated PowerShell prompt:

```powershell
# Confirm backup files exist from Lab 01
Get-ChildItem C:\CABackup -Recurse | Select-Object FullName, Length, LastWriteTime
```

**Expected:** .p12 file and DataBase folder with .edb file from Lab 01.

```
FullName                                  Length  LastWriteTime       
--------                                  ------  -------------       
C:\CABackup\DataBase                              6/30/2026 9:38:20 AM
C:\CABackup\CVI Issuing CA 1.p12          4631    6/30/2026 9:38:20 AM
C:\CABackup\DataBase\certbkxp.dat         398     6/30/2026 9:38:20 AM
C:\CABackup\DataBase\CVI Issuing CA 1.edb 1052672 6/30/2026 9:38:20 AM
C:\CABackup\DataBase\edb00005.log         1048576 6/30/2026 9:38:20 AM
```

**Lab 01 backup files are present in C:\CABackup:**
- [X] Yes — proceed to Part A
- [ ] No — Lab 01 must be completed before this lab can proceed

### Step 2 — Confirm CA Is Fully Operational

```powershell
certutil -ping
certutil -CRL
```

```
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (16ms)
CertUtil: -ping command completed successfully.
CertUtil: -CRL command completed successfully.
```

**CA is operational before starting the simulation:**
- [X] Yes — proceed to Part A
- [ ] No — resolve any CA issues before continuing

---

## Part A — Take a Pre-Failure Snapshot

You will take a VirtualBox snapshot of PKI-SRV01 while it is in its known-good state. This is the snapshot you will restore from in Part C.

### Step 1 — Note the Pre-Snapshot CA State

Before taking the snapshot, record the current CA state. These values will be your baseline for verification after recovery.

>**Note:** The original `findstr` and `Select-Object` filters did not return the required output in this lab environment. The commands below were used to capture the required verification data.

```powershell
# Record CRL publication status
certutil -CRL

# Record CA certificate thumbprint
certutil -store My

# Record the most recent certificate issued
certutil -view -restrict "Disposition=20" -out "RequestID,SerialNumber,CommonName,NotAfter"
```

```
CertUtil: -CRL command completed successfully.

Subject: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
Certificate Template: SubCA
Cert Hash (SHA1): 5137a597de2c3085ec5816c7f11edc18cfcdbaf8
Key Container = CVI Issuing CA 1

Issued Request ID: 0x10 (16)
Serial Number: 4400000010bf15486a229bae24000000000010
Issued Common Name: PKI-SRV01.corp.cvilab.local
Certificate Expiration Date: 4/25/2027 7:36 PM
```

**Pre-snapshot CA state — record for comparison after recovery:**

```
CRL publication: - [x] successful
CA certificate thumbprint (first 8 chars):  5137a597 
Most recent issued certificate RequestID: 0x10 (16)
```

### Step 2 — Shut Down PKI-SRV01 Cleanly

You must shut down the VM before taking a snapshot to ensure a consistent state.

From PKI-SRV01:
```powershell
Stop-Computer -Force
```

Wait for the VM to fully power off before proceeding.

**PKI-SRV01 has powered off completely:**
- [X] Yes
- [ ] No — describe:

### Step 3 — Take the Snapshot

On your **host machine** (not inside the VM). Follow the instructions for your hypervisor.

---

**If you are using VirtualBox:**

1. Open **VirtualBox Manager**
2. Select **PKI-SRV01** in the left panel
3. Click **Machine → Take Snapshot** (or press Ctrl+Shift+S)
4. Name the snapshot: `Week13-Lab02-PreFailure-<today's date>`
5. Add a description: `Known good state after Lab 01 backup. CA database, private key, and system state backed up to C:\CABackup.`
6. Click **OK**

> **Alternative — live snapshot:** VirtualBox supports snapshots while the VM is running. Use Machine → Take Snapshot with the VM running if preferred. The snapshot will include VM memory state.

---

**If you are using UTM (macOS):**

1. Open **UTM**
2. Make sure PKI-SRV01 is powered off (the VM should show as stopped in the sidebar)
3. Right-click **PKI-SRV01** in the UTM sidebar → select **Snapshots...**
4. In the Snapshots window, click the **+** button (bottom left)
5. Name the snapshot: `Week13-Lab02-PreFailure-<today's date>`
6. Click **OK** or press Enter

> **UTM snapshot note:** UTM snapshots are available for QEMU-based VMs only. If your PKI-SRV01 was created using the Apple Virtualization framework (check UTM settings → System → Architecture), snapshots may not be available. In that case, use a full VM clone as an alternative — contact your instructor.

---

**Snapshot name used:**

```
Week13-Lab02-PreFailure-7-1-26
```

**Hypervisor:**
- [X] VirtualBox
- [ ] UTM

**Snapshot taken successfully:**
- [X] Yes — snapshot appears in the snapshot list for PKI-SRV01
- [ ] No — describe the issue:

### Step 4 — Start PKI-SRV01

Start the VM and log back in as CORP\pki.admin before proceeding to Part B.

```powershell
# Confirm CA is running after restart
certutil -ping
```

```
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (0ms)
CertUtil: -ping command completed successfully.
```

---

## Part B — The Destructive Operation (Simulating Failure)

You will now perform a destructive operation to simulate a CA failure. Choose one of the two options below — Option 1 is recommended. Read both options before choosing.

> ⚠️ **This step intentionally breaks the CA.** The recovery in Part C will restore it. Do not attempt to repair the CA before completing Part C.

### Choose Your Failure Scenario

**Option 1 — Delete the CA database files (Recommended)**

This simulates corruption or accidental deletion of the CA database — a Scenario 2 failure where the OS is running but the CA cannot start.

From an elevated PowerShell prompt on PKI-SRV01:

```powershell
# Stop the CA service first (required to unlock the database files)
Stop-Service CertSvc

# Delete the CA database files
Remove-Item "C:\Windows\System32\CertLog\*.edb" -Force
Remove-Item "C:\Windows\System32\CertLog\*.log" -Force

# Confirm files are deleted
dir "C:\Windows\System32\CertLog\"

# Attempt to start the CA service — it should fail
Start-Service CertSvc
```

---

**Option 2 — Corrupt the CA database via registry modification**

This simulates a misconfiguration that prevents the CA from locating its database — producing a different error but the same recovery requirement.

```powershell
# Stop the CA service
Stop-Service CertSvc

# Rename the database directory to simulate it being inaccessible
Rename-Item "C:\Windows\System32\CertLog" "C:\Windows\System32\CertLog_BROKEN"

# Attempt to start the CA — it should fail
Start-Service CertSvc
```

> If using Option 2, rename the folder back BEFORE attempting certutil -recover or any repair. The snapshot restore in Part C will undo this automatically.

---

**Which option did you choose?**
- [X] Option 1 — CA database files deleted
- [ ] Option 2 — CertLog folder renamed

**Record the failure state:**

```powershell
# What does the CA service status show?
Get-Service CertSvc

# What does the event log show?
Get-WinEvent -LogName Application -Source "CertificationAuthority" -MaxEvents 5 |
    Select-Object TimeCreated, Id, Message | Format-List
```

```
Status   Name               DisplayName                           
------   ----               -----------                           
Stopped  CertSvc            Active Directory Certificate Services 

```

```

PS C:\Windows\system32> Get-WinEvent -FilterHashtable @{
    LogName='Application'
    ProviderName='Microsoft-Windows-CertificationAuthority'
} -MaxEvents 5


   ProviderName: Microsoft-Windows-CertificationAuthority

TimeCreated                      Id LevelDisplayName Message                                                                                                             
-----------                      -- ---------------- -------                                                                                                             
7/1/2026 8:20:06 AM              17 Error            Active Directory Certificate Services did not start: Unable to initialize the database connection for CVI Issuing...
7/1/2026 8:20:06 AM              38 Information      Active Directory Certificate Services for CVI Issuing CA 1 was stopped.                                             
7/1/2026 8:13:29 AM              26 Information      Active Directory Certificate Services for CVI Issuing CA 1 was started.  DC=DC01.corp.cvilab.local                  
7/1/2026 8:13:29 AM              44 Error            The "Windows default" Policy Module "Initialize" method returned an error. There is a time and/or date difference...
7/1/2026 8:09:12 AM              38 Information      Active Directory Certificate Services for CVI Issuing CA 1 was stopped
```

**Document the failure state in your own words:**

```
CA service status: Stopped

Event log errors observed (Event IDs and messages):
Event ID 17 (Error): Active Directory Certificate Services did not start because it was unable to initialize the database connection for CVI Issuing CA 1.
Event ID 38 (Information): Active Directory Certificate Services for CVI Issuing CA 1 was stopped.

What you believe is causing the CA failure:
The CertLog folder containing the CA database was renamed. When the CA service started, it attempted to access the configured database location but could not initialize the database because the required files were no longer available at that path.
```

**CA service is in a failed state and the failure is documented:**
- [X] Yes — proceed to Part C
- [ ] No — CA service started successfully (choose a more destructive option above)

---

## Part C — Recovery: Snapshot Restore

You will now restore PKI-SRV01 from the snapshot taken in Part A.

### Step 1 — Shut Down PKI-SRV01

The snapshot restore requires the VM to be powered off.

From PKI-SRV01 (if you can still log in):
```powershell
Stop-Computer -Force
```

If the VM is unresponsive, force power off from your hypervisor:
- **VirtualBox:** Machine → ACPI Shutdown, or as a last resort, Machine → Power Off
- **UTM:** Right-click the VM in the sidebar → Stop, or close the VM window

**PKI-SRV01 is powered off:**
- [X] Yes
- [ ] Forced power off used — describe:

### Step 2 — Restore the Snapshot

On your **host machine**. Follow the instructions for your hypervisor.

---

**If you are using VirtualBox:**

1. Open **VirtualBox Manager**
2. Select **PKI-SRV01**
3. Click the **Snapshots** tab (clock icon in the right panel, or View → Snapshots)
4. Locate your `Week13-Lab02-PreFailure-<date>` snapshot
5. Right-click → **Restore Snapshot**
6. When prompted: **Do NOT check "Create a snapshot of the current machine state"** — this is not necessary for a recovery simulation
7. Click **Restore**

> Typically 30–60 seconds. The VirtualBox status bar shows progress.

---

**If you are using UTM:**

1. Open **UTM**
2. Make sure PKI-SRV01 is powered off
3. Right-click **PKI-SRV01** in the UTM sidebar → select **Snapshots...**
4. Locate your `Week13-Lab02-PreFailure-<date>` snapshot in the list
5. Select it and click the **restore icon** (the curved arrow / back arrow button at the bottom of the snapshot list)
6. When prompted to confirm, click **Restore**
7. UTM will restore the VM to the snapshot state — this takes 15–60 seconds

> **UTM restore note:** After the restore, the VM will be in a powered-off state. You will need to start it manually from the UTM sidebar (click the Play button).

---

**Snapshot restore completed without errors:**
- [X] Yes
- [ ] No — describe the error:

### Step 3 — Start PKI-SRV01 and Log In

Start the VM and log in as CORP\pki.admin.

**VM started and login successful:**
- [X] Yes
- [ ] No — describe:

---

## Part D — Post-Recovery Verification

Recovery is not complete until every item on the verification checklist passes.

### Step 1 — Confirm CA Service Is Running

```powershell
Get-Service CertSvc
```

**Expected:** Status = Running

```
Status   Name               DisplayName                           
------   ----               -----------                           
Running  CertSvc            Active Directory Certificate Services 
```

**CA service is running:**
- [X] Yes
- [ ] No — attempt Start-Service CertSvc and document result:

### Step 2 — Confirm CA Responds

```powershell
certutil -ping
```

**Expected:** "Server 'CVI Issuing CA 1' ICertRequest2 interface is alive"

```
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (31ms)
CertUtil: -ping command completed successfully.
```

### Step 3 — Publish a New CRL

```powershell
certutil -CRL
```

**Expected:** "CRL published successfully. CertUtil: -CRL command completed successfully."

```
CertUtil: -CRL command completed successfully.
```

> **If certutil -CRL fails with "Access is denied":** Confirm you are running from an elevated prompt as CORP\pki.admin.

**CRL published successfully:**
- [X] Yes
- [ ] No — describe:

### Step 4 — Confirm CRL Is Accessible at the Distribution Point

```powershell
# Find the CDP URL from a certificate
certutil -dump revoked.cer | findstr "http"
# If no certificate file is available, check the CertEnroll folder directly:
certutil -URL http://pki-srv01.corp.cvilab.local/CertEnroll/"CVI Issuing CA 1.crl"
```

Alternatively, navigate to the CDP URL in a browser on PKI-SRV01:
`http://pki-srv01.corp.cvilab.local/CertEnroll/`

**CRL is accessible at the HTTP distribution point:**
- [X] Yes
- [ ] No — describe:

### Step 5 — Verify Database Is Intact

Confirm the CA database contains its issuance history.

```powershell
# Count issued certificates in the restored database
certutil -view -restrict "Disposition=20" -out "RequestID,CommonName" | Measure-Object -Line
```

```

PS C:\Windows\system32> certutil -view -restrict "Disposition=20" -out "RequestID,CommonName"
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  RequestID                     Issued Request ID             Long    4 -- Indexed
  CommonName                    Issued Common Name            String  8192 -- Indexed

Row 1:
  Issued Request ID: 0x3
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Row 2:
  Issued Request ID: 0x9
  Issued Common Name: "Svc Autoenroll"

Row 3:
  Issued Request ID: 0xb (11)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Row 4:
  Issued Request ID: 0xc (12)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Row 5:
  Issued Request ID: 0xd (13)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Row 6:
  Issued Request ID: 0xe (14)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Row 7:
  Issued Request ID: 0xf (15)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Row 8:
  Issued Request ID: 0x10 (16)
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"

Maximum Row Index: 8

8 Rows
  16 Row Properties, Total Size = 438, Max Size = 54, Ave Size = 27
   0 Request Attributes, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Certificate Extensions, Total Size = 0, Max Size = 0, Ave Size = 0
  16 Total Fields, Total Size = 438, Max Size = 54, Ave Size = 27
CertUtil: -view command completed successfully    
```

**Issued certificate count matches what you recorded before the failure:**
- [X] Yes
- [ ] Approximately — slight difference, explain:
- [ ] No — describe:

### Step 6 — Confirm Event Log Is Clean

```powershell
Get-WinEvent -LogName Application -Source "CertificationAuthority" -MaxEvents 10 |
    Where-Object { $_.LevelDisplayName -eq "Error" } |
    Select-Object TimeCreated, Id, Message | Format-List
```

```
TimeCreated : 7/1/2026 8:40:55 AM
Id          : 44
Message     : The "Windows default" Policy Module "Initialize" method returned an error. The specified domain either does not exist or could not be contacted. The returned status code is 0x8007054b (1355).  The Active Directory containing the Certification 
              Authority could not be contacted.
              

TimeCreated : 7/1/2026 8:40:12 AM
Id          : 91
Message     : Could not connect to the Active Directory.  Active Directory Certificate Services will retry when processing requires Active Directory access.

TimeCreated : 6/30/2026 12:12:01 PM
Id          : 44
Message     : The "Windows default" Policy Module "Initialize" method returned an error. The specified server cannot perform the requested operation. The returned status code is 0x8007003a (58).  The Active Directory containing the Certification Authority 
              could not be contacted.
```

**Event log shows no errors from CertificationAuthority after recovery:**
- [ ] Yes — no errors
- [X] Errors found — document and explain:
The event log shows intermittent errors (Event IDs 44 and 91) tied to Active Directory connectivity. These point to temporary issues reaching AD/DNS when the CA Policy Module initializes. Even with those errors, the CA service stayed up and responded normally to certutil -ping, which confirms the restore brought back the CA database and core services correctly. The errors are environmental (domain/DNS), not a sign of CA database corruption.

### Step 7 — Record Recovery Completion

**Recovery completed at (date and time):**

```
7/1/2026 12:10:45 PM
```

**Time from snapshot restore initiation to CA fully operational:**

```
2 minutes
```

---

## Part E — Lab Report

Answer all questions in complete sentences.

**1. Describe the failure state you simulated in Part B. What did Get-Service CertSvc show, and what Event IDs appeared in the event log? What would a real-world CA administrator see if they encountered this failure?**

```
I simulated a CA failure by deleting the CA database and log files inside the CertLog folder. After that, the CA could no longer start because it had no database to load.

Get-Service CertSvc showed the service as Stopped. In the event logs, I saw Event ID 17 showing the CA could not initialize the database, along with Events 38, 44, and 91 showing service stops and AD/policy initialization errors.

In a real environment, this would look like the CA is down because its database is missing or corrupted, and it would be unable to issue certificates until it is restored.
```

**2. Walk through the snapshot restore procedure step by step. What did VirtualBox do during the restore — and how did the restored VM state compare to the failure state you left it in?**

```
I shut down the VM, opened VirtualBox Manager, and took a snapshot called Week13-Lab02-PreFailure-7-1-26.

After breaking the CA, I went back into the Snapshots menu, selected that snapshot, and restored it. Then I powered the VM back on.

The restore rolled the system back completely to the saved state, meaning the CA database, configuration, and system state all returned to how they were before the failure.
```

**3. Walk through the post-recovery verification checklist. Which step confirmed that the CA was fully operational — not just running, but functional? Explain what each verification step tests and why it is not sufficient to just check that the CA service is running.**

```
I checked several things to confirm the CA was fully working again.

First, Get-Service CertSvc confirmed the service was running, but that alone doesn’t prove functionality.

Then certutil -ping confirmed the CA was actually responding.

Next, certutil -CRL confirmed the CA could publish a CRL, which proves core CA functions are working.

Finally, certutil -view confirmed the database still had all issued certificates.

Just checking the service isn’t enough because it doesn’t confirm the CA can actually issue certificates or communicate properly.
```

**4. The snapshot you restored from was taken immediately after Lab 01. If this were a production environment where the last snapshot was taken 72 hours ago, what operational data would be lost in this recovery — and why does that data loss matter to the organization?**

```
If this were a production system and the snapshot was from 72 hours ago, any changes made in those 3 days would be lost.

That includes issued certificates, certificate requests, revocations, and any updates to the CA database.

This matters because those changes are part of security and identity trust. Losing them could break authentication, cause missing certificates, or create gaps in revocation tracking.
```

**5. Compare snapshot restore to file-based restore (which you will perform in Lab 03). Based on what you experienced today, what is the primary advantage of snapshot restore — and in what failure scenario would snapshot restore NOT be available as an option?**

```
The main advantage of snapshot restore is speed. It restores the entire system state at once, including the OS, CA configuration, and database.

The limitation is that it only works if a snapshot exists. If the VM is gone, the hypervisor is unavailable, or there is no recent snapshot, then you can’t use this method and would need a file-based or system-state restore instead.
```

---

## Submission Checklist

- [X] Logged in as CORP\pki.admin — confirmed with whoami
- [X] Lab 01 backup files confirmed present in C:\CABackup before starting
- [X] Pre-failure CA state recorded (CRL status, certificate thumbprint, last issued RequestID)
- [X] PKI-SRV01 shut down cleanly before snapshot
- [X] Snapshot taken — name and description recorded
- [X] Destructive operation performed — failure state documented (service status + event log)
- [X] Option selected (delete database files OR rename CertLog folder) documented
- [X] PKI-SRV01 powered off before snapshot restore
- [X] Snapshot restore completed (VirtualBox or UTM) — no errors
- [X] Post-recovery verification: CA service running
- [X] Post-recovery verification: certutil -ping successful
- [X] Post-recovery verification: certutil -CRL successful
- [X] Post-recovery verification: CRL accessible at HTTP CDP
- [X] Post-recovery verification: database certificate count matches pre-failure baseline
- [X] Post-recovery verification: event log reviewed (non-CA AD connectivity errors present and explained in Step 6)
- [X] Recovery time recorded
- [X] All five lab report questions answered in complete sentences
- [X] Lab file committed to `labs/week-13/lab-02-ca-recovery-simulation.md`
