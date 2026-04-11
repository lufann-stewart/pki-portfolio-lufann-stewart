# Lab 03 — Diagnose a Hostname and SAN Mismatch

## Incident Summary

**Target System:** Staff scheduling portal (simulated via wrong.host.badssl.com)

**Reported Behavior:** Browser security errors after the portal was moved to a new hostname — staff cannot access the scheduling system

**Diagnostic Scope:** PKI Diagnostic Framework — all 4 steps

## Diagnostic Steps

Summarize what you checked at each step. Do not copy the lab instructions — describe what you actually did.

**Step 1 — Retrieve:**

**Step 2 — Parse:**

**Step 3 — Validate the Chain:**

**Step 4 — Check Revocation and Trust:**

## Evidence

- Hostname accessed (what the client expected):
- Subject CN (what the certificate says):
- SAN DNS entries (list all):
- Do any SAN entries match the hostname? (yes/no):
- Verify return code from openssl s_client:
- Does the chain validate independently of the hostname issue? (yes/no):
- OCSP URL present? (yes/no):

## Root Cause

Is this a certificate problem, a chain problem, or a configuration/deployment problem? Explain why the distinction matters.

## Remediation

Step-by-step path to resolve this incident:

1.
2.
3.

### Why a DNS CNAME alias would not fix this

Explain clearly — in terms a non-technical manager could follow — why adding a CNAME from `staff.metrogeneral.org` back to `scheduling.metrogeneral.org` does not resolve the TLS error.

## Key Findings

## Challenges / Troubleshooting

## Artifacts

- No certificate files required for this lab
