# FGT-BRANCH-A — DHCP Server Configuration (BR1)

## Overview

**FGT-BRANCH-A** acts as the DHCP server for every VLAN in **Branch-A (BR1)**.
Each VLAN sub-interface on `port3` has an independent DHCP pool.
PC/Linux hosts receive an IP, subnet mask, gateway, and DNS automatically.
**Server VLANs (40 & 50)** use **static IPs** — no DHCP pool is needed for them.

---

## VLAN → DHCP Scope Mapping

| VLAN | Name       | Network        | Gateway      | DHCP Range                    | Method  |
|------|------------|----------------|--------------|-------------------------------|---------|
| 10   | HR         | 10.10.10.0/24  | 10.10.10.1   | 10.10.10.100 – 10.10.10.200  | DHCP    |
| 20   | IT         | 10.10.20.0/24  | 10.10.20.1   | 10.10.20.100 – 10.10.20.200  | DHCP    |
| 30   | Finance    | 10.10.30.0/24  | 10.10.30.1   | 10.10.30.100 – 10.10.30.200  | DHCP    |
| 99   | Management | 10.10.99.0/24  | 10.10.99.1   | 10.10.99.100 – 10.10.99.200  | DHCP    |
| 40   | Server_1   | 10.10.40.0/24  | 10.10.40.1   | —                             | Static  |
| 50   | Server_2   | 10.10.50.0/24  | 10.10.50.1   | —                             | Static  |

---

## CLI Configuration

Connect to **FGT-BRANCH-A** via console or SSH, then run:

### VLAN 10 — HR

```
config system dhcp server
    edit 1
        set status enable
        set lease-time 86400
        set dns-server1 8.8.8.8
        set dns-server2 8.8.4.4
        set interface "port3.10"
        config ip-range
            edit 1
                set start-ip 10.10.10.100
                set end-ip   10.10.10.200
            next
        end
        set default-gateway 10.10.10.1
        set netmask 255.255.255.0
    next
end
```

### VLAN 20 — IT

```
config system dhcp server
    edit 2
        set status enable
        set lease-time 86400
        set dns-server1 8.8.8.8
        set dns-server2 8.8.4.4
        set interface "port3.20"
        config ip-range
            edit 1
                set start-ip 10.10.20.100
                set end-ip   10.10.20.200
            next
        end
        set default-gateway 10.10.20.1
        set netmask 255.255.255.0
    next
end
```

### VLAN 30 — Finance

```
config system dhcp server
    edit 3
        set status enable
        set lease-time 86400
        set dns-server1 8.8.8.8
        set dns-server2 8.8.4.4
        set interface "port3.30"
        config ip-range
            edit 1
                set start-ip 10.10.30.100
                set end-ip   10.10.30.200
            next
        end
        set default-gateway 10.10.30.1
        set netmask 255.255.255.0
    next
end
```

### VLAN 99 — Management

```
config system dhcp server
    edit 4
        set status enable
        set lease-time 86400
        set dns-server1 8.8.8.8
        set dns-server2 8.8.4.4
        set interface "port3.99"
        config ip-range
            edit 1
                set start-ip 10.10.99.100
                set end-ip   10.10.99.200
            next
        end
        set default-gateway 10.10.99.1
        set netmask 255.255.255.0
    next
end
```

---

## Verify DHCP Leases

```
show system dhcp server
diagnose test application dhcpd 3
get system dhcp status
```

Expected: each scope shows **enabled** with correct interface and IP range.

---

## Screenshots

> 📸 Screenshots will be added here.

<!-- Add screenshots in a screenshots/ folder -->

---

## Milestone Checklist

| Check                              | Expected                               |
|------------------------------------|----------------------------------------|
| DHCP scope VLAN10 HR active        | Enabled, interface port3.10            |
| DHCP scope VLAN20 IT active        | Enabled, interface port3.20            |
| DHCP scope VLAN30 Finance active   | Enabled, interface port3.30            |
| DHCP scope VLAN99 Mgmt active      | Enabled, interface port3.99            |
| PC in VLAN10 gets 10.10.10.x/24   | Via DHCP, gateway 10.10.10.1           |
| PC in VLAN20 gets 10.10.20.x/24   | Via DHCP, gateway 10.10.20.1           |
| PC in VLAN30 gets 10.10.30.x/24   | Via DHCP, gateway 10.10.30.1           |
| PC in VLAN99 gets 10.10.99.x/24   | Via DHCP, gateway 10.10.99.1           |
| Server_1 (VLAN40) — static IP     | 10.10.40.x/24, gateway 10.10.40.1     |
| Server_2 (VLAN50) — static IP     | 10.10.50.x/24, gateway 10.10.50.1     |

---

## Next Step

Verify each host receives its IP and can reach the gateway.

→ *See: `docs/02-topology-setup/HOSTS/BRANCH-A/01-linux-host-config.md`*
