# AS-REP ROASTING

AS-REP roasting works when pre-authentication is disabled for an account. When a user with pre-authentication disabled tries to authenticate with the KDC, it sends the username in plain text (AS-REQ) to the KDC. The KDC responds with AS-REP. This response contains a piece of data encrypted with a key derived from the target user's password. The attacker captures this response and tries to crack it offline with tools like hashcat.

## Sysadmin aspect:
- I disabled pre-authentication for the m.jay user. ADUC -> right clicked on the user (m.jay) -> properties -> Account -> Do not require Kerberos pre-authentication (checked).

## What I did:
1. **ran:** `ping <dc_ip>` to test connectivity with the domain controller. 
2. **ran:** `touch users.txt` and `open users.txt` to add the known usernames to a text file (including m.jay). 
3. **ran:** `impacket-GetNPUsers -h` to check for the command syntax.
4. **ran:** `impacket-GetNPUsers -dc-ip <dc_ip> lab.local/ -usersfile users.txt -request` . This returned the password hash (obtained from the AS-REP) for the m.jay user and also returned "User doesn't have UF_DONT_REQUIRE_PREAUTH set" for users that had pre-authentication enabled.

<img width="1199" height="320" alt="obtainedhash" src="https://github.com/user-attachments/assets/a59a6eac-0d0e-4861-bb9f-1c6ec50e1660" />


5. I saved the obtained hash to `hash.hash`. 
   The hash began with `$krb5asrep$23$`, the `23` showed it is a RC4 hash.
6. **ran:** `echo "m.jay password" >> rockyou.txt` which appended the actual m.jay user password to my copy of rockyou.txt file. This is because the m.jay user password does not exist in rockyou.txt. Since this is a lab environment with a custom password, I added it manually to simulate a successful crack.
7. **ran:** `hashcat -m 18200 hash.hash rockyou.txt` . `-m 18200` tells hashcat to use the mode 18200 which is for AS-REP RC4 cracking. 
   Initially, hashcat was unable to crack the hash due to a typo I made while saving the hash to a file, but then it succeeded when I re-obtained the hash and saved it properly.

<img width="1129" height="310" alt="crackedhash" src="https://github.com/user-attachments/assets/fa06eab1-2911-442b-9cac-0dfcf25c7dbf" />



## Why it worked:
1. Pre-authentication was disabled on the user account which made the KDC respond with AS-REP that contained a piece of data encrypted with the user's password hash. Normally, the AS-REP is a challenge for this user to decrypt with its password hash and get verified/authenticated.

## Lessons Learnt:
1. RC4 encryption was not allowed for the m.jay user, but it was allowed in the default domain policy. `impacket-GetNPUsers` still obtained an RC4 hash  for the m.jay user, which showed a "downgrade attack" by the impacket tool.
2. I configured the default domain policy to allow only AES 128, AES 256 and `impacket-GetNPUsers` was able to obtain the AES 256 password hash `$krb5asrep$18$`. (The issue with the Kerberoasting and `impacket-GetUserSPNs` still persists).
