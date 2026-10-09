# Lab Network Architecture

**Hypervisor:** VirtualBox
**Host machine:** Physical workstation running VirtualBox with two virtual networks attached to the lab VMs.

---

## Diagram

```mermaid
flowchart TB
    Internet((Internet))

    subgraph HOST["Host machine — VirtualBox hypervisor"]
        subgraph HOSTONLY["Host-only network — 192.168.56.0/24"]
            WS_HO["Wazuh-Server<br/>192.168.56.10"]
            WU_HO["WIN-USER<br/>no host-only IP (gap)"]
        end

        subgraph NATNET["NAT network — 10.0.2.0/24"]
            WS_NAT["Wazuh-Server<br/>10.0.2.100"]
            WU_NAT["WIN-USER<br/>10.0.2.18"]
        end
    end

    Internet --> NATNET
    WS_NAT <-. agent traffic 1514/1515 .-> WU_NAT

    style WU_HO stroke-dasharray: 4 4
```

---

## Network Summary

| Network | CIDR | Purpose |
|---|---|---|
| **NAT Network** (`NatNetwork`) | `10.0.2.0/24` | Shared internal network with outbound internet access. Carries all Wazuh agent ↔ manager traffic (ports 1514/1515) since both VMs sit on this subnet. |
| **Host-only Adapter** | `192.168.56.0/24` | Lets the physical host reach VMs directly — used to browse the Wazuh dashboard (443) and API (55000) without exposing the manager to the NAT network or internet. |

## Hosts

| Host | NAT Network IP | Host-only IP | Role |
|---|---|---|---|
| **Wazuh-Server** | `10.0.2.100` | `192.168.56.10` | Wazuh manager, dashboard, API |
| **WIN-USER** | `10.0.2.18` | — *(not assigned)* | Monitored endpoint running Sysmon + Wazuh agent |
