# SW1 — Access Switch Configuration (BR1)

## Overview

**SW1** is an access layer switch for **Branch-A (BR1)**.
It receives a trunk from **IOU1** and assigns each host-facing port to
the appropriate VLAN in access mode.

---

## Topology

```
        [ IOU1 ]
           |
         TRUNK
           |
        [ SW1 ]
       /   |   \   \
     V10  V20  V30  V99
     HR   IT  Finance Mgmt
```

### VLAN Assignment

| VLAN | Name       | Access Ports   |
|------|------------|----------------|
| 10   | HR         | Host ports     |
| 20   | IT         | Host ports     |
| 30   | Finance    | Host ports     |
| 99   | Management | Host ports     |

---

## Configuration

### Step 1 — Create VLANs on SW1

```
enable
configure terminal

vlan 10
 name HR
exit

vlan 20
 name IT
exit

vlan 30
 name FINANCE
exit

vlan 99
 name MANAGEMENT
exit

end
write memory
```

### Step 2 — Uplink Trunk to IOU1

```
configure terminal

interface Ethernet0/0
 description TRUNK_TO_IOU1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,99
 no shutdown
exit

end
write memory
```

### Step 3 — Access Ports for Hosts

Assign each host-connected port to its VLAN:

```
configure terminal

! HR hosts (VLAN 10)
interface Ethernet0/1
 description HOST_HR
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

interface Ethernet0/2
 description HOST_HR
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

! IT hosts (VLAN 20)
interface Ethernet0/3
 description HOST_IT
 switchport mode access
 switchport access vlan 20
 no shutdown
exit

interface Ethernet1/0
 description HOST_IT
 switchport mode access
 switchport access vlan 20
 no shutdown
exit

! Finance hosts (VLAN 30)
interface Ethernet1/1
 description HOST_FINANCE
 switchport mode access
 switchport access vlan 30
 no shutdown
exit

interface Ethernet1/2
 description HOST_FINANCE
 switchport mode access
 switchport access vlan 30
 no shutdown
exit

! Management hosts (VLAN 99)
interface Ethernet1/3
 description HOST_MGMT
 switchport mode access
 switchport access vlan 99
 no shutdown
exit

interface Ethernet2/0
 description HOST_MGMT
 switchport mode access
 switchport access vlan 99
 no shutdown
exit

end
write memory
```

Verify:

```
show vlan brief
show interfaces status
show interfaces trunk
```

## Milestone Checklist

| Check                             | Expected                          |
|-----------------------------------|-----------------------------------|
| VLAN 10, 20, 30, 99 active        | Active in VLAN DB                 |
| Ethernet0/0 trunk to IOU1         | Trunk mode, VLANs 10,20,30,99    |
| Host ports — correct access VLAN  | Access mode, correct VLAN         |
