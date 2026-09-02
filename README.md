# NOC Incident Triage & Network Design Lab

Hands-on network engineering labs built to simulate real Tier 1 NOC scenarios — VLAN segmentation, inter-VLAN routing, DHCP, ACL-based security, redundancy (STP), dynamic routing (OSPF), NAT, and proactive monitoring (Zabbix). Each lab includes deliberately injected faults, diagnosed and documented using real incident ticket format.

Built in Cisco Packet Tracer.

## Why This Repo Exists

Networking theory is easy to claim on a resume — this repo is the proof. Every fault below was actually broken, actually diagnosed using CLI tools and packet-level traffic analysis, and actually fixed, with the full diagnostic trail documented the way a real NOC ticket would be written.

## Skills Demonstrated

- VLAN design and segmentation
- Inter-VLAN routing (SVI on a Layer 3 switch)
- DHCP configuration (per-VLAN scoping, exclusions)
- ACL-based traffic filtering (extended ACLs, wildcard masking)
- Trunk configuration and troubleshooting (802.1Q)
- Packet-level traffic analysis (Packet Tracer Simulation Mode)
- Incident documentation and root cause analysis (RCA)

---

## Lab 1: Small Office Network — VLANs, Inter-VLAN Routing, DHCP, ACL Security

### Scenario
A small office with 3 departments — Sales, IT, and Finance — each requiring its own VLAN and subnet, automatic IP addressing, and traffic isolation between Sales and Finance (with IT able to reach both).

### Topology
- 1 Layer 3 (multilayer) switch — SVI-based inter-VLAN routing
- 1 Layer 2 access switch
- 3 VLANs: 10 (Sales), 20 (IT), 30 (Finance)
- Subnets: 192.168.10.0/26, 192.168.20.0/26, 192.168.30.0/26

  ![Topology Diagram](screenshots/topology.png)

### Design Decisions
**SVI over router-on-a-stick:** chosen because a Layer 3 switch was available. SVI routes in hardware, avoids funneling all inter-VLAN traffic through a single physical link, and is simpler to manage than sub-interface encapsulation.

### Configuration Summary
- Trunk link configured between access switch and multilayer switch, allowing VLANs 10, 20, 30
- SVIs configured per VLAN with `ip routing` enabled globally
- DHCP: one pool per VLAN on the L3 switch, gateway IPs excluded from each pool
- Extended ACL applied outbound on Sales and Finance VLAN interfaces, blocking Sales↔Finance while permitting all other traffic (including IT↔both)

### Faults Found & Fixed

#### Incident INC-001: Trunk Misconfiguration Isolating Sales VLAN
| Field | Detail |
|---|---|
| Severity | Sev2 — degraded, single VLAN isolated |
| Symptoms | Sales VLAN unable to reach gateway or any other VLAN |
| Diagnostic Steps | `show ip interface brief` confirmed SVI up/up → `show interfaces trunk` revealed VLAN 10 missing from allowed list on multilayer switch trunk port |
| Root Cause | Trunk configured with `switchport trunk allowed vlan 20,30` — VLAN 10 omitted |
| Resolution | Corrected to `switchport trunk allowed vlan 10,20,30` on both trunk ends; verified via ping |
| Prevention | Standardize a trunk provisioning checklist listing all active VLANs explicitly at setup time |

*[Insert Simulation Mode screenshot showing packet failing to cross the trunk]*

#### Incident INC-002: ACL Wildcard Mask Error Allowing Blocked Traffic
| Field | Detail |
|---|---|
| Severity | Sev2 — security/compliance exposure, Sales and Finance able to communicate despite segmentation policy |
| Symptoms | Ping tests Sales↔Finance unexpectedly succeeded |
| Diagnostic Steps | `show access-lists` showed match counts hitting `permit any any` instead of the deny lines → reviewed wildcard mask, found `0.0.0.64` instead of the correct `0.0.0.63` for a /26 subnet |
| Root Cause | Incorrect wildcard mask only covered 2 addresses instead of the full 64-address /26 range, allowing most traffic to fall through to the permit line |
| Resolution | Rebuilt ACL with corrected wildcard mask; verified Sales↔Finance blocked, IT↔both still permitted |
| Prevention | Use the standard wildcard calculation (255 − subnet mask octet) for any non-/24 subnet; verify ACL match counters after initial deployment, not just config syntax |

*[Insert `show access-lists` screenshot showing corrected match counts]*

### Verification Checklist
- [x] Each PC receives correct IP via DHCP for its VLAN
- [x] PCs within same VLAN can ping each other
- [x] IT can ping Sales and Finance
- [x] Sales cannot ping Finance (and vice versa)
- [x] Both faults diagnosed, fixed, and documented

---

## Lab 2: Redundant Network — STP, OSPF, Failover *(in progress)*

## Lab 3: Multi-Site WAN — NAT, ACLs, Zabbix Monitoring *(planned)*

## Lab 4: Full Incident Response Simulation *(planned)*

---

## Tools Used
- Cisco Packet Tracer
- Zabbix *(Lab 3+)*

## Repo Structure
```
/screenshots       — topology diagrams, Simulation Mode captures, config verification
/configs           — exported device configs
/tickets           — full incident tickets (markdown)
README.md
```
