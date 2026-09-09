# IPsec Site-to-Site VPN — Phase 2 (IPsec SA) Configuration

## Overview

**Phase 2** defines the **traffic selectors** — which subnets are allowed to
communicate through the IPsec tunnel. This is where you specify which
Branch-A VLAN(s) can reach which Branch-B VLAN(s).

Phase 2 depends on Phase 1 being established first.

→ *See: `docs/02-topology-setup/IPSEC-VPN/01-ipsec-phase1-config.md`*

---

## Traffic Selector Strategy

For the initial tunnel, **one Phase 2 SA** is created to allow traffic between
**all Branch-A subnets** and **all Branch-B subnets**.
Granular per-VLAN rules are then enforced by **firewall policies** (Phase 3).

| Local Subnets (BR-A)    | Remote Subnets (BR-B)   |
|-------------------------|-------------------------|
| 10.10.0.0/16 (all BRA)  | 10.20.0.0/16 (all BRB)  |

> **Note:** You can also create separate Phase 2 entries per VLAN pair for
> stricter control. This document uses a single broad selector for simplicity;
> policies restrict actual traffic.

---

## Phase 2 Parameters

| Parameter         | Value                         |
|-------------------|-------------------------------|
| Phase2 Name       | `P2-A-TO-B` / `P2-B-TO-A`   |
| Linked Phase1     | `SITE-A-TO-B` / `SITE-B-TO-A`|
| Proposal          | AES-256 / SHA-256             |
| PFS Group         | Group 14                      |
| Keylife           | 43200 seconds (12 h)          |
| Local Selector    | 10.10.0.0/255.255.0.0         |
| Remote Selector   | 10.20.0.0/255.255.0.0         |

---

## FGT-BRANCH-A Configuration

```
config vpn ipsec phase2-interface
    edit "P2-A-TO-B"
        set phase1name "SITE-A-TO-B"
        set proposal aes256-sha256
        set dhgrp 14
        set pfs enable
        set keylife-type seconds
        set keylifeseconds 43200
        set src-subnet 10.10.0.0 255.255.0.0
        set dst-subnet 10.20.0.0 255.255.0.0
    next
end
```

---

## FGT-BRANCH-B Configuration

```
config vpn ipsec phase2-interface
    edit "P2-B-TO-A"
        set phase1name "SITE-B-TO-A"
        set proposal aes256-sha256
        set dhgrp 14
        set pfs enable
        set keylife-type seconds
        set keylifeseconds 43200
        set src-subnet 10.20.0.0 255.255.0.0
        set dst-subnet 10.10.0.0 255.255.0.0
    next
end
```

---

## Add Static Routes for Tunnel Traffic

Both FortiGates need static routes pointing tunnel-bound traffic to the
IPsec interface.

### FGT-BRANCH-A — route to Branch-B subnets

```
config router static
    edit 10
        set dst 10.20.0.0 255.255.0.0
        set device "SITE-A-TO-B"
    next
end
```

### FGT-BRANCH-B — route to Branch-A subnets

```
config router static
    edit 10
        set dst 10.10.0.0 255.255.0.0
        set device "SITE-B-TO-A"
    next
end
```

---

## Verify Phase 2

```
diagnose vpn tunnel list
get vpn ipsec tunnel summary
```

Expected: Phase 2 SA shows as **up** with bytes in/out incrementing when
traffic is generated.

---

## Screenshots

> 📸 Screenshots will be added here.

---

## Milestone Checklist

| Check                                      | Expected                          |
|--------------------------------------------|-----------------------------------|
| Phase2 `P2-A-TO-B` created on FGT-A       | Listed in VPN phase2 table        |
| Phase2 `P2-B-TO-A` created on FGT-B       | Listed in VPN phase2 table        |
| Static route to 10.20.0.0/16 on FGT-A     | via SITE-A-TO-B interface         |
| Static route to 10.10.0.0/16 on FGT-B     | via SITE-B-TO-A interface         |
| Tunnel SA status                           | Up / Active                       |

---

## Next Step

Create firewall policies to allow specific inter-site VLAN traffic.

→ *See: `docs/02-topology-setup/IPSEC-VPN/03-firewall-policies.md`*
