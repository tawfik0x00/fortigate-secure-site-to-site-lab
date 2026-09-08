# GNS3 Topology Setup

## Overview

This document covers the creation of the GNS3 lab topology used in the
FortiGate Secure Site-to-Site project.

The topology consists of two FortiGate virtual machines, each connected
to a separate GNS3 Cloud node. Both Cloud nodes are bridged to the same
Linux network bridge (`br-wan`) on the host, simulating a shared WAN
segment.

---

## Lab Topology Diagram

```
  ┌─────────────┐                    ┌─────────────┐
  │  Cloud-WAN-A│                    │  Cloud-WAN-B│
  │  (br-wan)   │                    │  (br-wan)   │
  └──────┬──────┘                    └──────┬──────┘
         │ port1                            │ port1
         │ 172.31.255.10/24                 │ 172.31.255.20/24
  ┌──────┴──────┐                    ┌──────┴──────┐
  │ FGT-BRANCH-A│                    │ FGT-BRANCH-B│
  │  FortiOS    │                    │  FortiOS    │
  │  7.0.5      │                    │  7.0.5      │
  └─────────────┘                    └─────────────┘

  Host Linux bridge: br-wan — 172.31.255.1/24
```

---

## Network Addressing

| Device        | Interface | IP Address       | Role        |
|---------------|-----------|------------------|-------------|
| Host (Linux)  | br-wan    | 172.31.255.1/24  | WAN gateway |
| FGT-BRANCH-A  | port1     | 172.31.255.10/24 | WAN         |
| FGT-BRANCH-B  | port1     | 172.31.255.20/24 | WAN         |

---

## Step 1 — Create the Linux Bridge on the Host

Before opening GNS3, create the `br-wan` bridge on the Linux host.
This bridge acts as the simulated WAN segment shared by both FortiGates.

```bash
# Create the bridge
sudo ip link add br-wan type bridge

# Assign the gateway IP to the bridge
sudo ip addr add 172.31.255.1/24 dev br-wan

# Bring the bridge up
sudo ip link set br-wan up
```

Verify the bridge is running:

```bash
ip addr show br-wan
```

Expected output:

```
br-wan: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.31.255.1  netmask 255.255.255.0  broadcast 172.31.255.255
```

> **Note:** This bridge is not persistent across reboots. To make it
> permanent, add a systemd-networkd unit or equivalent for your
> distribution.

---

## Step 2 — Open GNS3 and Create a New Project

1. Launch **GNS3**.
2. Go to **File → New blank project**.
3. Name it `FortiGate-Secure-Site-to-Site-Lab` and click **OK**.

---

## Step 3 — Add the FortiGate Appliances

1. In the **Device** panel on the left, expand **Security**.
2. Drag **FortiGate 7.0.5** onto the canvas **twice**.
3. Rename the first instance to `FGT-BRANCH-A` and the second to `FGT-BRANCH-B`.

> Right-click a device → **Change hostname** to rename it.

---

## Step 4 — Add Cloud Nodes

Cloud nodes connect GNS3 devices to real host interfaces (including bridges).

1. In the Device panel, click **Browse all devices** or find **Cloud** under
   the **End devices** section.
2. Drag a **Cloud** node onto the canvas twice.
3. Rename them `Cloud-WAN-A` and `Cloud-WAN-B`.

---

## Step 5 — Configure the Cloud Nodes to Use `br-wan`

Each Cloud node must be configured to bridge to the `br-wan` interface.

1. **Right-click** `Cloud-WAN-A` → **Configure**.
2. In the **Ethernet interfaces** tab, select **`br-wan`** from the
   dropdown list.
3. Click **Add** → **OK**.

   > 📸 **Screenshot:** `screenshots/01-cloud-wan-a-config.png`
   > *Show the Cloud node properties dialog with `br-wan` selected and added.*

4. Repeat the same steps for `Cloud-WAN-B`.

   > 📸 **Screenshot:** `screenshots/02-cloud-wan-b-config.png`
   > *Show Cloud-WAN-B configured with `br-wan`.*

---

## Step 6 — Connect Devices

Draw the cables to connect the topology:

| From          | Interface | To           | Interface |
|---------------|-----------|--------------|-----------|
| FGT-BRANCH-A  | port1     | Cloud-WAN-A  | br-wan    |
| FGT-BRANCH-B  | port1     | Cloud-WAN-B  | br-wan    |

To draw a cable:
1. Click the **cable icon** (or press **C**) in the toolbar.
2. Click the source device, choose the interface, then click the
   destination device and choose its interface.

---

## Step 7 — Start All Devices

1. Click **Edit → Start all nodes** (or the ▶ play button in the toolbar).
2. Wait for both FortiGate nodes to finish booting (the status dot turns green).

   > 📸 **Screenshot:** `screenshots/03-gns3-topology-running.png`
   > *Show the complete GNS3 canvas with both FortiGates and Cloud nodes
   > connected, all status dots green.*

---

## Next Step

With the topology running, proceed to configure the FortiGate WAN
interfaces:

→ [01-fortigate-basic-config.md](./01-fortigate-basic-config.md)
