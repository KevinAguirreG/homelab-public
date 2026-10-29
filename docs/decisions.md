# Design decisions

Short log of decisions, the rejected alternative and why.

| # | Decision | Alternative | Reason |
|---|---|---|---|
| 1 | Proxmox VE as the base | Ubuntu + Docker | Per-service isolation, snapshots, VMs and LXC |
| 2 | ext4 + LVM-thin | ZFS | Single disk and 16 GB of RAM |
| 3 | Static IPs outside the DHCP range | DHCP reservations | Don't depend on the router |
| 4 | Tailscale subnet router on the host | Port forwarding / manual WireGuard | No open ports; access survives a broken container |
| 5 | OpenSSH with keys, no Tailscale SSH | Tailscale SSH | Less privileged software exposed |
| 6 | LXC for monitoring and DNS | VMs | Lightweight, stable services; low overhead |
| 7 | VM for applications | LXC | Kernel isolation and cloud parity |
| 8 | Prometheus + Grafana | Netdata / Zabbix | Industry standard |
| 9 | Native AdGuard Home in a dedicated LXC | Pi-hole / Docker | DNS is critical: isolated, fewer moving parts |
| 10 | `home.arpa` for internal names | `.local` / `.home` | Reserved by RFC 8375 |
| 11 | Caddy as reverse proxy | Nginx Proxy Manager / Traefik | Text config, automatic HTTPS |
| 12 | Media server (Jellyfin) postponed | Set it up first | Priority: hosting my own projects |
