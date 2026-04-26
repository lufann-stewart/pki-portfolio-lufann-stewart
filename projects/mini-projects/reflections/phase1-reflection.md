# Phase 1 Reflection Project

## 1. Your Phase 1 Journey

At the beginning of Phase 1, my understanding of PKI was basic and mostly conceptual. I knew certificates were used for HTTPS and that they helped secure communication, but I did not fully understand how the pieces worked together. In Week 1 and Week 2, I focused on foundational concepts like encryption, hashing, and digital signatures. At that stage, I struggled to connect how a private key, public key, and certificate all worked together to establish trust.

As the weeks progressed, my understanding became more practical. By Week 3 and Week 4, I was able to inspect certificates, understand certificate chains, and work with trust stores. I began to see how trust is not just theoretical, but enforced through root certificates and system-level validation.

By Week 5 and Week 6, my focus shifted to troubleshooting. I learned how to identify real-world failures such as expired certificates, missing intermediate certificates, and hostname mismatches. This was a turning point because I stopped just reading certificates and started diagnosing problems.

By Week 7, I was able to analyze a real enterprise TLS deployment and understand how large organizations manage certificates across multiple environments. At this point, I began thinking more like a PKI engineer rather than just a student learning concepts.

---

## 2. How the Pieces Connect

PKI works as a complete system that combines cryptographic foundations, certificate structure, trust distribution, lifecycle management, and troubleshooting.

It starts with cryptography. Encryption protects confidentiality, hashing ensures integrity, and digital signatures provide authenticity. These concepts come together in certificates, which bind a public key to an identity.

Certificates are issued by Certificate Authorities and form a chain of trust. A leaf certificate represents the server, an intermediate CA issues it, and a root CA is trusted by the system. This trust is enforced through the system’s trust store.

The lifecycle of a certificate includes generation (key and CSR), issuance, deployment, validation, renewal, and revocation. If any part of this lifecycle fails, it can lead to outages or security risks.

Troubleshooting ties everything together. In Week 6, I diagnosed issues such as missing intermediates, expired certificates, and SAN mismatches. These failures all relate back to how trust is established and validated.

In real-world systems, these components are used together in enterprise environments where certificates are deployed across load balancers, CDNs, and internal systems, often with automated renewal and monitoring.

---

## 3. A Lab That Made It Real
One lab that made PKI real for me was:

    `labs/week-06/diagnose-broken-chain`

In this lab, I diagnosed a TLS failure caused by a missing intermediate certificate. Even though the certificate itself was valid, the server failed to send the full chain, which prevented clients from building trust.

This lab showed me that TLS failures are not always caused by invalid certificates, but often by misconfigurations. It reinforced the importance of understanding how the full chain works rather than focusing only on the leaf certificate.

---

## 4. Explaining PKI to Someone Non-Technical

PKI can be explained as a system for proving identity on the internet.

When you visit a secure website, your computer needs to verify that the site is legitimate. It does this by checking a digital certificate, which acts like an ID card. That ID is issued by a trusted authority, similar to how a government issues identification.

Your computer trusts certain authorities in advance. If the website’s certificate traces back to one of those trusted authorities, the connection is allowed. If something is missing, expired, or doesn’t match, the connection is blocked.

---

## 5. Where You Go From Here

After completing Phase 1, I want to deepen my understanding of enterprise PKI systems. This includes learning how organizations automate certificate management, distribute trust across large environments, and monitor for failures.

I am also interested in how PKI is used in cybersecurity roles, particularly in areas like incident response and infrastructure security. Understanding how to diagnose TLS issues and validate trust is a practical skill that applies directly to real-world environments.

---

## Additional Lab Reference

Another lab that reinforced my understanding was:

`labs/week-07/analyze-enterprise-cert`

This lab helped me understand how large organizations use multiple certificate authorities and infrastructure layers such as load balancers and CDNs to manage TLS at scale.

---

## Required Visual

The PKI certificate lifecycle diagram is included as a separate file in the repository:

`reflections/visual/phase1-diagram.png`