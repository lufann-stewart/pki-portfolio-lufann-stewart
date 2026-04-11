# Week 01 Lab — Certificate Inspection

## Screenshot Evidence

1. Capture a screenshot of the certificate details in your browser.
2. Save it as:

assets/screenshots/week-01/certificate-inspection.png

3. Embed the screenshot below:

![Certificate Inspection](../../assets/screenshots/week-01/certificate-inspection.png)


## Website Information

**Website inspected:**    
https://www.google.com/

**Issuer (Certificate Authority):**    
SJ Pulse Root CA  

**Valid from:**      
November 19, 2024

**Valid until:**    
November 19, 2026

**Signature algorithm:**    
SHA-384 with RSA Encryption

---

## Subject Alternative Names (SAN Entries)

List at least 2–3 SAN entries:

- *.google.com

- *.appengine.google.com

- *.bdn.dev


## Observations

Document three observations about the certificate.

### Observation 1
The certificate uses Version 3.

### Observation 2
The Key Usage field is marked critical and specifies that the certificate can be used for digital signatures, non-repudiation, and key encipherment.


### Observation 3
The Basic Constraints field is marked as critical and indicates the the certificate is not a certificate authority.

---

## Reflection

Based on your inspection, explain how this certificate contributes to secure HTTPS communication.

Based on the inspection, this certificate contributes to secure HTTPS communication through the process of validation, starting with the server certificate and moving up the chain to the root. The browser validates it by checking that each certificate is valid, not expired, not revoked, and correctly issued. Once validated, the browser can confirm that the server is who it claims to be and safely perform the TLS handshake to ensure HTTPS communication.
