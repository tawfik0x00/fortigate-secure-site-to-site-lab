# VLAN Setup — FGT-BRANCH-B (BR2) on port3

## Overview

`port3` on **FGT-BRANCH-B** is the **trunk / LAN-facing** interface.  
It connects to **SW3** (the access switch) and carries all four branch VLANs as
802.1Q sub-interfaces (VLAN interfaces in FortiOS terminology).

---

## VLAN Addressing Plan — BR2

| VLAN ID | Name       | Network          | FortiGate Gateway |
|---------|------------|------------------|-------------------|
| 110     | Sales      | 10.20.10.0/24    | 10.20.10.1        |
| 120     | Support    | 10.20.20.0/24    | 10.20.20.1        |
| 130     | Guest      | 10.20.30.0/24    | 10.20.30.1        |
| 199     | Management | 10.20.99.0/24    | 10.20.99.1        |

---

## How FortiOS Implements VLANs on a Physical Interface

In FortiOS, 802.1Q VLANs are created as **VLAN sub-interfaces** parented to a
physical interface (`port3` here). Each sub-interface:

- Inherits the physical link from `port3`
- Carries a specific VLAN ID tag
- Has its own IP address (the gateway for that VLAN)
- Can have independent firewall policies and routing

```
port3 (trunk — no IP, carries all tags)
  ├── BR2-Sales    VLAN 110  Sales      10.20.10.1/24
  ├── BR2-Support  VLAN 120  Support    10.20.20.1/24
  ├── BR2-Guest    VLAN 130  Guest      10.20.30.1/24
  └── BR2-MGMT     VLAN 199  Management 10.20.99.1/24
```

---

## Step 1 — Bring Up port3 (No IP — Trunk Mode)

Before creating sub-interfaces, make sure `port3` itself has no IP assigned
and is up:

```
config system interface
    edit "port3"
        set mode static
        set ip 0.0.0.0 0.0.0.0
        set allowaccess ping
        set description "LAN-TRUNK-BRB"
    next
end
```

---

## Step 2 — Create VLAN Sub-Interfaces on port3

Paste the entire block into the FGT-BRANCH-B console:

```
config system interface

    edit "BRB-Sales"
        set vdom "root"
        set type vlan
        set vlanid 110
        set interface "port3"
        set mode static
        set ip 10.20.10.1 255.255.255.0
        set allowaccess ping
        set description "VLAN110 - Sales"
    next

    edit "BRB-Support"
        set vdom "root"
        set type vlan
        set vlanid 120
        set interface "port3"
        set mode static
        set ip 10.20.20.1 255.255.255.0
        set allowaccess ping
        set description "VLAN120 - Support"
    next

    edit "BRB-Guest"
        set vdom "root"
        set type vlan
        set vlanid 130
        set interface "port3"
        set mode static
        set ip 10.20.30.1 255.255.255.0
        set allowaccess ping
        set description "VLAN130 - Guest"
    next

    edit "BRB-MGMT"
        set vdom "root"
        set type vlan
        set vlanid 199
        set interface "port3"
        set mode static
        set ip 10.20.99.1 255.255.255.0
        set allowaccess ping http https ssh
        set description "VLAN199 - Management"
    next

end
```

> **Note:** `allowaccess ping http https ssh` is set on **VLAN199 (Management)**
> so that FortiGate can be reached over that VLAN for GUI / SSH access.
> User-facing VLANs only need `ping` for ICMP reachability checks.

---

## Step 3 — Verify the VLAN Interfaces

```
show system interface | grep -A 10 "BRB-"
```

Or use the full interface listing:

```
get system interface
```

Expected output (condensed):

```
== [ BR2-Sales ]
name: BR2-Sales   mode: static   ip: 10.20.10.1 255.255.255.0   status: up   vlan-id: 110
== [ BR2-Support ]
name: BR2-Support   mode: static   ip: 10.20.20.1 255.255.255.0   status: up   vlan-id: 120
== [ BR2-Guest ]
name: BR2-Guest   mode: static   ip: 10.20.30.1 255.255.255.0   status: up   vlan-id: 130
== [ BR2-MGMT ]
name: BR2-MGMT   mode: static   ip: 10.20.99.1 255.255.255.0   status: up   vlan-id: 199
```

---

## Milestone Checklist — BR2 VLANs

| Check                              | Expected Value         |
|------------------------------------|------------------------|
| BR2-Sales (VLAN 110) IP           | `10.20.10.1/24`        |
| BR2-Support (VLAN 120) IP         | `10.20.20.1/24`        |
| BR2-Guest (VLAN 130) IP           | `10.20.30.1/24`        |
| BR2-MGMT (VLAN 199) IP            | `10.20.99.1/24`        |
| All sub-interfaces parent = port3 | `port3`                |

---

## Next Step

Configure the corresponding trunk on **SW3** (Cisco IOU) to allow all VLANs
and set each access port to the correct VLAN ID.

→ *See: `docs/02-topology-setup/IOU-SWITCH/02-sw3-trunk-vlan-config.md`*
