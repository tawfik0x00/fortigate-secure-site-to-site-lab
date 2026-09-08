# Prerequisites & Required Downloads

Before setting up the lab, download the required images from the shared Google Drive:

## 📥 Download Link

**[Google Drive — Lab Images & Files](https://drive.google.com/drive/folders/1H5IPMfh7NootceapMAVdN-TgNxiy1eLs)**

---

## Required Files

| File | Purpose | Place after download |
|------|---------|----------------------|
| `fortios.qcow2` | FortiGate OS disk image | `~/GNS3/images/QEMU/` |
| `i86bi_LinuxL2-AdvEnterpriseK9-M_152_May_2018.bin` | Cisco IOU L2 switch binary | `~/GNS3/images/QEMU/` |
| `fortigate.gns3a` | GNS3 FortiGate appliance definition | `~/GNS3/appliances/` |

> **Note:** The IOU licence keygen script (`CiscoIOUKeygen.py`) is already included
> in this repository under `scripts/` — no need to download it separately.

> **Note:** Images are **not included** in this repository because of their size.
> All required files are available exclusively via the Drive link above.

---

## Versions

Make sure you download the exact versions listed below — the lab was built and
tested with these specific files.

| Component | Version |
|-----------|---------|
| FortiOS   | 7.0.5   |
| Cisco IOU L2 | 15.2 (May 2018) |
