# Week 6 Reflection

---

## Prompts

1. What did you learn this week?    
   > This week I learned how to diagnose different types of TLS/PKI failures using a structured approach. I practiced identifying issues like expired certificates, missing intermediate certificates, and hostname/SAN mismatches. I also learned how to use OpenSSL tools to retrieve, parse, and validate certificates, and how different failures can look similar but have completely different root causes.

2. What concept was most challenging?  
   > The most challenging part was distinguishing between chain validation errors and actual root causes. For example, I initially confused OpenSSL verification errors with the real issue, especially when my local environment trust store caused additional failures. Understanding that the same error message can come from different underlying problems took some effort.

3. Where does this concept appear in real-world systems?  
   > These concepts appear in any system that uses HTTPS or TLS, such as healthcare portals, banking apps, and internal enterprise systems. Misconfigured certificate chains, expired certificates, or missing trust roots can cause outages or security warnings, which directly impact users and business operations.


4. How would you explain this topic to a non-technical audience?  
   > I would explain it like a chain of trust. When you visit a secure website, your computer checks a “chain” of IDs to make sure the site is legitimate. If any part of that chain is missing, expired, or doesn’t match the website name, your computer blocks the connection to protect you.

5. What questions remain?  
   > I still want to better understand how different operating systems manage their trust stores and why the same certificate can validate on one system but fail on another. I also want more practice identifying failures faster without second-guessing the root cause.

---

## Professional Growth Check

- [X] I documented my work clearly and in my own words  
- [X] I used structured formatting in my submission files  
- [X] My commit message was meaningful and descriptive  
