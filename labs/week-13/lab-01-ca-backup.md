# Lab 01: Full CA Backup — Database, Keys, and System State

Lufann Stewart  
June 30, 2026  
**Phase:** 2 | **Week:** 13  
**Submission Path:** `labs/week-13/lab-01-ca-backup.md`

---

## Overview

In this lab, you perform a complete backup of PKI-SRV01, the CVI Issuing CA 1. A complete CA backup has three components: the CA database (all issued certificates and revocation records), the CA private key and certificate (the most critical component), and the Windows system state (CA configuration). You will use `certutil -backup` for the database and private key, and `wbadmin` for the system state. Your lab report documents every command and output — this IS the backup documentation.

**The backup files you create in this lab are required for Lab 03.** Record your backup password in a location separate from the backup folder.

---

## Lab Environment

| Component | Details |
|---|---|
| CA Server | PKI-SRV01 (192.168.10.20) |
| CA Name | CVI Issuing CA 1 |
| Domain | CORP\pki.admin |
| Backup Destination — Database | C:\CABackup |
| Backup Destination — System State | D:\ or a network path (separate volume required) |

> **Note on system state backup destination:** Windows Server Backup requires the system state backup target to be on a different volume than the OS (C:). If PKI-SRV01 does not have a second drive in your lab setup, see the instructor note at the end of Part C for alternatives.

---

## Pre-Lab Verification

Before starting, confirm the environment is healthy.

### Step 1 — Log in as CORP\pki.admin

Log into PKI-SRV01 using the domain account `CORP\pki.admin`. Confirm you are not using a local account.

```powershell
# Confirm your identity
whoami
```

**Expected output:** `corp\pki.admin`

```
corp\pki.admin
```

**Logged in as CORP\pki.admin (not a local account):**
- [X] Yes
- [ ] No — describe:

### Step 2 — Confirm the CA Service Is Running

```powershell
Get-Service CertSvc
```

**Expected output:**
```
Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services
```

```
Status   Name               DisplayName                           
------   ----               -----------                           
Running  CertSvc            Active Directory Certificate Services 

```

### Step 3 — Confirm the CA Responds

```powershell
certutil -ping
```

**Expected output:**
```
Connecting to PKI-SRV01\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive
CertUtil: -ping command completed successfully.
```

```
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (31ms)
CertUtil: -ping command completed successfully.
```

**CA is running and responding:**
- [X] Yes — proceed to Part A
- [ ] No — describe the issue and resolution:

---

## Part A — Create the Backup Destination

You need a dedicated folder for the CA backup files. The certutil command will write the CA database and private key here.

### Step 1 — Open an Elevated PowerShell Prompt

Right-click **Windows PowerShell** → **Run as Administrator**. Confirm the title bar shows **Administrator: Windows PowerShell**.

### Step 2 — Create the Backup Folder Using File Explorer

1. Press **Windows key + E** to open File Explorer (or click the folder icon on the taskbar).
2. In the left navigation panel, click **This PC**.
3. In the right panel, double-click **Local Disk (C:)** to open the C: drive.
4. Right-click in any empty area of the right panel (not on an existing file or folder).
5. Select **New → Folder** from the context menu.
6. A new folder appears with the name highlighted. Type exactly: `CABackup`
7. Press **Enter** to confirm the name.

The folder is now at `C:\CABackup`.

> **If the folder already exists from a prior backup attempt:** Right-click **CABackup** → **Delete** to remove it, then re-create it using the steps above. certutil will fail if the destination already contains a .p12 file with the same CA name from a previous run.

> **Prefer command line?** You can also run this in the elevated PowerShell prompt:
> `New-Item -ItemType Directory -Path "C:\CABackup" -Force`

### Step 3 — Verify the Folder Exists

Confirm the folder was created before running certutil:

```powershell
dir C:\CABackup
```

**Expected:** An empty directory listing with today's date. No files should be present.

```
PS C:\Windows\system32> dir C:\CABackup

```

**C:\CABackup folder exists and is empty:**
- [X] Yes — confirmed with `dir C:\CABackup`
- [ ] Folder contained old files — cleared before proceeding

---

## Part B — Back Up the CA Database and Private Key

`certutil -backup` captures both the CA database and the CA private key in one command. It uses Windows Volume Shadow Copy Service (VSS) — the CA service remains running during the backup.

### Step 1 — Choose a Backup Password

You will specify a password to protect the private key .p12 file. This password is as sensitive as the private key itself.

**Requirements:** At least 12 characters. Mix of uppercase, lowercase, numbers, and symbols.

> ⚠️ **Record this password in a location SEPARATE from C:\CABackup.** You will need it in Lab 03. If you lose the password, you cannot complete the file-based restore.

