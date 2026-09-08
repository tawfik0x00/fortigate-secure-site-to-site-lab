# SW4 — Access Switch Configuration (BR2)

## Overview

**SW4** is an access layer switch for **Branch-B (BR2)**.
It receives a trunk from **IOU2** and assigns each host-facing port to
the appropriate VLAN in access mode.

---

## Topology

```
        [ IOU2 ]
           |
         TRUNK
           |
        [ SW4 ]
           |
          V199
          Mgmt
```

### VLAN Assignment

| VLAN | Name       | Access Ports |
|------|------------|--------------|
| 199  | Management | Host ports   |

---

## Configuration

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

## Screenshots

> 📸 Screenshots will be added here.

<!-- Add screenshots in the screenshots/ folder -->

---

## Milestone Checklist

| Check                            | Expected               |
|----------------------------------|------------------------|
| VLAN 199 active                  | Active in VLAN DB      |
| Ethernet0/0 trunk to IOU2        | Trunk mode, VLAN 199   |
| Host ports — correct access VLAN | Access mode, VLAN 199  |
