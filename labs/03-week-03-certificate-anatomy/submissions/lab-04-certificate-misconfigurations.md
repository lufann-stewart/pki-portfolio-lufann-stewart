
# Lab 04 — Detect Certificate Misconfigurations

## Overview
Briefly describe what this lab was about in your own words.
  >This lab focused on lifecycle management and EKU, and why they are necessary and important.

What PKI concept were you investigating? 
  >Certificate Lifecycle Management

---

## Scenario 1 — Missing Subject Alternative Name

**Would modern browsers trust this certificate?**
  >No.

**Analysis:**
[Explain why SAN is required, why CN is not sufficient, and what error users would see] 

  >SAN is required because it lists multiple domains and subdomains covered by a single certificate, allowing browsers to securely connect to the domain being accessed. The CN is not sufficient because it only covers one domain and would require multiple certificates to handle additional domains. If modern browsers relied on the CN and tried to access a subdomain, users could see an error like `NET::ERR_CERT_COMMON_NAME_INVALID.`


---

## Scenario 2 — Incorrect Extended Key Usage

**Would a browser accept this certificate for a web server?**
  >Yes, because the EKU includes TLS Web Server Authentication, which is required for HTTPS connections.

**Analysis:**
[Explain what EKU defines, what value is required for HTTPS, and what error users would see]  

  >The EKU (Extended Key Usage) defines what a certificate is specifically allowed to be used for. For HTTPS, the certificate must include TLS Web Server Authentication. If a certificate did not include this value, modern browsers would display an error indicating the certificate cannot be used for this purpose, such as `ERR_CERT_INVALID` or a warning about an invalid certificate for the server.

---

## Scenario 3 — Expired Certificate

**What happens if this certificate is used today?**
  >If the certificate is expired, the trust chain is broken. The browser will issue a security warning and may block access to the site. Users will see an error such as `NET::ERR_CERT_DATE_INVALID` indicating that the certificate is no longer valid.

**Analysis:**
[Explain why expiration fails validation, why lifecycle management matters, and what users would see]
  >Because certificates are only valid within their specified validity period, an expired certificate breaks the trust chain and fails validation. Lifecycle management matters so that certificates can be renewed before expiration, preventing interrupted access. Users will see a browser warning indicating the certificate is expired, often with an option to bypass the warning.
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
  >The most important thing I learned is that modern browsers rely on the SAN instead of the CN, and a single SAN certificate can securely cover multiple domains and subdomains.
