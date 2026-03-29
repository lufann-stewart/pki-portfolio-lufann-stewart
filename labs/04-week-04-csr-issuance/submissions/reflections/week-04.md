# Week 4 Reflection

---

## Prompts

1. What did you learn this week?    
    >This week I learned how to create and install self-signed certificates and inspect them in Windows. I explored the system trust stores and learned the difference between the Local Machine store (certlm.msc) and the Current User store (certmgr.msc), including how a certificate can exist in one but not the other. I also learned about different certificate formats, their purposes, and how to convert between them. Working with trust chains in practice helped me understand why the root certificate is critical—removing it immediately breaks trust for any certificates it signed.

2. What concept was most challenging?  
     >The most challenging part was understanding why OpenSSL doesn't automatically use the Windows trust store. Once I realized OpenSSL has its own separate certificate store, I understood why I had to use the -CAfile switch to validate certificates. That was a key insight that changed how I approached the labs
   
3. Where does this concept appear in real-world systems?  
    >Trust chains and certificates are everywhere: when visiting secure websites, using VPNs, email encryption, or connecting devices in corporate networks. Any system that relies on encrypted communication depends on root certificates and their proper installation. If a root cert is missing or untrusted, secure connections can fail.
 
4. How would you explain this topic to a non-technical audience?  
   >Certificates are like digital ID cards for websites or devices. A root certificate is like the ID issuer. If your computer trusts the issuer, it will trust any ID they issue. If the issuer is removed, all IDs signed by them become untrusted—like everyone suddenly saying your ID is invalid. Trust chains are just the chain of people who vouch for each other to prove you can trust a certificate.

5. What questions remain?
   >I attempted Lab 3 a second time for practice. The first attempt was successful, but during the second attempt, I noticed some differences in behavior. When I manually deleted the CVI-Lab-Root-CA from the Local Machine root store in certmgr.msc, I received a warning about removing it. The first time I removed it using the command line, there was no prompt or warning. Why does Windows behave differently depending on whether you remove a certificate manually versus via the command line?
   
  >Are there best practices for using OpenSSL in Windows, especially in PowerShell, to avoid issues with path recognition or multi-line commands?

---

## Professional Growth Check


- [X] I documented my work clearly and in my own words
- [X] I used structured formatting in my submission files
- [X] My commit message was meaningful and descriptive
