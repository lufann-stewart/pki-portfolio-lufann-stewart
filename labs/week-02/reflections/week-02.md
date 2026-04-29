# Week 2 Reflection — Cryptography Fundamentals

---

## Prompts

### 1. What did you learn this week?

Explain in your own words:

- The difference between confidentiality, integrity, and authenticity
  > Confidentiality: Info stays secret, only authorized people can see it.  
  >Integrity: File hasn’t been changed or tampered with.  
  >Authenticity: File is real and comes from who it says it comes from.  
  
- How symmetric encryption works
  >Symmetric encryption works by using the same secret key to both encrypt and decrypt data, keeping the message private, but it does not check whether the file has been changed (hashing is used for that).
  
- How hashing detects tampering
  >Hashing detects tampering by generating a fixed-length hash of the file and checking whether it matches the original hash; if it doesn’t match, the file has been changed.  
  
- How digital signatures combine hashing and asymmetric cryptography
  >Digital signatures combine hashing and asymmetric cryptography by using a hash to detect changes (integrity) and a private key to sign the file (authenticity), which together help ensure confidentiality, integrity, and authenticity.  


Avoid copying definitions. Demonstrate understanding.

---

### 2. What concept was most challenging?

Consider:

- Understanding why hashing cannot be reversed
  >Hashing is one-way, so once data is turned into a hash, you can’t get the original data back. I understood it’s meant for integrity checks, not confidentiality, which helped me see why reversing isn’t possible.
- Grasping why digital signatures fail after tampering
  >Digital signatures fail when a file is changed because the hash no longer matches the signed value. Seeing this in practice helped me understand how integrity is enforced.  
  
- Differentiating encryption from signing
  >The concept I found most challenging was differentiating encryption from signing. I had to remember that encryption is mainly about confidentiality — keeping data secret from anyone who shouldn’t see it — while signing is about integrity and authenticity, proving the data hasn’t been changed and that it comes from the right person. Thinking about the goals of each process helped me understand the difference.
Explain what made it challenging and how you worked through it.

---

### 3. Where does this appear in real-world systems?

Provide specific examples such as:

- HTTPS connections
- Code signing
- Software updates
- Certificate Authorities
- Internal enterprise PKI systems

Be concrete. Avoid vague answers.

  >This appears in many real-world systems. For example, HTTPS connections use certificates to keep data private and verify a website’s identity, like when visiting your bank or making an online purchase. Code signing ensures software is authentic and hasn’t been tampered with, such as signing installers for Chrome or Zoom. Software updates, like Windows Update or iOS app updates, use digital signatures to prevent malicious changes. Certificate Authorities, such as DigiCert or Let’s Encrypt, issue and verify certificates for websites. Internal enterprise PKI systems control access, for example when employees log in to a company VPN or access internal servers.

---

### 4. How would you explain this topic to a non-technical audience?

Explain:

- What encryption does
- What hashing does
- What digital signatures prove

Keep it simple but accurate.

  >Encryption keeps information private so that only people who are supposed to see it can read it. Hashing creates a kind of “fingerprint” of a file to check whether it has been changed, helping detect if the file’s integrity has been compromised. Digital signatures prove both that the file has not been tampered with and that it really comes from the person who claims to have sent it, ensuring integrity and authenticity.
---

### 5. What questions remain?

List any uncertainties about:

- Key management
- Trust chains
- TLS
- Certificate signing
- Enterprise implementation

These questions will inform Week 3.

  >No, I think I have a somewhat solid conceptual understanding now. I’ve done similar labs in Security+ and Server+, so I’m familiar with how keys, certificates, and trust chains work. I also know that enterprises use tools like Server Manager or Active Directory Certificate Services to implement PKI, even if I’m not completely familiar with all the setup details.
---

## Professional Growth Check

- Did you explain concepts in your own words?
- Did you connect theory to practice?
- Did you reference your lab observations?
- Is your formatting clean and structured?
- Was your commit message meaningful?

---

## Forward Link

Week 2 introduced the mechanisms that enforce digital trust.

Week 3 will show how X.509 certificates combine encryption, hashing, and digital signatures into a structured trust model.

Be prepared to inspect certificates with deeper technical understanding.
