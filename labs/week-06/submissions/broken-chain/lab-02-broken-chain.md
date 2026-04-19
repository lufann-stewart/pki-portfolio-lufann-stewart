# Lab 02 — Diagnose a Broken Certificate Chain

## Incident Summary

**Target System:** Radiology imaging platform (simulated via incomplete-chain.badssl.com)

**Reported Behavior:** TLS failure after certificate renewal — vendor says certificate looks fine, connection still failing

**Diagnostic Scope:** PKI Diagnostic Framework — all 4 steps

## Diagnostic Steps

Summarize what you checked at each step. Do not copy the lab instructions — describe what you actually did.

**Step 1 — Retrieve:** 
  >Ran command to retrieve the full chain: `openssl s_client -connect incomplete-chain.badssl.com:443 -showcerts` then copied the leaf certificate into a file and saved it as leaf_cert.pem.

**Step 2 — Parse:**
  >Ran command `openssl x509 -in leaf_cert.pem -text -noout` to parse and display the certificate details.

**Step 3 — Validate the Chain:**
  >I checked if the certificate chain could be validated using `openssl verify leaf_cert.pem`, but it failed because the chain could not be built. I then retried using `openssl verify -untrusted issuer_cert.pem leaf_cert.pem`, but it still failed, confirming the intermediate certificate was missing.

**Step 4 — Check Revocation and Trust:**
  >I checked the certificate for revocation information using `openssl x509 -in leaf_cert.pem -text -noout | findstr OCSP`.

  >The certificate did not include an OCSP responder URL. However, this was not the cause of the failure. I also confirmed that the root CA is trusted by the system, so the failure was not due to trust store configuration. The failure was due to an incomplete certificate chain, not revocation or trust problems.

## Evidence

- Leaf certificate Subject: *.badssl.com 
- Issuer CN (the missing intermediate): R13
- Number of certificates the server sent: 1
- Verify return code from openssl s_client: 21
- openssl verify error before adding intermediate: error 20 at 0 depth lookup: unable to get local issuer certificate
- openssl verify result after adding intermediate with -untrusted: error 20 at 1 depth lookup: unable to get local issuer certificate
error leaf_cert.pem: verification failed
- Is the root CA trusted by your system? (yes/no): Yes — the root CA is present in the system trust store, but the certificate chain could not be validated due to a missing intermediate certificate.

## Root Cause

Is this a certificate problem or a server configuration problem? Explain the distinction clearly — this matters for how the fix is communicated to the team.
  >The root cause is a server configuration issue because the server fails to include the intermediate certificate in the TLS handshake. As a result, clients cannot build a complete chain of trust from the leaf certificate to a trusted root CA.

## Remediation

Step-by-step path to resolve this incident:

1. Download the missing intermediate certificate from the CA using: `curl -o issuer_cert.pem http://r13.i.lencr.org/`
2. Convert the downloaded certificate into PEM format to ensure it could be used by OpenSSL: `openssl x509 -inform der -in issuer_cert.pem -out issuer_cert.pem`  
3. Verify the certificate chain locally by supplying the intermediate certificate manually: `openssl verify -untrusted issuer_cert.pem leaf_cert.pem`

## Key Findings

  >During analysis, I found that the certificate only included a CA Issuers link (http://r13.i.lencr.org/), but did not include an OCSP responder URL. However, this was not related to the failure.

  >The real issue was that the server only provided the leaf certificate during the TLS handshake and did not include the intermediate certificate needed to build a complete trust chain. Because of this, clients were unable to validate the certificate even though the certificate itself was valid and within its validity period.

## Challenges / Troubleshooting

  >The main difficulty was confirming that the failure was caused specifically by the missing intermediate certificate rather than something like OCSP or system trust issues. On Windows, I also ran into issues using OpenSSL paths meant for Linux, which made some verification steps confusing at first.

  >Once I focused on the chain output from openssl s_client, it became clear that only the leaf certificate was being served, which confirmed the root cause.

  >I became confused from mixing s_client handshake errors with verify validation errors. I initially treated them as separate problems, but both were pointing to the same underlying issue: an incomplete certificate chain due to a missing intermediate.

## Artifacts

- leaf_cert.pem, issuer_cert.pem