**Password storage location (do not write the password here — write where you stored it):**

```
Password saved in Documents folder on Desktop machine - not in the backup folder.
```

### Step 2 — Run certutil -backup

Replace `<YourBackupPassword>` with the password you chose above.

```powershell
certutil -backup -p <YourBackupPassword> C:\CABackup
```

This command backs up:
- The CA database → `C:\CABackup\DataBase\<CAName>.edb` (and log files)
- The CA private key and certificate → `C:\CABackup\<CAName>.p12`

**Expected output (may take 30–60 seconds):**
```
Backup directory: C:\CABackup
Key Container Name: <CAName>
Backing up CA database...
Database backed up successfully.
Backing up private key and certificate...
Private key and certificate backed up successfully.
CertUtil: -backup command completed successfully.
```

> **If you see "Access is denied":** You are not running from an elevated prompt or not logged in as CORP\pki.admin. Right-click PowerShell → Run as Administrator. Run `whoami` to confirm `corp\pki.admin`.

> **If you see "The process cannot access the file because it is being used by another process":** Another backup is in progress or the folder is locked. Wait 60 seconds and retry.

```
Backed up keys and certificates for PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 to C:\CABackup\CVI Issuing CA 1.p12.
Full database backup for PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1.

Backing up Database files: 100%

Backing up Log files: 100%

Truncating Logs: 100%
Backed up database to C:\CABackup.
Database logs successfully truncated.
CertUtil: -backup command completed successfully.
```

**certutil -backup completed without errors:**
- [X] Yes
- [ ] No — error message and resolution:

### Step 3 — Verify Backup Files Exist

```powershell
# List all files in the backup folder
Get-ChildItem C:\CABackup -Recurse | Select-Object FullName, Length, LastWriteTime
```

**Expected files:**

