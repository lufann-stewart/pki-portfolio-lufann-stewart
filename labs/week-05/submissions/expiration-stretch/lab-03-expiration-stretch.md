# Lab 03 — Simulate a Certificate Expiration Scenario (Stretch)

## Overview
Briefly describe what this lab was about in your own words. What PKI concept or operational scenario were you investigating? 
  >This lab was about working with expired certificates, replacing them, and understanding the workflow. I analyzed the output of expired certificates to understand validity periods and expiration behavior.

## Environment
- Operating System: Windows
- Terminal Used: cmd.exe
- OpenSSL Version (openssl version): OpenSSL 3.6.1

## Steps Performed
Summarize the key steps you performed. Do not copy the lab instructions — describe what you actually did.
1. Created a short-lived certificate and checked its validity period using OpenSSL.
2. Used `-checkend` to test whether the certificate would expire within certain timeframes.
3. Created an already expired certificate and verified its dates.
4. Ran verification on the expired certificate and observed the failure errors.
5. Generated a replacement key, CSR, and issued a new certificate.

## Results
- What output did `openssl x509 -checkend` produce for the short-lived certificate?
  >When I ran it with 86400 seconds, it returned “Certificate will expire.” When I ran it with 3600 seconds, it returned “Certificate will not expire.” This shows how the expiration window changes based on the time being checked.
  
- What error appeared when you ran `openssl verify` on the expired certificate?
  > The verification failed because the certificate was expired. It showed:
  error 18: self-signed certificate  
  error 10: certificate has expired  
  This confirms the certificate is no longer trusted.
  
- What is the exit code behavior of `-checkend 0` vs `-checkend 86400`, and how would you use this in a script?
  > `-checkend 0` checks if the certificate is already expired right now, while `-checkend 86400` checks if it will expire within the next 24 hours. The command returns an exit code where 0 means the certificate will NOT expire within that time, and 1 means it WILL expire. This can be used in scripts to trigger alerts or automate certificate renewal.
  
- Walk through the replacement workflow steps you performed: new key → new CSR → new cert.
  >I generated a new private key, then created a new CSR using that key, and finally issued a new certificate from that CSR.


## Key Findings
- Expired certificates immediately fail validation and break trust.
- `-checkend` is useful for proactively monitoring expiration in scripts.
- Certificate replacement involves generating a completely new key and certificate, not just extending validity.
- The exit code behavior of `openssl x509 -checkend` is critical for automation: a return value of 0 indicates the certificate is still valid within the defined time window, while 1 indicates the certificate will expire within that window. This allows monitoring systems and scripts to proactively detect impending certificate expiration and trigger alerts or renewal workflows.
- In production environments, expired certificates cause immediate TLS validation failures. Systems will reject connections with errors such as “certificate has expired (error 10),” resulting in service outages. This demonstrates why expiration monitoring is critical in enterprise environments where certificate-based authentication is required for secure communication.
  
## Explanation
- What is the difference between certificate renewal and certificate replacement?
  > Renewal extends the expiration of an existing certificate, while replacement generates a new key and a completely new certificate.

- When would you choose replacement over renewal, even if the certificate hasn't expired?
  > I would choose replacement if something changes like the organization, domain, or security concerns (like a compromised key), or if it's close to expiration and easier to reissue cleanly.

- Why does certificate expiration still cause enterprise outages despite being predictable?
  > It usually comes down to poor tracking, lack of automation, or unclear ownership of certificate management.
 

## Challenges / Troubleshooting
  >The OpenSSL command using `-startdate` and `-enddate` failed with an "Extra option" error in OpenSSL 3.6.1 on Windows. I switched to using `-days 0` to create an already expired certificate, which achieved the same objective.


>The `-checkend` command only returned text output in the terminal ("Certificate will expire" / "Certificate will not expire") and did not directly display an exit code. I ran the command in cmd.exe using `& echo %errorlevel%` to surface the exit code returned by OpenSSL.

## Artifacts
- test_cert_short.pem, test_cert_expired.pem, test_cert_replacement.pem
