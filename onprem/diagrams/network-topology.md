# Network Topology

**Last updated:** 2026-06-11

Solid outlines and lines are built; dashed are planned or in flight. Port labels appear only where the physical cabling is documented (2026-05-06 journal): ASA p2 → SW p1 (802.1Q trunk), SW p2 → primary desktop (VLAN 50), SW p3 → Hyper-V host (802.1Q trunk).

```mermaid
flowchart TB
    classDef planned stroke-dasharray: 5 5

    INET(["Internet via upstream home network<br/>double-NAT; ASA not yet true edge"])
    ASA["Cisco ASA 5506-X<br/>perimeter firewall, inter-VLAN router, NAT"]
    SW["Cisco SG350-10<br/>L2 802.1Q VLAN tagging"]

    INET --- ASA
    ASA ---|"ASA p2 to SW p1, 802.1Q trunk"| SW

    subgraph HV["Dell OptiPlex 9020 - production Hyper-V host"]
        DC01["DC01 - AD DS, DNS, DHCP<br/>SERVERS, VLAN 20"]
        SRV01["SRV01 - member server, file shares<br/>SERVERS, VLAN 20"]
        JUMP["Jumpbox - admin workstation<br/>MGMT, VLAN 10 - planned"]
    end

    subgraph DESK["Primary desktop - USER VLAN 50 - not domain-joined, ADR-0009"]
        subgraph LAB["Lab - internal-only vSwitch, no uplink - planned, WS4"]
            DCL["DC01-Lab"]
            W10["Win10-Lab"]
            KALI["Kali"]
        end
    end

    WAZ["Micro #1 - Wazuh SIEM/XDR<br/>MGMT, VLAN 10 - staged, not deployed"]
    PAV["HP Pavilion x360 - field workstation<br/>CLIENTS, VLAN 30 - join in progress"]
    LAT["Dell Latitude E6500 - contractor workstation<br/>CLIENTS, VLAN 30"]

    SW ---|"SW p3, 802.1Q trunk"| HV
    SW ---|"SW p2, VLAN 50"| DESK
    SW -.- WAZ
    SW --- PAV
    SW --- LAT

    subgraph SWANN["Swann camera building - remote site - planned, WS2"]
        DVR["Swann DVR"]
        M2["Micro #2 - DVR integration host"]
        DVR --- M2
    end
    M2 -.->|"Wazuh agent, encrypted, over building internet"| WAZ

    ENTRA(["Entra ID tenant - planned, WS3"])
    SRV01 -.->|"Entra Connect Sync"| ENTRA

    class JUMP,LAB,DCL,W10,KALI,WAZ,M2,DVR,SWANN,ENTRA,PAV planned
```

## VLANs

| VLAN | ID | Subnet | Purpose |
|---|---|---|---|
| MGMT | 10 | 10.10.10.0/24 | Infrastructure management; monitoring host (planned) |
| SERVERS | 20 | 10.10.20.0/24 | Domain controller, member server; document and video management (planned) |
| CLIENTS | 30 | 10.10.30.0/24 | Business workstations |
| USER | 50 | 10.10.50.0/24 | Development workstation |

## Notes

- The ASA performs all inter-VLAN routing and ACL enforcement via 802.1Q subinterfaces on the trunk to the SG350 (router-on-a-stick). The switch does L2 tagging only.
- The network currently double-NATs through the upstream home network, so the ASA is not yet the true edge. This must be resolved before Workstream 3 remote access work (2026-05-06 journal).
- The USER VLAN is running a temporarily permissive ACL during buildout, to be re-restricted at the end of Workstream 1 (ADR-0008).
