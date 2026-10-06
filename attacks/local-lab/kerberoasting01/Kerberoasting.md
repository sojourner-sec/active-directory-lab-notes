# Kerberoasting

- created a service account on my Domain Controller with a weak password. See [sysadmin notes](./syadmin-notes.md) for account creation and SPN registration steps.
- I was provided with a valid username (m.jay) and password to perform kerberoasting.
## What I did 1: 
1. ran `ping <dc_ip>` : to test connectivity with the Domain Controller.
2. ran `impacket-GetUserSPNs lab.local/username:'password' -dc-ip <dc_ip> -request` : the service account name was gotten but an error was returned (Kerberos SessionError: KDC_ERR_ETYPE_NOSUPP(KDC has no support for encryption type))
3. ran `sudo ntpdate <dc_ip>` to sync my attacking machine clock to the DC's clock. The issue still persists. 
4. On my DC
   Navigated to Local Security Policy (secpol.msc) - Local Policies - Security options  to check for "Network security: Configure encryption types allowed for Kerberos". It is not defined, which was not the issue, because windows would fall back to the default encryption which includes AES and RC4.
5. Ran a lot of troubleshooting for hours: 
   On the Domain Controller
   - configured the default domain policy (Network security: Configure encryption types allowed for Kerberos) to allow RC4 encryption. 
   - set the allowed encryption for the svc_sql account: In ADUC, right clicked on svc_sql -> properties -> Account. Ensured AES 128, AES 256 and DES was unticked. This would allow the kdc to fall back to using RC4 for the svc_sql account. 
   - reset the svc_sql account password: so that the corresponding hash can be generated using the allowed encryption. 
   - ran: `gpupdate /force` to force an immediate policy refresh. 
6. ran `impacket-GetUserSPNs lab.local/username:'password' -dc-ip 10.10.10 -request` but the same error was returned: (Kerberos SessionError: KDC_ERR_ETYPE_NOSUPP(KDC has no support for encryption type))

## Problems and Fixes:
### Problem: 
1. Decided to find out the encryption type supported by the KDC for some user accounts: 
   - ran `Get-ADObject -Filter 'sAMAccountName -eq "ServiceName"' -Properties msDS-SupportedEncryptionTypes`
   - Finding: the svc_sql and m.jay "msDS-SupportedEncryptionTypes" were set to 0.
    This meant there was no supported encryption for these accounts.  
### Fix:
1. ran `Set-ADUser -Identity "ServiceName" -KerberosEncryptionType <encryptiontype eg RC4,AES128,AES256>` which allowed me to set the encryption type for the svc_sql to RC4. I also set the encryption type of m.jay to AES128 and AES256. 
2. Navigated to the default domain policy from the Group Policy Management tool. 
   I edited the "Network security: Configure encryption types allowed for Kerberos" to allow AES 128, AES 256 and RC4. 
   ran `gpupdate /force` to force an immediate policy refresh.
3. Opened ADUC (dsa.msc) to reset passwords for svc_sql and m.jay accounts. 

## What I did 2:
This was after the above fix.
1. ran `sudo ntpdate <dc_ip>` to sync my attacking machine time to the DC.
2. ran `impacket-GetUserSPNs lab.local/username:'password' -dc-ip 10.10.10 -request` which successfully returned the hash for the svc_sql service account. 
3. The hash began with `$krb5tgs$23$`. The "23" shows it is a RC4 hash. 
4. I saved the hash using a GUI text editor because `echo` command removed the `$krb5tgs$23$` from the hash, which would render the hash uncrackable by `hashcat`. 
5. ran `hashcat -m 13100 svc.hash /usr/share/wordlists/rockyou.txt` to crack the hash and obtained the plaintext password: CHICKEN123!
   

## Lesson learnt:
1. Disabling AES 128 and AES 256 on an account does not necessarily mean the KDC would fall back to RC4. 
2. For the "msDS-SupportedEncryptionTypes" result: 4 - RC4, 8 - AES 128, and 16 - AES 256. Therefore a result of 24 means that only AES 128 and AES 256 are the supported encryption type for that account. 
3. A specific encryption type must be enabled on an account, and the encryption type must also be allowed by the Group Policy (Default Domain policy in this case) before it can take effect.
4. `impacket-GetUserSPNs` can not fetch the svc_sql's password hash when RC4 was disabled, and AES 128, AES 256 were enabled.
   The command only worked when svc_sql encryption was set to RC4 (and the Group policy also allowed RC4).
   Note: I should look into this sooner. 

