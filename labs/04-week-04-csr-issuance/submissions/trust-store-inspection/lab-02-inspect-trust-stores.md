# Lab 02 — Inspect Your Trust Store

## Overview
Briefly describe what this lab was about in your own words. What PKI concept or system behavior were you investigating?
  >This lab explored the trusted root certificate store for my user session. I checked the listed certificates, their details, and compared familiar vs. unfamiliar root CAs to see their purposes and how expired certificates still appear.

## Environment
- Operating System: Windows
- Terminal Used: cmd, PowerShell
- OpenSSL Version (openssl version): OpenSSL 3.6.1 

## Steps Performed
### To view and retrieve certification details:
1. Went to certmgr on my local machine > Trusted Root Certification Authorities > Certificates
2. Viewed the details of the certificates I chose to examine (DigiCert Global Root G3 and Microsoft RSA Root Certificate Authority 2017) using the Details, General, and Certification Path tabs

### To verify root certificates for Google:
4. Ran `openssl s_client -connect google.com:443 -verify_return_error` and received verify error:num=20:unable to get local issuer certificate
5. Downloaded the root and intermediate certificate bundle for GlobalSign Root CA, saved it as bundle.pem on my machine
6. Ran `openssl s_client -connect google.com:443 -CAfile bundle.pem -verify_return_error` and received Verify return code: 0 (ok)

## Results
- How many trusted root CAs did you find on your system?
  >67
- Name at least one specific root CA you inspected. Include its Subject and expiration date.
  
 > **DigiCert Global Root G3**  
> **Subject:** DigiCert Global Root G3  
> **Issuer:** DigiCert Global Root G3  
> **Valid From / Valid To:** ‎Thursday, ‎August ‎1, ‎2013 8:00:00 AM until Thursday, ‎Friday, ‎January ‎15, ‎2038 8:00:00 AM
> **Public Key Algorithm:** ECC (384 bits)

> **Microsoft RSA Root Certificate Authority 2017**  
> **Subject:** Microsoft RSA Root Certificate Authority 2017  
> **Issuer:** Microsoft RSA Root Certificate Authority 2017  
> **Valid From / Valid To:** Wednesday, December 18, 2019 6:51:22 PM until Friday, July 18, 2042 7:00:23 PM  
> **Public Key Algorithm:** RSA (4096 bits)
  
- What did the verify return code output tell you?
 >The return code showed all the Common Names (CNs) listed in the machine's certificate store.

  - List of inspected root CAs (first few shown; click to expand full list):

Issuer: CN=Microsoft Root Certificate Authority, DC=microsoft, DC=com  
Subject: CN=Microsoft Root Certificate Authority, DC=microsoft, DC=com  
Issuer: CN=Thawte Timestamping CA, OU=Thawte Certification, O=Thawte, L=Durbanville, S=Western Cape, C=ZA  
Subject: CN=Thawte Timestamping CA, OU=Thawte Certification, O=Thawte, L=Durbanville, S=Western Cape, C=ZA  

<details>
<summary>Click to expand full list</summary>

Issuer: CN=Microsoft Root Authority, OU=Microsoft Corporation, OU=Copyright (c) 1997 Microsoft Corp.  
Subject: CN=Microsoft Root Authority, OU=Microsoft Corporation, OU=Copyright (c) 1997 Microsoft Corp.  
Issuer: CN=SJ Pulse Root CA, OU=Survey Junkie, O=DISQO, Inc., L=California, S=California, C=US  
Subject: CN=SJ Pulse Root CA, OU=Survey Junkie, O=DISQO, Inc., L=California, S=California, C=US  
Issuer: CN=Symantec Enterprise Mobile Root for Microsoft, O=Symantec Corporation, C=US  
Subject: CN=Symantec Enterprise Mobile Root for Microsoft, O=Symantec Corporation, C=US  
Issuer: CN=Microsoft Root Certificate Authority 2011, O=Microsoft Corporation, L=Redmond, S=Washington, C=US  
Subject: CN=Microsoft Root Certificate Authority 2011, O=Microsoft Corporation, L=Redmond, S=Washington, C=US  
Issuer: CN=Microsoft Authenticode(tm) Root Authority, O=MSFT, C=US  
Subject: CN=Microsoft Authenticode(tm) Root Authority, O=MSFT, C=US  
Issuer: CN=Microsoft Root Certificate Authority 2010, O=Microsoft Corporation, L=Redmond, S=Washington, C=US  
Subject: CN=Microsoft Root Certificate Authority 2010, O=Microsoft Corporation, L=Redmond, S=Washington, C=US  
Issuer: CN=Microsoft ECC TS Root Certificate Authority 2018, O=Microsoft Corporation, L=Redmond, S=Washington, C=US  
Subject: CN=Microsoft ECC TS Root Certificate Authority 2018, O=Microsoft Corporation, L=Redmond, S=Washington, C=US  
Issuer: CN=Microsoft ECC Product Root Certificate Authority 2018, O=Microsoft Corporation, L=Redmond, S=Washington, C=US  
Subject: CN=Microsoft ECC Product Root Certificate Authority 2018, O=Microsoft Corporation, L=Redmond, S=Washington, C=US  
Issuer: CN=Microsoft Time Stamp Root Certificate Authority 2014, O=Microsoft Corporation, L=Redmond, S=Washington, C=US  
Subject: CN=Microsoft Time Stamp Root Certificate Authority 2014, O=Microsoft Corporation, L=Redmond, S=Washington, C=US

