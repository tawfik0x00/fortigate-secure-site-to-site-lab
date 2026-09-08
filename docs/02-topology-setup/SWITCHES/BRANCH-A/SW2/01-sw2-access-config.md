# SW2 — Access Switch Configuration (BR1)

## Overview

**SW2** is an access layer switch for **Branch-A (BR1)**.
It receives a trunk from **IOU1** and assigns each host-facing port to
the appropriate VLAN in access mode.

---

## Topology

```
        [ IOU1 ]
           |
         TRUNK
           |
        [ SW2 ]
        /      \
      V40      V50
    Server_1  Server_2
```

### VLAN Assignment

| VLAN | Name     | Access Ports |
|------|----------|--------------|
| 40   | Server_1 | Host ports   |
| 50   | Server_2 | Host ports   |

---

## Configuration

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

## Screenshots

> 📸 Screenshots will be added here.

<!-- Add screenshots in the screenshots/ folder -->

---

## Milestone Checklist

| Check                            | Expected                    |
|----------------------------------|-----------------------------|
| VLAN 40, 50 active               | Active in VLAN DB           |
| Ethernet0/0 trunk to IOU1        | Trunk mode, VLANs 40,50     |
| Host ports — correct access VLAN | Access mode, correct VLAN   |
