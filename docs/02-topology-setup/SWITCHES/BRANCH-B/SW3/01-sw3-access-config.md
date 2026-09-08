# SW3 — Access Switch Configuration (BR2)

## Overview

**SW3** is an access layer switch for **Branch-B (BR2)**.
It receives a trunk from **IOU2** and assigns each host-facing port to
the appropriate VLAN in access mode.

---

## Topology

```
        [ IOU2 ]
           |
         TRUNK
           |
        [ SW3 ]
       /   |   \
    V110  V120  V130
    Sales  Sup  Guest
```

### VLAN Assignment

| VLAN | Name    | Access Ports |
|------|---------|--------------|
| 110  | Sales   | Host ports   |
| 120  | Support | Host ports   |
| 130  | Guest   | Host ports   |

---

## Configuration

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

## Screenshots

> 📸 Screenshots will be added here.

<!-- Add screenshots in the screenshots/ folder -->

---

## Milestone Checklist

| Check                             | Expected                          |
|-----------------------------------|-----------------------------------|
| VLAN 110, 120, 130 active         | Active in VLAN DB                 |
| Ethernet0/0 trunk to IOU2         | Trunk mode, VLANs 110,120,130     |
| Host ports — correct access VLAN  | Access mode, correct VLAN         |
