# Lab 01 — Diagnose an Expired Certificate

## Incident Summary

**Target System:** portal.metrogeneral.org (simulated via expired.badssl.com)

**Reported Behavior:** TLS failure — patients seeing security warnings when accessing the appointment portal

**Diagnostic Scope:** PKI Diagnostic Framework — all 4 steps

## Diagnostic Steps

Summarize what you checked at each step. Do not copy the lab instructions — describe what you actually did.

**Step 1 — Retrieve:**
  >Used command `openssl s_client -connect expired.badssl.com:443 -showcerts` to pull the server certificate and saved it as a PEM file.

**Step 2 — Parse:**
  >Ran `openssl x509 -in leaf.pem -text -noout` to view the certificate details in a readable format. Confirmed the certificate was expired based on the Not After: Apr 12 23:59:59 2015 GMT date. Also identified the subject as `*.badssl.com` and issuer as COMODO RSA Domain Validation Secure Server CA.

**Step 3 — Validate the Chain:**
  >I ran openssl s_client -connect expired.badssl.com:443 -showcerts to retrieve the full certificate chain. There were 3 certificates in the output (leaf, intermediate, and root), showing the chain was structurally complete. An intermediate CA was present, and the chain correctly terminated at a trusted root.

  >There were no missing certificates in the chain itself. The only validation failure shown was that the certificate had expired, with a verify return code: 10 (certificate has expired). I also saw a local issuer warning (unable to get local issuer certificate), but that did not reflect a missing server chain—just a local validation issue. The main issue was clearly the expired certificate, not a broken chain.

**Step 4 — Check Revocation and Trust:**
  >Ran `openssl x509 -in leaf.pem -text -noout` to check revocation-related fields. Found the OCSP responder URL `http://ocsp.comodoca.com` and CA Issuers URL `http://crt.comodoca.com/COMODORSADomainValidationSecureServerCA.crt`. No OCSP validation was performed, and revocation was not the cause of the failure.

## Evidence

- Not Before date: Apr  9 00:00:00 2015 GMT
- Not After date: Apr 12 23:59:59 2015 GMT
- Days since expiration: 3,650
- Subject (entity the certificate was issued to): *.badssl.com
- Issuer: COMODO RSA Domain Validation Secure Server CA
- Chain status (complete / incomplete): complete 
- OCSP URL present? (yes/no): yes

## Root Cause

What caused the TLS failure? Be specific — is this a certificate problem, a chain problem, or a configuration problem?
  >The TLS failure was caused by an expired certificate. Even though the certificate chain and trust were valid, the certificate was no longer within its validity period, which caused the connection to be rejected.

## Remediation

Step-by-step path to resolve this incident:

  >1. Generate a new key and create a CSR, then submit it to the CA to issue a new certificate with valid dates.
  >2. Replace the expired certificate on the server with the newly issued one.
  >3. Restart the TLS service and verify the new certificate is being served using `openssl s_client -connect expired.badssl.com:443 -showcerts`.

## Key Findings
  >The certificate was structurally valid and issued by a trusted CA, but expired outside its validity window. This alone caused the TLS failure, not a chain or trust issue.

## Challenges / Troubleshooting
  >The main challenge was distinguishing between different types of TLS failures, especially separating certificate expiration from chain validation errors. Initially, chain verification errors (“unable to get local issuer certificate”) made it seem like a trust or chain issue, but further inspection of the certificate validity period confirmed that expiration was the actual root cause. Another challenge was ensuring the correct certificate fields (Not After date, issuer, and SAN) were interpreted correctly to rule out other possible issues.

## Artifacts

- expired_cert.pem
