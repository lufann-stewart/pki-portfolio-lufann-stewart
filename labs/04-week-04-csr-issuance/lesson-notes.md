# Week 4 Notes — Certificates, Trust Chains, and PKI in Windows

## 1. Core Concept

Explain the foundational concept clearly and practically.
  >This week I learned how to create and install self-signed certificates and inspect them in Windows. I explored the system trust stores and learned the difference between the Local Machine store (certlm.msc) and the Current User store (certmgr.msc), including how a certificate can exist in one but not the other. I also learned about different certificate formats, their purposes, and how to convert between them. Working with trust chains in practice helped me understand why the root certificate is critical—removing it immediately breaks trust for any certificates it signed.
---

## 2. Why It Matters

How this concept appears in real enterprise environments.
  >Certificates and trust chains are used anywhere secure communication is required: websites, VPNs, email encryption, and corporate networks. Enterprises typically manage root CAs centrally using Group Policy, ensuring that all machines trust the same roots without needing manual installs.

  >Trust chains prevent impersonation and man-in-the-middle attacks. If a root certificate is missing or compromised, any certificates it signed become untrusted, breaking secure connections.

  >Understanding the difference between Current User and Local Machine stores is important: Local Machine certificates apply to all users on a machine, while Current User certificates apply only to that user's session.
---

## 3. Technical Breakdown

- Definition
    >A self-signed certificate is a certificate that signs itself, meaning the Issuer and Subject are identical. Trust is established only if the certificate is explicitly added to the system’s trust store.
    
- Components
  >Root CA (self-signed, trusted manually or via GPO)
Signed certificate (leaf)
Client system (verifies the certificate chain)
  

- Flow
  >1. Client requests a secure connection to a server.
  >2. Server responds with its certificate and any intermediate certificates.
  >3. Client checks the certificate chain against its trusted root store.
  >4. If the chain reaches a trusted root, the connection continues securely.
  >5. If the chain is broken or a certificate is invalid, the client displays a warning or blocks the connection.
  
- Trust implications
   >Trust depends entirely on the root CA being in the system’s trusted store. Removing the root breaks trust. Command line removal may not prompt warnings, but manual removal through certmgr.msc may show warnings.
---

## 4. Common Misconceptions

- Misconception 1
  >Some people think certificates are automatically trusted by the OS if they exist anywhere. In reality, a certificate must be in the correct store (Local Machine or Current User) to be trusted.
  
- Misconception 2
  >People sometimes assume that OpenSSL automatically uses the Windows trust store. OpenSSL uses its own store unless you specify the CA with -CAfile.
---

## 5. Where This Shows Up

- Web security
  >HTTPS websites, browser warnings for untrusted certificates.
  
- Enterprise systems
  >Internal login, portals, VPNs, device authentication; internal CA gives control.
  
- Personal systems / labs
  >Testing self-signed certificates to learn about trust chains and certificate management.
  

---

## Mental Model

Short summary that ties the week back to:

Identity + Trust + Verification
  >Certificates establish identity and trust. The root certificate is the anchor. If the root is removed, trust breaks immediately. The chain of trust is what validates that a certificate came from a trusted source. Local Machine vs Current User stores control scope of trust on the system.
