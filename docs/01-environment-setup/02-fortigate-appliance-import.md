# Importing Appliances into GNS3

Download the required images before proceeding — see [00-prerequisites.md](./00-prerequisites.md).

---

## 1. Import the FortiGate Appliance

1. In GNS3, click **File** → **Import appliance**.
2. Select the appliance file:
   ```
   ~/GNS3/appliances/fortigate.gns3a
   ```
3. Choose **Install the appliance on your local computer** → click **Next**.
4. Click **Create a new version** and enter `7.0.5`.
5. In the image list, find **FortiGate 7.0.5** and select `hda_disk` row.
6. Click **Import** (bottom-left) and choose the file you downloaded:
   ```
   ~/GNS3/images/QEMU/fortios.qcow2
   ```
7. Click **Next** → **Next** → **Finish**.

The FortiGate appliance is now available in the GNS3 device panel.

---

## 2. Import the Cisco IOU Layer-2 Switch

1. In GNS3, click **Edit** → **Preferences**.
2. In the left panel, select **IOU Devices**.
3. Click **New** and fill in:
   - **Name:** `Cisco-IOU-L2` (or any name you prefer)
   - **Image:** browse to
     ```
     ~/GNS3/images/QEMU/i86bi_LinuxL2-AdvEnterpriseK9-M_152_May_2018.bin
     ```
4. Click **Finish**.

---

## 3. Activate the IOU Licence

IOU requires a valid `iourc` licence file. A keygen helper script is included in this
repository at `scripts/CiscoIOUKeygen.py`. Run it as root:

```bash
# Run from the repo root
sudo python3 scripts/CiscoIOUKeygen.py
```

The script generates and writes the licence directly to `~/.iourc`.
Verify it was created:

```bash
cat ~/.iourc
```

> **Note:** The `iourc` file must be at `~/.iourc` (i.e. `/home/<your-user>/.iourc`).
> GNS3 looks for it in that exact location.

> **Note:** Make sure the 32-bit x86 libraries are installed on your system before
> running IOU — see [01-gns3-installation.md](./01-gns3-installation.md#x86-environment-setup-required-for-cisco-iou).
