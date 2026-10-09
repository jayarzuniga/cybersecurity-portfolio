# Cybersecurity Portfolio

This is my collection of hands-on labs, write-ups, and projects from my ongoing cybersecurity learning journey, it is covering offensive, defensive, and analysis work across multiple disciplines.

![Status](https://img.shields.io/badge/status-in%20progress-yellow)
![Last Updated](https://img.shields.io/badge/last%20updated-2026--10-blue)

---

## About

This repository documents my practical learning in cybersecurity through structured labs, test executions, and investigation write-ups. Each folder represents a different domain. The goal is to build and demonstrate real, reproducible skills rather than just theory.

Before this repo, my earlier work was spread across a few standalone repositories:
- [`enterprise-soc-homelab`](https://github.com/jayarzuniga/enterprise-soc-homelab)
- [`junior-soc-analyst-guide`](https://github.com/jayarzuniga/junior-soc-analyst-guide)

A scattered labs and one-off exercises where I was still building fundamentals.They're part of how I got here, but they weren't built with a consistent structure.
 
**This repository is where that foundation gets made concrete.** Going forward, every lab, test execution, and investigation follows with an improving structure.

> **Disclaimer:** All testing shown here was performed in isolated lab environments (personal VMs / sandboxes) that I own and control. Nothing in this repository was run against systems I don't have explicit authorization to test.

---

## Table of Contents

- [Repository Structure](#repository-structure)
- [Project Folders](#project-folders)
- [Tools & Technologies](#tools--technologies)
- [Skills Demonstrated](#skills-demonstrated)
- [How Each Project Is Organized](#how-each-project-is-organized)
- [Roadmap](#roadmap)
- [Connect](#connect)

---

## Repository Structure

```text
cybersecurity-portfolio/
├── 01_wazuh-soc-detection-lab/
├── 02_phishing-analysis/
├── 03_packet-analysis/
├── 04_malware-analysis/
├── 05_detection-engineering/
├── 06_ctf-writeups/
└── 07_notes-and-cheatsheets/
```

*(Folder names are placeholders — rename/add as your projects grow.)*

---

## Project Folders

| Folder | Description | Status |
|---|---|---|
| [`red-teaming/`](./red-teaming) | Adversary emulation using Atomic Red Team — technique execution, MITRE ATT&CK mapping, and execution logs. | 🟡 In progress |
| [`blue-teaming/`](./blue-teaming) | SOC-side detection and triage work in Wazuh — alert investigation, Sysmon correlation, rule tuning. | 🟡 In progress |
| [`phishing-analysis/`](./phishing-analysis) | Email header analysis, IOC extraction, and phishing campaign breakdowns. | ⚪ Planned |
| [`packet-analysis/`](./packet-analysis) | Wireshark/tcpdump captures and traffic analysis for common attack patterns. | ⚪ Planned |
| [`malware-analysis/`](./malware-analysis) | Static/dynamic analysis of sample malware in an isolated sandbox. | ⚪ Planned |
| [`detection-engineering/`](./detection-engineering) | Custom Sigma/Wazuh rules, MITRE ATT&CK coverage mapping, and detection validation. | 🟡 In progress |
| [`ctf-writeups/`](./ctf-writeups) | Solutions and methodology notes from CTF challenges. | ⚪ Planned |
| [`notes-and-cheatsheets/`](./notes-and-cheatsheets) | Quick-reference notes on tools, commands, and concepts. | ⚪ Planned |

**Status legend:** 🟢 Complete · 🟡 In progress · ⚪ Planned

---

## Tools & Technologies

- **SIEM / Detection:** Wazuh, Sysmon
- **Adversary Emulation:** Atomic Red Team, MITRE ATT&CK Navigator
- **Network Analysis:** Wireshark, tcpdump
- **Environments:** VirtualBox (NAT Network + Host-only lab topology)
- **Scripting:** PowerShell, Bash, Python


---

## Skills Demonstrated

- Mapping adversary behavior to MITRE ATT&CK tactics, techniques, and sub-techniques
- Writing and tuning custom detection rules (Wazuh/Sysmon-based)
- Alert triage workflows — filtering, grouping, and correlating high-volume SOC data
- Red Team / Blue Team (Purple Team) validation loops: execute → detect → close gaps
- Documenting findings in clear, reproducible investigation reports

---

## How Each Project Is Organized

Every project folder follows the same structure so write-ups stay consistent and easy to navigate:

```text
project-name/
├── README.md                    # Overview, objective, and summary of findings
├── Checklist.md                 # Steps and requirements checklist
├── execution-log.md             # Commands run / steps taken
├── screenshots/                 # Supporting evidence
└── <process_done_name>.md       # Observations, gaps, and lessons learned
```

---

## Roadmap

- [ ] Finish Wazuh detection coverage for T1059.001 (PowerShell) and T1021.006 (WinRM)
- [ ] Add first phishing analysis write-up
- [ ] Add first packet analysis case study
- [ ] Create a red team process documentation
---

## Connect

- **LinkedIn:** https://www.linkedin.com/in/jonhzuniga
- **Email:** jonhronelzuniga@gmail.com

---

*This repository is a living document — updated as labs are completed and new domains are explored.*