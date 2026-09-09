# VLAN Setup — FGT-BRANCH-A (BR1) on port3

## Overview

`port3` on **FGT-BRANCH-A** is the **trunk / LAN-facing** interface.  
It connects to **SW1** (the access switch) and carries all six branch VLANs as
802.1Q sub-interfaces (VLAN interfaces in FortiOS terminology).

---

## VLAN Addressing Plan — BR1

| VLAN ID | Name       | Network          | FortiGate Gateway |
|---------|------------|------------------|-------------------|
| 10      | HR         | 10.10.10.0/24    | 10.10.10.1        |
| 20      | IT         | 10.10.20.0/24    | 10.10.20.1        |
| 30      | Finance    | 10.10.30.0/24    | 10.10.30.1        |
| 99      | Management | 10.10.99.0/24    | 10.10.99.1        |
| 40      | Server_1   | 10.10.40.0/24    | 10.10.40.1        |
| 50      | Server_2   | 10.10.50.0/24    | 10.10.50.1        |

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
  ├── BR1-HR      VLAN 10  HR         10.10.10.1/24
  ├── BR1-IT      VLAN 20  IT         10.10.20.1/24
  ├── BR1-Finance VLAN 30  Finance    10.10.30.1/24
  ├── BR1-MGMT    VLAN 99  Management 10.10.99.1/24
  ├── BR1-SRV1    VLAN 40  Server_1   10.10.40.1/24
  └── BR1-SRV2    VLAN 50  Server_2   10.10.50.1/24
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
        set description "LAN-TRUNK-BRA"
    next
end
```

---

## Step 2 — Create VLAN Sub-Interfaces on port3

Paste the entire block into the FGT-BRANCH-A console:

```
config system interface

    edit "BR1-HR"
        set vdom "root"
        set type vlan
        set vlanid 10
        set interface "port3"
        set mode static
        set ip 10.10.10.1 255.255.255.0
        set allowaccess ping
        set description "VLAN10 - HR"
    next

    edit "BR1-IT"
        set vdom "root"
        set type vlan
        set vlanid 20
        set interface "port3"
        set mode static
        set ip 10.10.20.1 255.255.255.0
        set allowaccess ping
        set description "VLAN20 - IT"
    next

    edit "BR1-Finance"
        set vdom "root"
        set type vlan
        set vlanid 30
        set interface "port3"
        set mode static
        set ip 10.10.30.1 255.255.255.0
        set allowaccess ping
        set description "VLAN30 - Finance"
    next

    edit "BR1-MGMT"
        set vdom "root"
        set type vlan
        set vlanid 99
        set interface "port3"
        set mode static
        set ip 10.10.99.1 255.255.255.0
        set allowaccess ping http https ssh
        set description "VLAN99 - Management"
    next

    edit "BR1-SRV1"
        set vdom "root"
        set type vlan
        set vlanid 40
        set interface "port3"
        set mode static
        set ip 10.10.40.1 255.255.255.0
        set allowaccess ping
        set description "VLAN40 - Server_1"
    next

    edit "BR1-SRV2"
        set vdom "root"
        set type vlan
        set vlanid 50
        set interface "port3"
        set mode static
        set ip 10.10.50.1 255.255.255.0
        set allowaccess ping
        set description "VLAN50 - Server_2"
    next

end
```

> **Note:** `allowaccess ping http https ssh` is set on **VLAN99 (Management)**
> so that FortiGate can be reached over that VLAN for GUI / SSH access.
> User-facing VLANs only need `ping` for ICMP reachability checks.

---

## Step 3 — Verify the VLAN Interfaces

```
show system interface | grep -A 10 "BR1-"
```

Or use the full interface listing:

```
get system interface
```

Expected output (condensed):

```
== [ BR1-HR ]
name: BR1-HR   mode: static   ip: 10.10.10.1 255.255.255.0   status: up   vlan-id: 10
== [ BR1-IT ]
name: BR1-IT   mode: static   ip: 10.10.20.1 255.255.255.0   status: up   vlan-id: 20
== [ BR1-Finance ]
name: BR1-Finance   mode: static   ip: 10.10.30.1 255.255.255.0   status: up   vlan-id: 30
== [ BR1-MGMT ]
name: BR1-MGMT   mode: static   ip: 10.10.99.1 255.255.255.0   status: up   vlan-id: 99
== [ BR1-SRV1 ]
name: BR1-SRV1   mode: static   ip: 10.10.40.1 255.255.255.0   status: up   vlan-id: 40
== [ BR1-SRV2 ]
name: BR1-SRV2   mode: static   ip: 10.10.50.1 255.255.255.0   status: up   vlan-id: 50
```

---

## Milestone Checklist — BR1 VLANs

| Check                              | Expected Value         |
|------------------------------------|------------------------|
| BR1-HR (VLAN 10) IP               | `10.10.10.1/24`        |
| BR1-IT (VLAN 20) IP               | `10.10.20.1/24`        |
| BR1-Finance (VLAN 30) IP          | `10.10.30.1/24`        |
| BR1-MGMT (VLAN 99) IP             | `10.10.99.1/24`        |
| BR1-SRV1 (VLAN 40) IP            | `10.10.40.1/24`        |
| BR1-SRV2 (VLAN 50) IP            | `10.10.50.1/24`        |
| All sub-interfaces parent = port3 | `port3`                |

---

## Next Step

Configure the corresponding trunk on **SW1** (Cisco IOU) to allow all VLANs
and set each access port to the correct VLAN ID.

→ *See: `docs/02-topology-setup/IOU-SWITCH/01-sw1-trunk-vlan-config.md`*
