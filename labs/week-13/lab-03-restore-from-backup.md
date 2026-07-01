# Lab 03: Restore from Backup Files *(Stretch)*

Lufann Stewart  
July 1, 2026   
**Phase:** 2 | **Week:** 13  
**Submission Path:** `labs/week-13/lab-03-restore-from-backup.md`

---

## Overview

This stretch lab has you restore the CA from the backup files you created in Lab 01 — not from a snapshot. This is a file-based restore: you will import the CA private key from the .p12 file, restore the CA database using `certutil -restoredb`, and verify the CA returns to full operational state. Lab 03 simulates a Scenario 2b or Scenario 3 recovery (Lesson 3) — the procedure that applies when no recent snapshot is available or when hardware must be replaced.

**Requirements before starting:**
- Lab 01 must be complete — you need the C:\CABackup files and your backup password
- Lab 02 must be complete — Lab 03 uses a different starting state than Lab 02

**If you do not have the Lab 01 backup password:** Lab 03 cannot be completed without it. Contact your instructor.

---

## Lab Environment

| Component | Details |
|---|---|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| Domain Account | CORP\pki.admin |
| Backup Source | C:\CABackup (from Lab 01) |
| Backup Password | The password you used in Lab 01 for certutil -backup -p |

---

## Pre-Lab Setup

### Step 1 — Confirm Lab 01 Backup Files Are Present

The Lab 03 restore requires the backup files from Lab 01 to still be present.

```powershell
Get-ChildItem C:\CABackup -Recurse | Select-Object FullName, Length, LastWriteTime
```

**Confirm these files exist:**

| File | Present? |
|---|---|
| C:\CABackup\<CAName>.p12 | [X] Yes / [ ] No |
| C:\CABackup\DataBase\<CAName>.edb | [X] Yes / [ ] No |

> **If the backup files were removed by the Lab 02 snapshot restore:** The snapshot restored PKI-SRV01 to its Lab 01 post-backup state — the backup files should be present. If they are not, you need to re-run Lab 01 before proceeding.

```
FullName                                  Length  LastWriteTime       
--------                                  ------  -------------       
C:\CABackup\DataBase                              6/30/2026 9:38:20 AM
C:\CABackup\CVI Issuing CA 1.p12          4631    6/30/2026 9:38:20 AM
C:\CABackup\DataBase\certbkxp.dat         398     6/30/2026 9:38:20 AM
C:\CABackup\DataBase\CVI Issuing CA 1.edb 1052672 6/30/2026 9:38:20 AM
C:\CABackup\DataBase\edb00005.log         1048576 6/30/2026 9:38:20 AM
```

### Step 2 — Confirm Your Backup Password

You will need the password you specified in Lab 01's `certutil -backup -p <password>` command. Do not proceed without it.

**I have the backup password from Lab 01:**
- [X] Yes — password is in hand (do not record it here)
- [ ] No — I need to re-run Lab 01 before continuing

### Step 3 — Take a New Snapshot Before Starting

Before performing any destructive operations, take a fresh snapshot of PKI-SRV01. This gives you a rollback point if the restore procedure encounters unexpected issues.

Shut down PKI-SRV01 cleanly:
```powershell
Stop-Computer -Force
```

Then take the snapshot on your **host machine**. Follow the instructions for your hypervisor:

**VirtualBox:** Machine → Take Snapshot → Name it `Week13-Lab03-PreRestore-<today's date>` → Click OK

**UTM:** Right-click **PKI-SRV01** in the UTM sidebar → **Snapshots...** → click **+** → Name it `Week13-Lab03-PreRestore-<today's date>` → click OK

Then start PKI-SRV01 and log back in as CORP\pki.admin.

**Hypervisor:**
- [X] VirtualBox
- [ ] UTM

**Lab03 pre-restore snapshot taken:**
- [X] Yes — snapshot name recorded: Week13-Lab03-PreRestore-7-1-2026
- [ ] No — reason:

