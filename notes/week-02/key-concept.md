# Week 02 — Key Concepts
# Lab 1:

Document the following in your Week 2 lab notes:

Why the encrypted file is unreadable
 >The encrypted file is unreadable because it has been scrambled by encryption, so it looks like random characters instead of the original content.
    
What would happen if the wrong password were used
  > If the wrong password is used, the decryption fails and OpenSSL gives an error because the key doesn’t match, so the file cannot be read.  
  
  > Error Message Received: B4450000:error:1C800064:Provider routines:ossl_cipher_unpadblock:bad decrypt:providers\implementations\ciphers\ciphercommon_block.c:107:  

What security property symmetric encryption provides
  > Symmetric encryption keeps information secret, so only people with the right password can read it.

Why TLS uses symmetric encryption for data transfer
> TLS uses symmetric encryption to send data quickly and securely because it’s fast and can handle a lot of information, but it doesn’t prove who sent it or protect against tampering — other TLS features do that  


# Lab 2:

Document the following in your Week 2 notes:

Why the hash changed after a small modification  
 >Even a tiny change in the file, like adding a word, completely changes the hash because hashing is designed so that any difference in the input produces a very different output.

Why hashing does NOT provide confidentiality
 >Hashing only creates a unique fingerprint of the data; it doesn’t hide the content. Anyone who can see the file can read it. Confidentiality requires encryption, which scrambles the data so only someone with the key can read it.

What security property hashing provides  
 >Hashing provides integrity, meaning you can detect if the file was changed. It can also support identity in digital signatures, because the hash proves the data came from someone who signed it.

Where hashing is used in PKI systems
 >In PKI, hashes are used in digital signatures. Instead of signing the whole document, the sender hashes the document and signs the hash. The receiver can hash the document themselves and verify the signature to ensure the data wasn’t tampered with and came from the correct sender.
