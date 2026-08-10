# Lab Writeups & Solutions

A personal collection of documented walkthroughs and solutions for Hack The Box, TryHackMe, and other cybersecurity training platforms. Each writeup covers the full process — reconnaissance, exploitation, privilege escalation, and remediation — with the reasoning behind every step.

## Disclaimer

All content in this repository is for educational purposes only and was produced against isolated, intentionally-vulnerable lab environments that I am authorized to test.

- Nothing here should be used against systems you do not own or lack explicit written permission to test. Unauthorized access to computer systems is illegal.
- Writeups for Hack The Box machines are published only after the machine has retired, in line with HTB's content policy. Active-machine writeups, if any, are kept private or redacted.
- Flags, credentials, and IP addresses shown are specific to a lab instance and rotate on reset. They are included only to illustrate methodology.

## About

This repository is where I keep my notes and full walkthroughs as I work through labs. Each writeup documents the complete path from the first scan to root, along with the tools used and how the underlying vulnerabilities could be fixed. The goal is to reinforce my own learning by writing things down properly, and to help others who get stuck on the same challenges understand the reasoning, not just the commands.

## Repository Structure

Each machine or room lives in its own folder with a self-contained writeup, so everything renders correctly on its own. A typical layout looks like this:

```
.
├── HackTheBox/
│   └── <Machine-Name>/
│       ├── README.md
│       └── images/
├── TryHackMe/
│   └── <Room-Name>/
│       ├── README.md
│       └── images/
├── Other-Labs/
│   └── <Lab-Name>/
│       ├── README.md
│       └── images/
└── README.md
```

The `Other-Labs` folder is for anything outside the main platforms, such as PortSwigger, Proving Grounds, VulnHub, or custom labs.

## What Each Writeup Contains

- Overview — target information, exposed services, and a short summary of the attack path.
- Reconnaissance — scanning and enumeration, with commands and output.
- Foothold — how initial access was obtained, whether through an exploit, misconfiguration, or known CVE.
- Privilege Escalation — the path from a low-privilege user to root or administrator.
- Proof — the user and root flags.
- Findings and Remediation — the root-cause issues and how to address them.

## Common Tools and Topics

Recurring tools and subjects across these writeups include nmap, gobuster and ffuf, Burp Suite, curl, netcat, hydra, hashcat, and the PEAS enumeration scripts, alongside topics such as web exploitation, Active Directory, privilege escalation, reverse shells, and CVE research.

## Contributing and Feedback

These are personal notes, so the approach won't always be the intended or most elegant path — that is part of the learning process. If you spot a mistake or know a cleaner method, feel free to open an issue or a pull request. Corrections and alternative solutions are welcome.

## Contact

Replace the placeholders below with your own details:

- GitHub: your-username
- LinkedIn: your-profile

If any of these writeups helped you, consider starring the repository.
