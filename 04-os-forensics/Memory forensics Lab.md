Important notes are in the course. For the Lab I dived into the Windows memory dumps practical example.
### RAM vs Disk Analysis

| Aspect                | RAM Analysis                                            | Disk Analysis                                          |
| --------------------- | ------------------------------------------------------- | ------------------------------------------------------ |
| **Volatility**        | Extremely volatile – lost at shutdown                   | Persistent – survives reboot                           |
| **Content**           | Running processes, network connections, encryption keys | Files, deleted data, file system metadata              |
| **Anti-Forensics**    | Harder to manipulate in real-time                       | Easier to tamper (timestomping, wiping)                |
| **Malware Detection** | Can find fileless malware, injected code                | Can find files on disk, but misses memory-only threats |
| **Speed**             | Acquisition: minutes                                    | Acquisition: hours (large drives)                      |
| **Encryption**        | Keys may be present in memory                           | Encrypted data on disk requires key                    |


https://cyberdefenders.org/blueteam-ctf-challenges/reveal/
Official write up is available too ( CHECK IT ) 
So basically you are going to reconstruct a multi-stage attack by analyzing memory dumps of windows system using Volatility 3 to identify malicious processes, command lines, and correlate findings with threat intelligence.

**Cheat sheets can help you with Volatility 3:**

- [Ashley Pearson cheatsheet](https://blog.onfvp.com/post/volatility-cheatsheet/) 
- [HackTricks cheatsheet](https://book.hacktricks.xyz/generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet)
- [Find evil on windows host ](https://sansorg.egnyte.com/dl/WFdH1hHnQI)
## Full Attack Chain 
Always after finishing a lab I recommend reconnect all the dots because you may get lost in the chain, especially if you are beginner like me.
```
Phishing / Initial Access
   ↓
PowerShell launched under user "Elon" (PID 3692)
Parent PID: 4120  ← the process that triggered it      ↓
windowstyle hidden  ← completely invisible to user
	Launches PowerShell with a hidden window, concealing its execution from    the user.
   ↓
net.exe (PID 2416) connects to attacker WebDAV server
\\45.9.74.32@8888\davwwwroot\
	( Maps a remote shared directory located on `45.9.74.32` (an external server) to the local system, allows the attacker to access files on the remote server as if they were local, facilitating the transfer or loading of malicious payloads)
   ↓
rundll32 loads 3435.dll directly from remote share
	(Executes the rundll32.exe utility, which is commonly used to load and run DLL files, it loads `3435.dll` from the remote server's shared directory)
   ↓
DLL "entry" function executes → StrelaStealer activates
   ↓
Credential & email data theft (C2: 45.9.74.32)
```




## Important Volatility Commands used in this lab

```bash
# Step 1 - Confirm OS
vol3 -f 192-Reveal.dmp windows.info

# Step 2 - Find suspicious processes
vol3 -f 192-Reveal.dmp windows.psscan

# Step 3 - See full command lines
vol3 -f 192-Reveal.dmp windows.cmdline

# Step 4 - Check user sessions
vol3 -f 192-Reveal.dmp windows.sessions

# Step 5 - Detect injected code
vol3 -f 192-Reveal.dmp windows.malware.malfind
```