---

## Part A — Simulate the Failure State

To demonstrate a file-based restore, you need the CA to be in a broken state — one that cannot be fixed with a snapshot restore alone. You will delete the CA database, the CA private key from the Windows certificate store, and clear the CertLog folder.

> ⚠️ **This is the point of no return for snapshot-based recovery.** After Part A, the only path back is the file-based restore in Parts B and C. The Lab03 pre-restore snapshot from Step 3 above is your safety net — it uses the Lab03 snapshot, not the Lab01 snapshot.

### Step 1 — Stop the CA Service

```powershell
Stop-Service CertSvc
Get-Service CertSvc
```

**CA service is stopped:**
- [X] Yes — status shows Stopped

### Step 2 — Delete the CA Database Files

```powershell
# Remove all CA database files
Remove-Item "C:\Windows\System32\CertLog\*" -Force -Recurse
dir "C:\Windows\System32\CertLog\"
```

**Expected after deletion:** Empty directory or "File Not Found."

```
PS C:\Windows\system32>  dir "C:\Windows\System32\CertLog\"

PS C:\Windows\system32> 
```

### Step 3 — Remove the CA Private Key from the Windows Certificate Store

This simulates key material being deleted or unavailable — the more severe failure scenario.

```powershell
# List the CA certificate in the local machine store
certlm.msc
# Navigate to: Certificates (Local Computer) → Personal → Certificates
# Locate the "CVI Issuing CA 1" certificate
# Right-click → Delete
```

Alternatively, from PowerShell (find and remove the CA cert):

```powershell
# Find the CA certificate thumbprint
$caCert = Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.Subject -like "*CVI Issuing CA*" }
$caCert | Select-Object Subject, Thumbprint

# Remove the certificate (including private key if present)
Remove-Item "Cert:\LocalMachine\My\$($caCert.Thumbprint)"
```

```
The CA certificate was removed using certlm.msc (Certificates (Local Computer) → Personal → Certificates) as instructed.

PS C:\Windows\system32> Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.Subject -like "*CVI Issuing CA*" }

PS C:\Windows\system32>
```

**CA certificate removed from local machine certificate store:**
- [X] Yes
- [ ] Not found — document and continue (the certutil -restorekey step will handle this)

### Step 4 — Attempt to Start the CA and Document the Failure

```powershell
Start-Service CertSvc
Get-Service CertSvc

# Check event log for errors
Get-WinEvent -FilterHashtable @{
    LogName='Application'
    ProviderName='Microsoft-Windows-CertificationAuthority'
} -MaxEvents 5
```

```

Status   Name               DisplayName                           
------   ----               -----------                           
Running  CertSvc            Active Directory Certificate Services 

TimeCreated  : 7/1/2026 1:50:04 PM
ProviderName : Microsoft-Windows-CertificationAuthority
Id           : 17
Message      : Active Directory Certificate Services did not start: Unable to initialize the database connection for CVI Issuing CA 1.  File not found 0xc8000713 (ESE: -1811 JET_errFileNotFound).


TimeCreated  : 7/1/2026 1:50:03 PM
ProviderName : Microsoft-Windows-CertificationAuthority
Id           : 91
Message      : Could not connect to the Active Directory.  Active Directory Certificate Services will retry when processing requires Active Directory access.


TimeCreated  : 7/1/2026 12:42:43 PM
ProviderName : Microsoft-Windows-CertificationAuthority
Id           : 17
Message      : Active Directory Certificate Services did not start: Unable to initialize the database connection for CVI Issuing CA 1.  File not found 0xc8000713 (ESE: -1811 JET_errFileNotFound).


TimeCreated  : 7/1/2026 12:40:02 PM
ProviderName : Microsoft-Windows-CertificationAuthority
Id           : 38
Message      : Active Directory Certificate Services for CVI Issuing CA 1 was stopped.


TimeCreated  : 7/1/2026 12:39:28 PM
ProviderName : Microsoft-Windows-CertificationAuthority
Id           : 26
Message      : Active Directory Certificate Services for CVI Issuing CA 1 was started.  DC=DC01.corp.cvilab.local

```

