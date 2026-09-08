# SW3 & SW4 — Access Switch Configuration (BR2)

## Overview

**SW3** and **SW4** are the access layer switches for **Branch-B (BR2)**.
They receive a trunk from **IOU2** and assign each host-facing port to
the appropriate VLAN in access mode.

---

## Topology

```
          [ IOU2 ]
          /       \
    TRUNK           TRUNK
      /               \
   [ SW3 ]          [ SW4 ]
  /   |   \              |
V110 V120 V130          V199
Sales Sup  Guest        Mgmt
```

### VLAN-to-Switch Assignment

| VLAN | Name       | Switch | Access Ports   |
|------|------------|--------|----------------|
| 110  | Sales      | SW3    | Host ports     |
| 120  | Support    | SW3    | Host ports     |
| 130  | Guest      | SW3    | Host ports     |
| 199  | Management | SW4    | Host ports     |

---

## SW3 Configuration

### Step 1 — Create VLANs on SW3

```
enable
configure terminal

vlan 110
 name SALES
exit

vlan 120
 name SUPPORT
exit

vlan 130
 name GUEST
exit

end
write memory
```

### Step 2 — Uplink Trunk to IOU2

```
configure terminal

interface Ethernet0/0
 description TRUNK_TO_IOU2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 110,120,130
 no shutdown
exit

end
write memory
```

### Step 3 — Access Ports for Hosts

```
configure terminal

! Sales hosts (VLAN 110)
interface Ethernet0/1
 description HOST_SALES
 switchport mode access
 switchport access vlan 110
 no shutdown
exit

interface Ethernet0/2
 description HOST_SALES
 switchport mode access
 switchport access vlan 110
 no shutdown
exit

! Support hosts (VLAN 120)
interface Ethernet0/3
 description HOST_SUPPORT
 switchport mode access
 switchport access vlan 120
 no shutdown
exit

interface Ethernet1/0
 description HOST_SUPPORT
 switchport mode access
 switchport access vlan 120
 no shutdown
exit

! Guest hosts (VLAN 130)
interface Ethernet1/1
 description HOST_GUEST
 switchport mode access
 switchport access vlan 130
 no shutdown
exit

interface Ethernet1/2
 description HOST_GUEST
 switchport mode access
 switchport access vlan 130
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

---

## SW4 Configuration

### Step 1 — Create VLANs on SW4

```
enable
configure terminal

vlan 199
 name MANAGEMENT
exit

end
write memory
```

### Step 2 — Uplink Trunk to IOU2

```
configure terminal

interface Ethernet0/0
 description TRUNK_TO_IOU2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 199
 no shutdown
exit

end
write memory
```

### Step 3 — Access Ports for Hosts

```
configure terminal

! Management hosts (VLAN 199)
interface Ethernet0/1
 description HOST_MGMT
 switchport mode access
 switchport access vlan 199
 no shutdown
exit

interface Ethernet0/2
 description HOST_MGMT
 switchport mode access
 switchport access vlan 199
 no shutdown
exit

end
write memory
```

Verify:

```
show vlan brief
show interfaces status
```

---

## Milestone Checklist — SW3 / SW4

| Check                                | Expected                           |
|--------------------------------------|------------------------------------|
| SW3 — VLAN 110, 120, 130 active      | Active in VLAN DB                  |
| SW4 — VLAN 199 active                | Active in VLAN DB                  |
| SW3 Ethernet0/0 trunk to IOU2        | Trunk mode, VLANs 110,120,130      |
| SW4 Ethernet0/0 trunk to IOU2        | Trunk mode, VLAN 199               |
| Host ports on SW3                    | Access mode, correct VLAN          |
| Host ports on SW4                    | Access mode, VLAN 199              |
