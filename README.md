# FortiGate Secure Site-to-Site Lab

**Final Project — NTI Fortinet Cybersecurity Track**
Creativa Innovation Hub

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

### BR1 VLAN Addressing (FGT-BRANCH-A — port3)

| VLAN | Name       | Network        | Gateway     |
|------|------------|----------------|-------------|
| 10   | HR         | 10.10.10.0/24  | 10.10.10.1  |
| 20   | IT         | 10.10.20.0/24  | 10.10.20.1  |
| 30   | Finance    | 10.10.30.0/24  | 10.10.30.1  |
| 99   | Management | 10.10.99.0/24  | 10.10.99.1  |
| 40   | Server_1   | 10.10.40.0/24  | 10.10.40.1  |
| 50   | Server_2   | 10.10.50.0/24  | 10.10.50.1  |

### BR2 VLAN Addressing (FGT-BRANCH-B — port3)

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
| 7 | Switch trunk & access port config (SW1 / SW3) | 🔜 Next |
| 8 | IPsec Site-to-Site VPN tunnel | 🔜 Upcoming |
| 9 | Firewall policies | 🔜 Upcoming |
| 10 | Full end-to-end validation | 🔜 Upcoming |

---

## Documentation

```
docs/
├── 01-environment-setup/
│   ├── 00-prerequisites.md               — Downloads & required files
│   ├── 01-gns3-installation.md           — GNS3 install on Linux (Fedora)
│   └── 02-fortigate-appliance-import.md  — Import FortiGate & IOU appliances
│
└── 02-topology-setup/
    └── FORITGATE/
        ├── 01-CLOUD_CONNECTION/
        │   ├── 00-gns3-topology.md           — Cloud nodes + FortiGate topology
        │   └── 01-fortigate-basic-config.md  — WAN interface config & ping proof
        │
        ├── 02-NAT_CONNECTION/
        │   └── 02-nat-management-access.md   — port2 DHCP + GUI access
        │
        └── 03-VLANS/
            ├── BRANCH-A/
            │   └── 01-vlan-setup-br1.md      — BR1 VLAN sub-interfaces on port3
            └── BRANCH-B/
                └── 01-vlan-setup-br2.md      — BR2 VLAN sub-interfaces on port3
```

---

## Quick Start

1. Read [prerequisites](docs/01-environment-setup/00-prerequisites.md) and download required images.
2. Follow [GNS3 installation](docs/01-environment-setup/01-gns3-installation.md).
3. Follow [appliance import](docs/01-environment-setup/02-fortigate-appliance-import.md).
4. Build the [GNS3 topology](docs/02-topology-setup/FORITGATE/01-CLOUD_CONNECTION/00-gns3-topology.md).
5. Configure [FortiGate WAN interfaces](docs/02-topology-setup/FORITGATE/01-CLOUD_CONNECTION/01-fortigate-basic-config.md).
6. Enable [management GUI access](docs/02-topology-setup/FORITGATE/02-NAT_CONNECTION/02-nat-management-access.md) via port2.
7. Configure [BR1 VLANs](docs/02-topology-setup/FORITGATE/03-VLANS/BRANCH-A/01-vlan-setup-br1.md) on FGT-BRANCH-A port3.
8. Configure [BR2 VLANs](docs/02-topology-setup/FORITGATE/03-VLANS/BRANCH-B/01-vlan-setup-br2.md) on FGT-BRANCH-B port3.

---

## Tech Stack

| Component   | Version                         |
|-------------|---------------------------------|
| OS          | Fedora Linux                    |
| GNS3        | 2.2.61                          |
| FortiOS     | 7.0.5                           |
| QEMU/KVM    | Host virtualization             |
| Cisco IOU   | 15.2 (May 2018) — L2 switching  |