**Failure state documented:**

```
CA service status: Running
Event IDs observed: 17, 91, 38, and 26
Failure description in your own words: Database files were missing from CertLog, so ADCS couldn't initialize and failed to start.
```

---

## Part B — Restore the CA Private Key

The first step of a file-based restore is restoring the CA private key from the .p12 backup. Without the private key, the CA service cannot sign certificates or CRLs.

### Step 1 — Run certutil -restorekey

Replace `<YourBackupPassword>` with the password from Lab 01.

```powershell
certutil -restorekey -p "YOUR_PASSWORD_HERE" "C:\CABackup\CVI Issuing CA 1.p12"
```

**Expected output:**
```
Restoring to container: <ContainerName>
CertUtil: -restorekey command completed successfully.
```

> **If you see "Cannot find object or property":** The path to the .p12 file is incorrect. The CA name contains spaces — the path must be in quotes. Use the exact form: `certutil -restorekey "C:\CABackup\CVI Issuing CA 1.p12" <YourBackupPassword>`. Verify the exact filename first: `dir C:\CABackup\*.p12`

> **If you see "The password does not meet the password policy requirements" or "The credentials supplied to the package were not recognized":** The password is incorrect. Re-check your Lab 01 notes. The password is case-sensitive.

> **If you see "The system cannot find the file specified":** The .p12 file does not exist at the specified path. Run `dir C:\CABackup` to confirm the filename and path.

```
Restored keys and certificates for PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 from C:\CABackup\CVI Issuing CA 1.p12.
CertUtil: -restoreKey command completed successfully.
The CertSvc service may need to be restarted for changes to take effect.
```

**certutil -restorekey completed successfully:**
- [X] Yes
- [ ] No — error message and resolution:

### Step 2 — Verify the CA Certificate Is Back in the Certificate Store

```powershell
# Confirm the CA certificate and key are now in the local machine store
Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.Subject -like "*CVI Issuing CA*" } |
    Select-Object Subject, Thumbprint, HasPrivateKey
```

**Expected:** `HasPrivateKey = True` — confirming the key was restored.

```

Subject                                           Thumbprint                               HasPrivateKey
-------                                           ----------                               -------------
CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local 5137A597DE2C3085EC5816C7F11EDC18CFCDBAF8          True
```

**CA certificate is in the local machine store with private key:**
- [X] Yes — HasPrivateKey = True
- [ ] No — describe:

> **If HasPrivateKey shows False despite certutil -restorekey reporting success:** The key was restored to a different CSP container than the CA service expects. Run `certutil -store My` to see all certificates in the personal store and verify the correct CA certificate is listed. If the thumbprint does not match the CA certificate, re-run certutil -restorekey and confirm you are using the correct .p12 file.

---

## Part C — Restore the CA Database

With the private key restored, you now restore the CA database from the certutil backup.

### Step 1 — Run certutil -restoredb

```powershell
certutil -restoredb "C:\CABackup"
```

**Expected output:**
```
Restoring CA database...
Database restored successfully.
CertUtil: -restoredb command completed successfully.
```

> **If you see "The directory is not empty" or "File already exists":** The CertLog directory may have residual files. Clear it first:
> `Remove-Item "C:\Windows\System32\CertLog\*" -Force -Recurse`
> Then re-run certutil -restoredb.

> **If you see "The backup directory does not contain a valid database backup":** The path to the backup is incorrect, or the DataBase subfolder is missing. Verify: `dir C:\CABackup\DataBase\`

```
Restoring database for PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1.

Restoring Database files: 100%

