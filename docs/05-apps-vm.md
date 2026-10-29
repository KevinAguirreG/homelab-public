# 5. Application VM

## Why a VM and not an LXC

- **Real isolation:** its own kernel. If a project breaks or gets compromised, the host is unaffected.
- **Cloud parity:** an Ubuntu VM is essentially an EC2 instance.
- **Docker without workarounds:** no nesting or keyctl needed.

## Configuration (VM 200 `apps`)

| Setting | Value | Why |
|---|---|---|
| OS | Ubuntu Server 26.04 LTS | Most common in the cloud; 5 years of support |
| CPU | 4 cores, type `host` | Exposes the Ryzen's real instruction set |
| RAM | 6144 MB, ballooning with a 2048 MB floor | Proxmox can reclaim RAM when the host needs it |
| Disk | 64 GB on `local-lvm`, `discard=on`, `iothread=on` | Freed space returns to the host; I/O on its own thread |
| Network | VirtIO on `vmbr0` | Paravirtualized driver, faster than emulated hardware |
| Agent | QEMU guest agent | Reports IP and state, clean shutdown, consistent backups |
| IP | `192.168.0.53/24`, DNS `1.1.1.1, 8.8.8.8` | Static IP outside the DHCP range |

## Key-only SSH

- Public keys were imported from GitHub (`github.com/<user>.keys`) during installation, and password login is disabled.
- **ED25519** keys.
- To authorize a new machine: add its public key to GitHub, then on the VM:

```bash
ssh-import-id gh:<user>
```

GitHub acts as a central store of public keys: any new server can be authorized for every machine with a single command.

## Setup

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y qemu-guest-agent
sudo systemctl enable --now qemu-guest-agent
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER   # log out and back in
docker run --rm hello-world
```

Docker's official install script was used instead of Ubuntu's `docker.io` package, which is older and doesn't always ship Compose v2.

> Membership in the `docker` group is effectively root. Acceptable on a personal VM.
