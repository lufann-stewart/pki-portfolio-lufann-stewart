
# Lab 1 — Symmetric Encryption
 
## Overview
Briefly describe the purpose of this lab in your own words.
  >The purpose of this lab was to create, exncryot, decrypt and inspect the decrypted file to see if there were and changes made to the file.

What PKI concept or system behavior were you investigating?
  >Symmetric Encryption.
 
---
 
## Environment
Document the environment used to complete the lab.
 
- Operating System: Windows
- Terminal Used: Command Prompt (cmd.exe)
- OpenSSL Version (if applicable): OpenSSL 3.6.1 
 
---
 
## Steps Performed
Summarize the key steps you performed to complete the lab.
 
Do **not copy the lab instructions**.
Describe what you actually did.  

*Using commands I:
 
1. Created the labs directory structure for submissions.

2. Created a plaintext file named plaintext.txt containing the text: "Week 2 Symmetric Encryption Lab - CVI".

3. Encrypted the plaintext.txt file with a password, resulting in plaintext.txt.enc.

4. Decrypted plaintext.txt.enc using the same password, producing plaintext.decrypted.txt.

5. Verified that the decrypted file matched the original plaintext file using the diff command, confirming that no changes occurred during encryption and decryption.
 
---
 
## Results
Include the important outputs or findings from the lab.
 
Examples may include:
 
- command outputs
- certificate fields
- verification results
- screenshots (if applicable)
 
If you include screenshots, store them in the **assets folder** and reference them here.
 
Example:
 
![Certificate Output](assets/certificate-output.png)
 
---
 
## Key Findings
Document the most important observations from the lab.
 
Examples:
 
- Certificate issuer
- Public key algorithm used
- Certificate extensions present
- Trust chain relationships
- Validation results
 
•
•
•
 
---
 
## Explanation
Explain **why the results matter**.
 
Examples:
 
- Why the issuer is important in PKI
- Why SAN is required for modern TLS validation
- Why the certificate chain validates successfully
- Why a misconfiguration would cause a failure
 
---
 
## Challenges / Troubleshooting
Document any issues encountered during the lab and how you resolved them.
 
Examples:
 
- command errors
- missing intermediate certificates
- verification failures
 
---
 
## Artifacts
List the files generated during this lab.
 
Examples:
 
- leaf_cert.pem
- server.pem
- intermediate.pem
- root.pem
- screenshots stored in assets/
 
---
 
CVI PKI Career Pathway — Foundations Phase
