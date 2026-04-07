# Lab 02 — Diagnose a Broken Certificate Chain

## Incident Summary

**Target System:** Radiology imaging platform (simulated via incomplete-chain.badssl.com)

**Reported Behavior:** TLS failure after certificate renewal — vendor says certificate looks fine, connection still failing

**Diagnostic Scope:** PKI Diagnostic Framework — all 4 steps

## Diagnostic Steps

Summarize what you checked at each step. Do not copy the lab instructions — describe what you actually did.

**Step 1 — Retrieve:**

**Step 2 — Parse:**

**Step 3 — Validate the Chain:**

**Step 4 — Check Revocation and Trust:**

## Evidence

- Leaf certificate Subject:
- Issuer CN (the missing intermediate):
- Number of certificates the server sent:
- Verify return code from openssl s_client:
- openssl verify error before adding intermediate:
- openssl verify result after adding intermediate with -untrusted:
- Is the root CA trusted by your system? (yes/no):

## Root Cause

Is this a certificate problem or a server configuration problem? Explain the distinction clearly — this matters for how the fix is communicated to the team.

## Remediation

Step-by-step path to resolve this incident:

1.
2.
3.

## Key Findings

## Challenges / Troubleshooting

## Artifacts

- leaf_cert.pem, issuer_cert.pem