| File | What It Is |
|---|---|
| `C:\CABackup\<CAName>.p12` | CA private key + certificate (PKCS#12 format) |
| `C:\CABackup\DataBase\<CAName>.edb` | CA database file |
| `C:\CABackup\DataBase\*.log` | Database transaction log files |

```
FullName                                  Length  LastWriteTime       
--------                                  ------  -------------       
C:\CABackup\DataBase                              6/5/2026 11:08:45 AM
C:\CABackup\CVI Issuing CA 1.p12          4631    6/5/2026 11:08:45 AM
C:\CABackup\DataBase\certbkxp.dat         398     6/5/2026 11:08:45 AM
C:\CABackup\DataBase\CVI Issuing CA 1.edb 1052672 6/5/2026 11:08:45 AM
C:\CABackup\DataBase\edb00003.log         1048576 6/5/2026 11:08:45 AM
C:\CABackup\DataBase\edb00004.log         1048576 6/5/2026 11:08:45 AM
```

**Record the backup file details:**

```
CA private key file (.p12):
  Full path: C:\CABackup\CVI Issuing CA 1.p12 
  File size: 4631
  Last write time: 6/5/2026 11:08:45 AM

CA database file (.edb):
  Full path: C:\CABackup\DataBase\certbkxp.dat         398     6/5/2026 11:08:45 AM
C:\CABackup\DataBase\CVI Issuing CA 1.edb 
  File size: 1052672 
  Last write time: 6/5/2026 11:08:45 AM
```

**All expected files are present:**
- [X] Yes — all three file types confirmed
- [ ] No — describe what is missing:

### Step 4 — Verify the .p12 File Is Readable

This confirms the private key backup is not corrupt.

```powershell
certutil -dump -p <YourBackupPassword> "C:\CABackup\<CAName>.p12"
```

**Expected output (partial):**
```
================ Certificate 0 ================
...
Subject: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
...
CertUtil: -dump command completed successfully.
```

> **If you see "Cannot find object or property" or a password error:** The .p12 file may be corrupt or the password is wrong. Re-run certutil -backup with the correct password.

```
================ Certificate 0 ================
================ Begin Nesting Level 1 ================
Element 0:
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
----------------  End Nesting Level 1  ----------------
  Provider = Microsoft Software Key Storage Provider
Private key is NOT plain text exportable
Encryption test passed

================ Certificate 1 ================
================ Begin Nesting Level 1 ================
Element 1:
Serial Number: 26373e51a6ab669340c47caef2232ce1
Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
 NotBefore: 4/25/2026 6:15 PM
 NotAfter: 4/25/2046 6:25 PM
Subject: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
CA Version: V0.0
Signature matches Public Key
Root Certificate: Subject matches Issuer
Cert Hash(sha1): b805e6ab548f6e7c57d3989f61de7fe6a51031d1
----------------  End Nesting Level 1  ----------------
No key provider information
Cannot find the certificate and private key for decryption.
CertUtil: -dump command completed successfully.
```

**certutil -dump confirms .p12 is readable with the backup password:**
- [X] Yes
- [ ] No — describe:

---

## Part C — Back Up the Windows System State

The system state backup captures the CA's Windows registry configuration, local certificate store, and Active Directory configuration objects. Run this on PKI-SRV01.

### Step 1 — Install Windows Server Backup Feature (if not present)

```powershell
# Check if Windows Server Backup is installed
Get-WindowsFeature Windows-Server-Backup

# Install if not present
Install-WindowsFeature Windows-Server-Backup
```

```
Display Name                                            Name                       Install State
------------                                            ----                       -------------
[ ] Windows Server Backup                               Windows-Server-Backup          Available

```

**Windows Server Backup feature is installed:**
- [ ] Yes — already installed
- [X] Installed now — describe any prompts:

```
When I first ran wbadmin start systemstatebackup, Windows reported that Windows Server Backup was not installed. I installed the feature using Install-WindowsFeature Windows-Server-Backup -IncludeAllSubFeature, restarted PowerShell, and reran the backup successfully.

```

### Step 2 — Identify a Backup Target Volume

The system state backup requires a target volume different from C:. 

```powershell
# List available drives
Get-PSDrive -PSProvider FileSystem
```

```
Name           Used (GB)     Free (GB) Provider      Root                                                                                                 CurrentLocation
----           ---------     --------- --------      ----                                                                                                 ---------------
C                  31.44         27.95 FileSystem    C:\                                                                                                 Windows\system32
D                                      FileSystem    D:\                                                                                                                 
F                   8.08         11.91 FileSystem    F:\                                                                                                                 
Z                                      FileSystem    \\VBoxSvr\Shared             
```

> **If only C: is available:** The system state backup to a local drive requires a second volume. In this situation, use one of the alternatives below:
> - **Option A:** Create a VHD on C: and mount it as a separate volume — `New-VHD -Path C:\BackupVHD.vhdx -SizeBytes 20GB -Fixed; Mount-VHD -Path C:\BackupVHD.vhdx`
> - **Option B:** Skip the wbadmin step and document the limitation in your lab report. Your instructor may accept this with explanation.
> - **Option C:** Ask your instructor for a second disk configuration in VirtualBox.

**System state backup target drive/path:**

```
F:\

wbadmin requires the backup target to be on a separate disk from the operating system. In my VM, C: and E: were on the same virtual disk, so E: could not be used as the backup target.

I powered off the VM, added a new 20 GB virtual hard disk in Oracle VirtualBox, then initialized and formatted it as an NTFS volume in Disk Management and assigned it drive letter F:. I then used F: as the backup target for the system state backup.
```

### Step 3 — Run the System State Backup

Replace `D:` with your target drive letter:

```powershell
wbadmin start systemstatebackup -backuptarget:D: -quiet
```

This takes 10–30 minutes depending on system configuration.

> **The -quiet flag suppresses the "Are you sure?" confirmation prompt.** Without it, wbadmin will wait for interactive input.

> **If the command fails with "The backup storage location is invalid":** The target must be a fixed local disk or a network share (UNC path). USB drives and mapped drives may not work. Try a different target.

```
The backup operation successfully completed.
The backup of the system state successfully completed [6/30/2026 7:50 PM].
Log of files successfully backed up:
C:\Windows\Logs\WindowsServerBackup\Backup-30-06-2026_19-26-26.log
```

**System state backup completed without errors:**
- [X] Yes — output included above
- [ ] No — error and resolution:
- [ ] Skipped — reason documented:

### Step 4 — Verify the System State Backup

```powershell
wbadmin get versions
```

**Expected output (partial):**
```
Backup time: <today's date and time>
Backup target: <drive>:\WindowsImageBackup\PKI-SRV01\
Version identifier: <version ID>
Can recover: Volume(s), File(s), Application(s), System State, Bare Metal Recovery
```

```
wbadmin 1.0 - Backup command-line tool
(C) Copyright Microsoft Corporation. All rights reserved.

Backup time: 6/30/2026 12:26 PM 
Backup target: Fixed Disk labeled F:
Version identifier: 06/30/2026-19:26
Can recover: Volume(s), File(s), Application(s), System State
Snapshot ID: {fae1a66c-0917-4ec1-8940-a82c22959cde}

```

**System state backup appears in wbadmin get versions output:**
- [X] Yes — timestamp matches today
- [ ] No — describe:

---

## Part D — Confirm CA Is Still Operational After Backup

The backup runs online — the CA should remain operational throughout. Verify before closing the lab.

```powershell
# Confirm CA service is running
Get-Service CertSvc

# Confirm CA responds
certutil -ping

# Publish a fresh CRL (verifies private key is functional)
certutil -CRL
```

```

PS C:\Windows\system32> Get-Service CertSvc

Status   Name               DisplayName                           
------   ----               -----------                           
Running  CertSvc            Active Directory Certificate Services 



PS C:\Windows\system32> certutil -ping
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (31ms)
CertUtil: -ping command completed successfully.

PS C:\Windows\system32> certutil -CRL
CertUtil: -CRL command completed successfully.

```

**CA is operational after backup — all three commands succeeded:**
- [X] Yes
- [ ] No — describe the issue:

---

## Part E — Lab Report

Answer all questions in complete sentences.

**1. Describe the three components of a complete CA backup and explain what would happen during recovery if each one were missing.**

```
A complete CA backup consists of three components: the **CA database**, the **CA certificate and private key**, and the **System State**.

The **CA database** contains all the issued certificates, revoked certificates, and pending certificate requests. If this is missing during recovery, you would lose the CA's history, including issued and revoked certificates and any pending requests that had not been completed.

The **CA certificate and private key** are needed to restore the CA's identity. The CA cannot be restored as the same CA without its private key because it would no longer be able to sign certificates or certificate revocation lists (CRLs). Even if the database is restored, the CA would not be able to function properly without the private key.

The System State contains the server's core operating system configuration. It contains the Windows Registry, the Active Directory database (if the CA is on a domain controller), IIS configuration (if installed), and the local certificate store. If the System State is missing, the server's configuration cannot be fully restored, which can prevent the CA from working correctly even if the database and private key are available.

```

**2. certutil -backup uses the Volume Shadow Copy Service (VSS). What does this mean operationally — specifically, why is it better than stopping the CA service before copying the database files?**

```
VSS (Volume Shadow Copy Service) allows `certutil -backup` to create a point-in-time snapshot of the CA’s volumes so the backup can be taken from a consistent state while the CA is still running.

This is better than stopping the CA service because it avoids downtime and service disruption, and it ensures the database and related files are backed up in a consistent state without being affected by ongoing changes during the backup process.

```

**3. The CA private key backup (.p12 file) is protected by a password you chose. Where did you store the password, and why is storing it in the same folder as the .p12 file a security problem?**

```
I stored the password in a text file on my desktop Documents folder, separate from the .p12 file.

Storing the password in the same folder as the .p12 file would be a security risk because if an attacker or unauthorized user gains access to that directory, they would immediately have both the private key and the password needed to unlock it. This would fully compromise the CA private key.

Keeping them separate reduces the chance of both being exposed together.


```

**4. What does the Windows system state backup capture that the certutil -backup does not? If the system state backup had been skipped, what would a recovery operator need to do manually that they would not need to do if the system state backup were present?**

```
The Windows System State backup captures the server's core configuration, including the Windows Registry, system files, the local certificate store, and the Active Directory database if the CA is installed on a domain controller. `certutil -backup` only backs up the CA database, certificate, and private key.

If the System State backup was skipped, the recovery operator would have to manually rebuild and reconfigure the server before restoring the CA. This would take more time and increase the chance of missing or misconfiguring something.

```

**5. Explain the relationship between backup frequency and Recovery Point Objective (RPO). If this CA performs daily backups and the CA fails on day six of a seven-day backup cycle, what is the maximum data loss — and what specifically is lost?**

```

The Recovery Point Objective (RPO) is the most data you could lose if the CA fails, and it depends on how often backups are done. The more often you back up the CA, the less data you could lose.

If the CA is backed up every day and it fails on day six, the most you would lose is one day's worth of data. That would include any certificate requests, newly issued certificates, certificate revocations, CRLs generated after the last backup, and any pending requests created since the previous backup.

```

---

## Submission Checklist

- [X] Logged in as CORP\pki.admin (not a local account) — whoami output included
- [X] CA service confirmed running — Get-Service CertSvc output included
- [X] CA confirmed responding — certutil -ping output included
- [X] C:\CABackup folder created via File Explorer and confirmed empty before running certutil
- [X] certutil -backup -p run without errors — full output included
- [X] .p12 file confirmed present — file name, path, and size recorded
- [X] .edb database file confirmed present — file name, path, and size recorded
- [X] certutil -dump confirms .p12 is readable with backup password
- [X] Private key backup password stored SEPARATELY from C:\CABackup — storage location documented (not the password itself)
- [X] Windows Server Backup feature installed
- [X] wbadmin system state backup run (or limitation documented with instructor approval)
- [X] wbadmin get versions confirms backup completed with today's timestamp
- [X] CA confirmed still operational after backup — Get-Service, certutil -ping, certutil -CRL all succeeded
- [X] All five lab report questions answered in complete sentences
- [X] Lab file committed to `labs/week-13/lab-01-ca-backup.md`
