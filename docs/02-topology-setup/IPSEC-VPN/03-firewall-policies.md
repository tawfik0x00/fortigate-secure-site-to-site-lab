# IPsec VPN — Firewall Policies (Inter-Site VLAN Access)

## Overview

Firewall policies control **which VLANs can communicate through the IPsec tunnel**.
Even though the tunnel is up, all traffic is **denied by default** until explicit
`ACCEPT` policies are created.

This document implements the initial policy:
> **Branch-B Sales (VLAN 110)** can reach **Branch-A HR (VLAN 10)** — and vice versa.

Additional per-VLAN pairs can be added using the same pattern below.

---

## Policy Architecture

```
Branch-B Sales (10.20.10.0/24)
        │
        ▼  [tunnel interface SITE-B-TO-A]
  FGT-BRANCH-B
        │
   IPsec Tunnel
        │
  FGT-BRANCH-A
        │
        ▼  [port3.10 — VLAN10 HR]
Branch-A HR (10.10.10.0/24)
```

Policies are required **in both directions** on both FortiGates:
- FGT-A: `port3.10 → SITE-A-TO-B` (outbound from HR to tunnel)
- FGT-A: `SITE-A-TO-B → port3.10` (inbound from tunnel to HR)
- FGT-B: `port3.110 → SITE-B-TO-A` (outbound from Sales to tunnel)
- FGT-B: `SITE-B-TO-A → port3.110` (inbound from tunnel to Sales)

---

## Address Objects

### FGT-BRANCH-A

```
config firewall address
    edit "BRA-VLAN10-HR"
        set subnet 10.10.10.0 255.255.255.0
    next
    edit "BRB-VLAN110-SALES"
        set subnet 10.20.10.0 255.255.255.0
    next
end
```

### FGT-BRANCH-B

```
config firewall address
    edit "BRB-VLAN110-SALES"
        set subnet 10.20.10.0 255.255.255.0
    next
    edit "BRA-VLAN10-HR"
        set subnet 10.10.10.0 255.255.255.0
    next
end
```

---

## Firewall Policies — FGT-BRANCH-A

### Policy 1: HR → Tunnel (outbound to Branch-B Sales)

```
config firewall policy
    edit 10
        set name "HR-to-BRB-Sales"
        set srcintf "port3.10"
        set dstintf "SITE-A-TO-B"
        set srcaddr "BRA-VLAN10-HR"
        set dstaddr "BRB-VLAN110-SALES"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

### Policy 2: Tunnel → HR (inbound from Branch-B Sales)

```
config firewall policy
    edit 11
        set name "BRB-Sales-to-HR"
        set srcintf "SITE-A-TO-B"
        set dstintf "port3.10"
        set srcaddr "BRB-VLAN110-SALES"
        set dstaddr "BRA-VLAN10-HR"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

---

## Firewall Policies — FGT-BRANCH-B

### Policy 1: Sales → Tunnel (outbound to Branch-A HR)

```
config firewall policy
    edit 10
        set name "Sales-to-BRA-HR"
        set srcintf "port3.110"
        set dstintf "SITE-B-TO-A"
        set srcaddr "BRB-VLAN110-SALES"
        set dstaddr "BRA-VLAN10-HR"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

### Policy 2: Tunnel → Sales (inbound from Branch-A HR)

```
config firewall policy
    edit 11
        set name "BRA-HR-to-Sales"
        set srcintf "SITE-B-TO-A"
        set dstintf "port3.110"
        set srcaddr "BRA-VLAN10-HR"
        set dstaddr "BRB-VLAN110-SALES"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

---

## End-to-End Test

From an **AlpineLinux1 (Sales, VLAN110)** host in Branch-B:

```sh
# Ping a Linux host in Branch-A VLAN10 (HR)
ping -c 4 <Linux-1-IP>     # e.g. 10.10.10.101

# From Branch-A Linux-1 (HR) ping back
ping -c 4 <AlpineLinux1-IP>  # e.g. 10.20.10.101
```

Expected: **0% packet loss** in both directions.

---

## Verify Policies & Traffic

On either FortiGate:

```
get firewall policy
diagnose firewall iprope list 100004
```

Check logs:
```
execute log filter category 1
execute log display
```

---

## Screenshots

> 📸 Screenshots will be added here.

---

## Milestone Checklist

| Check                                             | Expected                         |
|---------------------------------------------------|----------------------------------|
| Address object `BRA-VLAN10-HR` on FGT-A          | 10.10.10.0/24                    |
| Address object `BRB-VLAN110-SALES` on FGT-A      | 10.20.10.0/24                    |
| Policy HR→Tunnel on FGT-A                        | Accept, logtraffic all           |
| Policy Tunnel→HR on FGT-A                        | Accept, logtraffic all           |
| Address objects mirrored on FGT-B                 | Same subnets, opposite roles     |
| Policy Sales→Tunnel on FGT-B                     | Accept, logtraffic all           |
| Policy Tunnel→Sales on FGT-B                     | Accept, logtraffic all           |
| Ping: Sales host → HR host                       | 0% packet loss                   |
| Ping: HR host → Sales host                       | 0% packet loss                   |
| Traffic visible in FortiGate logs                 | Entries for both directions      |

---

## Extending to More VLANs

To allow additional VLAN pairs, repeat the address object and policy pattern
above, incrementing policy IDs for each new pair.

Example pairs to add in future iterations:
- IT (VLAN20) ↔ Support (VLAN120)
- Finance (VLAN30) ↔ Guest (VLAN130)
- Management (VLAN99) ↔ Management (VLAN199)
