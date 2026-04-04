# Lab 01 — Generate a CSR

## Overview
Briefly describe what this lab was about in your own words. What PKI concept or workflow were you investigating?

## Environment
- Operating System: Windows
- Terminal Used: cmd.exe
- OpenSSL Version (openssl version): OpenSSL 3.6.1

## Steps Performed

1. Generated a 2048-bit RSA private key using `openssl genrsa -out test_key.pem 2048` and checked its size and structure with `openssl rsa -in test_key.pem -text -noout`.

2. Created a Certificate Signing Request (CSR) using `openssl req -new -key test_key.pem -out test_csr.pem`, filling in all required Subject fields (CN, O, OU, C, ST, L). Verified the CSR using `openssl req -in test_csr.pem -text -noout` to confirm the Subject and the embedded public key.

3. Self-signed the CSR using `openssl x509 -req -in test_csr.pem -signkey test_key.pem -out test_cert.pem -days 365` to simulate a Certificate Authority issuing a certificate. Checked the certificate with `openssl x509 -in test_cert.pem -text -noout` to see the Subject, Issuer, validity period, and public key.

4. Extracted the public key from the private key using `openssl rsa -in test_key.pem -pubout -out key_pub.pem`.

5. Extracted the public key from the certificate using `openssl x509 -in test_cert.pem -pubkey -noout > cert_pub.pem`.

6. Compared both public keys using the `fc` command and got "FC: no differences encountered," showing that the certificate and private key share the same cryptographic key pair.

## Results
- What Subject fields did you include in your CSR, and what does each field communicate to a CA?
    >I included Country (C), State (ST), Locality (L), Organization (O), Organizational Unit (OU), Common Name (CN), and email address. These fields identify the subject (the entity requesting the certificate). The CA uses this information to verify the identity of the requester before issuing a signed certificate.
    
- What output did `openssl req -text` show when you inspected your CSR?
  >The certificate output includes the X.509 structure, which contains the version, serial number, signature algorithm, issuer, validity period, subject, and subject public key information (including the modulus and exponent). It also includes extensions and a digital signature created using the private key. The PEM-encoded certificate block at the bottom represents the same certificate in Base64 format for storage and transmission.
  
- How did the CSR differ from the signed certificate when you compared them?
  >The signed certificate now includes the issuer information and the validity period, which the CSR didn’t have. The CSR only had the subject info and the public key, so the certificate shows that it has been officially issued and can be used.
  
- What did the diff output show when you compared the public key in the CSR vs the signed cert? The fc command showed “no differences encountered,” which means the public key from the certificate matches the public key from the private key. This confirms that they form the same key pair.
  
## Key Findings
- After signing the certificate, the issuer information and validity period were added. The CSR alone only contained the subject and public key.  
- Comparing the public key from the certificate to the one from the private key showed they are identical, confirming the certificate was generated from the correct key pair.  
- The private key must stay private; if someone else had it, they could impersonate the certificate owner.  
- Self-signed certificates can be used for internal testing, but for public trust, a certificate should be signed by a trusted CA.

## Explanation
- Why must the private key never leave the requestor's machine — even when submitting a CSR to a CA?
  >The private key must stay private because it proves your identity. If someone else gets it, they could read information meant only for you, intercept communications (MITM attacks), or impersonate you to create certificates or sign data.
  
- What is the difference between a CSR and a signed certificate?
   >A CSR is a request to get a signature on a cert and a signed certificate is after the request has been fulfilled.
    
- In what real-world scenario would self-signing be appropriate vs submitting to a trusted CA?
  >A self-signed certificate is appropriate for internal networks, testing, or lab environments where public trust isn’t required.

## Challenges / Troubleshooting
>No challenges during this lab.

## Artifacts
- test_csr.pem, test_cert.pem