Restoring Log files: 100%
Full database restore for PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1.
Stop and Start Active Directory Certificate Services to complete database restore from C:\CABackup.
CertUtil: -restoreDB command completed successfully.
The CertSvc service may need to be restarted for changes to take effect.
```

**certutil -restoredb completed successfully:**
- [X] Yes
- [ ] No — error message and resolution:

### Step 2 — Verify the Database Files Were Restored

```powershell
dir "C:\Windows\System32\CertLog\"
```

**Expected:** .edb file and log files are present.

```

    Directory: C:\Windows\System32\CertLog


Mode                 LastWriteTime         Length Name                                                                                                                   
----                 -------------         ------ ----                                                                                                                   
-a----          7/1/2026   2:21 PM        1052672 CVI Issuing CA 1.edb                                                                                                   
-a----          7/1/2026   2:21 PM        1048576 edb00005.log                                                                                                      
```

---

## Part D — Start the CA Service and Verify

### Step 1 — Start the CA Service

```powershell
Start-Service CertSvc
Get-Service CertSvc
```

**Expected:** Status = Running

```
Status   Name               DisplayName                           
------   ----               -----------                           
Running  CertSvc            Active Directory Certificate Services 
```

> **If the service fails to start with Event ID 100 — "CA certificate not found":** certutil -restorekey was not run before certutil -restoredb, or the restorekey step failed silently. Confirm HasPrivateKey = True in the certificate store (Step 2 above). If not, re-run certutil -restorekey first, then Start-Service CertSvc again.

> **If the service fails to start with Event ID 100 — "CA database could not be opened":** The CertLog directory still has residual files. Run `Remove-Item "C:\Windows\System32\CertLog\*" -Force -Recurse` and re-run certutil -restoredb, then Start-Service CertSvc.

> **If the service starts but certutil -CRL fails with "The system cannot find the path specified":** The CertEnroll folder may be missing or the service account lacks write access. Check: `dir "C:\Windows\System32\CertSrv\CertEnroll\"`. If the folder does not exist, create it: `New-Item -ItemType Directory -Path "C:\Windows\System32\CertSrv\CertEnroll" -Force`. Then retry certutil -CRL.

> **If Event ID 100 shows a CA name mismatch:** The CA name in the restored database does not match the installed CA configuration. Restore the system state backup from Lab 01 (wbadmin start systemstaterecovery) and retry.

**CA service started without errors:**
- [X] Yes — proceed to verification
- [ ] No — error and resolution:

### Step 2 — Run the Full Post-Recovery Verification Checklist

```powershell
# 1. CA responds
certutil -ping

# 2. Publish CRL
certutil -CRL

# 3. Check event log for errors
Get-WinEvent -FilterHashtable @{
    LogName = 'Application'
    ProviderName = 'Microsoft-Windows-CertificationAuthority'
} -MaxEvents 10 |
Where-Object { $_.LevelDisplayName -eq "Error" } |
Select-Object TimeCreated, Id, Message | Format-List
```

```
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (16ms)
CertUtil: -ping command completed successfully.
CertUtil: -CRL command completed successfully.


TimeCreated : 7/1/2026 2:16:19 PM
Id          : 17
Message     : Active Directory Certificate Services did not start: Unable to initialize the database connection for CVI Issuing CA 1.  File not found 0xc8000713 (ESE: 
              -1811 JET_errFileNotFound).

TimeCreated : 7/1/2026 1:51:52 PM
Id          : 17
Message     : Active Directory Certificate Services did not start: Unable to initialize the database connection for CVI Issuing CA 1.  File not found 0xc8000713 (ESE: 
              -1811 JET_errFileNotFound).

TimeCreated : 7/1/2026 1:51:52 PM
Id          : 91
Message     : Could not connect to the Active Directory.  Active Directory Certificate Services will retry when processing requires Active Directory access.

TimeCreated : 7/1/2026 1:50:04 PM
Id          : 17
Message     : Active Directory Certificate Services did not start: Unable to initialize the database connection for CVI Issuing CA 1.  File not found 0xc8000713 (ESE: 
              -1811 JET_errFileNotFound).

TimeCreated : 7/1/2026 1:50:03 PM
Id          : 91
Message     : Could not connect to the Active Directory.  Active Directory Certificate Services will retry when processing requires Active Directory access.

TimeCreated : 7/1/2026 12:42:43 PM
Id          : 17
Message     : Active Directory Certificate Services did not start: Unable to initialize the database connection for CVI Issuing CA 1.  File not found 0xc8000713 (ESE: 
              -1811 JET_errFileNotFound).

All errors shown are timestamped before the private key and database restore completed (before 2:21 PM). No new errors appeared after CertSvc was restarted.

```

