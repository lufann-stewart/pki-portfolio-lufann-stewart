# Week 02 — Key Concepts

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
