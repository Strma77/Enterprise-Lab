# Enterprise Network Lab — Cisco Packet Tracer

A production-grade enterprise network simulation built in Cisco Packet Tracer, covering the majority of CCNA exam topics in a single cohesive topology. Designed as both a hands-on learning project and a portfolio piece demonstrating real-world networking knowledge.

---

## Topology Overview

Three-tier hierarchical design:

```
                          [ISP]
                            |
                          [R3]
                         /     \
                       [R1]   [R2]
                        |       |
                      [DS1]   [DS2]
                     / | \ \ / | \ \
                  AS1 AS2 AS3 AS4 AS5 AS6 AS7 AS8
```

| Layer | Devices | Role |
|-------|---------|------|
| Edge | R3 | Internet gateway, NAT/PAT |
| Distribution (Routers) | R1, R2 | OSPF backbone, WAN routing |
| Distribution (L3 Switches) | DS1, DS2 (Cisco 3560) | Inter-VLAN routing, HSRP, STP root |
| Access | AS1–AS8 (Cisco 2960) | Host connectivity, port security |

---

## Technologies Implemented

- **VLANs** — 6 VLANs including data, voice, and native VLAN with 802.1Q trunking
- **RPVST+** — Rapid Per-VLAN Spanning Tree with deliberate root bridge election per VLAN
- **SVIs** — Inter-VLAN routing via Switched Virtual Interfaces on DS1 and DS2
- **HSRP** — Redundant default gateways per VLAN, load balanced across DS1 and DS2
- **OSPF** — Single area (Area 0) dynamic routing across 5 devices
- **DHCP** — One pool per VLAN with HSRP virtual IPs as default gateways
- **NAT/PAT** — PAT overload on R3 for internet access
- **SSH** — Hardened remote management on all devices (SSHv2, Telnet disabled)
- **Port Security** — Sticky MAC on all access ports with shutdown violation mode
- **Voice VLAN** — VLAN 50 with IP phones on HR and Sales departments
- **ACLs** — Designed and documented (PT simulator limitation on 3560 SVIs — see documentation)

---

## VLAN Design

| VLAN | Name | Department | Hosts | Color |
|------|------|------------|-------|-------|
| 10 | IT | IT Department | 105 | Blue |
| 20 | SALES | Sales Department | 47 | Purple |
| 30 | HR | HR Department | 20 | Red |
| 40 | MGMT | Management | 10 | Green |
| 50 | VOICE | VoIP | 4 phones | — |
| 99 | NATIVE | Native VLAN (trunk hardening) | — | — |

---

## IP Addressing Plan

Base address space: **10.0.0.0/8** — VLSM applied

| Segment | Subnet | Network | First Usable | Last Usable | Hosts |
|---------|--------|---------|--------------|-------------|-------|
| IT (VLAN 10) | /25 | 10.0.0.0 | 10.0.0.1 | 10.0.0.126 | 126 |
| Sales (VLAN 20) | /26 | 10.0.0.128 | 10.0.0.129 | 10.0.0.190 | 62 |
| HR (VLAN 30) | /27 | 10.0.0.192 | 10.0.0.193 | 10.0.0.222 | 30 |
| MGMT (VLAN 40) | /28 | 10.0.0.224 | 10.0.0.225 | 10.0.0.238 | 14 |
| R1–R2 | /30 | 10.0.0.240 | 10.0.0.241 | 10.0.0.242 | 2 |
| R1–R3 | /30 | 10.0.0.244 | 10.0.0.245 | 10.0.0.246 | 2 |
| R2–R3 | /30 | 10.0.0.248 | 10.0.0.249 | 10.0.0.250 | 2 |
| R3–ISP | /30 | 10.0.0.252 | 10.0.0.253 | 10.0.0.254 | 2 |
| DS1–R1 | /30 | 10.0.1.0 | 10.0.1.1 | 10.0.1.2 | 2 |
| DS2–R2 | /30 | 10.0.1.4 | 10.0.1.5 | 10.0.1.6 | 2 |
| Voice (VLAN 50) | /29 | 10.0.1.8 | 10.0.1.9 | 10.0.1.14 | 6 |

### HSRP Virtual IPs (Default Gateways)

| VLAN | DS1 SVI | DS2 SVI | HSRP VIP | Active |
|------|---------|---------|----------|--------|
| VLAN 10 | 10.0.0.1 | 10.0.0.2 | 10.0.0.3 | DS1 |
| VLAN 20 | 10.0.0.129 | 10.0.0.130 | 10.0.0.131 | DS2 |
| VLAN 30 | 10.0.0.193 | 10.0.0.194 | 10.0.0.195 | DS1 |
| VLAN 40 | 10.0.0.225 | 10.0.0.226 | 10.0.0.227 | DS2 |

---

## Key Design Decisions

### STP + HSRP Alignment
DS1 is STP root AND HSRP active for VLANs 10 and 30. DS2 is STP root AND HSRP active for VLANs 20 and 40. This prevents suboptimal traffic paths where STP and HSRP disagree on which switch to use — a common real-world misconfiguration.

### L2 Cross-Links for RPVST+
Access switches AS4 and AS7 are cross-connected to both DS1 and DS2, creating redundant L2 paths across the full switching domain. This allows RPVST+ to operate correctly across both distribution switches and enables proper root bridge election per VLAN.

### HSRP Load Balancing
Rather than making one DS switch active for all VLANs, HSRP is load balanced — DS1 handles IT and HR traffic, DS2 handles Sales and MGMT traffic. Both switches actively forward traffic under normal operation.

### PAT Overload on R3
All internal RFC1918 traffic is translated to R3's single outside interface IP using PAT overload. No NAT pool required — port multiplexing handles the translation for all internal hosts simultaneously.

### ACL Design (PT Limitation)
Inter-VLAN ACL policy was fully designed and the correct commands were written and tested. Packet Tracer does not support ACL application on SVI interfaces on the 3560 model — commands are accepted but silently discarded. The policy, logic, and commands are documented in full in the technical documentation. This works correctly on real hardware.

---

## Known Packet Tracer Limitations Encountered

| Feature | Issue | Status |
|---------|-------|--------|
| EtherChannel | PT incompatibility between 3560 and 2960 mixed models | Scrapped — documented |
| ACLs on SVIs | PT silently discards ACL application on 3560 SVI interfaces | Documented with full commands |
| HSRP VLAN 20 | PT not propagating HSRP multicast hellos for VLAN 20 | Documented — other VLANs working |

---

## Repository Structure

```
enterprise-network-lab/
├── README.md
├── documentation/
│   └── Enterprise_Network_Lab.docx
├── topology/
│   └── practiceTopology.pkt
├── ip-plan/
│   └── ip-addressing.txt
├── screenshots/
│   └── (verification screenshots)
└── configs/
    └── (running configs for all 13 devices)
```

---

## Tools Used

- Cisco Packet Tracer 8.x
- Cisco IOS (simulated)
- Devices: ISR4300 routers, Catalyst 3560 L3 switches, Catalyst 2960 access switches

---

*Full technical documentation including per-stage configuration details, troubleshooting notes, and design decisions available in `/documentation/Enterprise_Network_Lab.docx`*
