# Lab 04 — Full Diagnostic Scenario: Multi-Failure Investigation (Stretch)

## Incident Report

**System:** Metro General EHR portal — ehr.metrogeneral.org

**Reported Behavior:** TLS failure for clinical staff on the 10.22.0.0/24 subnet; main office network unaffected

**Infrastructure Team's Note:** "The certificate was renewed last week. It was working fine before."

## Diagnostic Steps

Work through all four steps. For each, document what you checked and what it ruled in or out.

**Step 1 — Retrieve:**
  >Ran `openssl s_client -connect ehr.metrogeneral.org:443 -servername ehr.metrogeneral.org -showcerts` to pull the server certificate and view what was being presented.

**Step 2 — Parse:**
  >Used openssl x509 -in cert.pem -text -noout to parse the certificate. Checked that the certificate wasn't expired and that the hostname matched. Since the renewal was a full key replacement, I looked at the public key field to confirm a new key was actually used. The Issuer field showed the internal Metro General CA — which is the key detail here, because any device that doesn't have that CA root trusted is going to reject the certificate even if everything else is fine.

**Step 3 — Validate the Chain:**
  >Ran openssl verify -CAfile ca-bundle.pem leaf.pem to test chain validation. The biggest clue in this whole scenario is that office devices work and clinical devices don't — same certificate, same server, different result. That can only mean the problem is on the client side. The CA root was pushed via Group Policy six months ago, but the clinical subnet was added two weeks ago, so those devices never got it. This rules out the certificate and chain being the problem — it's a trust store gap on the clinical devices.

**Step 4 — Check Revocation and Trust:**
  >Checked for OCSP info using openssl x509 -in cert.pem -text -noout | grep -A1 OCSP. Since this was a full certificate and key replacement, the old certificate is still technically valid until it expires — which means it should be revoked so nobody can use it. That's not what's causing the current failure, but it's still something that needs to be handled. You'd verify revocation status by checking the OCSP responder or CRL listed in the old certificate.

## Findings — In Diagnostic Order

List all failures and contributing factors in the order a PKI engineer should address them.

**Finding 1 — Primary Failure**

- Type: Trust Store
- Severity: High 
- Detail: Clinical subnet devices do not trust the internal Metro General CA root certificate due to missing Group Policy distribution after the subnet expansion.
- Evidence: Office subnet works normally, while clinical subnet devices fail TLS validation.
  
**Finding 2 — Contributing Factor**

- Type: Configuration
- Severity: Medium
- Detail: The new 10.22.0.0/24 subnet was added without ensuring those devices were included in the existing PKI trust deployment process.
- Evidence: Subnet was created after CA rollout, but not added to GPO scope.

**Finding 3 — Additional Issue to Address**

- Type: Revocation
- Severity: Low
- Detail: Previous certificate was fully replaced but revocation status must be verified to avoid fallback risk.
- Evidence: Full certificate replacement noted in incident description.

## Root Cause

Go beyond the technical failure. What process gap allowed this to happen? Why did clinical subnet devices end up in a different trust state than office devices?
  >The issue was caused by a gap in PKI trust distribution after the new clinical subnet was introduced. The Group Policy that deploys the internal CA root certificate did not include the new subnet, so those devices never received the trusted root. As a result, clinical devices could not validate the certificate chain, while office devices continued working normally.

## Remediation

Immediate steps (resolve the active TLS failure):

  >1. Update Group Policy to include the clinical subnet and ensure the internal CA root certificate is deployed to all affected endpoints.
  >2. Force policy update on clinical devices to restore trust.

Follow-up steps (address contributing factors and prevent recurrence):

  >1. Assign ownership for PKI and network onboarding so new subnets are included in trust policies by default.
  >2. Add PKI trust distribution to the standard checklist for new network segments.
  >3. Periodically verify that all endpoints across subnets trust the internal CA.

## Key Findings
  >The outage was caused by inconsistent PKI trust distribution across network segments. Clinical devices were not included in the Group Policy scope for the internal CA root certificate, which caused TLS failures even though the certificate itself was valid.

## Challenges / Troubleshooting
  >No issues to report.

## Artifacts

- No certificate files required for this lab
