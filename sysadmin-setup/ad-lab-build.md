# Building My First Active Directory Lab

I wanted to understand Active Directory, so I'm setting up a lab. This is in preparation for Active Directory Enumerations and Attacks. 

## Goal: 

- Server 2022 (AD-DC01) + Windows 11 Client(CLIENT01). 
- Promote the server to a Domain Controller.
- Add the client to the domain. 
- Create OUs(IT, Sales and Workstations) - Groups(IT-Admins, Sales-Team) - Users.
- Create GPOs and check if they apply.

## Problems and Fixes:
### 1. AD DS install failed - 0x800f0819 error.

**What happened:**
Installing the Active Directory Domain Service failed with this error.

**Why:**
I interrupted a stuck windows update earlier, which left the component servicing store in a bad state.

**Fix:** ran: "DISM /Online /Cleanup-Image /RestoreHealth" in cmd(admin) and restarted.

### 2. Server Manager failed to start

**What happened:** Server Manager could not load server list.

**Why:** I renamed the server and changed the IP address.

**Fix:** Click "Ok/reset" to reset the server list.

### 3. Client VM failed to boot

**What happened:** No bootable option or device found.

**Why:** VirtualBox's EFI firmware had trouble booting the ISO.

**Fix:** Disabled UEFI and TPM 2.0. Disabling it let Windows use legacy BIOS instead. 

### 4. PC doesn't meet windows requirement issue

**What happened:** windows installation stopped while displaying this. 

**Why:** My VM doesn't meet the system requirements.

**Fix:** 1. Shift + F10 to open cmd.
	2. regedit (to launch the registry editor)
	3.  navigate to: HKEY_LOCAL_MACHINE\SYSTEM\Setup. Right-click 'Setup' → New → Key → name it LabConfig.
	4. Click into the new 'LabConfig' key, then right-click in the empty space on the right → New → DWORD (32-bit) Value, and create each of these, setting each value to 1: BypassTPMCheck, BypassSecureBootCheck, BypassRAMCheck, BypassStorageCheck, BypassCPUCheck. You don't have to need all five though. 
	5. Close regedit and cmd, then click back and try again. 

### 5. Interactive logon banner GPO didn't display

**What happened:** I created a GPO with a logon banner message and linked
it to the IT OU, but the banner never showed when logging into CLIENT01.

**Why:** The banner setting lives under Computer Configuration, which
applies based on where the *computer* object sits in AD, not the user.
My computer object (CLIENT01) was in the Workstations OU, not IT, so the
GPO never reached it. I correctly separated users and computers into
different OUs earlier, but I did not link this particular GPO to both.

**Fix:** Created a separate GPO specifically for the Workstations OU
and linked it there, rather than reusing the IT-linked one. After that,
the banner still didn't appear until I also confirmed both
"Interactive logon: Message title" and "Interactive logon: Message text"
were set; on some Windows versions, only one of the two isn't enough to
trigger the banner. Restarted CLIENT01 (a `gpupdate /force` refresh
wasn't sufficient for this particular policy type) and the banner
appeared correctly before login.

## Reflection

Setting up the lab was easier than I expected, and I enjoyed the "sysadmin" side of it: creating users, groups, and watching GPOs actually take effect. One thing that clicked technically: Computer Configuration GPOs apply based on where the *computer* sits in AD, not the user, which explains why my logon banner didn't work until I fixed the OU it was linked to. The bigger realization was about scope. I caught myself starting to chase "industry-standard" perfection: hardening the lab, trying to make it feel like a real organization, before I actually needed any of that for what I'm doing next. AD is huge and I realized I only need to understand what my current path (enumeration and attacks) actually requires, not the whole field. I also used to think attacking would mostly be blind guessing, but I now understand it's built on understanding the system first; though I don't need full sysadmin-level mastery of AD to get there, just enough to know what I'm looking at.

## What's next

Optionally exploring basic enumeration against my own lab using tools like BloodHound, then starting HTB's Active Directory Enumeration and Attacks course later.

