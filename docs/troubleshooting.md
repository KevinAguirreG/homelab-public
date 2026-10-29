# Troubleshooting

Real issues hit while building this, and how they were diagnosed.

## `ERR_CONNECTION_REFUSED` when opening Proxmox from a laptop

**Diagnosis:**
1. On the server, `systemctl status pveproxy` → active; `ss -tlnp | grep 8006` → listening. The server was fine.
2. On the laptop, `ipconfig getifaddr en0` → `192.168.3.30`.

**Cause:** the laptop was on a second router's network (`192.168.3.x`). The "connection refused" came from that router, not Proxmox.
**Fix:** join the laptop to the server's network. Long term: Tailscale.

## VM boot loop with "No bootable device"

**First hypothesis (wrong):** CPU type `host`. Switching to `x86-64-v2-AES` changed nothing.
**Diagnosis:** `ls -lh /var/lib/vz/template/iso/` → the ISO was **87 KB**.
**Cause:** the URL given to "Download from URL" was the download *page*, not the file; Proxmox saved HTML.
**Fix:** delete it and use the direct link from `releases.ubuntu.com`. *Query URL* should report ~3 GB before downloading.
**Lesson:** check the size and hash of any image before using it.

## SSH: `Permission denied (publickey)` from Windows

**Diagnosis:** `ssh -v` showed the client offering `id_ed25519` with a fingerprint that matched neither key imported from GitHub. `Authentications that can continue: publickey` confirmed the server doesn't accept passwords.
**Fix:** add the Windows public key to GitHub and run `ssh-import-id gh:<user>` from the Proxmox console.

## `nslookup` ignored AdGuard

**Symptom:**

```
DNS request timed out.
Server:  UnKnown
Address:  fe80::1
```

**Cause:** manual DNS was set for IPv4 only, but the router advertised a DNS server over IPv6 (`fe80::1`, link-local). Windows prefers IPv6, so it queried the router first — which didn't answer.
**Fix:** disable TCP/IPv6 on the PC's adapter.
**Pending:** solve it network-wide (AdGuard listening on IPv6, or stop the router advertising IPv6 DNS).

## DNS setting in the wrong field

While creating the `dns` LXC, `1.1.1.1` was entered as *DNS domain* (shown as `searchdomain`) instead of *DNS servers* (`nameserver`). Caught on the confirmation screen.
- **nameserver:** who you ask.
- **searchdomain:** suffix appended to short names.

## *Tailworm* vulnerability in Tailscale (Oct 2026)

**Assessment:** the critical finding was in Tailscale SSH, which is disabled on the server (`RunSSH: false`). No `tailscale serve`, app connectors or headscale in use.
**Action:** server already on 1.104.4 via auto-update; desktop clients updated.
