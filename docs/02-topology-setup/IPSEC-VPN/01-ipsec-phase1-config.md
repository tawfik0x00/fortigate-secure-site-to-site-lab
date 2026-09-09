# IPsec Site-to-Site VPN — Phase 1 (IKE) Configuration

## Overview

This document configures **IKE Phase 1** on both **FGT-BRANCH-A** and
**FGT-BRANCH-B** to establish the IPsec tunnel foundation over the simulated WAN.

```
FGT-BRANCH-A (port1)          WAN 172.31.255.0/24        FGT-BRANCH-B (port1)
172.31.255.10  ◄─────────── IPsec Tunnel ───────────►  172.31.255.20
```

---

## Tunnel Parameters

| Parameter            | Value                    |
|----------------------|--------------------------|
| Tunnel Name          | `SITE-A-TO-B` / `SITE-B-TO-A` |
| IKE Version          | IKEv1                    |
| Authentication       | Pre-Shared Key (PSK)     |
| Pre-Shared Key       | `Fortigate@NTI2025!`     |
| Encryption           | AES-256                  |
| Authentication Hash  | SHA-256                  |
| DH Group             | Group 14 (2048-bit)      |
| Lifetime             | 86400 seconds (24 h)     |
| Local Gateway        | port1 IP                 |
| Remote Gateway       | peer port1 IP            |
| Mode                 | Main Mode                |

---

## FGT-BRANCH-A Configuration

Connect to FGT-BRANCH-A CLI:

```
config vpn ipsec phase1-interface
    edit "SITE-A-TO-B"
        set interface "port1"
        set ike-version 1
        set peertype any
        set net-device disable
        set proposal aes256-sha256
        set dhgrp 14
        set remote-gw 172.31.255.20
        set psksecret Fortigate@NTI2025!
        set keylife 86400
    next
end
```

---

## FGT-BRANCH-B Configuration

Connect to FGT-BRANCH-B CLI:

```
config vpn ipsec phase1-interface
    edit "SITE-B-TO-A"
        set interface "port1"
        set ike-version 1
        set peertype any
        set net-device disable
        set proposal aes256-sha256
        set dhgrp 14
        set remote-gw 172.31.255.10
        set psksecret Fortigate@NTI2025!
        set keylife 86400
    next
end
```

---

## Verify Phase 1

```
diagnose vpn ike gateway list
diagnose debug application ike -1
diagnose debug enable
```

Expected: tunnel status transitions to **`established`**.

---

## Screenshots

> 📸 Screenshots will be added here.

---

## Milestone Checklist

| Check                                    | Expected                          |
|------------------------------------------|-----------------------------------|
| Phase1 created on FGT-A                  | `SITE-A-TO-B` in VPN list        |
| Phase1 created on FGT-B                  | `SITE-B-TO-A` in VPN list        |
| IKE negotiation succeeds                 | Status: established               |
| No PSK mismatch errors in debug output   | Clean IKE exchange                |

---

## Next Step

Configure **Phase 2** (IPsec SA) to define the traffic selectors.

→ *See: `docs/02-topology-setup/IPSEC-VPN/02-ipsec-phase2-config.md`*
