
# Lab 02 — Investigate Certificate Extensions

## Overview
Briefly describe what this lab was about in your own words.
  >This lab was about pulling and inspecting the X.509 extension fields and understanding how to find and interpret what each section is used for in a certificate.

What PKI concept were you investigating?
  >Certificate extensions, what they are, and how they control how a certificate is used and validated in TLS.

---

## Environment
- OS: Windows
- Terminal used (Mac Terminal / Git Bash / WSL): Command Prompt(cmd.exe)
- OpenSSL version (`openssl version`): OpenSSL 3.6.1 

---

## Extensions Found

### Subject Alternative Name (SAN)
Paste the value from your output:
  >`DNS:www.t-mobile.com, DNS:b2b.t-mobile.com, DNS:t-mobile.com`

### Key Usage
Paste the value from your output:
  >Critical: Digital Signature, Key Encipherment

### Extended Key Usage (EKU)
Paste the value from your output:
   >TLS Web Server Authentication, TLS Web Client Authentication 

### Basic Constraints
Paste the value from your output:
  > critical CA:FALSE
              
---

## Observations

1. What domains appear in the SAN field? `DNS:www.t-mobile.com, DNS:b2b.t-mobile.com, DNS:t-mobile.com`
2. What is this certificate authorized to do based on Key Usage? It used for digital signatures and key encipherment.
3. What does the EKU field tell you about this certificate's purpose? It tells us the certificate is specifically used for TLS web server and client authentication.
4. Is this a CA certificate? How can you tell? No, because it says CA:FALSE, which means it cannot sign other certificates.
5. Why does SAN matter more than the Subject CN field in modern TLS? Modern TLS uses the SAN field to verify domain names, and the CN is no longer relied on for that. The SAN field can list multiple domains and subdomains, while the CN only supports one, which makes it more limited. Because of that, SAN became the standard for hostname validation.
