# 6. Local DNS with AdGuard Home

## Goal

Resolve internal names (`grafana.home.arpa`) for any device on the network, without editing every device's `hosts` file.

## Why AdGuard Home

| Option | Notes |
|---|---|
| **AdGuard Home** | Wildcard rewrites, DNS over HTTPS upstream, ad blocking, clean UI |
| Pi-hole | Very popular; wildcards are less straightforward |
| Technitium | Full authoritative DNS server; more than needed |
| dnsmasq | Minimal, no UI |

## Why its own LXC

DNS is critical infrastructure: if it goes down, nobody can browse. It shouldn't depend on the apps VM, which is where experimentation happens. AdGuard is installed **natively** (a single binary) rather than in Docker, to keep one fewer moving part.

LXC 102 `dns`: Debian 13, unprivileged, 1 vCPU, 512 MB, 4 GB, IP `192.168.0.54`, nameserver `1.1.1.1`.

```bash
apt update && apt full-upgrade -y
apt install -y curl
curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
```

The setup wizard runs on `:3000`; afterwards the dashboard lives on port 80 and DNS on port 53.

## Configuration

**Upstreams (DNS over HTTPS), parallel requests:**

```
https://cloudflare-dns.com/dns-query
https://dns.google/dns-query
```

Outbound queries are encrypted, so the ISP can't see which domains are resolved.

**Rewrites:**

| Domain | Answer |
|---|---|
| `*.home.arpa` | `192.168.0.53` (Caddy) |
| `pve.home.arpa` | `192.168.0.50` (more specific, takes precedence) |

`home.arpa` is the domain reserved for home networks (RFC 8375): it never collides with a real domain, unlike `.local` (mDNS) or `.home`.

AdGuard's DHCP server is **not** enabled: the router already hands out IPs, and two DHCP servers on one network compete with each other.

## Clients

For now only one Windows PC points to `192.168.0.54` (manual IPv4 DNS). IPv6 had to be **disabled** on that adapter: Windows preferred the DNS server the router advertised over IPv6 and ignored AdGuard. See [troubleshooting](troubleshooting.md).

```powershell
nslookup whoami.home.arpa 192.168.0.54   # query AdGuard directly
ipconfig /flushdns
nslookup whoami.home.arpa                # with AdGuard as the default resolver
```

## Known risk

Once the whole network points to AdGuard (pending), the `dns` LXC becomes a single point of failure. A public DNS as secondary isn't a fix: clients don't respect the order and local names would fail at random. Planned mitigation: a DNS monitor in Uptime Kuma.
