# FortiGate Secure Site-to-Site Lab

**Final Project — NTI Fortinet Cybersecurity Track**
Creativa Innovation Hub

> 🏁 **Project Status: COMPLETE**

---

## Overview

This lab builds a fully emulated enterprise multi-site network security
environment using GNS3 and QEMU/KVM on a Linux host. Two FortiGate virtual
machines act as branch-site firewalls connected over a shared simulated WAN
segment, with segmented LAN VLANs on each branch and a secure IPsec tunnel
between them.

---

## Objectives

- Deploy and configure two FortiGate 7.0.5 VM instances in GNS3
- Simulate WAN connectivity using GNS3 Cloud nodes bridged to a Linux bridge
- Configure WAN interfaces and verify layer-3 reachability between sites
- Segment branch LANs into multiple VLANs (HR, IT, Finance, Sales, etc.)
- Build a site-to-site IPsec VPN between FGT-BRANCH-A and FGT-BRANCH-B
- Apply firewall policies to control encrypted traffic
- Validate and document end-to-end connectivity

---

## Lab Architecture

```
  BR1 (FGT-BRANCH-A)                          BR2 (FGT-BRANCH-B)
  ┌─────────────────────┐                    ┌─────────────────────┐
  │ VLAN 10  HR         │                    │ VLAN 110 Sales      │
  │ VLAN 20  IT         │                    │ VLAN 120 Support    │
  │ VLAN 30  Finance    │                    │ VLAN 130 Guest      │
  │ VLAN 99  Management │                    │ VLAN 199 Management │
  │ VLAN 40  Server_1   │                    └──────────┬──────────┘
  │ VLAN 50  Server_2   │                               │ port3 (trunk)
  └──────────┬──────────┘                        ┌──────┴──────┐
             │ port3 (trunk)           IPsec VPN │ FGT-BRANCH-B│
      ┌──────┴──────┐              ◄────────────►│  FortiOS    │
      │ FGT-BRANCH-A│              WAN            │  7.0.5      │
      │  FortiOS    │◄────────────────────────────┘             │
      │  7.0.5      │ port1                    port1            │
      └─────────────┘ 172.31.255.10/24   172.31.255.20/24      │
             │                                                   │
      ┌──────┴───────────────────────────────────────────────────┘
      │          Host Linux bridge: br-wan — 172.31.255.1/24
      └──────────────────────────────────────────────────────────┘
```

---

## IP Addressing

### WAN Interfaces

| Device       | Interface | IP Address        | Role         |
|--------------|-----------|-------------------|--------------|
| Host (Linux) | br-wan    | 172.31.255.1/24   | WAN gateway  |
| FGT-BRANCH-A | port1     | 172.31.255.10/24  | WAN (Site A) |
| FGT-BRANCH-B | port1     | 172.31.255.20/24  | WAN (Site B) |

### Management Interfaces (port2 — GNS3 NAT)

| Device       | Interface | IP Address        | Access URL              |
|--------------|-----------|-------------------|-------------------------|
| FGT-BRANCH-A | port2     | 192.168.122.69    | `http://192.168.122.69` |
| FGT-BRANCH-B | port2     | 192.168.122.6     | `http://192.168.122.6`  |

### BR-A VLAN Addressing (FGT-BRANCH-A — port3)

| VLAN | Name       | Network        | Gateway     |
|------|------------|----------------|-------------|
| 10   | HR         | 10.10.10.0/24  | 10.10.10.1  |
| 20   | IT         | 10.10.20.0/24  | 10.10.20.1  |
| 30   | Finance    | 10.10.30.0/24  | 10.10.30.1  |
| 99   | Management | 10.10.99.0/24  | 10.10.99.1  |
| 40   | Server_1   | 10.10.40.0/24  | 10.10.40.1  |
| 50   | Server_2   | 10.10.50.0/24  | 10.10.50.1  |

### BR-B VLAN Addressing (FGT-BRANCH-B — port3)

| VLAN | Name       | Network        | Gateway     |
|------|------------|----------------|-------------|
| 110  | Sales      | 10.20.10.0/24  | 10.20.10.1  |
| 120  | Support    | 10.20.20.0/24  | 10.20.20.1  |
| 130  | Guest      | 10.20.30.0/24  | 10.20.30.1  |
| 199  | Management | 10.20.99.0/24  | 10.20.99.1  |