</details>

## Key Findings

* Some certificates are expired but still show up in the store. For example, the certificate status for Microsoft Root Authority 2017 says "This certificate has expired or is not yet valid."

* I looked at DigiCert Global Root G3 and Microsoft Root Authority 2017 to compare.  

  **DigiCert Global Root G3 purposes:**  
  - Proves your identity to a remote computer  
  - Ensures software came from the publisher  
  - Protects software from being altered after publication  
  - Protects email messages  
  - Ensures identity of a remote computer  
  - Allows data to be signed with the current time  

  **Microsoft Root Authority 2017 purposes:**  
  - Proves your identity to a remote computer  
  - Ensures the identity of a remote computer  
  - All issuance policies  

* The DigiCert certificate has more intended purposes, like verifying software and email, while Microsoft Root Authority 2017 is simpler. This shows that different root CAs have different roles.  

* I picked DigiCert because I’d examined its details before, then compared it to Microsoft Root Authority 2017, which I hadn’t examined. This helped me identify the differences between a large general CA and a Microsoft-specific one. My research confirmed that DigiCert is a larger, more general CA with more purposes, while Microsoft Root Authority 2017 is specifically for Microsoft's ecosystem and mainly trusted on Windows systems. 

## Explanation
- Why does your browser trust a certificate from a website you have never visited before?
    >Because the root CA that issued the certificate is already installed in my machine's trust store. My browser validates the certificate chain up to that root CA, and if it's valid, it trusts the certificate — even for sites I've never visited.
    
- What would happen if an enterprise's internal root CA was NOT in the trust store?
  >The certificate chain would break and the connection wouldn't work. Users would get errors because the internal CA wouldn't be in the trust store. Even if the certificate is technically valid, it won't be trusted because the root CA isn't recognized by the system.
  
- What surprised you about how many roots are pre-installed on your system?
  >I was surprised by how many root CAs come pre-installed. I didn't expect a single machine to have that many trusted roots already built in.

## Challenges / Troubleshooting
I tried getting the Root CA 2 ways:

  >While running `openssl s_client -connect google.com:443 -verify_return_error`, I got an error: verify error:num=20:unable to get local issuer certificate. This meant OpenSSL couldn't find the root CA (GlobalSign Root CA) in my trust store.

I first tried exporting the GlobalSign Root CA from my local machine trust store (certlm) and importing it into Trusted Root Certification Authorities in certmgr, but running the command again still gave me code 20.

I ran a command openssl x509 -inform DER -in GlobalSignRoot.cer -out GlobalSignRoot.pem
openssl s_client -connect google.com:443 -CAfile GlobalSignRoot.pem -verify_return_error and received 

Then I went to the GlobalSign support page, downloaded the root and intermediate certificate bundle, saved it as bundle.pem, and reran the command with the CAfile switch:
`openssl s_client -connect google.com:443 -CAfile bundle.pem -verify_return_error`

This time it worked — returned code 0, which meant successful verification.

I remembered from feedback on a previous lab that when a certificate was missing, using the -CAfile switch to point to a file with the certificate PEM info fixed it. I applied that same logic here when I hit this issue, and it worked.

While attempting to screenshot the certification information from certmgr, I ran into trouble with Print Screen not working. I learned that Windows has a security control that prevents Print Screen from capturing certain sensitive areas like Certificate Manager. I had to use Windows + Shift + S instead to override the security restriction and take the screenshots.

## Artifacts
- root_cas.pem (macOS) or equivalent, screenshots stored in assets/screenshots/week-04/
