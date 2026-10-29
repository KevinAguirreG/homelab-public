# 1. Hypervisor

## Why Proxmox VE

A **type 1 hypervisor** runs directly on the hardware and splits CPU, RAM and storage across isolated systems.

| Option | Ruled out because |
|---|---|
| Ubuntu Server + Docker | Everything shares one OS; no snapshots or per-service isolation |
| TrueNAS Scale | Storage-focused |
| Unraid | Paid |
| ESXi | Free tier mostly gone |

Proxmox is free, combines VMs (KVM) and containers (LXC), ships with snapshots and backups, and its model closely resembles operating real infrastructure.

## Installation

1. Proxmox VE 9.2 ISO verified against its SHA256 checksum and written to USB with balenaEtcher (raw image write, not a file copy).
2. BIOS: **SVM Mode** enabled (AMD virtualization), Secure Boot disabled.
3. Graphical installer: NVMe disk with **ext4**, timezone `America/Monterrey`, hostname `pve.home.arpa`, static IP `192.168.0.50/24`.

### ext4 instead of ZFS

ZFS protects against corruption, compresses and snapshots natively, but it is RAM-hungry and its main advantage (RAID) needs multiple disks. With a single disk and 16 GB of RAM, **ext4 + LVM-thin** is the efficient choice.

Resulting layout (238.5 GiB):

| Volume | Size | Purpose |
|---|---|---|
| `pve-root` | 69 GB | OS, ISOs, templates |
| `pve-data` (thin pool) | 141 GB | VM and LXC disks |
| `pve-swap` | 8 GB | Swap |

*Thin provisioning*: virtual disks only consume the space they actually use.

## Post-install

- **Enterprise** repositories (pve and ceph) disabled, **no-subscription** repository added. Enterprise requires a license; no-subscription gets the same updates slightly earlier.
- `apt update && apt full-upgrade`.

Hardware verification:

```bash
grep -c svm /proc/cpuinfo   # 8 → virtualization enabled on all 8 threads
ls /dev/dri                 # renderD128 → iGPU available for transcoding
free -h                     # 13 GiB → 16 GB minus iGPU-reserved memory
lsblk                       # 238.5 GiB → the advertised 256 GB
```

## VM vs. LXC

| | VM (KVM) | LXC |
|---|---|---|
| Kernel | Its own | Shared with the host |
| Isolation | Strong | Good (namespaces + cgroups) |
| Overhead | Higher | Minimal; boots in seconds |
| Used here for | Own applications (experimentation) | Lightweight, stable services |

LXCs are created **unprivileged**: UID 0 inside the container maps to UID 100000 on the host, so escaping the container does not grant root on the server.
