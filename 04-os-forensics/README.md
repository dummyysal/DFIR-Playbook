# 🖥️ Operating System Forensics

> OS-specific forensic analysis techniques and lab guides.

## 📁 Contents

| File | Description |
|------|-------------|
| [Windows Forensics.md](Windows%20Forensics.md) | NTFS internals, $MFT, Event Logs, persistence mechanisms |
| [Linux Forensics Lab.md](Linux%20Forensics%20Lab.md) | Linux forensic artifacts, CyberDefenders Insider challenge |
| [Memory forensics Lab.md](Memory%20forensics%20Lab.md) | Volatility memory analysis practice |
| [Network Forensics Lab - BlueSky Ransomware.md](Network%20Forensics%20Lab%20-%20BlueSky%20Ransomware.md) | Network forensics scenario |
| [cloud forensics.md](cloud%20forensics.md) | Cloud platform forensics (AWS, Azure, GCP) |

## 🔑 Key Topics

### Windows Forensics
- NTFS metadata files ($MFT, $LogFile, $UsnJrnl)
- MACB timestamps
- Alternate Data Streams (ADS)
- File Slack
- Event Log analysis (Security.evtx, System.evtx)
- Persistence detection (Run keys, Services)

### Linux Forensics
- File system artifacts
- Log locations (/var/log)
- Bash history analysis
- Process accounting

### Memory Forensics
- Volatility framework
- Process injection detection
- Network connections analysis

### Network Forensics
- PCAP analysis
- Protocol decoding

---

*See parent [README](../README.md) for full repository structure.*