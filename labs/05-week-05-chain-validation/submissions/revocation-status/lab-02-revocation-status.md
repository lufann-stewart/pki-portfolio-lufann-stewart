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
  > They show when the status was last updated and when the next update is scheduled, basically how long this info is valid before it refreshes.

- Where was the CRL Distribution Point located in the certificate?  
  > I couldn’t find a CRL URL for GitHub — looks like they just rely on OCSP instead.

## Key Findings
- Not all certificates have a CRL URL — GitHub uses OCSP instead.  
- OCSP provides real-time certificate validation, which is better for high-traffic systems.  
- The issuer certificate is always required to verify the authenticity of the OCSP response.  

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
