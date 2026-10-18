# VirtualBox and VM Preparation

**Guide 01 of 10** - *Install Oracle VirtualBox, create the LAB-LAN network and prepare the virtual machines*

| Item | Detail |
| --- | --- |
| Goal | VirtualBox installed and tuned, an isolated internal network, a sysprepped Windows Server template, and the four lab VMs ready for installation. |
| Estimated time | 2 hours (mostly downloads and installs) |
| VMs used | TEMPLATE-WS2022, DC01, FS01, CLIENT1, CLIENT2 (DC02 and extras are cloned later) |
| Prerequisite | Windows, Linux or macOS (Intel) host with 16 GB RAM (32 GB recommended), 250 GB free SSD space, virtualization enabled in BIOS/UEFI |
| Snapshot at the end | 01-Clean-Install on every VM |

> **Concept:** In VirtualBox an **Internal Network** is a private virtual switch that only VMs using the same network name can reach. It is invisible to your host and your home network, and it has **no DHCP server** unless you add one. That makes it ideal for an AD lab.


## Build Steps


### Step 1: Check that virtualization is on

On a Windows host open Task Manager > Performance > CPU and read **Virtualization**. It must say Enabled. If not, enable Intel VT-x or AMD-V (SVM) in your BIOS/UEFI.


### Step 2: Install Oracle VirtualBox

- Download the current **VirtualBox 7.x platform package** for your host from virtualbox.org and run the installer with the default options. Accept the network driver prompt (your network drops for a few seconds).

- The Extension Pack is **not required** for this lab. Guest Additions (installed later inside each VM) give you the screen resize and clipboard.

- Reboot the host if the installer asks.


### Step 3: Avoid the Hyper-V conflict (Windows hosts)

VirtualBox can run next to Windows' own hypervisor, but then every VM runs slowly (a green turtle icon appears in the VM status bar). For a smooth lab, switch the Windows hypervisor off:

- Windows Features: untick **Hyper-V**, **Virtual Machine Platform** and **Windows Hypervisor Platform**.

- Windows Security > Device security > Core isolation > set **Memory integrity** to Off.

- Open an elevated Command Prompt and run the command below, then reboot.

**To undo later: bcdedit /set hypervisorlaunchtype auto**

```powershell
bcdedit /set hypervisorlaunchtype off
```

> **Tip:** WSL2 and Docker Desktop also use the Windows hypervisor. Turning it off pauses them until you set the value back to `auto`.


### Step 4: Choose where the VMs are stored

In VirtualBox open **File > Tools > Preferences > General** (or File > Preferences) and set **Default Machine Folder** to a folder on your fastest drive with at least 250 GB free, for example `D:\VMs`.


### Step 5: Download the installation media

- From the Microsoft Evaluation Center download **Windows Server 2022 (ISO, 64-bit)**. The evaluation lasts 180 days.

- Download **Windows 10 Enterprise** (easiest) or **Windows 11 Enterprise** evaluation ISO (90 days). Home edition cannot join a domain.

- Save the ISOs in one folder, for example `D:\ISOs`.


### Step 6: Create the template VM

Click **Machine > New** (or the New button) and fill in the wizard:

| Wizard field | Value |
| --- | --- |
| Name | TEMPLATE-WS2022 |
| Folder | Your default machine folder |
| ISO Image | The Windows Server 2022 ISO |
| Type / Version | Microsoft Windows / **Windows 2022 (64-bit)** |
| Skip Unattended Installation | **Tick this box** so you control the install and the Administrator password |
| Base Memory / Processors | 4096 MB / 2 CPUs |
| Hard Disk | Create a Virtual Hard Disk Now, **60 GB**, leave Pre-Allocate Full Size **unticked** (dynamic VDI) |

Before starting the VM open **Settings** and change:

| Settings page | Change |
| --- | --- |
| System > Motherboard | Boot order: Optical first, then Hard Disk. Untick Floppy |
| Display > Screen | Video Memory **128 MB**; Graphics Controller **VBoxSVGA**; 3D acceleration off |
| Network > Adapter 1 | Attached to: **Internal Network**; Name: `LAB-LAN` (type it exactly, it is case-sensitive; later VMs pick it from the dropdown) |
| Network > Adapter 2 | Leave **disabled** for now (see Step 12 for temporary internet) |
| General > Advanced | Shared Clipboard: **Bidirectional** so you can paste scripts. Drag and Drop: Disabled |

