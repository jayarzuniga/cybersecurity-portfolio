## 📋 Phase 1: Infrastructure Preparation
- [/] **Check host resources:** 16 GB RAM recommended (8 GB minimum), 4+ CPU cores, ~120 GB free disk.
- [/] **Choose a Hypervisor:** Install VirtualBox or VMware Workstation Player.
- [/] **Download ISOs:**
  - [/] Linux: Ubuntu Server 22.04 LTS (confirm the Wazuh version you pick supports your Ubuntu release first)
  - [/] Windows: Windows 11 Enterprise Evaluation from Microsoft (Windows 10 is past end of support)
  - [/] *(Optional)* Kali Linux VM for attacker simulation
- [/] **Size the VMs:**
  - [/] Wazuh VM: 4 vCPU, 8 GB RAM, 50 GB disk (4 GB is the bare minimum and gets sluggish)
  - [/] Windows VM: 2 vCPU, 4 GB RAM, 50 GB disk
- [/] **Network Configuration (keep it isolated):**
  - [/] Use a **NAT Network or Host-Only network** so the lab is isolated from your home LAN. Avoid Bridged mode, since you'll be running attack simulations.
  - [/] Assign **static IPs** and record them in a table (needed later for your diagram).
  - [/] Confirm VMs can ping each other.
- [ ] **Take a baseline snapshot** of each clean VM so you can roll back after testing.

## 🐧 Phase 2: Wazuh Installation & Setup (Linux VM)
- [/] **Deploy Wazuh Manager:**
  - [ ] Option A: Wazuh OVA (fastest)
  - [/] Option B: Official all-in-one installer script on Ubuntu Server (more realistic and a better portfolio story)
- [/] **Save the credentials:** the installer prints a generated `admin` password at the end (the OVA uses its own documented default). Store it in a password manager and change defaults.
- [/] **Access the Dashboard:** browse to `https://<Linux_VM_IP>` and accept the self-signed cert warning.
- [/] **Verify services are healthy:**
  - [/] `systemctl status wazuh-manager wazuh-indexer wazuh-dashboard`
- [/] **Generate Agent Deployment Script:** Dashboard → **Agents** → **Deploy new agent** → Windows → copy the PowerShell command.
- [/] **Snapshot** the Wazuh VM after a successful install.

## 🪟 Phase 3: Windows VM Configuration & Log Collection
- [/] **Install Wazuh Agent:**
  - [/] Run the generated PowerShell command in an **elevated** PowerShell.
  - [/] Start the service: `NET START Wazuh` (or `Start-Service Wazuh`).
  - [/] Verify the agent shows **Active** in the Dashboard.