### Step 3 — Confirm Certificate History Is Intact

```powershell
# View issued certificates from the restored database
certutil -view -restrict "Disposition=20" -out "RequestID,SerialNumber,CommonName,NotAfter"
```

```
Schema:
  Column Name                   Localized Name                Type    MaxLength
  ----------------------------  ----------------------------  ------  ---------
  RequestID                     Issued Request ID             Long    4 -- Indexed
  SerialNumber                  Serial Number                 String  128 -- Indexed
  CommonName                    Issued Common Name            String  8192 -- Indexed
  NotAfter                      Certificate Expiration Date   Date    8 -- Indexed

Row 1:
  Issued Request ID: 0x3
  Serial Number: "44000000030d20ca71500ba5c4000000000003"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 2:
  Issued Request ID: 0x9
  Serial Number: "440000000911304cdb8a83f133000000000009"
  Issued Common Name: "Svc Autoenroll"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 3:
  Issued Request ID: 0xb (11)
  Serial Number: "440000000ba5579effd72b412500000000000b"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/13/2026 12:25 PM

Row 4:
  Issued Request ID: 0xc (12)
  Serial Number: "440000000c6c0163ea8cb879e700000000000c"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/13/2026 12:26 PM

Row 5:
  Issued Request ID: 0xd (13)
  Serial Number: "440000000d5ff28d7e92a9e01f00000000000d"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 6/13/2026 12:39 PM

Row 6:
  Issued Request ID: 0xe (14)
  Serial Number: "440000000ef02bb109f34551ad00000000000e"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 7:
  Issued Request ID: 0xf (15)
  Serial Number: "440000000f6660a3690731094b00000000000f"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Row 8:
  Issued Request ID: 0x10 (16)
  Serial Number: "4400000010bf15486a229bae24000000000010"
  Issued Common Name: "PKI-SRV01.corp.cvilab.local"
  Certificate Expiration Date: 4/25/2027 7:36 PM

Maximum Row Index: 8

8 Rows
  32 Row Properties, Total Size = 1110, Max Size = 76, Ave Size = 34
   0 Request Attributes, Total Size = 0, Max Size = 0, Ave Size = 0
   0 Certificate Extensions, Total Size = 0, Max Size = 0, Ave Size = 0
  32 Total Fields, Total Size = 1110, Max Size = 76, Ave Size = 34
CertUtil: -view command completed successfully.

```

**Issued certificate history is present in the restored database:**
- [X] Yes — records are present
- [ ] No — database appears empty (document and explain):

### Step 4 — Confirm CRL Is Accessible

```powershell
certutil -URL http://pki-srv01.corp.cvilab.local/CertEnroll/"CVI Issuing CA 1.crl"
```

Or navigate to the URL in a browser on PKI-SRV01.

```
Opened the CRL URL using certutil -URL. The CRL loaded successfully and the status showed "OK," confirming the CRL was accessible from the HTTP distribution point.
```

**CRL is accessible at the HTTP CDP:**
- [X] Yes
- [ ] No — describe:

**Recovery timestamp:**

```
7/1/2026 2:25 PM
```

**Estimated time from Part A (simulated failure) to CA fully operational:**

