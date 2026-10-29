# 3. Remote access with Tailscale

## The problem

The classic way to reach a home server from outside is port forwarding on the router, which exposes services to the entire internet.

## The solution

Tailscale builds a **mesh VPN** on top of WireGuard: each of my devices connects to the others over encrypted tunnels, with no open ports. It traverses NAT (including CGNAT) and falls back to encrypted relays (DERP) when a direct connection isn't possible.

| Alternative | Why not |
|---|---|
| Plain WireGuard | Requires opening a port and managing keys by hand |
| Cloudflare Tunnel | Better suited to publishing websites than admin access |
| OpenVPN | Older and heavier |

## Where it runs

On the **Proxmox host**, as a **subnet router**: installed once, it gives access to the whole `192.168.0.0/24` network. If a container breaks, remote access keeps working.

```bash
curl -fsSL https://tailscale.com/install.sh | sh

# Allow the host to forward packets between interfaces
echo 'net.ipv4.ip_forward = 1'          >> /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' >> /etc/sysctl.d/99-tailscale.conf
sysctl -p /etc/sysctl.d/99-tailscale.conf

tailscale up --advertise-routes=192.168.0.0/24
```

In the admin console:
- The `192.168.0.0/24` route is **approved manually** (no node can advertise networks on its own).
- **Key expiry disabled** for the server so it doesn't lose access after 180 days.

## Tailscale SSH: intentionally disabled

Machines are reached with **OpenSSH and keys**, not Tailscale SSH (`RunSSH: false`). Less software running as root means a smaller attack surface.

That kept the server out of scope for the most severe *Tailworm* finding (Oct 2026, TS-2026-014), which lived in Tailscale SSH socket forwarding. The client was still upgraded to **1.104.4**, which fixes all client-side findings.

```bash
tailscale version
tailscale debug prefs | grep -i runssh   # "RunSSH": false
```

## Known limitation

If a device is on another network that also uses `192.168.0.x`, the advertised route can clash with that local network. In that case, use the server's `100.x.y.z` tailnet IP.
