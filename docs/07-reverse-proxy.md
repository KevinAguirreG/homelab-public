# 7. Reverse proxy with Caddy

## What it does

Receives every web request on ports 80/443 and forwards it to the right service based on the `Host` header. One entry point, centralized HTTPS (pending) and no exposed ports on the apps themselves. It plays the role of an ALB in AWS.

## Options

| Option | Configuration | Pros | Cons |
|---|---|---|---|
| **Caddy** | `Caddyfile` | Automatic HTTPS, minimal syntax, version-controllable | Fewer plugins than Nginx |
| Nginx Proxy Manager | Web UI | Very easy | Config lives in a database, hard to version |
| Traefik | Docker labels | Auto-discovery | Steep learning curve |
| Plain Nginx | `nginx.conf` | Familiar | Manual certificates |

Caddy was chosen because its configuration is plain text that lives in Git.

## Shared network

```bash
docker network create proxy
```

Caddy and the apps share the `proxy` network. Inside it, containers find each other by name (Docker's embedded DNS), so apps don't publish ports — only Caddy exposes 80/443. In `docker-compose.yml` files the network is declared `external: true` so Compose never creates or deletes it.

## Routing

- Containers on the same Docker network → by **name**: `reverse_proxy whoami:80`
- Services on other hosts → by **IP:port**: `reverse_proxy 192.168.0.51:3000`

Caddy handles WebSockets (needed by Uptime Kuma) with no extra configuration.

Files: [`apps/proxy/docker-compose.yml`](../apps/proxy/docker-compose.yml), [`apps/proxy/Caddyfile`](../apps/proxy/Caddyfile).

```bash
cd ~/infra/proxy
docker compose up -d
# apply Caddyfile changes without dropping connections
docker exec caddy caddy reload --config /etc/caddy/Caddyfile
```

## Why plain HTTP for now

With `.home.arpa` names, Caddy can only sign certificates with its own internal CA, which every device would have to trust. The planned fix is a real domain on Cloudflare DNS with Let's Encrypt certificates via the *DNS challenge*. See the [roadmap](roadmap.md).
