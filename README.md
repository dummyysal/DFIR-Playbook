# Digital Forensics & Incident Response (DFIR) Playbook

> Course Notes & Digital Forensics Practical Labs - [Redteamleaders](https://courses.redteamleaders.com/courses)

[![Contributions welcome](https://img.shields.io/badge/Contributions-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Last updated](https://img.shields.io/badge/Last%20updated-April%202026-blue.svg)](#)
[![Focus](https://img.shields.io/badge/Focus-DFIR%20%26%20Threat%20Hunting-red.svg)](#)
---
## Overview
The course is available on the Red Team Leaders Learning Academy. I highly recommend it as it offers a free certificate and provides valuable information for beginners, intermediate learners, and those seeking a structured reference for DFIR concepts. 
This guide is complemented by technical labs mostly from CyberDefenders for hands-on practice.

NOTE: By working through this repository, I am building practical skills in:

- Digital Forensics investigation methodology
- Incident Response lifecycle execution
- Windows / Linux / Memory / Network analysis
- Malware behavior and persistence detection
- Evidence handling & forensic reporting
- Cloud and container forensic concepts

## DFIR Certificate Content
It covers 4 modules:
1. Foundations of Digital Forensics
2. Operating System Forensics
3. Incident & Malware Investigation
4. Professional Reporting & Presentation
5. Module Bonus

## 📚 Repo Contents

```
DFIR-notes/
├── 01-forensic-process.md       # The Forensic Process phases
├── 02-essential-tools.md       # Essential Tools
├── 03-malware-forensics.md     # Malware Forensics
├── 03-credential-dumping.md    # Credential Dumping Tools
├── 04-os-forensics/            # OS Forensics
│   ├── Windows Forensics.md
│   ├── Linux Forensics Lab.md
│   ├── Memory forensics Lab.md
│   ├── Network Forensics Lab.md
│   └── cloud forensics.md
└── labs/                      
```

| Section | Description |
|---------|-------------|
| [01-forensic-process.md](01-forensic-process.md) | The forensic process phases: Identification → Preservation → Acquisition → Analysis → Documentation |
| [02-essential-tools.md](02-essential-tools.md) | Core forensic tools: Autopsy, FTK Imager, Volatility, Wireshark, KAPE, Plaso |
| [03-malware-forensics.md](03-malware-forensics.md) | PE headers, static/dynamic analysis, strings, sandboxing |
| [03-credential-dumping.md](03-credential-dumping.md) | Credential dumping tools: Mimikatz, LaZagne - attacker techniques & forensic detection |
| [04-os-forensics/](04-os-forensics/) | OS-specific forensics: Windows, Linux, Memory, Network, Cloud |


## 🔧 Tools Covered

| Tool | Purpose | Installation |
|------|---------|--------------|
| **Autopsy** | Disk analysis, file recovery | [Download](https://www.autopsy.com/download/) |
| **FTK Imager** | Forensic disk imaging | [Download](https://accessdata.com/product-download) |
| **Volatility** | Memory forensics framework | `pip install volatility3` |
| **Wireshark** | Network traffic analysis | `sudo apt install wireshark` |
| **KAPE** | Rapid triage & artifact collection | [GitHub](https://github.com/ericzimmerman/KapeFiles) |
| **Plaso** | Timeline generation | [GitHub](https://github.com/log2timeline/plaso) |



## Practical Labs & Case Studies

- BlueSky Ransomware - Network Forensics lab 
- Insider Threat Investigation - Endpoint Forensics lab
- Reveal - windows memory dumps lab
- PE Header Analysis – Static malware inspection

> Labs are inspired by real DFIR workflows and CTF-style investigations (CyberDefenders, TryHackMe)
> CyberDefenders offers free retired challenges! Check their [challenge archive](https://cyberdefenders.org/blueteam-ctf-challenges/) for practice.

## Purpose of This Repository

This project is part of my continuous development in DFIR field , aiming to:

- Strengthen forensic investigation skills
- Bridge the gap between theory and SOC/DFIR practice
This repository is continuously evolving as new labs, investigations, and forensic techniques are added.

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on how to contribute to those notes.

---

*Last updated: April 2026*
