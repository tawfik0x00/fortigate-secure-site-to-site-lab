# Branch-A — VLAN Inter-Routing Design

## Overview

This document describes how traffic flows **between VLANs** on **FGT-BRANCH-A**
and explains the routing and policy design that enforces segmentation.

FortiGate acts as the **Layer 3 gateway** for all Branch A VLANs via VLAN
subinterfaces on `port3`. Traffic between VLANs always traverses the FortiGate
and is subject to firewall policy evaluation.

---

## VLAN Interface Reference

| Interface    | VLAN | Subnet          | Gateway IP   |
|--------------|------|-----------------|--------------|
| `BRA-HR`     | 10   | 10.10.10.0/24   | 10.10.10.1   |
| `BRA-IT`     | 20   | 10.10.20.0/24   | 10.10.20.1   |
| `BRA-FINANCE`| 30   | 10.10.30.0/24   | 10.10.30.1   |
| `BRA-SRV1`   | 40   | 10.10.40.0/24   | 10.10.40.1   |
| `BRA-SRV2`   | 50   | 10.10.50.0/24   | 10.10.50.1   |
| `BRA-MGMT`   | 99   | 10.10.99.0/24   | 10.10.99.1   |

---

## Traffic Flow

Because FortiGate is the router for all VLANs, inter-VLAN traffic flows as:

```
Host (VLAN 10)
      │
      ▼
BRA-HR (10.10.10.1)
      │
   [FortiGate evaluates policy]
      │
BRA-SRV1 (10.10.40.1)
      │
      ▼
Server 1 (10.10.40.x)
```

The FortiGate does **not** require separate static routes for connected subnets.
Each VLAN interface is a directly connected network.

---

## Access Control Summary

### ✅ Allowed

| Source         | Destination  | Services              |
|----------------|--------------|-----------------------|
| HR (10/24)     | Server 1     | HTTPS, PING           |
| HR (10/24)     | Server 2     | HTTPS, PING           |
| IT (20/24)     | Server 1     | SSH, HTTPS, PING      |
| IT (20/24)     | Server 2     | SSH, HTTPS, PING      |
| Finance (30/24)| Server 1     | HTTPS, PING           |
| Finance (30/24)| Server 2     | HTTPS, PING           |
| Mgmt (99/24)   | Server 1     | SSH, HTTPS, PING      |
| Mgmt (99/24)   | Server 2     | SSH, HTTPS, PING      |

### ❌ Denied (Implicit Deny)

| Source     | Destination | Reason                            |
|------------|-------------|-----------------------------------|
| HR         | IT          | No policy — implicit deny applies |
| HR         | Finance     | No policy — implicit deny applies |
| IT         | HR          | No policy — implicit deny applies |
| IT         | Finance     | No policy — implicit deny applies |
| Finance    | HR          | No policy — implicit deny applies |
| Finance    | IT          | No policy — implicit deny applies |
| Any user   | Management  | No policy — implicit deny applies |

---

## NAT Design Note

NAT is **disabled** on all internal VLAN-to-VLAN policies.

```
Correct:
10.10.10.10  ──────►  10.10.40.10
(HR host)              (Server 1)
```

```
Wrong (would break internal routing and future IPsec VPN):
10.10.10.10  ──NAT──►  FortiGate IP  ──►  10.10.40.10
```

NAT is only enabled on policies where traffic exits to the **Internet** via `port2`.

---

## Connectivity Test Commands

### From an HR host — test server access

```sh
# Should succeed — policy allows HTTPS
ping -c 4 10.10.40.1       # Server 1 gateway

# Should succeed
ping -c 4 10.10.50.1       # Server 2 gateway
```

### From an HR host — test denied access

```sh
# Should fail — no policy allows HR → IT
ping -c 4 10.10.20.1

# Should fail — no policy allows HR → Finance
ping -c 4 10.10.30.1

# Should fail — no policy allows users → Management
ping -c 4 10.10.99.1
```

### From an IT host — test admin access

```sh
# Should succeed — A-ADMIN allows SSH + HTTPS + PING
ping -c 4 10.10.40.1
ssh admin@10.10.40.10
```

---

## Verification Commands

```sh
# View all firewall policies
show firewall policy

# View routing table (connected subnets should all appear)
get router info routing-table all

# Check active sessions
diagnose sys session list

# Debug a specific traffic flow
diagnose debug reset
diagnose debug flow filter clear
diagnose debug flow filter addr 10.10.10.10
diagnose debug flow show console enable
diagnose debug enable
diagnose debug flow trace start 20

# Stop debugging
diagnose debug disable
diagnose debug flow trace stop
diagnose debug reset
```

---

## Screenshots

> 📸 Screenshots will be added here.

---

## Milestone Checklist

| Check                                      | Expected             |
|--------------------------------------------|----------------------|
| HR can ping Server 1 gateway               | 0% packet loss       |
| HR can ping Server 2 gateway               | 0% packet loss       |
| IT can ping Server 1 gateway               | 0% packet loss       |
| IT can SSH to Server 1                     | Connection accepted  |
| Finance can ping Server 1 gateway          | 0% packet loss       |
| Management can SSH to Server 1             | Connection accepted  |
| HR cannot ping IT gateway (10.10.20.1)     | 100% packet loss     |
| HR cannot ping Finance gateway (10.10.30.1)| 100% packet loss     |
| HR cannot ping Management (10.10.99.1)     | 100% packet loss     |

---

## Next Step

Once Branch A internal policies are verified, proceed to configure the IPsec VPN.

→ *See: [`docs/02-topology-setup/IPSEC-VPN/01-ipsec-phase1-config.md`](../../02-topology-setup/IPSEC-VPN/01-ipsec-phase1-config.md)*
