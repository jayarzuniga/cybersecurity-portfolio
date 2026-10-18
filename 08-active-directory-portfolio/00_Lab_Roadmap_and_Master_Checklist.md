# Lab Roadmap and Master Checklist

**Roadmap** - *Savora Hospitality Group - Active Directory Home Lab*

| Item | Detail |
| --- | --- |
| Goal | Understand the whole build, the order of the guides, and the settings shared by every guide. |
| Estimated time | 15 minutes to read |
| VMs used | None yet |
| Prerequisite | None |
| Snapshot at the end | Not applicable |

This lab turns the requirements document into a hands-on build. It is split into **10 short guides** so you can finish one, take a snapshot, and continue another day. Every guide has the same layout: goal, numbered steps, expected result, a portfolio screenshot to capture, a checklist and a troubleshooting table.


## Guide Index

| # | File | What you build | Time |
| --- | --- | --- | --- |
| 01 | VirtualBox and VM Preparation | VirtualBox install, LAB-LAN internal network, ISOs, sysprepped template, VMs | 2 h |
| 02 | Build DC01 and Create Forest | First domain controller, DNS, forest corp.savorahg.com | 1.5 h |
| 03 | Build OUs, Groups and Users | OU tree, groups, admin accounts, bulk users | 2 h |
| 04 | Add DC02, Sites and Replication | Second DC, sites, subnets, FSMO, failover test | 2 h |
| 05 | File Server, DHCP and Client Join | FS01, shares with AGDLP, DHCP, CLIENT1 and CLIENT2 | 2.5 h |
| 06 | Delegation and Group Policy | Brand IT admins, GPOs, drive maps, lockdown | 2.5 h |
| 07 | Security Hardening and Auditing | Password policy, PSOs, LAPS, signing, audit policy | 2.5 h |
| 08 | Kerberos Lab | Trace tickets, events, Wireshark, skew, SPNs, Protected Users, krbtgt | 3 h |
| 09 | Backup, Recovery and Forest Trust | System state backup, Recycle Bin, FSMO seize, partner.local trust | 3 h |
| 10 | Portfolio Evidence and Troubleshooting | Screenshots, GitHub layout, write-ups, fixes | 1.5 h |


## Master Settings (use these everywhere)


### Servers and clients

| VM | Hostname | IP address | DNS servers | Role |
| --- | --- | --- | --- | --- |
| DC01 | DC01 | 10.10.1.10/24 | 10.10.1.10 now; after Guide 04: 10.10.1.11 first, 10.10.1.10 second | First DC, DNS, FSMO |
| DC02 | DC02 | 10.10.1.11/24 | 10.10.1.10, then 10.10.1.11 | Additional DC, DNS, GC |
| FS01 | FS01 | 10.10.1.20/24 | 10.10.1.10, 10.10.1.11 | File server, DHCP |
| CLIENT1 | CF-MKT-PC01 | DHCP 10.10.1.101 | from DHCP | Casa Fuego Makati PC |
| CLIENT2 | SB-ORT-PC01 | DHCP 10.10.1.102 | from DHCP | Sakura Bay Ortigas PC |
| FORESTB-DC | PDC-B | 10.10.1.50/24 | 10.10.1.50 | Optional partner.local forest |

The gateway is left empty because the lab network is internal only. Subnet mask is 255.255.255.0 everywhere.


### Names and accounts

| Item | Value |
| --- | --- |
| Domain / forest | corp.savorahg.com (NetBIOS SAVORA) |
| Domain admin (built-in) | SAVORA\\Administrator - used only to build, then replaced by adm- accounts |
| Your admin account | adm-\<yourname> (created in Guide 03), member of GG-HQ-ITAdmins |
| DSRM (recovery) password | Choose one, write it in your private lab notes |
| Lab password rule | Use a lab-only passphrase of 16+ characters. Never reuse a real password. |
| Second forest (optional) | partner.local (NetBIOS PARTNER) |


### Reusable variables file

Several scripts reuse the same list of brands and branches. Save this once as `C:\Lab\vars.ps1` on every server where you run scripts, then load it at the top of a session with `. C:\Lab\vars.ps1` (dot, space, path).

**vars.ps1**

```powershell
# vars.ps1  - save as C:\Lab\vars.ps1 on every server; load with:  . C:\Lab\vars.ps1
$base = "DC=corp,DC=savorahg,DC=com"
$brands = @{
  CasaFuego      = @{ Code="CF"; Branches=@{ Makati="MKT"; BGC="BGC"; QuezonCity="QC" } }
  SakuraBay      = @{ Code="SB"; Branches=@{ Ortigas="ORT"; Alabang="ALB"; Cebu="CEB" } }
  TrattoriaBella = @{ Code="TB"; Branches=@{ Greenhills="GRH"; Eastwood="EST"; Davao="DVO" } }
  GoldenWok      = @{ Code="GW"; Branches=@{ Binondo="BIN"; Pasay="PSY"; Baguio="BAG" } }
}
```


## Snapshot Naming

After each guide, shut down cleanly and snapshot every VM that changed (VirtualBox: select the VM, open the **Snapshots** tab, click **Take**). Name it `NN-description`, for example `02-DC01-Forest`. Snapshots let you redo a risky step and give you a clean state for screenshots.

> **Warning:** Once you have two domain controllers, never revert only one of them. Revert both together, or rebuild the reverted one. A lone revert causes a USN rollback and broken replication.


## Working Rules

- Do every step on VMs only. Keep every VM on the VirtualBox **Internal Network** named `LAB-LAN`; do not use Bridged Adapter for the lab.

- Type commands rather than only pasting at first. You learn Kerberos faster by typing and reading the output.

- Take the screenshot listed in each step before moving on. Hide anything personal.

- Write one or two sentences in your notes about what each step proved. Those notes become the portfolio write-up.

![Figure A - Target lab topology](images/diagram2_lab.png)

*Figure A - Target lab topology*


## Completion Checklist

- [ ] I read all 10 guide names and understand the order
- [ ] I installed Oracle VirtualBox 7.x and confirmed virtualization is on in BIOS
- [ ] I saved vars.ps1 somewhere I can copy from
- [ ] I created a notes file for passwords and observations


## Next

Open [01_VirtualBox_and_VM_Preparation.md](01_VirtualBox_and_VM_Preparation.md).
