# Lab 04 — Full Diagnostic Scenario: Multi-Failure Investigation (Stretch)

## Incident Report

**System:** Metro General EHR portal — ehr.metrogeneral.org

**Reported Behavior:** TLS failure for clinical staff on the 10.22.0.0/24 subnet; main office network unaffected

**Infrastructure Team's Note:** "The certificate was renewed last week. It was working fine before."

## Diagnostic Steps

Work through all four steps. For each, document what you checked and what it ruled in or out.

**Step 1 — Retrieve:**
Retrieved the certificate information running command: openssl s_client -connect ehr.metrogeneral.org:443 -showcerts

**Step 2 — Parse:**
Would parse lal cetifiate fields using command: openssl x509 -in cert.pem -text -noout

**Step 3 — Validate the Chain:**
Validate the chain with the full CA bundle : openssl verify -CAfile ca-bundle.pem leaf.pem

**Step 4 — Check Revocation and Trust:**
Locate the OCSP responder URL openssl x509 -text -noout | grep -A1 "OCSP and then query the OCSP responder directly using openssl ocsp -issuer issuer.pem -cert leaf.pem -url [ocsp
url] -resp_text

## Findings — In Diagnostic Order

List all failures and contributing factors in the order a PKI engineer should address them.

**Finding 1 — Primary Failure**

- Type: Trust Store
- Severity: High 
- Detail: Clinical subnet devices do not trust Metro General Internal CA - G2 due to missing Group Policy distribution after subnet expansion.
- Evidence: Office subnet works; clinical subnet fails; GPO applied before subnet existed
  
**Finding 2 — Contributing Factor**

- Type: Configuration
- Severity: Medium
- Detail: New subnet introduced without ensuring endpoint enrollment into existing PKI trust policy.
- Evidence: Subnet added 2 weeks ago after CA rollout 6 months ago

**Finding 3 — Additional Issue to Address**

- Type: Revocation
- Severity: Low
- Detail: Previous certificate was fully replaced but revocation status must be verified to avoid fallback risk.
- Evidence: Scenario states full replacement of cert and key

## Root Cause

Go beyond the technical failure. What process gap allowed this to happen? Why did clinical subnet devices end up in a different trust state than office devices?
When the new subnet was created the root didnt push to this via Group policy.

## Remediation

Immediate steps (resolve the active TLS failure):

1. Add the subdomain to trust the root in other words get it to trust the root becuase it was never told to.
2. push the deployment 

Follow-up steps (address contributing factors and prevent recurrence):

1. CHange policy or create a owner 
2.
3.

## Key Findings

## Challenges / Troubleshooting

## Artifacts

- No certificate files required for this lab
