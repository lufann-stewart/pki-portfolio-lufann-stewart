
# Lab 01 — Inspect X.509 Certificate Fields

## Overview
Briefly describe what this lab was about in your own words.
  >This lab was about retrieving and inspecting the information in a certificate, including who issued it, the version number, the expiration date, and other key details.

What PKI concept were you investigating?
  >The PKI concepts we were investigating were authenticity and integrity. By inspecting the certificate information, we could confirm the authenticity of the issuer and the integrity of the certificate because of the signature and public key algorithms used.

---

## Environment
- OS: Windows
- Terminal used (Mac Terminal / Git Bash / WSL): Command Prompt (cmd.exe)
- OpenSSL version (`openssl version`): OpenSSL 3.6.1 

---
## Steps Performed
Summarize the key steps you performed. Do not copy the lab instructions — describe what you actually did.

1. Used openssl s_client -connect google.com:443 -showcerts to get the certificate for T-mobile.
2. Copied the leaf cert and pasted in notepad and saved as leaf_cert.pem and then used a command openssl x509 -in leaf_cert.pem -text -noout in open ssl to view the leaf cert in readble format
3. Verified the PEM and DER versions using `openssl x509 -text -noout` to inspect certificate details.
4. Converted the DER file back to PEM format (`leaf_cert_restored.pem`) to ensure integrity.
5. Used the Windows `fc` command to compare the original PEM file and the restored PEM file.
6. Generated a new RSA private key (`test_key.pem`) for lab testing.
7. Created a self-signed certificate (`test_cert.pem`) using the newly generated private key.
8. Bundled the test certificate and private key into a PFX file (`test_bundle.pfx`) with a password using OpenSSL.
9. Verified the contents of the PFX file with `openssl pkcs12 -info -noout`, confirming the presence of the certificate and encrypted private key.
---
## Certificate Fields

| Field                | Value from your output |
|----------------------|------------------------|
| Version              |  3                     |
| Serial Number        |  09:78:16:af:a4:ed:7c:4b:54:17:b0:48:0b:05:a7:3a  |
| Signature Algorithm  |  sha256WithRSAEncryption   |
| Issuer               |  DigiCert Inc              |
| Subject              |  T-Mobile USA, Inc.        |
| Not Before           |  Feb  9 00:00:00 2026 GMT  |
| Not After            |  Feb  8 23:59:59 2027 GMT  |
| Public Key Algorithm |  rsaEncryption (2048 bit)  |

---

## Observations

1. Who issued the certificate? DigiCert, Inc
2. What domain or organization does it represent? T-Mobile USA, Inc.   
3. When does it expire? Feb 8, 2027 23:59:59 GMT
4. What public key algorithm is used? rsaEncryption (2048 bit)
5. Why does the Issuer field matter in a PKI system? Because trust in the issuer equals trust in the certificate; if we trust the issuer, we can trust that the certificate is authentic.


## Artifacts
- leaf_cert.pem
