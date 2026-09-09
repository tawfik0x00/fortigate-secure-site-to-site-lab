# FGT-BRANCH-B — DHCP Server Configuration (BR2)

## Overview

**FGT-BRANCH-B** acts as the DHCP server for every VLAN in **Branch-B (BR2)**.
Each VLAN sub-interface on `port3` has an independent DHCP pool.
PC/Alpine Linux hosts receive an IP, subnet mask, gateway, and DNS automatically.
**All four VLANs** in BR2 use DHCP — no static-only server VLANs exist here.

---

## VLAN → DHCP Scope Mapping

| VLAN | Name       | Network        | Gateway      | DHCP Range                    | Method  |
|------|------------|----------------|--------------|-------------------------------|---------|
| 110  | Sales      | 10.20.10.0/24  | 10.20.10.1   | 10.20.10.100 – 10.20.10.200  | DHCP    |
| 120  | Support    | 10.20.20.0/24  | 10.20.20.1   | 10.20.20.100 – 10.20.20.200  | DHCP    |
| 130  | Guest      | 10.20.30.0/24  | 10.20.30.1   | 10.20.30.100 – 10.20.30.200  | DHCP    |
| 199  | Management | 10.20.99.0/24  | 10.20.99.1   | 10.20.99.100 – 10.20.99.200  | DHCP    |

---

## CLI Configuration

Connect to **FGT-BRANCH-B** via console or SSH, then run:

### VLAN 110 — Sales

```
config system dhcp server
    edit 1
        set status enable
        set lease-time 86400
        set dns-server1 8.8.8.8
        set dns-server2 8.8.4.4
        set interface "port3.110"
        config ip-range
            edit 1
                set start-ip 10.20.10.100
                set end-ip   10.20.10.200
            next
        end
        set default-gateway 10.20.10.1
        set netmask 255.255.255.0
    next
end
```

### VLAN 120 — Support

```
config system dhcp server
    edit 2
        set status enable
        set lease-time 86400
        set dns-server1 8.8.8.8
        set dns-server2 8.8.4.4
        set interface "port3.120"
        config ip-range
            edit 1
                set start-ip 10.20.20.100
                set end-ip   10.20.20.200
            next
        end
        set default-gateway 10.20.20.1
        set netmask 255.255.255.0
    next
end
```

### VLAN 130 — Guest

```
config system dhcp server
    edit 3
        set status enable
        set lease-time 86400
        set dns-server1 8.8.8.8
        set dns-server2 8.8.4.4
        set interface "port3.130"
        config ip-range
            edit 1
                set start-ip 10.20.30.100
                set end-ip   10.20.30.200
            next
        end
        set default-gateway 10.20.30.1
        set netmask 255.255.255.0
    next
end
```

### VLAN 199 — Management

```
config system dhcp server
    edit 4
        set status enable
        set lease-time 86400
        set dns-server1 8.8.8.8
        set dns-server2 8.8.4.4
        set interface "port3.199"
        config ip-range
            edit 1
                set start-ip 10.20.99.100
                set end-ip   10.20.99.200
            next
        end
        set default-gateway 10.20.99.1
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

| Check                               | Expected                               |
|-------------------------------------|----------------------------------------|
| DHCP scope VLAN110 Sales active     | Enabled, interface port3.110           |
| DHCP scope VLAN120 Support active   | Enabled, interface port3.120           |
| DHCP scope VLAN130 Guest active     | Enabled, interface port3.130           |
| DHCP scope VLAN199 Mgmt active      | Enabled, interface port3.199           |
| PC in VLAN110 gets 10.20.10.x/24   | Via DHCP, gateway 10.20.10.1           |
| PC in VLAN120 gets 10.20.20.x/24   | Via DHCP, gateway 10.20.20.1           |
| PC in VLAN130 gets 10.20.30.x/24   | Via DHCP, gateway 10.20.30.1           |
| PC in VLAN199 gets 10.20.99.x/24   | Via DHCP, gateway 10.20.99.1           |

---

## Next Step

Verify each host receives its IP and can reach the gateway.

→ *See: `docs/02-topology-setup/HOSTS/BRANCH-B/01-linux-host-config.md`*
