# Lab 01 — Diagnose an Expired Certificate

## Incident Summary

**Target System:** portal.metrogeneral.org (simulated via expired.badssl.com)

**Reported Behavior:** TLS failure — patients seeing security warnings when accessing the appointment portal

**Diagnostic Scope:** PKI Diagnostic Framework — all 4 steps

## Diagnostic Steps

Summarize what you checked at each step. Do not copy the lab instructions — describe what you actually did.

**Step 1 — Retrieve:**

**Step 2 — Parse:**

**Step 3 — Validate the Chain:**

**Step 4 — Check Revocation and Trust:**

## Evidence

- Not Before date:
- Not After date:
- Days since expiration:
- Subject (entity the certificate was issued to):
- Issuer:
- Chain status (complete / incomplete):
- OCSP URL present? (yes/no):

## Root Cause

What caused the TLS failure? Be specific — is this a certificate problem, a chain problem, or a configuration problem?

## Remediation

Step-by-step path to resolve this incident:

1.
2.
3.

## Key Findings

## Challenges / Troubleshooting

## Artifacts

- expired_cert.pem