```
Approximately 35 minutes.
This estimate excludes time spent troubleshooting incorrect lab commands and reflects the actual recovery procedure.
```

---

## Part E — Snapshot vs. File-Based Restore Comparison

Reflect on your experience with both Lab 02 (snapshot restore) and Lab 03 (file-based restore).

### Direct Comparison

| Factor | Lab 02 — Snapshot Restore | Lab 03 — File-Based Restore |
|---|---|---|
| Time to complete |Approx. 5 mins |Approx. 35 mins |
| Password required? |No |Yes |
| Data currency |Restores the VM exactly to the point when the snapshot was taken |Restores only the data contained in the backup files; anything after the backup is not restored |
| Hardware dependency |Requires access to the original VM or disk containing the snapshot |Can be restored to a different server as long as the AD CS role is installed and the backup files are available |
| Steps required |Few steps (restore snapshot and boot the VM) |More steps (restore private key, restore database, restart the service, and verify CA functionality) |

Fill in this table based on your direct experience.

---

## Part F — Lab Report

Answer all questions in complete sentences.

**1. Describe the file-based restore procedure you performed in Parts B and C. Walk through each command — certutil -restorekey and certutil -restoredb — and explain what each one does and why the order matters.**

```
For this recovery I followed the file-based restore order: private key first, then database, then service checks.

Restore CA Private Key

First I restored the CA's private key and cert from the backup PFX:

certutil -restorekey -p "<backup password>" "C:\CABackup\CVI Issuing CA 1.p12"

This puts the CA's signing key back into the Local Machine cert store — the service can't start without it.

Then I checked it actually restored:

Get-ChildItem Cert:\LocalMachine\My |
Where-Object { $_.Subject -like "*CVI Issuing CA*" } |
Select-Object Subject, Thumbprint, HasPrivateKey

HasPrivateKey = True came back, so the key restored fine.

Restore CA Database

With the key back, I restored the database from backup:

certutil -restoredb "C:\CABackup"

This brings back the ESE database and transaction logs the CA needs to run.

I confirmed the files were actually there:

dir "C:\Windows\System32\CertLog"

Saw the .edb file and logs, so the database restore worked.

Service Recovery and Validation

Restarted the service and checked status:

Start-Service CertSvc
Get-Service CertSvc

Came back Running, meaning the CA started fine with the restored key and database.

Then I ran the functional checks:

certutil -ping
certutil -CRL

Both worked — CA was responding and able to publish a CRL.

Post-Recovery Verification

Checked the event log for anything new:

Get-WinEvent -FilterHashtable @{
    LogName='Application'
    ProviderName='Microsoft-Windows-CertificationAuthority'
} -MaxEvents 10 |
Where-Object { $_.LevelDisplayName -eq "Error" } |
Select-Object TimeCreated, Id, Message | Format-List

No new failures after recovery.

Checked the database still had the issued cert history:

certutil -view -restrict "Disposition=20" -out "RequestID,SerialNumber,CommonName,NotAfter"

Records came back intact — previously issued certs were all still there.

Last, checked the CRL was reachable:

certutil -URL http://pki-srv01.corp.cvilab.local/CertEnroll/"CVI Issuing CA 1.crl"

Came back OK.
```

**2. In Part B, you restored the CA private key using certutil -restorekey. What would have happened if you had restored the CA database first (certutil -restoredb) without restoring the private key first — specifically, would the CA service have started? Why or why not?**

```
The restore order matters because the CA database depends on the private key being available in the local machine store. If the database is restored before the key, the CA service cannot initialize because it has no signing context to bind to.

Restoring the private key first ensures the certificate and key container exist before the database is loaded. Once both components are in place, the CA can start normally and use the restored key to sign certificates and CRLs.

If the order is reversed, the service will fail to start until the private key is restored.
```

**3. Compare your experience with Lab 02 (snapshot restore) and Lab 03 (file-based restore). Which was faster? Which required more steps? Which required the backup password? Which procedure would apply if the CA server hardware had been destroyed and you were restoring to a new machine?**

