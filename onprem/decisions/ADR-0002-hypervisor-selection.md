# ADR-0002: Use Hyper-V Over Proxmox for the Production Hypervisor

**Date:** 2026-06-03
**Status:** Accepted
**Workstream:** 1

## Context

The production VMs (domain controller, member server, business workstations) need a hypervisor to run on, hosted on a Dell OptiPlex 9020. This was an early foundational decision, made before the rest of the stack was built, so it effectively set the base operating environment that everything else would be engineered around. The choice came down to whether that base environment should be Windows or Linux.

## Options Considered

### Option A: Hyper-V (Windows base)
- Run the production VMs on Hyper-V, the hypervisor built into Windows Server. Choosing it commits the environment to a Windows base, since Active Directory and the rest of the production stack would then be built on Windows.
- Pros: Native integration with a Windows-centric stack and with the equipment on hand. Aligns with Entra ID as the planned cloud gateway, since the on-prem-to-cloud identity path is Microsoft-native, and I had already targeted Entra ID before settling the hypervisor. No additional hypervisor licensing cost beyond the Windows Server licensing the environment would use anyway. Windows integrates natively with a large number of tools and programs, which keeps engineering straightforward. I can still run individual workloads Linux-native where it makes sense (for example Wazuh bare metal on an OptiPlex); going Windows-base-then-Linux-where-needed is easier than the inverse.
- Cons: Windows as a hypervisor base carries the usual criticisms (heavier footprint, less favored in pure-infrastructure circles). Some advanced virtualization features are weaker or absent compared to a dedicated Linux virtualization platform.

### Option B: Proxmox (Linux base)
- Run the production VMs on Proxmox VE, a Linux-based virtualization platform.
- Pros: Open source, strong feature set (clustering, ZFS, snapshots), well regarded for homelab and small-infrastructure use.
- Cons: Simultaneously niche and feature-heavy in ways that do not map to the stack I am building. The features that distinguish it are largely irrelevant to this environment, while a Linux base makes integration with the Windows-centric and Entra-aligned toolchain less direct. Choosing it would mean fighting the grain of the rest of the design.

ESXi was a passing consideration only. With no free tier available under current licensing, it was dismissed quickly on cost grounds and was not weighed as a genuine third option.

## Decision

Hyper-V is the production hypervisor. The deciding factors were native integration with a Windows-based toolchain and the equipment on hand, and alignment with Entra ID as the cloud gateway, which I had already targeted before settling the hypervisor. A Microsoft-native base is the path of least resistance for the hybrid identity work coming in Workstream 3. Proxmox's distinguishing features do not serve this stack, and a Windows base still leaves me free to run specific workloads Linux-native where that is the better choice, whereas the reverse would be harder.

## Consequences

- The production base environment is Windows, so AD, Hyper-V, and the eventual Entra Connect path all sit in one integrated, Microsoft-native toolchain.
- Workloads that are better off on Linux (such as Wazuh) can still run bare metal or as Linux VMs; the Windows base does not lock everything into Windows.
- Accepts the heavier footprint and the loss of Proxmox-specific features, none of which this environment needs.
- The lab on the primary desktop also uses Hyper-V, keeping the production and lab virtualization platforms consistent so cloned VMs behave the same in both.