> **Warning:** The `LAB-LAN` name must be identical on every VM or they cannot see each other. Do not choose NAT Network or Bridged Adapter for Adapter 1; you want the lab isolated.


### Step 7: Install Windows Server on the template

- Start the VM, choose language and click Install now.

- Select **Windows Server 2022 Standard Evaluation (Desktop Experience)**. Do not choose the Core option for the first DC.

- Choose Custom install, pick the empty disk and wait for the reboots.

- Set a lab Administrator password and sign in.


### Step 8: Install VirtualBox Guest Additions

- In the VM window use **Devices > Insert Guest Additions CD image**.

- Open File Explorer > the CD drive and run `VBoxWindowsAdditions.exe`; keep the defaults and reboot when asked.

- Set the time zone (Settings > Time and language).


### Step 9: Update the template, then generalize it

- Power off the VM. In Settings > Network enable **Adapter 2** and attach it to **NAT**. Start the VM and run Windows Update until it reports up to date.

- Shut down, **disable Adapter 2** again (untick Enable Network Adapter) so no clone gets a second NIC.

- In the Snapshots tab take a snapshot named `pre-sysprep`. This lets you reopen and re-patch the template later.

- Start the VM and run Sysprep in an elevated Command Prompt:

```powershell
C:\Windows\System32\Sysprep\sysprep.exe /generalize /oobe /shutdown
```

> **Warning:** After Sysprep the template shuts down. Do not boot it again. Clone from it only. Cloning a machine that was NOT sysprepped duplicates the SID and breaks domain joins and replication. If you must patch it later, revert to `pre-sysprep`, update, snapshot again and re-run Sysprep. if Error arise you must run this to remove the edge which is not installed globally:
``` 
Get-AppxPackage -AllUsers Microsoft.MicrosoftEdge.Stable | Remove-AppxPackage -AllUsers 
```




### Step 10: Clone the template into DC01 and FS01

Right-click `TEMPLATE-WS2022` > **Clone...** and use:

| Clone wizard field | Value |
| --- | --- |
| Name | DC01 (then repeat with FS01) |
| MAC Address Policy | **Generate new MAC addresses for all network adapters** |
| Clone type | **Full clone** |
| Snapshots | Current machine state |

| VM name | Source | RAM | CPUs | Disk |
| --- | --- | --- | --- | --- |
| DC01 | Full clone of TEMPLATE-WS2022 | 4096 MB | 2 | 60 GB |
| FS01 | Full clone of TEMPLATE-WS2022 | 4096 MB | 2 | 60 GB (add a second disk later if you want a data drive) |

Optional command-line shortcut (run in an elevated PowerShell on a Windows host; adjust the path if VirtualBox is installed elsewhere):

```powershell
cd "C:\Program Files\Oracle\VirtualBox"
foreach ($vm in "DC01","FS01") {
    .\VBoxManage.exe clonevm "TEMPLATE-WS2022" --name $vm --register
    .\VBoxManage.exe modifyvm $vm --memory 4096 --cpus 2 --nic1 intnet --intnet1 "LAB-LAN"
}
```


### Step 11: Stop VirtualBox from fighting AD over the clock

Guest Additions copies the host clock into the guest. Inside a domain, the PDC emulator must be the time authority, and Guide 08 needs you to change a client clock on purpose. Turn host time sync **off** for every lab VM. The VM must be powered off:

**Run after the client VMs exist (Step 13); repeat for DC02 and PDC-B when you create them**

```powershell
cd "C:\Program Files\Oracle\VirtualBox"
foreach ($vm in "DC01","FS01","CLIENT1","CLIENT2") {
    .\VBoxManage.exe setextradata $vm "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled" 1
}
```


### Step 12: Temporary internet access (only when needed)

