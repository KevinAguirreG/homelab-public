# Roadmap

## Done

- [x] Proxmox VE installed, no-subscription repos, fully updated
- [x] Static IP plan
- [x] Tailscale subnet router (on 1.104.4)
- [x] Monitoring: node-exporter, Prometheus, Grafana, Uptime Kuma
- [x] `apps` VM with Ubuntu Server, Docker and key-only SSH
- [x] Local DNS with AdGuard Home and DNS over HTTPS
- [x] Caddy reverse proxy
- [x] Configuration versioned in this repo

## Networking and HTTPS

- [ ] Own domain with DNS on Cloudflare
- [ ] Let's Encrypt certificates via DNS challenge (Caddy + Cloudflare plugin)
- [ ] AdGuard as DNS for the whole network via router DHCP
- [ ] Solve the IPv6 DNS preference network-wide
- [ ] DNS monitor in Uptime Kuma

## Operations

- [ ] Scheduled VM and LXC backups
- [ ] Off-box backup copy
- [ ] Alerting (Grafana or Alertmanager)
- [ ] Extra disk for data
- [ ] Lower iGPU reserved memory (UMA Frame Buffer 512M)

## Projects

- [ ] CI/CD: build on GitHub Actions → GHCR → deploy via self-hosted runner on the `apps` VM
- [ ] First project: an outfit-planning app (PWA, requires HTTPS)
