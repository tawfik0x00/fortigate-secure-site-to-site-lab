# SW1 & SW2 — Access Switch Configuration (BR1)

## Overview

**SW1** and **SW2** are the access layer switches for **Branch-A (BR1)**.
They receive a trunk from **IOU1** and assign each host-facing port to
the appropriate VLAN in access mode.

---

## Topology

```
          [ IOU1 ]
          /       \
    TRUNK           TRUNK
      /               \
   [ SW1 ]          [ SW2 ]
   /  |  \           /    \
 V10 V20 V30       V40    V50
 HR  IT  Finance  SRV1   SRV2

 VLAN 99 (Management) trunk on both SW1 and SW2
```

### VLAN-to-Switch Assignment

| VLAN | Name       | Switch | Access Ports   |
|------|------------|--------|----------------|
| 10   | HR         | SW1    | Host ports     |
| 20   | IT         | SW1    | Host ports     |
| 30   | Finance    | SW1    | Host ports     |
| 99   | Management | SW1    | Host ports     |
| 40   | Server_1   | SW2    | Host ports     |
| 50   | Server_2   | SW2    | Host ports     |

---

## SW1 Configuration

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

---

## SW2 Configuration

### Step 1 — Create VLANs on SW2

```
enable
configure terminal

vlan 40
 name SERVER_1
exit

vlan 50
 name SERVER_2
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
 switchport trunk allowed vlan 40,50
 no shutdown
exit

end
write memory
```

### Step 3 — Access Ports for Hosts

```
configure terminal

! Server_1 hosts (VLAN 40)
interface Ethernet0/1
 description HOST_SERVER1
 switchport mode access
 switchport access vlan 40
 no shutdown
exit

! Server_2 hosts (VLAN 50)
interface Ethernet0/2
 description HOST_SERVER2
 switchport mode access
 switchport access vlan 50
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

## Milestone Checklist — SW1 / SW2

| Check                              | Expected                |
|------------------------------------|-------------------------|
| SW1 — VLAN 10, 20, 30, 99 active  | Active in VLAN DB       |
| SW2 — VLAN 40, 50 active          | Active in VLAN DB       |
| SW1 Ethernet0/0 trunk to IOU1     | Trunk mode, VLANs 10,20,30,99 |
| SW2 Ethernet0/0 trunk to IOU1     | Trunk mode, VLANs 40,50 |
| Host ports on SW1                  | Access mode, correct VLAN |
| Host ports on SW2                  | Access mode, correct VLAN |
