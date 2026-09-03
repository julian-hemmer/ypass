# Security

## Server

The server shouldn't know anything about user, it should only have his information encrypted but nothing in plain text like in the database


## Client

Local vault fetch from the server should still be encrypted  
The vault is unlock and decrypted only when the right pin is given  
The client should be protected enough so the pin cannot be bruteforce  
Pin entering should be secured and if the client is open for too long without any interaction for some moment should lock back itself.  
When a password is copied into the clipboard it should be removed after sometime.  
No feature should allow a user to comprise is data, bash history shouldn't have any data. (No shortcut to add password).

## Database

Never store unencrypted blob, a password should never be in plain text inside the database  

## Network trafic

Every connection should be encrypted  
If the trafic in intercepted an hacker shouldn't be able to see any confidential information  

## Crypto

Argon2id for password to key ?  
AES-256 for encryption.  
Safe random number generator.  
TLS for network.  
