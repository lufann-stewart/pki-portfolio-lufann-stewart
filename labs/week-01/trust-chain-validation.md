# Week 01 Mini Lab — Trust Chain Validation

## Screenshot Evidence

Capture a screenshot of the Certification Path (certificate chain) from your browser.

Save it as:

assets/screenshots/week-01/trust-chain-validation.png

Embed the screenshot below:

![Trust Chain Validation](../../assets/screenshots/week-01/trust-chain-validation.png)


## Website Information

**Website inspected:**  
(https://www.google.com/)

---

## Certificate Chain Breakdown

**Leaf (Server) Certificate**  
    *.google.com

**Intermediate Certificate Authority**  
    GTS WR2

**Root Certificate Authority (Trust Anchor)**  
    GTS Root R1

---

## Trust Anchor Verification

Is the Root CA marked as trusted by your system?

Yes.

If yes, explain where that trust comes from (OS/browser root store). 

GTS Root R1 is trusted by my system because it is listed in my browser's root certificate store.

If no, explain what warning or behavior occurred. 

N/A

---

## Observations

Document three observations about the certificate.

### Observation 1
The certificate structure is hierarchical with three levels: the root CA (GTS Root R1) at the top, the intermediate CA (GTS WR2) in the middle, and the server certificate (*.google.com) at the bottom. 
I checked the certificate in both Chrome and Firefox. In Firefox, the certificates are displayed as separate tabs rather than a visual tree diagram, but they still form the same chain of trust, with each certificate issued by the one above it.

### Observation 2
The root CA is self-signed, meaning it issues itself. It's the foundation of trust and pre-installed in the browser's root certificate store. Unlike the other certificates, it doesn't need to be verified by another authority becasue it's the top-level authority.

### Observation 3
The browser determines trust by checking if the root CA is in its pre-installed root certificate store. If it finds the root CA there, it automatically trusts the entire chain below it. If the root CA isn't trusted, the browser shows a security warning.

---

## Reflection

In 3–5 sentences, explain:

The root certificate is called the trust anchor because it's the foundation of trust. 
Validation starts at the server and if it checks out, checks the intermediate all the way to the root certificate. If the root CA is trusted, then the entire chain is trusted.
If the Root CA wasn't trusted the webpage would be inaccessible because the chain of trust would be broken and a security warning would be issued.