- [ ] **Confirm Windows Event Logs:**
  - [ ] Open `C:\Program Files (x86)\ossec-agent\ossec.conf`.
  - [ ] Ensure `Security`, `System`, and `Application` `<localfile>` entries exist (they're on by default, so verify).
- [ ] **Enable extra auditing (needed for good detections):**
  - [ ] Enable *Audit Process Creation* and *Include command line in process creation events* via Local Security Policy / `auditpol`.
  - [ ] Enable PowerShell Script Block Logging (Event ID 4104).
- [/] **Install Sysmon:**
  - [/] Download Sysmon from Microsoft Sysinternals.
  - [/] Download a community config (SwiftOnSecurity or Olaf Hartong's sysmon-modular).
  - [/] Install (elevated prompt): `sysmon64.exe -accepteula -i sysmonconfig.xml`
  - [/] Verify: Event Viewer → *Applications and Services Logs → Microsoft → Windows → Sysmon → Operational*.
- [/] **Integrate Sysmon with Wazuh:**
  - [ ] Add this to `ossec.conf` (`eventchannel` format):
    ```xml
    <localfile>
      <location>Microsoft-Windows-Sysmon/Operational</location>
      <log_format>eventchannel</log_format>
    </localfile>
    ```
  - [ ] Restart the agent: `Restart-Service -Name wazuh`
  - [ ] Confirm Sysmon events (Event ID 1, 3, 11, etc.) appear in the Dashboard.
- [ ] **Snapshot** the Windows VM ("clean + instrumented").
- [ ] Add this to `ossec.conf` (`eventchannel` format):
  ```xml
  <localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
  </localfile>
- [ ] Restart the agent: `Restart-Service -Name wazuh`
- [ ] **Back up the current `local_rules.xml`:**
  ```bash
  sudo cp /var/ossec/etc/rules/local_rules.xml \
          /var/ossec/etc/rules/local_rules.xml.backup
  ```
- [ ] **Download the Sysmon `local_rules.xml`** ruleset.
- [ ] **Replace the default `local_rules.xml`:**
  ```bash
  sudo cp /tmp/local_rules.xml /var/ossec/etc/rules/local_rules.xml
  ```
- [ ] **Set permissions:**
  ```bash
  sudo chown root:wazuh /var/ossec/etc/rules/local_rules.xml
  sudo chmod 640 /var/ossec/etc/rules/local_rules.xml
  ```
- [ ] **Test the rules:** `sudo /var/ossec/bin/wazuh-logtest`
- [ ] **Restart the Wazuh Manager:** `sudo systemctl restart wazuh-manager`
- [ ] **Verify the Wazuh Manager is running:**
  ```bash
  sudo systemctl status wazuh-manager
  ```
- [ ] Confirm Sysmon events (Event ID 1, 3, 7, 8, 10, 11, etc.) appear in the Dashboard.
- [ ] Verify the custom Sysmon rules trigger correctly using actual Sysmon events.
- [ ] Check Wazuh Manager logs for rule/configuration errors:
  ```bash
  sudo tail -50 /var/ossec/logs/ossec.log
  ```
  - [ ] **Snapshot** the Windows VM ("clean + instrumented").


## 🔍 Phase 4: Practice & Detection Engineering
### 4A. Generate Baseline Telemetry
- [ ] Create failed login attempts on Windows (Event ID 4625).
- [ ] Run harmless recon commands (`whoami`, `ipconfig /all`, `net user`, `Get-Process`).
- [ ] Create a new local user, then add it to the Administrators group (Event IDs 4720, 4732).

### 4B. Investigate Alerts Like an Analyst
- [ ] Dashboard → **Threat Hunting / Security Events**; find the alerts from your actions.
- [ ] Open an alert and read the raw log, `rule.id`, `rule.level`, and MITRE fields.
- [ ] Practice filters (e.g., `agent.name`, `rule.mitre.id`, `data.win.system.eventID`).
- [ ] Write a short triage note for 3 alerts: *what happened, is it malicious, what's the next step*.

### 4C. Simulate Attacks (Atomic Red Team, safe tests in the isolated lab only)
- [ ] Install Atomic Red Team on the Windows VM (add a Defender exclusion for the atomics folder if needed, in the lab only).
- [ ] Run and record results for these techniques:
  - [ ] **T1136.001**: Create Local Account
  - [ ] **T1059.001**: PowerShell execution (encoded command)
  - [ ] **T1053.005**: Scheduled Task persistence
  - [ ] **T1547.001**: Registry Run Key persistence
  - [ ] **T1003**: Credential dumping attempt (observe only, in a snapshot)
- [ ] Note which techniques Wazuh detected **out of the box** and which it **missed** (gaps = detection opportunities).

### 4D. Write Custom Detection Rules
- [ ] Edit `/var/ossec/etc/rules/local_rules.xml` (custom rule IDs must be **100000+**).
- [ ] Write at least 5 rules, for example:
  - [ ] User added to Administrators group (Event 4732)
  - [ ] Encoded PowerShell command line (Sysmon Event 1)
  - [ ] New scheduled task created
  - [ ] Registry Run key modified (Sysmon Event 13)
  - [ ] Multiple failed logons followed by a success
- [ ] Add MITRE mapping to each rule using the `<mitre><id>Txxxx</id></mitre>` block.
- [ ] Validate syntax and logic with `/var/ossec/bin/wazuh-logtest` **before** restarting.
- [ ] Restart: `systemctl restart wazuh-manager`
- [ ] Re-trigger each attack and **verify the custom alert fires**. Screenshot it.
- [ ] **Tune one noisy rule** (e.g., add an exclusion for a known-good process) and document before/after alert counts.

### 4E. Stretch Goals (pick 1–2 for extra impact)
- [ ] Add a **Linux agent** (the Wazuh VM itself or a second Ubuntu VM) and detect SSH brute force (Kali + Hydra against your own lab).
- [ ] Enable **File Integrity Monitoring** on a sensitive folder.
- [ ] Configure **Active Response** to block a brute-force source IP.
- [ ] Build a custom **Dashboard** (top rules, MITRE heatmap, failed logins over time).
- [ ] Integrate **VirusTotal** or another threat-intel enrichment.

## 📝 Phase 5: Documentation & Portfolio Building
Goal: a hiring manager should understand your skills in 5 minutes without cloning anything.

- [ ] **Create a public GitHub repo** (e.g., `mini-soc-lab-wazuh`) with this structure:
  ```
  mini-soc-lab-wazuh/
  ├── README.md            # pitch, architecture, results, key takeaways
  ├── docs/
  │   ├── setup-guide.md   # step-by-step with screenshots
  │   ├── lessons-learned.md
  │   └── incident-report-01.md
  ├── detections/
  │   ├── local_rules.xml
  │   └── detection-catalog.md
  └── images/              # diagram + screenshots
  ```
- [ ] **Network Diagram:** Host, Wazuh VM, Windows VM (and Kali if used), with IPs, ports (1514/1515/443), and data flow.
- [ ] **Setup Guide:** step-by-step with screenshots of install and agent deployment. **Redact** passwords, tokens, and real IPs.
- [ ] **Detection Catalog:** a table for every rule:

  | Rule ID | Detects | MITRE Technique | Log Source | Test Used | Result | FP Notes |
  |---|---|---|---|---|---|---|
  | 100001 | User added to Administrators | T1098 | Security 4732 | `net localgroup administrators` | ✅ Fired | Whitelist IT admin account |

- [ ] **Detection Logic Write-ups:** for each rule, explain *why it matters*, what the attacker is doing, and how a SOC analyst would respond.
- [ ] **Mock Incident Report:** one end-to-end write-up (alert → investigation → timeline → verdict → containment → recommendations).
- [ ] **ATT&CK Coverage Summary:** which techniques you detected out of the box, which you added, and which remain gaps.
- [ ] **Lessons Learned:** errors you hit (agent not connecting, logs not parsing, rules not firing) and how you fixed them. Hiring managers value this highly.
- [ ] **README polish:** lead with the pitch, a diagram, 2–3 screenshots, a results table, and a "what I'd do next" section.
- [ ] **Short demo (optional):** a 2–3 minute screen recording of an attack triggering your alert.
- [ ] **Publish & promote:**
  - [ ] Add the project to your resume with metrics (see below).
  - [ ] Pin the repo on GitHub.
  - [ ] Post a short LinkedIn write-up linking the repo.

### Resume bullet templates (fill in your real numbers)
- Built an isolated SOC lab (Wazuh, Sysmon, Windows 11) ingesting endpoint telemetry from X sources.
- Authored X custom detection rules mapped to MITRE ATT&CK, validated with Atomic Red Team simulations.
- Reduced false positives on a PowerShell detection by X% through rule tuning.
- Documented a full incident investigation and published the lab with reproducible setup guides.

---

## ⚠️ Safety & Hygiene
- [ ] Keep the lab **isolated** from your home network; run attack simulations only on your own VMs.
- [ ] Never expose the Wazuh Dashboard to the internet.
- [ ] Snapshot before each risky test; roll back afterward.
- [ ] Don't commit passwords, API keys, or agent keys to GitHub.
