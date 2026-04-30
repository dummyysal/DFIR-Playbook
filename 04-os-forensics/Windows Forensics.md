
	Important Course Notes since Windows forensics is an essential area in Digital Forensics, as Windows systems are widely used everywhere, enterprise and personal environments..THAT'S why  Understanding Windows artifacts is critical for investigations.
Ps: For the practical lab, click here for the [[Memory forensics Lab]] which can be used also for hands-on practice.

## 1. Core Windows Forensic Priorities
When an incident occurred and you are working on forensic copy of the compromised  system, those following questions guide you to determine the scope chain better
1. Who used the system?
2. What was executed?
3. What files were accessed / modified / deleted?
4. When did events happen?
5. Was persistence established?
6. Was data exfiltrated?
7. Were external devices used?
8. Was evidence tampered with?


## 2. NTFS Internals

**1-Key NTFS Metadata Files**

| File       | Purpose                               |
| ---------- | ------------------------------------- |
| `$MFT`     | Master File Table – every file/folder |
| `$LogFile` | File system journal                   |
| `$UsnJrnl` | Change journal                        |
| `$Bitmap`  | Cluster allocation map                |
| `$Boot`    | Boot sector                           |
| `$Secure`  | Security descriptors                  |
 **2-High-Value Evidence**
- Alternate Data Streams (ADS) : it contains hidden data attached to files. 
It's important to check :
```powershell
Get-Item file.txt -Stream *
dir /r
Get-Content file.txt -Stream hidden
```

- File Slack : NTFS allocates space in clusters (typically 4KB). If a 1KB file is written, the remaining 3KB in that cluster is "slack space" 
	Now Because this space is not directly managed by the OS for new data, it often contains hidden or deleted data from previous file operations

- MACB timestamps: 
	Modified, Accessed, Changed, and Birth, why important ? they can be used as an artifacts to reconstruct timeline events  such file creation, user activity or even malware execution



---

## 3. Master File Table ($MFT)

All you have to know here about MFT that it is the heart of NFTS.

|Attribute|Meaning|
|---|---|
|`$STANDARD_INFORMATION`|Main timestamps|
|`$FILE_NAME`|Name + parent + timestamps|
|`$DATA`|Content|
|`$INDEX_ROOT`|Directory entries|
Those attributes are way important is the following forensic significance :
- Detect timestomping ($SI vs $FN) 
    Compare `$STANDARD_INFORMATION` vs `$FILE_NAME` timestamps. Attackers often modify $SI but forget $FN.
- Recover deleted metadata
- Resident small files inside MFT
```bash
# Parse MFT info with analyzeMFT and turn it into human-readable format
analyzeMFT.py -f \$MFT -o mft_output.csv

# Using MFTECmd (Eric Zimmerman)
MFTECmd.exe -f "\$MFT" --csv output_dir
```


## 4. Event Logs

**Location:** `C:\Windows\System32\winevt\Logs\` stored in EVTX format

**Valuable logs** that are worth to check during an investigation

- Security.evtx
     Logon/logoff (4624/4634), failed logons (4625), privilege use (4672)
- System.evtx
     Service installs (7045), system time changes (4616), shutdown/startup
- Application.evtx
     Application errors, installations
- PowerShell Operational
     PowerShell script execution (4104), module logging
- Sysmon Operational
     Process creation, network connections, file creation time changes

```bash
# parse logs to readable csv
EvtxECmd.exe -d Logs --csv out
```

## 5. Persistence Checks
Persistence = techniques used by attackers to **maintain access after reboot or logout** ( interesting IK )
- Run keys
    those are  Registry entries that automatically execute programs at user login
	 on the system location can be found :
	    HKCU\Software\Microsoft\Windows\CurrentVersion\Run
- Services
     Windows background processes that can auto-start at boot ( ransomware )
- Startup folder
     Programs that automatically executed when user logs in 
     C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
- Browser extensions (people  be blindly installing browser like they are trusted ALWAYS check the vendor  )



Resources: 

- SANS Windows Forensic Analysis Poster
- Eric Zimmerman's Tools: [https://ericzimmerman.github.io](https://ericzimmerman.github.io)
- NTFS Documentation: [https://flatcap.github.io/linux-ntfs/](https://flatcap.github.io/linux-ntfs/)





