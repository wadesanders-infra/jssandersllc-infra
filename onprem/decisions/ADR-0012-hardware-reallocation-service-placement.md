# ADR-0012: Place Wazuh on Bare-Metal Micro #1, Micro #2 at the Camera Building, and the Donated Laptops as Endpoints

**Date:** 2026-07-01
**Status:** Accepted
**Workstream:** 1

## Context

This buildout is opportunistic on hardware: mid-project I acquired two OptiPlex Micros (needing RAM and SSDs) and two free laptops, which changed a plan written around less equipment (2026-05-29 journal). Before that, the SIEM was going to run on an old repurposed laptop (2026-05-08 journal), and the production Hyper-V host (OptiPlex 9020) was already at capacity with DC01 and SRV01. The new hardware needed role assignments covering the SIEM host, DVR event integration at the remote camera building, and two business endpoints.

## Options Considered

### Option A: Wazuh as a VM on the production Hyper-V host
- Add the SIEM as a third VM on the OptiPlex 9020.
- Pros: No additional hardware; one host to manage.
- Cons: The 9020 does not have the headroom; SIEM ingestion and indexing are resource-heavy. It also puts monitoring inside the failure domain it is supposed to observe: if the host or hypervisor goes down, the record of what happened goes down with it.

### Option B: Wazuh on a repurposed old laptop (the original plan)
- Reimage an old laptop with a GUI-less Linux distribution as a dedicated SIEM host.
- Pros: Uses hardware already owned; keeps the SIEM off the production host.
- Cons: Marginal hardware for a retention and indexing workload. Displaced when the Micros arrived: same dedicated-host benefit on better equipment.

### Option C: Keep Micro #2 as a spare; ship DVR events over a site-to-site VPN
- Hold Micro #2 as a cold spare and connect the camera building's DVR to Wazuh by linking the building's network to production.
- Pros: Retains a spare against hardware failure.
- Cons: The camera building has its own internet and no link to production; a site-to-site VPN adds cost, complexity, and attack surface to solve a one-way event-shipping problem. A cold spare is idle hardware with a hypothetical job.

### Option D: The chosen allocation
- Micro #1 as bare-metal Wazuh host on the MGMT VLAN; Micro #2 as a local DVR integration host at the camera building, shipping events to Wazuh encrypted over the building's own internet; the two laptops as the business endpoints on the CLIENTS VLAN.
- Pros: Every device has a real, defined role. The SIEM is outside the production host's failure domain. No new network paths into production. Real laptops generate real endpoint telemetry, which is worth more to the monitoring program than additional VMs.
- Cons: No spare hardware remains; a failure means reallocating or buying.

## Decision

Option D. The driving constraint for Wazuh was capacity: the production host cannot absorb a SIEM workload, and the acquired Micro fit the dedicated-host role better than the laptop plan it replaced. Micro #2 goes to the camera building because a local integration host is the only clean path for DVR events given the building's separate internet; a VPN was evaluated and rejected as disproportionate. The laptop role split (Pavilion x360 as field workstation JSS-WS01, Latitude E6500 as contractor workstation JSS-WS02) was a practical call made on availability and timing, not a strong technical preference; role restrictions live in OU placement and GPO (ADR-0010), so the assignment could be swapped without redesign.

## Consequences

- The SIEM survives production host failure, which is the point of having one: the monitoring record must outlive the thing it monitors.
- The Windows 11 VM endpoint was cut; the laptops replace it and only support Windows 10. Endpoint telemetry comes from real hardware.
- No cold spare exists. Hardware failure means reallocation or purchase, an accepted risk in a budget-constrained build.
- Micro #2's placement commits the surveillance integration to an agent-push model over the building's internet; no network path from the camera building into production is created.
- Both Micros need RAM and SSDs before deployment; Wazuh remains staged, not deployed, until then.
- The role-neutral naming and OU/GPO model (ADR-0010) is what makes the arbitrary laptop assignment safe: if the split ever needs to reverse, it is an OU move, not a rebuild.
