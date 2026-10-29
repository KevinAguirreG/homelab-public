# 2. Networking

## Subnet

- Network: `192.168.0.0/24` (254 usable addresses)
- Gateway and DHCP: router at `192.168.0.1`
- Router DHCP range: `192.168.0.100 – 192.168.0.199`

## Static IPs outside the DHCP range

If the server relied on DHCP its address could change and access would break. Infrastructure IPs are assigned manually and kept outside the router's DHCP range so they never collide with other devices.

| IP | Host |
|---|---|
| `.50` | Proxmox host |
| `.51` | monitoring LXC |
| `.52` | Reserved (media, postponed) |
| `.53` | apps VM |
| `.54` | dns LXC |

Each IP was checked with `ping` before use ("Destination host unreachable" reported by the local machine means nobody holds it).

The alternative is a **DHCP reservation** (the router always gives the same IP to a MAC address). It's more centralized, but depends on the router.

## Bridge

`vmbr0` is a Linux bridge inside Proxmox — a virtual switch. The physical NIC (`nic0`) is attached to it, and every VM or LXC plugs into the bridge as if it had its own cable to the router.

## Physical link

The server is wired via **Ethernet** to a LAN port on the router (not WAN). Proxmox isn't designed to run over Wi-Fi: bridging doesn't work well on wireless interfaces.
