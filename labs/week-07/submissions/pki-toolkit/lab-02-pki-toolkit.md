# My PKI Toolkit

**CVI PKI Career Pathway — Phase 1 Foundations**

Completed: April 2026

---

## About This Document

[2–3 sentences describing what this document is and why you built it. Write in your own words.]
  >This document is a reference of the commands and tools I used throughout Phase 1 of the labs. It explains what each tool does and when I used it in practice. I built it to reinforce what I learned and to create a personal guide I can use for future PKI and security work.

---

## Core Command-Line Tools

### openssl x509 — Inspect X.509 certificate fields
**What it does:**
  >Reads and displays certificate details like issuer, validity, SANs, and extensions.
 
**When to use it:**
  >When you already have a certificate file and need to inspect what’s inside it.

**Example command from my labs:**

```bash
`openssl x509 -text -noout`
```

**What the output tells you:**
  >It shows whether the certificate is valid, who issued it, and what domains it applies to.

**Phase 1 source:** Week 3, Inspect X.509 Certificate Fields

### openssl s_client — View live certificate chain from a server
**What it does:**
  >Opens a live TLS connection to a server and shows the certificate chain being used.

**When to use it:**
  >When you want to see what certificate a real website is actually serving.

**Example command from my labs:**

```bash
`openssl s_client -connect google.com:443 -showcerts`
```

**What the output tells you:**
  >It shows the full certificate chain, TLS version, and cipher being used.

**Phase 1 source:** Week 3, Inspect X.509 Certificate Fields

### openssl req — Generate a CSR (Certificate Signing Request)

**What it does:**
  >Generates a certificate signing request (CSR) used to request a signed certificate from a CA.

**When to use it:**
  >When creating a new certificate for a server or service.

**Example command from my labs:**

```bash
`openssl req -new -key test_key.pem -out test_csr.pem`
```

**What the output tells you:**
  >Produces a CSR file that gets sent to a certificate authority for signing.

**Phase 1 source:** Week 5, Generate a CSR

### openssl verify — Validate certificate trust chain

**What it does:**
  >Checks whether a certificate is trusted and correctly chained to a root CA.

**When to use it:**
  >When validating if a certificate is valid or expired.

**Example command from my labs:**

```bash
`openssl verify labs/week-06/submissions/expired-cert/expired_cert.pem`
```

**What the output tells you:**
Shows whether trust succeeds or fails, and why (expired, untrusted, missing chain, etc.).

**Phase 1 source:** Week 5, Simulate a Certificate Expiration Scenario (Stretch)

### openssl ocsp — Check certificate revocation status

**What it does:**
  >Queries an OCSP responder to check if a certificate has been revoked.

**When to use it:**
  >When you need real-time revocation status instead of waiting for CRL updates.

**Example command from my labs:**

```bash
`openssl ocsp -issuer labs/week-05/submissions/revocation-status/issuer_cert.pem -cert labs/week-05/submissions/revocation-status/leaf_cert.pem -url http://ocsp.sectigo.com -resp_text -noverify 2>/dev/null | tee labs/week-05/submissions/revocation-status/ocsp_response.txt`
```

**What the output tells you:**
  >Whether the certificate is good, revoked, or unknown according to the CA.

**Phase 1 source:** Week 5, Check Certificate Revocation Status with OCSP

### openssl pkcs12 — Work with PFX/P12 certificate bundles

**What it does:**
  >Handles .pfx/.p12 files that contain certificates and private keys together.

**When to use it:**
  >When exporting or inspecting certificates used in Windows or enterprise systems.

**Example command from my labs:**

```bash
`openssl pkcs12 -info -noout`
```

**What the output tells you:**
  >Whether the file contains a valid cert chain and private key.

**Phase 1 source:** Week 3, Inspect X.509 Certificate Fields

[Add any additional openssl subcommands from your lab history]
### openssl rsa — Extract public key from private key
```bash
openssl rsa -in test_key.pem -pubout -out key_pub.pem
```
Used to generate a public key from a private key during CSR and key generation labs.

---

### fc — Compare two certificate files (Windows file compare)
```bash
fc leaf_cert.pem leaf_cert_restored.pem
```
Used to compare two certificate files to confirm they were identical after conversion.  
Result: no differences found.

---

### Import-Certificate (PowerShell) — Install certificate into Windows trust store
```powershell
Import-Certificate -FilePath "test-root-ca.crt" -CertStoreLocation Cert:\LocalMachine\Root
```
Used to install a root CA certificate into the local machine trusted root store for trust validation testing.

---

### Get-ChildItem + Remove-Item — Verify and remove test certificate
```powershell
Get-ChildItem Cert:\LocalMachine\Root | Where-Object { $_.Subject -like "*CVI-lab*" } | Remove-Item
```
Used to verify the installed test root certificate and then remove it after completing validation.

Week 4 Install a Certificate and Validate Trust (Stretch)
---

## Network and Inspection Tools

### curl — Download certificate or intermediate CA

**What it does:**
  >Downloads data from a URL, often used to retrieve intermediate certificates.
**When to use it:**
  >When a certificate chain is missing an intermediate CA.
**Example command from my labs:**

```bash
`curl -o issuer_cert.pem http://r13.i.lencr.org/`
```

**What the output tells you:**
Saves the downloaded certificate file locally.
**Phase 1 source:** Week 6, Diagnose a Broken Certificate Chain

### openssl genrsa — Generate a RSA key

**What it does:**
  >Generated a 2048-bit RSA private key
**When to use it:**
  >when Generated a 2048-bit RSA private key for a csr requests
**Example command from my labs:**

```bash
`openssl genrsa -out test_key.pem 2048`
```

**What the output tells you:**
That it generated a key
**Phase 1 source:** Week 5, Generate a CSR

### openssl verify — Use to vaidate the certificate chain

**What it does:**
  >Use to verification to see if t cert is valid or not
**When to use it:**
  >To see if the cert can connet
**Example command from my labs:**

```bash
[real command from your Phase 1 submissions]
```

**What the output tells you:**

**Phase 1 source:** Week [X], [Lab name]

---

## Reference and Analysis Services

### SSL Labs SSL Server Test — Get tls and cipher suite info

**What it does:**
  >Searches Certificate Transparency logs to show issued certificates for a domain.
**When to use it:**
  >When analyzing how many certificates exist or which CAs are used.
**What to look for in the output:**
  >Shows issuance history, domains, validity periods, and issuing CAs.

**Phase 1 source:** Week 7, Analyze a Live Enterprise Certificate Deployment

### crt.sh — Certificate Transparency Search — [Brief description]

**What it does:**
Shows cert ingo about how many certs were inssued over the past and the root cas the crt.sh id numbers the comman name the not before and not after dates nad matching identites info
**When to use it:**
When you need to look at patterns to deteremine what kijnd of deployment enviorment this is?
**What to look for in the output:**
the fields i listed aboue
**Phase 1 source:** Week 7, Analyze a Live Enterprise Certificate Deployment

---

## Phase 1 Skills Summary

[3–5 sentences summarizing what Phase 1 built as a whole and how these tools connect to real PKI operational work. Written in your own words.]
  >Phase 1 built the foundational skills and tools needed for thos eto loof kfor the foundational information about a cert , how to generate keys, spot diferences and search key information that is necessary to learn about a vert.
