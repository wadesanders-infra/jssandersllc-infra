# ADR-0011: Run the Lab as an Isolated Replica on the Dedicated Desktop, Not on the Production Network or in the Cloud

**Date:** 2026-07-01
**Status:** Superseded by ADR-0014
**Workstream:** 1

## Context

Workstream 4's attack simulation and detection engineering require an environment that mirrors production and can be attacked, broken, and rebuilt freely. The production Hyper-V host (OptiPlex 9020) is at capacity running DC01 and SRV01 and cannot absorb a lab workload. The outline originally envisioned cloned production VMs carrying the real domain and SIDs; the lab is instead built as a generic replica (same topology, same design decisions, its own domain name and fresh SIDs), because a clone would place production identity and data on a personal machine, extending the production trust boundary onto hardware that sits outside it (ADR-0009). The question was where the lab should run.

## Options Considered

### Option A: Lab VLAN on the production network
- Add a fifth VLAN behind the ASA and run lab VMs on the production Hyper-V host, with ACLs separating lab from business segments.
- Pros: No additional hardware. One management plane for everything.
- Cons: The production host does not have the capacity, which rules this out on its own. Beyond that, isolation would be procedural rather than structural: one misconfigured ACL puts attack traffic one hop from business systems, and under the original clone plan a duplicate domain and SIDs on a routed network would have risked replication and identity conflicts with production.

### Option B: Cloud lab (Azure/AWS)
- Build the lab as cloud VMs in a personal tenant.
- Pros: No local hardware or capacity constraints. Snapshots and teardown are trivial.
- Cons: Recurring cost against a buildout deliberately assembled from hardware already owned. Fidelity suffers: the production environment's behavior is shaped by the ASA and router-on-a-stick L2 topology, which a cloud VNet does not reproduce. Running attack tooling also adds provider acceptable-use considerations that a local, disconnected lab avoids entirely.

### Option C: Internal-only vSwitch on the primary desktop
- Run the replica VMs plus Kali on the i7-12700K desktop, attached to a Hyper-V internal-only vSwitch with no uplink and no path to any physical network.
- Pros: The desktop is the only machine available with the spare compute. No uplink means isolation is structural, a property of the topology rather than a rule that must be maintained. The desktop is already permanently outside production AD (ADR-0009), so the lab host is not a domain member of the environment it simulates.
- Cons: Lab availability is tied to a personal machine and contends with personal use. No remote access to the lab.

## Decision

The lab runs on the primary desktop as a generic replica of production, on an internal-only Hyper-V vSwitch with no uplink. The initial driver was capacity: the production host cannot fit a lab and the desktop is the only machine that can. The isolation properties turned out to reinforce the choice rather than merely tolerate it: no uplink means attack traffic has no possible path to business systems, and the replica approach means no production SIDs, credentials, or data ever sit on a machine outside the production boundary.

## Consequences

- Isolation is structural, not procedural. There is no firewall rule whose misconfiguration could expose production to lab traffic, because there is no path.
- The replica replaces the clone plan. Lab fidelity now comes from rebuilding to spec from the ADRs, diagrams, and journals rather than from copying VMs. This doubles as a test of the documentation itself: if the replica cannot be rebuilt from the repo, the repo is incomplete.
- Drift is the accepted trade-off. A clone tracks production automatically; a replica only tracks it as well as the documentation does. Any future production change must be reflected in the docs for the replica to stay faithful.
- Follow-up: the README and outline still describe the lab as cloned production VMs with the same domain and SIDs, and the Workstream 1 exit criterion reads "Production VMs cloned to desktop." Both need updating to replica language to match this decision.
- The desktop remains permanently off the production domain per ADR-0009; this ADR depends on and reinforc