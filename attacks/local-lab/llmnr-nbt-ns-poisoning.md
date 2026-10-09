# LLMNR, NBT-NS POISONING

LLMNR and NBT-NS are older protocols for name resolution.\
LLMNR (Local Link Multicast Name Resolution): port 5355\
NBT-NS (NetBIOS Name Service): port 137

LLMNR/NBT-NS Poisoning works when DNS fails to resolve a name which makes Windows fall back to LLMNR/NBT-NS. An attacker on the same network can set up a rogue server, listen for broadcasts with a tool like `responder`, pretend to be the requested service and capture the NTLMv2 password hash when the victim tries to authenticate against the rogue server.

## To disable/enable LLMNR/NBT-NS

### LLMNR: 
- Start the gpmc.msc (Group Policy Management).
- Edit the Default Domain Policy and navigate to computer configuration -> Policies -> Administrative Templates -> Network -> DNS Client -> "Turn off multicast name resolution". Enabling this policy disables LLMNR.
### NBT-NS:
- Open Network Connections (ncpa.cpl) in the control panel -> right click on the network interface (Ethernet for me) -> click on Properties -> select "Internet Protocol Version 4" -> click on Properties -> Advanced -> WINS -> NetBIOS settings (Default/Enable/Disable). 

## What I did:

1. **ran:** `ip a` to check if my Kali is on the DC network and to know the interface (eth0).
2. **ran:** `sudo responder -I eth0 -v`. The `-I` is to specify the interface to be used and `-v` is for verbosity. 
3. On my Client machine (CLIENT01), logged in as user **m.jay**. I started `PowerShell` and **ran:** `net use \\serverme\help` to connect to **serverme** and use the **help** share. **serverme** does not exist, so DNS is unable to resolve it, causing the system to fall back to LLMNR/NBT-NS.
4. `responder` captured the NTLMv2 password hash for **m.jay**.

<img width="1319" height="861" alt="nthashobtained" src="https://github.com/user-attachments/assets/526cbbc0-ec14-4319-8342-5ba97f29c1f3" />


5. I saved the obtained hash to **nthash.hash**.
6. **ran:** `hashcat -m 5600 nthash.hash rockyou.txt`. `-m 5600` tells `hashcat` to use the mode 5600 which is for NTLMv2 cracking. The hash was cracked in seconds and the m.jay user password was returned.  

<img width="1450" height="909" alt="crackednt" src="https://github.com/user-attachments/assets/cfa1331e-fc9b-45ab-9c31-b86c52e873bf" />




## Why it worked:

- LLMNR/NBT-NS was enabled in the domain, so when DNS failed to resolve the hostname, Windows fell back to one of them. 

## Lessons Learnt:

- **When LLMNR/NBT-NS is disabled in the domain:** if DNS fails to resolve a hostname, the system drops the connection and throws a standard network error.