---

## Progress

| # | Phase | Status |
|---|-------|--------|
| 1 | Environment Setup (GNS3 install + appliance import) | ✅ Done |
| 2 | GNS3 Topology — Cloud nodes + FortiGate instances | ✅ Done |
| 3 | FortiGate WAN interface configuration | ✅ Done |
| 4 | WAN reachability verified (ping between FortiGates) | ✅ Done |
| 5 | Management access via GUI (port2 NAT) | ✅ Done |
| 6 | VLAN segmentation on port3 (BR1 × 6 VLANs, BR2 × 4 VLANs) | ✅ Done |
| 7 | Switch trunk & access port config (IOU1, IOU2, SW1–SW4) | ✅ Done |
| 8 | FortiGate DHCP servers per VLAN (BR1 & BR2) | ✅ Done |
| 9 | Linux/Alpine host IP assignment & gateway reachability | ✅ Done |
| 10 | Screenshots — device configs & ping proofs | ✅ Done |
| 11 | Firewall address objects & groups (BR1 & BR2) | ✅ Done |
| 12 | Branch A internal policies (8 policies — users → servers) | ✅ Done |
| 13 | Branch B isolation & Internet/NAT policies | ✅ Done |
| 14 | IPsec Site-to-Site VPN tunnel (FGT-A ↔ FGT-B) | ✅ Done |
| 15 | VPN firewall policies — Management VLAN ↔ Management VLAN | ✅ Done |
| 16 | End-to-end cross-site ping validation (MGMT-A ↔ MGMT-B) | ✅ Done |

---

## Documentation

```
docs/
├── 01-environment-setup/
│   ├── 00-prerequisites.md               — Downloads & required files
│   ├── 01-gns3-installation.md           — GNS3 install on Linux (Fedora)
│   └── 02-fortigate-appliance-import.md  — Import FortiGate & IOU appliances
│
├── 02-topology-setup/
│   ├── FORTIGATE/
│   │   ├── 01-CLOUD_CONNECTION/
│   │   │   ├── 00-gns3-topology.md           — Cloud nodes + FortiGate topology
│   │   │   └── 01-fortigate-basic-config.md  — WAN interface config & ping proof
│   │   ├── 02-NAT_CONNECTION/
│   │   │   └── 02-nat-management-access.md   — port2 DHCP + GUI access
│   │   ├── 03-VLANS/
│   │   │   ├── BRANCH-A/
│   │   │   │   └── 01-vlan-setup-br1.md      — BR1 VLAN sub-interfaces on port3
│   │   │   └── BRANCH-B/
│   │   │       └── 01-vlan-setup-br2.md      — BR2 VLAN sub-interfaces on port3
│   │   └── 04-DHCP/
│   │       ├── BRANCH-A/
│   │       │   └── 01-dhcp-br1.md            — DHCP server per VLAN on FGT-A
│   │       └── BRANCH-B/
│   │           └── 01-dhcp-br2.md            — DHCP server per VLAN on FGT-B
│   ├── SWITCHES/
│   │   ├── BRANCH-A/
│   │   │   ├── IOU1/  — Distribution trunk to FGT-A + SW1 + SW2
│   │   │   ├── SW1/   — Access switch: HR, IT, Finance, Management
│   │   │   └── SW2/   — Access switch: Server_1, Server_2
│   │   └── BRANCH-B/
│   │       ├── IOU2/  — Distribution trunk to FGT-B + SW3 + SW4
│   │       ├── SW3/   — Access switch: Sales, Support, Guest
│   │       └── SW4/   — Access switch: Management
│   ├── HOSTS/
│   │   ├── BRANCH-A/
│   │   │   └── 01-linux-host-config.md       — Alpine/Linux DHCP + static setup
│   │   └── BRANCH-B/
│   │       └── 01-linux-host-config.md       — Alpine/Linux DHCP + static setup
│   └── IPSEC-VPN/                            — ✅ Complete
│       ├── 01-ipsec-phase1-config.md          — IKE Phase 1 (PSK, AES-256, DH14)
│       ├── 02-ipsec-phase2-config.md          — Phase 2 selectors & static routes
│       ├── 03-firewall-policies.md            — VPN firewall policy pattern
│       └── 04-cross-site-mgmt-poc.md         — ✅ Proof-of-concept: MGMT↔MGMT ping
│
├── 03-branch-a-firewall-policy/             — ✅ Complete
│   ├── ADDRESSES/
│   │   └── 01-branch-a-address-objects.md   — Address objects & A-SERVERS group
│   ├── POLICIES/
│   │   └── 01-branch-a-firewall-policies.md — 8 internal policies + service groups
│   └── VLAN-INTER-ROUTING/
│       └── 01-branch-a-vlan-inter-routing.md — Routing design, access matrix, tests
│
└── 04-branch-b-firewall-policy/             — ✅ Complete
    ├── ADDRESSES/
    │   └── 01-branch-b-address-objects.md   — Address objects & B-USERS group
    └── POLICIES/
        └── 01-branch-b-firewall-policies.md — Isolation deny + Internet/NAT policies
```

