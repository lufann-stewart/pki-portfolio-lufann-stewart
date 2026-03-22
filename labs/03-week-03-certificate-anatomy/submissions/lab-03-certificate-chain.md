
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
  >C=US, ST=Washington, L=Bellevue, O=T-Mobile USA, Inc., CN=www.t-mobile.com
error 20 at 0 depth lookup: unable to get local issuer certificate
error C:\Users\lmste\Documents\leaf_cert.pem: verification failed

---

## Certificate Roles

| Certificate  | Role                        | Key Indicator                    |
|--------------|-----------------------------|----------------------------------|
| root.pem     |    Root CA                         |   Self-signed, trusted by browser                               |
| intermediate.pem |    Intermediate CA                     |  Signed by root, signs server cert                                |
| server.pem   |          Server CA                   |          `CN = www.t-mobile.com/ `signed by intermediate                     |

---

## Observations

1. Did the chain verify successfully? What did the output say?  
> The output was C=US, ST=Washington, L=Bellevue, O=T-Mobile USA, Inc., `CN=www.t-mobile.com`  
> error 20 at 0 depth lookup: unable to get local issuer certificate  
> error C:\Users\lmste\Documents\leaf_cert.pem: verification failed  
> The chain didn’t verify successfully because the intermediate certificate wasn't provided, which broke the chain.

2. How did you identify the root CA?  
> I could tell it was the root because it's self-signed, so the Issuer and Subject are the same. Plus, the Basic Constraints were set to True. Those two things told me it was the root.

3. How did you identify the intermediate CA?  
> I looked at the CN in the Subject field, and it didn’t match the CN in the Issuer field, which helped me figure out it was the intermediate. Also, the Basic Constraints being True made it clear it was an intermediate since it could issue certificates. Those two things made it pretty clear.

4. What field confirms whether a certificate can issue other certificates?  
> The Basic Constraints field. Root and intermediate certificates have CA:TRUE, while the leaf/server certificate has CA:FALSE.

5. Why does removing the intermediate certificate break the chain?  
> Because each part of the chain must be valid and trusted. If the intermediate certificate is missing, the chain is broken and the browser/OpenSSL cannot trust the server certificate.
