
# Lab 03 — Verify a Certificate Chain

## Overview
Briefly describe what this lab was about in your own words.
  >This lab was about observing the certificate chain and identifying each certificate by looking at key fields. It also focused on understanding whether the chain is valid or broken, and if it’s broken, why.
  
What PKI concept were you investigating?
  >The certificate chain and the differences between the root, intermediate, and server certificates.

---

## Environment
- OS: Windows
- Terminal used (Mac Terminal / Git Bash / WSL): Command Prompt (cmd.exe)
- OpenSSL version (`openssl version`): OpenSSL 3.6.1 
- Website used: tmobile.com

---

## Chain Verification Result
Paste the output of your `openssl verify` command:
  >leaf_cert.pem: OK

---

## Certificate Roles

| Certificate  | Role                        | Key Indicator                    |
|--------------|-----------------------------|----------------------------------|
| root.pem     |    Root CA                         |   CN=DigiCert Assured ID Root G3 (Self-signed)                               |
| intermediate.pem |    Intermediate CA                     |  CN=DigiCert Global G3 TLS ECC SHA384 2020 CA1 (Signed by root.pem)                          |
| server.pem   |          Server/Leaf  |          CN=www.t-mobile.com (Signed by intermediate.pem)                     |

---

## Observations

1. Did the chain verify successfully? What did the output say?  
> Yes, the certificate verified successfully. The output returned leaf_cert.pem: OK.

2. How did you identify the root CA?  
> I could tell it was the root because it's self-signed, so the Issuer and Subject are the same. Plus, the Basic Constraints were set to True. Those two things told me it was the root.

3. How did you identify the intermediate CA?  
> The intermediate certificate is identified because its Subject matches the Issuer of the leaf certificate, confirming it acts as the bridge between the server and root certificates. Additionally, the Basic Constraints field shows CA:TRUE, meaning it can issue certificates. However, this alone does not distinguish it from the root certificate. The root certificate is identified as self-signed because its Subject and Issuer are the same, which differentiates it from the intermediate certificate.

4. What field confirms whether a certificate can issue other certificates?  
> The Basic Constraints field. Root and intermediate certificates have CA:TRUE, while the leaf/server certificate has CA:FALSE.

5. Why does removing the intermediate certificate break the chain?  
> Because the intermediate certificate links the server certificate to the trusted root. If it’s missing, OpenSSL can’t build a complete trust chain, so the server certificate can’t be verified.

---

## Challenges / Troubleshooting
Document any issues encountered and how you resolved them.
  >The verification initially failed because the intermediate certificate was missing, so OpenSSL couldn’t complete the chain. After adding the trusted CA bundle and the intermediate certificate using the -CAfile and -untrusted options, the command worked and returned leaf_cert.pem: OK.

---
## Artifacts
leaf_cert.pem, intermediate.pem, DigiCertAssuredIDRootG3.crt.pem, cacert.pem