---


## Quick Start

1. Read [prerequisites](docs/01-environment-setup/00-prerequisites.md) and download required images.
2. Follow [GNS3 installation](docs/01-environment-setup/01-gns3-installation.md).
3. Follow [appliance import](docs/01-environment-setup/02-fortigate-appliance-import.md).
4. Build the [GNS3 topology](docs/02-topology-setup/FORTIGATE/01-CLOUD_CONNECTION/00-gns3-topology.md).
5. Configure [FortiGate WAN interfaces](docs/02-topology-setup/FORTIGATE/01-CLOUD_CONNECTION/01-fortigate-basic-config.md).
6. Enable [management GUI access](docs/02-topology-setup/FORTIGATE/02-NAT_CONNECTION/02-nat-management-access.md) via port2.
7. Configure [BR1 VLANs](docs/02-topology-setup/FORTIGATE/03-VLANS/BRANCH-A/01-vlan-setup-br1.md) on FGT-BRANCH-A port3.
8. Configure [BR2 VLANs](docs/02-topology-setup/FORTIGATE/03-VLANS/BRANCH-B/01-vlan-setup-br2.md) on FGT-BRANCH-B port3.
9. Configure [BR1 DHCP servers](docs/02-topology-setup/FORTIGATE/04-DHCP/BRANCH-A/01-dhcp-br1.md) on FGT-BRANCH-A.
10. Configure [BR2 DHCP servers](docs/02-topology-setup/FORTIGATE/04-DHCP/BRANCH-B/01-dhcp-br2.md) on FGT-BRANCH-B.
11. Verify [Branch-A host connectivity](docs/02-topology-setup/HOSTS/BRANCH-A/01-linux-host-config.md).
12. Verify [Branch-B host connectivity](docs/02-topology-setup/HOSTS/BRANCH-B/01-linux-host-config.md).
13. Create [Branch-A address objects](docs/03-branch-a-firewall-policy/ADDRESSES/01-branch-a-address-objects.md) on FGT-BRANCH-A.
14. Apply [Branch-A internal firewall policies](docs/03-branch-a-firewall-policy/POLICIES/01-branch-a-firewall-policies.md) (8 policies).
15. Create [Branch-B address objects](docs/04-branch-b-firewall-policy/ADDRESSES/01-branch-b-address-objects.md) on FGT-BRANCH-B.
16. Apply [Branch-B isolation & NAT policies](docs/04-branch-b-firewall-policy/POLICIES/01-branch-b-firewall-policies.md).
17. Configure [IPsec Phase 1](docs/02-topology-setup/IPSEC-VPN/01-ipsec-phase1-config.md) on both FortiGates.
18. Configure [IPsec Phase 2](docs/02-topology-setup/IPSEC-VPN/02-ipsec-phase2-config.md) selectors and static routes.
19. Create [VPN firewall policies](docs/02-topology-setup/IPSEC-VPN/03-firewall-policies.md) for cross-site access.
20. Validate with the [cross-site Management PoC](docs/02-topology-setup/IPSEC-VPN/04-cross-site-mgmt-poc.md) ping test.

---


## Tech Stack

| Component   | Version                         |
|-------------|---------------------------------|
| OS          | Fedora Linux                    |
| GNS3        | 2.2.61                          |
| FortiOS     | 7.0.5                           |
| QEMU/KVM    | Host virtualization             |
| Cisco IOU   | 15.2 (May 2018) — L2 switching  |