The lab network has no internet. When you need to download something (for example Wireshark in Guide 08), **power off** the VM, enable **Adapter 2 > NAT**, start the VM, download, then power off and disable Adapter 2 again.

> **Warning:** Do not leave a NAT adapter on a domain controller. A second NIC registers an extra address in DNS and breaks name resolution for clients.


### Step 13: Create CLIENT1 and CLIENT2

Create each with Machine > New, ticking **Skip Unattended Installation**:

| Setting | Windows 10 Enterprise | Windows 11 Enterprise |
| --- | --- | --- |
| Type / Version | Windows 10 (64-bit) | Windows 11 (64-bit) |
| Memory / CPUs | 4096 MB / 2 | 4096 MB / 2 |
| Disk | 60 GB dynamic | 80 GB dynamic |
| System > Motherboard | Defaults | Tick **Enable EFI**, set **TPM: v2.0**, tick **Enable Secure Boot** |
| Network > Adapter 1 | Internal Network, `LAB-LAN` | Internal Network, `LAB-LAN` |
| Display | 128 MB, VBoxSVGA | 128 MB, VBoxSVGA |

> **Tip:** Windows 11 needs TPM 2.0 and Secure Boot, which VirtualBox 7 provides when you tick the options above. If setup still complains, use Windows 10 Enterprise; everything in the guides works the same.

Install Windows on both. When asked about the account choose **Domain join instead** (or the offline account option) and create a local user named `labadmin`. Do not use a Microsoft account and do not join a domain yet. Install Guest Additions in each as in Step 8.


### Step 14: Group the VMs and snapshot

- Select all lab VMs, right-click > **Group**, and rename the group to `Savora-Lab`.

- Shut down every VM from inside Windows (not Power Off).

- Select each VM > **Snapshots** tab > **Take** and name it `01-Clean-Install`. Do this for DC01, FS01, CLIENT1 and CLIENT2.

> **Portfolio screenshot:** VirtualBox Manager listing the Savora-Lab group, and the Network settings of one VM showing Internal Network LAB-LAN.

![Figure 1 - Target lab topology](images/diagram2_lab.png)

*Figure 1 - Target lab topology*


## Completion Checklist

- [ ] Virtualization shows Enabled and no green turtle in VM status bars
- [ ] VirtualBox 7.x installed with a fast Default Machine Folder
- [ ] Every VM uses Adapter 1 = Internal Network LAB-LAN
- [ ] Template was patched, snapshotted as pre-sysprep, sysprepped and is never booted again
- [ ] DC01 and FS01 are full clones with new MAC addresses
- [ ] Host time sync disabled with VBoxManage on every lab VM
- [ ] CLIENT1 and CLIENT2 installed with a local labadmin user and Guest Additions
- [ ] Snapshot 01-Clean-Install taken


## Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| VT-x / AMD-V not available, or VERR_VMX / VERR_SVM error | Virtualization off in BIOS, or Windows hypervisor holds it | Enable VT-x/SVM in BIOS; follow Step 3 and reboot |
| Green turtle icon, VM very slow | VirtualBox running on top of Hyper-V / WSL2 | Turn off Hyper-V, Virtual Machine Platform and Memory integrity; run the bcdedit command |
| VMs cannot ping each other | Network name typo or not set to Internal Network | Use the dropdown to pick the exact same LAB-LAN on every VM |
| Windows 11 setup says this PC cannot run Windows 11 | EFI, TPM or Secure Boot not enabled | Enable them in System > Motherboard (VM powered off) or use Windows 10 |
| Small screen or no auto resize | Guest Additions missing | Reinstall Guest Additions; set Video Memory 128 MB and VBoxSVGA |
| Clone has the same computer name or SID problems | Template was not sysprepped, or a full clone was not used | Re-clone from the sysprepped template as a Full clone with new MAC addresses |
| VBoxManage setextradata fails | VM is running or name has a typo | Power off the VM and match the exact VM name |
| Host runs out of RAM | Too many VMs started | Start only the VMs needed for each guide; lower RAM per VM; DC02 can run on 2 GB |


## Next

Open [02_Build_DC01_and_Create_Forest.md](02_Build_DC01_and_Create_Forest.md).
