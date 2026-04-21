# Lab 02 — Check Certificate Revocation Status with OCSP

## Overview
Briefly describe what this lab was about in your own words. What PKI concept or system behavior were you investigating?
  >This lab is about checking certificate revocation status with OCSP and looking for CRL Distribution Points to verify if a certificate is legitimate.

## Environment
- Operating System: Windows
- Terminal Used: cmd.exe
- OpenSSL Version (openssl version): OpenSSL 3.6.1 
- Website Used for Certificate Retrieval: Github.com

## Steps Performed
Summarize the key steps you performed. Do not copy the lab instructions — describe what you actually did.
1. Retrieved the leaf certificate from GitHub and checked its subject, issuer, and validity dates.
2. Pulled the full certificate chain from GitHub, extracted the issuer certificate, and confirmed it was a CA certificate.
3. Inspected the leaf certificate for revocation info, found the OCSP responder URL, and noted that there was no CRL Distribution Point URL.
4. Queried the OCSP responder using the leaf and issuer certificates to check the real-time status of the certificate.
5. Verified the OCSP response, confirmed the certificate status as good, and looked at the This Update and Next Update fields to understand how OCSP caching works.

## Results
- What OCSP URL did you find in the certificate's Authority Information Access extension?  
  > http://ocsp.sectigo.com

- What was the OCSP response status (`good`, `revoked`, or `unknown`) and what does it mean?  
  > Good. That means the certificate is still valid and hasn’t been revoked.

- What were the `This Update` and `Next Update` values in the OCSP response, and what do they indicate?  
  > These fields define the validity window of the OCSP response. “This Update” indicates when the response was generated, and “Next Update” defines when a fresh response should be retrieved. Between these timestamps, clients can safely reuse the cached response instead of repeatedly querying the OCSP responder.

- Where was the CRL Distribution Point located in the certificate?  
  > The certificate did not include a CRL Distribution Point, indicating reliance on OCSP for revocation checking rather than traditional CRL-based distribution.

## Key Findings
- Not all certificates use CRL Distribution Points; GitHub relies on OCSP for real-time revocation checking, demonstrating a design choice favoring live validation over periodic revocation lists.
- OCSP responses include This Update and Next Update fields, which define a validity window for cached revocation data. This enables clients to reuse responses instead of querying the OCSP responder on every TLS handshake, improving performance.  
- The issuer certificate is required to validate the OCSP response because it is used to verify the responder’s digital signature, ensuring the revocation status has not been tampered with.
- OCSP provides real-time certificate status checking, making it more efficient than CRLs in high-traffic environments where frequent revocation checks are required.
- The “This Update” and “Next Update” timestamps in the OCSP response define a validity window (e.g., Apr 2–Apr 9), enabling OCSP response caching. Clients reuse the cached response until the “Next Update” time instead of querying the OCSP responder during every TLS handshake, improving performance and reducing latency. This behavior is the basis for OCSP stapling as a performance optimization in TLS.
  
## Explanation
- What is the difference between OCSP and CRL as revocation checking methods?
  >OCSP gives real-time status for a specific certificate, while CRL is a list of all revoked certificates published on a schedule.
  
- Why does an OCSP query require both the leaf certificate and the issuer certificate?
  >An OCSP query needs both the leaf certificate and the issuer certificate because the issuer’s public key is used to verify that the OCSP response is authentic. The responder signs the response using the issuer’s private key, so without the issuer certificate, you can’t confirm that the “good” or “revoked” status is legitimate. It’s not just about checking the leaf certificate—it’s about proving that the status itself can be trusted.
  
- In what scenario would a certificate show `unknown` status from an OCSP responder?
  >The responder may not have information on that certificate yet, or it could be down/unreachable.
  
## Challenges / Troubleshooting
- When I tried to query the OCSP responder at first for cdc.gov, I got an error (`Responder Error: unauthorized (6)`). I switched to using GitHub for the certificate and OCSP check, and that worked fine.

## Artifacts
- leaf_cert.pem, issuer_cert.pem, ocsp_response.txt
