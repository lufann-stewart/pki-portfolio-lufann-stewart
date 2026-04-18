# Lab 01 — Analyze a Live Enterprise Certificate Deployment

## Overview

Briefly describe what this lab was about in your own words. What were you analyzing and why?
  >This lab focused on analyzing a real-world website’s SSL/TLS certificate setup. The goal was to understand how a large enterprise (ServiceNow) secures its public web traffic, including who issues their certificates, how long they are valid for, and how the overall security configuration is structured.

## Environment

- Operating System: Windows
- Terminal Used: Cmd.exe
- OpenSSL Version (openssl version): OpenSSL 3.6.1 
- Target Hostname Chosen: www.servicenow.com

## Target

**Hostname analyzed:**
  >www.servicenow.com

**Why I chose this target:**
  >I chose ServiceNow because it is a major enterprise SaaS platform used by many organizations. I wanted to understand how large companies manage SSL/TLS certificates across different services like production, staging, and documentation environments.
## Certificate Summary

- Issuer (CA name and type — public CA / internal CA):
    >Amazon RSA 2048 M04 (Amazon Trust Services) — public Certificate Authority
- Validity window (Not Before → Not After):
    >Not Before: May 21, 2025 → Not After: June 19, 2026
- Approximate remaining validity (days):
    >62 days  
- Certificate type (DV / OV / EV — and how you determined this):
    >Likely OV, not DV. It’s not 100% confirmed from the certificate alone, but given this is a large enterprise platform using a commercial CA, OV is the more reasonable assumption. DV is typically used for simpler or automated setups, which doesn’t really match here.
- Number of SAN entries:
    >2
- Wildcard entries present? If yes, list them and describe what they suggest about the architecture:
    >No, they issue specific certificates for specific domains instead of using one blanket wildcard cert.

## Chain Analysis

- Number of certificates in the chain:
    >3
- Intermediate CA subject:
    >Amazon RSA 2048 M04 (Amazon Trust Services intermediate CA)
- Root CA name:
    >Starfield Services Root CA - G2
- Is the chain complete (leaf → intermediate → root)?
    >Yes (Leaf → Intermediate → Root all present and valid).
- Any missing or unexpected certificates in the chain?
    >No issues found.
  
## Termination Analysis

- Where does TLS appear to terminate — application server, load balancer, or CDN edge?
   >Likely at the load balancer or CDN edge.
- Evidence supporting this conclusion (server headers, issuer identity, SAN pattern, or other indicators):
   >Based on the certificate structure and use of a public Amazon Trust Services CA chain, TLS likely terminates at a front-door infrastructure layer such as a CDN or load balancer rather than directly at the application server. The certificate is scoped only to public-facing domains (`servicenow.com` and `www.servicenow.com`
), which is consistent with edge termination in a distributed enterprise SaaS architecture.


## TLS Configuration

- SSL Labs overall grade:
    >A+
- TLS versions supported:
    >TLS 1.2 and TLS 1.3
- Deprecated TLS (1.0 or 1.1) still supported? (yes/no):
    >No
- HSTS configured? (yes/no):
    >Yes
- OCSP stapling enabled? (yes/no):
    >Yes

## CT Log Analysis

- Approximate number of certificates issued for this domain:
    >~100+ certificates (likely more when including all subdomains and environments).
- Is the issuer consistent across recent certificates, or have multiple CAs been used?
    >The issuer is not fully consistent across all historical certificates. Recent certificates show multiple trusted CAs, primarily Entrust, DigiCert, and Amazon. This variation is expected for a large enterprise that uses multiple infrastructure providers and environments (production, staging, and documentation).
- Any unexpected or unfamiliar issuers? If yes, possible explanation:
    >No suspicious issuers found. The mix of CAs is expected for a large enterprise.
- Certificate validity period pattern (90-day Let's Encrypt / 1–2 year paid CA):
    >Most certificates last around 6 months to 1 year, which is typical for enterprise-grade public TLS certificates.



## Architecture Assessment
 
In 2–3 sentences, describe what this certificate deployment tells you about the organization's PKI architecture and operational approach. This is not a grade — it is an observation. Write it the way a PKI engineer would write it in a pre-migration audit.
  >This appears to be a large-scale enterprise public TLS deployment using multiple commercial Certificate Authorities. The use of different issuers (Amazon, Entrust, DigiCert) suggests a distributed infrastructure spanning cloud services and third-party providers.

  >TLS likely terminates at edge infrastructure such as load balancers or CDN layers, which is typical for modern SaaS environments.

## Key Findings
> -Strong modern TLS configuration (A+ rating).    
  -No weak or deprecated TLS versions.    
  -Multiple trusted public CAs in use.   
  -Short-to-medium certificate lifespans (good security practice).    
  -Likely edge-based TLS termination architecture.    

## Challenges / Troubleshooting
  >The hardest part was interpreting the CT log data because multiple certificates and issuers appeared across different environments (production, staging, documentation). It was initially unclear if this was inconsistent, but it actually reflects a multi-infrastructure setup.

## Artifacts

- enterprise_cert.pem
