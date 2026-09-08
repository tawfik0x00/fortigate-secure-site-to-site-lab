# GNS3 Installation

## Purpose

This project uses GNS3 to emulate a complete FortiGate network environment locally on Linux.

The lab was developed and tested with the following stack:

| Component        | Version / Notes              |
|------------------|------------------------------|
| Operating System | Fedora Linux                 |
| GNS3             | 2.2.61                       |
| GNS3 Server      | 2.2.61                       |
| Virtualization   | QEMU/KVM                     |
| FortiGate        | FortiOS 7.0.5                |
| Switching        | Cisco IOU L2 (x86 / x86_64) |

> **Important:** Use the exact versions listed above to ensure the lab is reproducible.

---

## Architecture

All virtual network devices run locally on the Linux host — no GNS3 VM is needed.

The environment is composed of:

- **GNS3 GUI** — graphical front-end
- **GNS3 Server** — local back-end server
- **QEMU/KVM** — hypervisor for FortiGate VMs
- **Cisco IOU** — software-based Layer-2 switching
- **GNS3 Cloud nodes** — represent the simulated WAN link

GNS3 supports QEMU/KVM natively on Linux, so a separate GNS3 VM is not required.

---

## Installation

Follow the official GNS3 documentation for your Linux distribution:

- <https://docs.gns3.com/docs/getting-started/installation/linux>
- <https://docs.gns3.com/docs/getting-started/setup-wizard-local-server>

After installation, verify both components are running:

```bash
gns3 --version
gns3server --version
```

Both should report version **2.2.61**.

---

## x86 Environment Setup (required for Cisco IOU)

Cisco IOU binaries are compiled for **32-bit x86**. On a 64-bit Fedora host you must
install the 32-bit compatibility libraries before IOU will run.

```bash
# Fedora
sudo dnf install glibc.i686 libgcc.i686
```

Verify that the IOU binary is executable:

```bash
file ~/GNS3/images/QEMU/i86bi_LinuxL2-AdvEnterpriseK9-M_152_May_2018.bin
# expected: ELF 32-bit LSB executable, Intel 80386 ...

~/GNS3/images/QEMU/i86bi_LinuxL2-AdvEnterpriseK9-M_152_May_2018.bin --help
```

If you see `No such file or directory` despite the file existing, the 32-bit libs
are missing — install them with the command above.
