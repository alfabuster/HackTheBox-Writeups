<p align="center">
  <img src="https://img.shields.io/badge/Platform-Hack_The_Box-9FEF00?style=for-the-badge&logo=hackthebox&logoColor=black" alt="Platform">
  <img src="https://img.shields.io/badge/Focus-Offensive_Security-critical?style=for-the-badge&logo=kalilinux&logoColor=white" alt="Focus">
  <img src="https://img.shields.io/badge/OS-Linux_|_Windows-informational?style=for-the-badge&logo=linux&logoColor=white" alt="OS">
</p>

# 🟢 Hack The Box — Writeups

Writeups for retired machines from the [Hack The Box](https://www.hackthebox.com) platform. Each writeup documents the full attack chain — from initial reconnaissance to root flag.

> **Note:** Writeups are published only for **retired machines** in accordance with HTB rules.

---

## 📂 Repository Structure

```
htb-writeups/
│
├── <machine_name>/
│   └── writeup.md
│
├── <machine_name>/
│   └── writeup.md
│
└── README.md
```

---

## 📋 Machines

| Machine | Difficulty | OS | Key Topics | Writeup |
|---------|------------|-----|------------|---------|
| *coming soon* | — | — | — | — |

<!-- Example row:
| Lame | 🟢 Easy | Linux | FTP misconfiguration, Samba CVE | [writeup](./lame/writeup.md) |
-->

---

## ✍️ Writeup Format

Every writeup follows a consistent methodology:

- **Summary** — one-liner with the attack chain overview
- **Reconnaissance** — port scanning, service detection
- **Enumeration** — directory fuzzing, version fingerprinting, vulnerability research
- **Exploitation** — initial foothold, CVE / custom exploit
- **Privilege Escalation** — local enumeration, escalation path
- **Flags** — user.txt & root.txt
- **Lessons Learned** — key takeaways and techniques worth remembering

---

## 🛠 Toolbox

| Phase | Tools |
|---|---|
| **Scanning** | Nmap, Rustscan |
| **Web** | Burp Suite, ffuf, Gobuster, curl |
| **Exploitation** | SQLMap, Metasploit, custom scripts |
| **Post-Exploitation** | LinPEAS, WinPEAS, GTFOBins, pspy |
| **Cracking** | John the Ripper, Hashcat |
| **General** | Python, CyberChef |

---

## ⚠️ Disclaimer

These writeups are published strictly for **educational and documentation purposes**.  
Only retired machines are covered. Unauthorized access to computer systems is illegal.  
Always obtain proper authorization before testing.

---

<p align="center">
  <a href="https://github.com/alfabuster"><img src="https://img.shields.io/badge/GitHub-@alfabuster-181717?style=flat-square&logo=github" alt="GitHub"></a>
  <a href="https://app.hackthebox.com/users/3071989"><img src="https://img.shields.io/badge/Hack_The_Box-alfabuster-9FEF00?style=flat-square&logo=hackthebox&logoColor=black" alt="HTB"></a>
  <a href="https://alfabuster.com"><img src="https://img.shields.io/badge/Blog-alfabuster.com-00FF41?style=flat-square&logo=ghost&logoColor=white" alt="Blog"></a>
</p>
