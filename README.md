# FortiGate Secure Site-to-Site Lab

**Final Project — NTI Fortinet Cybersecurity Track**
Creativa Innovation Hub

---

## Overview

This lab builds a fully emulated FortiGate site-to-site IPsec VPN environment
using GNS3 and QEMU/KVM on a Linux host. Two FortiGate virtual machines act
as branch-site firewalls connected over a shared simulated WAN segment, with
the goal of establishing a secure IPsec tunnel between them.

---

## Objectives

- Deploy and configure two FortiGate 7.0.5 VM instances in GNS3
- Simulate WAN connectivity using GNS3 Cloud nodes bridged to a Linux bridge
- Configure WAN interfaces and verify layer-3 reachability between sites
- Build a site-to-site IPsec VPN between FGT-BRANCH-A and FGT-BRANCH-B
- Apply firewall policies to control encrypted traffic
- Validate and document end-to-end connectivity

---

## Lab Architecture

```
  ┌─────────────┐                    ┌─────────────┐
  │  Cloud-WAN-A│                    │  Cloud-WAN-B│
  │  (br-wan)   │                    │  (br-wan)   │
  └──────┬──────┘                    └──────┬──────┘
         │ port1                            │ port1
         │ 172.31.255.10/24                 │ 172.31.255.20/24
  ┌──────┴──────┐                    ┌──────┴──────┐
  │ FGT-BRANCH-A│                    │ FGT-BRANCH-B│
  │  FortiOS    │◄── IPsec VPN ──►   │  FortiOS    │
  │  7.0.5      │                    │  7.0.5      │
  └─────────────┘                    └─────────────┘

  Host Linux bridge: br-wan — 172.31.255.1/24
```

---

## IP Addressing

| Device       | Interface | IP Address       | Role        |
|--------------|-----------|------------------|-------------|
| Host (Linux) | br-wan    | 172.31.255.1/24  | WAN gateway |
| FGT-BRANCH-A | port1     | 172.31.255.10/24 | WAN (Site A)|
| FGT-BRANCH-B | port1     | 172.31.255.20/24 | WAN (Site B)|

---

## Progress

| # | Phase | Status |
|---|-------|--------|
| 1 | Environment Setup (GNS3 install + appliance import) | ✅ Done |
| 2 | GNS3 Topology — Cloud nodes + FortiGate instances | ✅ Done |
| 3 | FortiGate WAN interface configuration | ✅ Done |
| 4 | WAN reachability verified (ping between FortiGates) | ✅ Done |
| 5 | IPsec Site-to-Site VPN tunnel | 🔜 Next |
| 6 | Firewall policies | 🔜 Upcoming |
| 7 | Full end-to-end validation | 🔜 Upcoming |

---

## Documentation

```
docs/
├── 01-environment-setup/
│   ├── 00-prerequisites.md          — Downloads & required files
│   ├── 01-gns3-installation.md      — GNS3 install on Linux (Fedora)
│   └── 02-fortigate-appliance-import.md — Import FortiGate & IOU appliances
│
└── 02-topology-setup/
    ├── 00-gns3-topology.md          — Create topology: Cloud nodes + FortiGates
    ├── 01-fortigate-basic-config.md — WAN interface config & connectivity proof
    └── screenshots/                 — Evidence screenshots for each step
```

---

## Quick Start

1. Read [prerequisites](docs/01-environment-setup/00-prerequisites.md) and download required images.
2. Follow [GNS3 installation](docs/01-environment-setup/01-gns3-installation.md).
3. Follow [appliance import](docs/01-environment-setup/02-fortigate-appliance-import.md).
4. Build the [GNS3 topology](docs/02-topology-setup/00-gns3-topology.md).
5. Configure [FortiGate WAN interfaces](docs/02-topology-setup/01-fortigate-basic-config.md).

---

## Tech Stack

| Component | Version |
|-----------|---------|
| OS        | Fedora Linux |
| GNS3      | 2.2.61 |
| FortiOS   | 7.0.5 |
| QEMU/KVM  | Host virtualization |
| Cisco IOU | 15.2 (May 2018) — L2 switching |