```
The snapshot restore (Lab 02) was faster and required fewer steps. It also did not require a backup password since it restores the entire VM state as it existed at the time of the snapshot.

The file-based restore (Lab 03) took longer and required more manual steps, including restoring the private key, restoring the database, and restarting the CA service. It also required the backup password from Lab 01.

If the CA server hardware had been destroyed, the file-based restore would be the correct method. A snapshot would not be available in that scenario, so recovery would depend entirely on the backup files to rebuild the CA on new hardware.
```

**4. Explain the "environment mismatch" failure mode described in Lesson 3. What would cause certutil -restoredb to succeed but the CA service to fail to start? What would the event log show, and how would you resolve it?**

```
The CA service can fail even after certutil -restoredb succeeds when the restored database doesn’t match the environment the CA is trying to run in.

In my case, the most likely cause would be missing or mismatched cryptographic material (like the CA private key not being restored correctly, or not being in the expected certificate store/container). The database can be present on disk, but the CA still needs that key to initialize its identity and sign operations.

When this happens, the database restore completes successfully, but the CA service fails during startup because it can’t bind to its key material or complete initialization. In the event logs, you typically see errors about being unable to open the database, missing files, or failures initializing the CA.

To fix it, I would verify the CA certificate in the Local Computer store, confirm HasPrivateKey = True, and if needed re-run certutil -restorekey using the correct .p12 backup. After that, restarting the CertSvc service allows the CA to start normally.
```

**5. If this were a production CA with an RTO of 4 hours, would the file-based restore procedure you performed today meet that RTO? What factors could cause it to take longer — or shorter — in a production environment compared to your lab?**

```
It might meet a 4-hour RTO, but it depends heavily on how prepared the environment is.

In this lab, the process took longer mainly because I had to pause to verify commands, fix syntax issues, and locate the correct backup files and key material. In production, those delays could be smaller if the team already has documented runbooks, validated backup locations, and knows exactly where the CA key and database backups are stored.

However, there are also factors that could easily push it past 4 hours. If the private key backup is missing or the password is unavailable, the restore stops immediately. If the database backup is incomplete or corrupted, that adds investigation and re-recovery steps. Storage issues, permission problems, or Active Directory connectivity issues could also slow things down. On top of that, coordination and validation steps (like CRL publishing and service validation) take longer in real environments because you can’t just assume everything is correct.

Overall, it can meet the RTO only if backups, credentials, and restore steps are well-documented and tested. Without that, it can easily exceed 4 hours.
```

---

## Submission Checklist

- [X] Logged in as CORP\pki.admin (not a local account)
- [X] Lab 01 backup files confirmed present in C:\CABackup before starting
- [X] Backup password available from Lab 01 records
- [X] Lab03 pre-restore snapshot taken (VirtualBox or UTM) — snapshot name documented
- [X] Failure state simulated — CA database deleted AND CA private key removed from cert store (both operations required)
- [X] Failure state documented — CA service status and event log errors recorded
- [X] certutil -restorekey run successfully — output included
- [X] CA certificate confirmed back in local machine store with HasPrivateKey = True
- [X] certutil -restoredb run successfully — output included
- [X] CA database files confirmed in C:\Windows\System32\CertLog\ after restore
- [X] CA service started without errors — Get-Service CertSvc shows Running
- [X] certutil -ping successful after recovery
- [X] certutil -CRL successful after recovery
- [X] Event log clean (no CA errors after recovery)
- [X] Issued certificate history confirmed present in restored database
- [X] CRL accessible at HTTP CDP
- [X] Recovery timestamp and estimated duration recorded
- [X] Snapshot vs. file-based comparison table completed
- [X] All five lab report questions answered in complete sentences
- [X] Lab file committed to `labs/week-13/lab-03-restore-from-backup.md`
