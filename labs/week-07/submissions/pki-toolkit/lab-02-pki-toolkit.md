# My PKI Toolkit

**CVI PKI Career Pathway — Phase 1 Foundations**

Completed: April 2026

---

## About This Document

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
openssl x509 -in leaf.pem -text -noout
```

**What the output tells you:**
  >It shows whether the certificate is valid, who issued it, and what domains it applies to.

**Phase 1 source:** Week 3, Inspect X.509 Certificate Fields

---

### openssl s_client — View live certificate chain from a server
**What it does:**
  >Opens a live TLS connection to a server and shows the certificate chain being used.

**When to use it:**
  >When you want to see what certificate a real website is actually serving.

**Example command from my labs:**

```bash
openssl s_client -connect google.com:443 -showcerts
```

**What the output tells you:**
  >It shows the server’s presented certificate chain, TLS version, and cipher being used.

**Phase 1 source:** Week 3, Inspect X.509 Certificate Fields

---

### openssl req — Generate a CSR (Certificate Signing Request)

**What it does:**
  >Generates a certificate signing request (CSR) used to request a signed certificate from a CA.

**When to use it:**
  >When creating a new certificate for a server or service.

**Example command from my labs:**

```bash
openssl req -new -key test_key.pem -out test_csr.pem
```

**What the output tells you:**
  >Produces a CSR file that gets sent to a certificate authority for signing.

**Phase 1 source:** Week 5, Generate a CSR

---

### openssl verify — Validate certificate trust chain

**What it does:**
  >Checks whether a certificate is trusted and correctly chained to a root CA.

**When to use it:**
  >When checking whether a certificate chains correctly to a trusted root CA.

**Example command from my labs:**

```bash
openssl verify labs/week-06/submissions/expired-cert/expired_cert.pem
```

**What the output tells you:**  
  >Shows whether trust succeeds or fails, and why (expired, untrusted, missing chain, etc.).

**Phase 1 source:** Week 5, Simulate a Certificate Expiration Scenario (Stretch)

---

### openssl ocsp — Check certificate revocation status

**What it does:**
  >Queries an OCSP responder to check if a certificate has been revoked.

**When to use it:**
  >When you need real-time revocation status instead of waiting for CRL updates.

**Example command from my labs:**

```bash
openssl ocsp -issuer labs/week-05/submissions/revocation-status/issuer_cert.pem -cert labs/week-05/submissions/revocation-status/leaf_cert.pem -url http://ocsp.sectigo.com -resp_text -noverify 2>/dev/null | tee labs/week-05/submissions/revocation-status/ocsp_response.txt
```

**What the output tells you:**
  >Whether the certificate is good, revoked, or unknown according to the CA.

**Phase 1 source:** Week 5, Check Certificate Revocation Status with OCSP

---

### openssl pkcs12 — Work with PFX/P12 certificate bundles

**What it does:**
  >Handles .pfx/.p12 files that contain certificates and private keys together.

**When to use it:**
  >When exporting or inspecting certificates used in Windows or enterprise systems.

**Example command from my labs:**

```bash
openssl pkcs12 -in bundle.pfx -info -noout
```

**What the output tells you:**
  >Displays the contents of a PKCS#12 bundle, including certificates and (if accessible) the private key.

**Phase 1 source:** Week 3, Inspect X.509 Certificate Fields

---

### openssl genrsa — Generate a RSA key

**What it does:**
  >Generates an RSA private key of a specified bit size (commonly 2048-bit for lab and production use).

**When to use it:**
  >When creating a private key for a CSR or certificate.

**Example command from my labs:**

```bash
openssl genrsa -out test_key.pem 2048
```

**What the output tells you:**
  >Creates a private key file that will be used to generate a CSR.

**Phase 1 source:** Week 5, Generate a CSR

---

### openssl rsa — Extract public key from private key

**What it does:**
  >Extracts the public key from an existing private key.

**When to use it:**
  >When you need to extract and inspect the public key from an existing private key file during key or CSR work.

**Example command:**
```bash
openssl rsa -in test_key.pem -pubout -out key_pub.pem
```

**What the output tells you:**
  >Extracts or displays RSA key details and can output the public key from a private key.

**Phase 1 source:** Week 5, Generate a CSR

---

## Additional Tools and Commands

### fc — Compare two certificate files (Windows file compare)

**What it does:**
  >Compares two files and checks whether they are identical or different.

**When to use it:**
  >When validating that a certificate or file conversion did not change the contents.

**Example command from my labs:**
```bash
fc leaf_cert.pem leaf_cert_restored.pem
```

**What the output tells you:**
  >Shows whether files match exactly or highlights differences between them.

**Phase 1 source:** Week 5, Generate a CSR & Week 4, Convert Certificate Formats 

---

### Import-Certificate (PowerShell) — Install certificate into Windows trust store

**What it does:**
  >Installs a certificate into the Windows certificate store.

**When to use it:**
  >When testing trust validation or adding a root CA for local verification.

**What the output tells you:**
  >Confirms the certificate was successfully added to the specified certificate store.

**Example command from my labs:**
```powershell
Import-Certificate -FilePath "test-root-ca.crt" -CertStoreLocation Cert:\LocalMachine\Root
```

**Phase 1 source:** Week 4, Install a Certificate and Validate Trust (Stretch)

---

### Get-ChildItem + Remove-Item — List and remove test certificates

**What it does:**
  >Lists certificates in a Windows certificate store and removes selected entries.

**When to use it:**
  >When verifying that a certificate was installed correctly and then cleaning up after testing.

**Example command from my labs:**
```powershell
Get-ChildItem Cert:\LocalMachine\Root | Where-Object { $_.Subject -like "*CVI-lab*" } | Remove-Item
```

**What the output tells you:**
  >Shows whether the certificate exists, and confirms removal when deleted.

**Phase 1 source:** Week 4, Install a Certificate and Validate Trust (Stretch)

---

## Network and Inspection Tools

### curl — Download certificate or intermediate CA

**What it does:**
  >Downloads data from a URL, often used to retrieve intermediate certificates.

**When to use it:**
  >When a certificate chain is missing an intermediate CA.

**Example command from my labs:**
```bash
curl -o issuer_cert.pem http://r13.i.lencr.org/
```

**What the output tells you:**
  >Saves the downloaded certificate file locally.

**Phase 1 source:** Week 6, Diagnose a Broken Certificate Chain

---

## Reference and Analysis Services

### SSL Labs SSL Server Test — Analyze TLS configuration

**What it does:**
  >Analyzes a public website’s TLS configuration and assigns a security grade.

**When to use it:**
  >When you want a full external view of a site’s TLS setup, including protocols, ciphers, and misconfigurations.

**What to look for in the output:**
  >Overall grade, supported TLS versions, weak ciphers, and certificate details.

**Phase 1 source:** Week 7, Analyze a Live Enterprise Certificate Deployment

---

### crt.sh — Certificate Transparency Search

**What it does:**
  >Shows publicly logged certificates issued for a domain, including issuers, dates, and matching names.

**When to use it:**
  >When you want to see certificate history and patterns across environments.

**What to look for in the output:**
  >Issuer names, number of certificates, validity dates, and domain variations.

**Phase 1 source:** Week 7, Analyze a Live Enterprise Certificate Deployment

---

## Phase 1 Skills Summary

  >Phase 1 was hands-on work with certificates and PKI tools. I learned how to inspect certificates, generate keys and CSRs, validate trust chains, and troubleshoot certificate issues like expiration and revocation. These are the same kinds of tools used when diagnosing real TLS problems in systems.
