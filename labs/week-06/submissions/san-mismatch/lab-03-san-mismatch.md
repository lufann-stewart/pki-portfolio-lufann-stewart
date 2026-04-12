# Lab 03 — Diagnose a Hostname and SAN Mismatch

## Incident Summary

**Target System:** Staff scheduling portal (simulated via wrong.host.badssl.com)

**Reported Behavior:** Browser security errors after the portal was moved to a new hostname — staff cannot access the scheduling system

**Diagnostic Scope:** PKI Diagnostic Framework — all 4 steps

## Diagnostic Steps

Summarize what you checked at each step. Do not copy the lab instructions — describe what you actually did.

**Step 1 — Retrieve:**
  >Ran openssl s_client -connect wrong.host.badssl.com:443 -servername wrong.host.badssl.com -showcerts to retrieve the certificate.
Saved the leaf as leaf_cert.pem file

**Step 2 — Parse:**
  >Ran openssl x509 -in leaf_cert.pem -text -noout to view the SAN information within the certificate.

**Step 3 — Validate the Chain:**
Ran openssl verify -untrusted intermediate.pem leaf_cert.pem to check the certificate chain using the intermediate manually. The verification failed with an “unable to get local issuer certificate” error, meaning the chain still couldn’t be fully validated even after adding the intermediate.

**Step 4 — Check Revocation and Trust:**
  >First ran openssl x509 -in leaf_cert.pem -text -noout | findstr /i OCSP to check for an OCSP responder URL. No OCSP URL was returned in the output. Then ran openssl x509 -in leaf_cert.pem -text -noout to review the full certificate details and confirm OCSP information was not present or relevant to the issue.

## Evidence

- Hostname accessed (what the client expected): wrong.host.badssl.com
- Subject CN (what the certificate says): *.badssl.com
- SAN DNS entries (list all): *.badssl.com, DNS:badssl.com
- Do any SAN entries match the hostname? (yes/no): no
- Verify return code from openssl s_client: 20
- Does the chain validate independently of the hostname issue? (yes/no): no
- OCSP URL present? (yes/no): no

## Root Cause

Is this a certificate problem, a chain problem, or a configuration/deployment problem? Explain why the distinction matters.
  >The main issue was a hostname mismatch between wrong.host.badssl.com and what was listed in the certificate SAN. The certificate only covered *.badssl.com and badssl.com, so TLS rejected it even though the cert itself was valid and issued by a trusted CA. I also saw OCSP info in the certificate, but it didn’t matter for this failure. The problem was strictly the SAN not matching the hostname, not a revocation or trust chain issue.

## Remediation

Step-by-step path to resolve this incident:

1. Reissue or update the certificate so it includes the correct hostname (wrong.host.badssl.com) in the SAN field.
   
2. Make sure all the actual hostnames the service uses are included in the SAN going forward so this doesn’t happen again.
   
3. Install the updated cert on the server and restart the TLS service so the new certificate is what gets presented during connection.

### Why a DNS CNAME alias would not fix this

Explain clearly — in terms a non-technical manager could follow — why adding a CNAME from `staff.metrogeneral.org` back to `scheduling.metrogeneral.org` does not resolve the TLS error.
  >A CNAME is like updating the directions to send people to a different building. But when they arrive, security still checks the ID badge (SAN). If the badge doesn’t match the name they used to get there, they get denied entry. So even if the redirect works, the certificate SAN check will still fail.

## Key Findings
  >The certificate presented for the portal did not include the hostname wrong.host.badssl.com in its SAN field. Instead, it only covered *.badssl.com and badssl.com, which does not match the requested hostname. I also confirmed that OCSP information was not relevant to this issue, and the failure was strictly due to hostname mismatch, not chain or trust problems.

## Challenges / Troubleshooting
  >interpreting wildcard behavior in the SAN field and understanding why *.badssl.com does not cover multi-level subdomains like wrong.host.badssl.com.

## Artifacts

- No certificate files required for this lab
