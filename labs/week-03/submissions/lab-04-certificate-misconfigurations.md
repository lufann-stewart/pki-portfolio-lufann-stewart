
# Lab 04 — Detect Certificate Misconfigurations

## Overview
Briefly describe what this lab was about in your own words.
  >This lab focused on identifying and analyzing common certificate misconfigurations, including missing SAN, incorrect Extended Key Usage (EKU), expired certificates, and missing intermediate certificates, and how these issues affect browser trust and certificate validation.

What PKI concept were you investigating? 
  >Certificate validation and trust chain management

---

## Scenario 1 — Missing Subject Alternative Name

**Would modern browsers trust this certificate?**
  >No.

**Analysis:**
[Explain why SAN is required, why CN is not sufficient, and what error users would see] 

  >SAN is required because it specifies the valid domain names a certificate can be used for. Modern browsers ignore the CN if no SAN is present, so a certificate with only a CN will be rejected. Users would see an error such as `NET::ERR_CERT_COMMON_NAME_INVALID.`


---

## Scenario 2 — Incorrect Extended Key Usage

**Would a browser accept this certificate for a web server?**
  >No.

**Analysis:**
[Explain what EKU defines, what value is required for HTTPS, and what error users would see]  

  >The EKU (Extended Key Usage) defines what a certificate is allowed to be used for. For HTTPS, the certificate must include TLS Web Server Authentication. In this scenario, the certificate only includes Client Authentication, which is intended for identifying clients, not servers. Because of this, the browser will reject the certificate as invalid for a web server and display an error such as `ERR_CERT_INVALID.`

---

## Scenario 3 — Expired Certificate

**What happens if this certificate is used today?**
  >If the certificate is expired, it fails validation even if the trust chain is otherwise intact. The browser will issue a security warning and may block access to the site. Users will see an error such as `NET::ERR_CERT_DATE_INVALID` indicating that the certificate is no longer valid.

**Analysis:**
[Explain why expiration fails validation, why lifecycle management matters, and what users would see]
  >Because certificates are only valid within a specific period, an expired certificate fails validation even if the rest of the trust chain is intact. Lifecycle management matters so that certificates can be renewed before expiration, preventing interrupted access. Users will see a browser warning indicating the certificate is expired, often with an option to bypass the warning.
---

## Scenario 4 — Missing Intermediate Certificate

**Can the browser build a complete trust chain?**
  >No. Even if the root CA is trusted, if an intermediate certificate is missing, the browser cannot complete the chain. The entire chain — server certificate, any intermediate certificates, and the root CA — must be valid for the trust chain to work.

**Analysis:**
[Explain why the full chain must be served, what happens when the intermediate is missing, and how this is fixed]
  >Each part of the trust chain must be valid. If an intermediate certificate is missing, the entire trust chain fails and browsers will not trust the site. To fix this, the missing intermediate certificate must be installed on the server along with the server certificate, so the full chain is served to the browser.
---

## Key Takeaway
What is the most important thing you learned about certificate misconfigurations from this lab?
  >The most important thing I learned is that certificate validation relies on multiple factors: SAN, EKU, expiration, and the full trust chain. Misconfiguration in any of these can cause browsers to reject a certificate, even if it otherwise looks valid.
