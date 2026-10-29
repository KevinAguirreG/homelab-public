# 4. Monitoring

## Components

| Component | Where | What it does |
|---|---|---|
| node-exporter | `pve` host (`:9100`) | Exposes server metrics (CPU, RAM, disk, network) |
| Prometheus | `monitoring` LXC (`:9090`) | Scrapes exporters every 30 s and stores time series (30-day retention) |
| Grafana | `monitoring` LXC (`:3000`) | Dashboards. Imported: *Node Exporter Full* (ID 1860) |
| Uptime Kuma | `monitoring` LXC (`:3001`) | Availability checks: is it up? |

node-exporter runs on the **host** because inside an LXC it would only see the container.

## Concepts

- **Pull vs. push:** Prometheus fetches metrics (*pull*). In the push model (InfluxDB, Datadog) each service sends its own data.
- **Metrics vs. availability:** Grafana answers "how is it doing?"; Uptime Kuma answers "is it responding?" (*blackbox* monitoring).

Alternatives considered: Netdata (all-in-one) and Zabbix (enterprise, heavier). Prometheus + Grafana won as the industry standard in DevOps and Kubernetes.

## `monitoring` LXC

- Debian 13, unprivileged, 2 vCPU, 2 GB RAM, 16 GB disk.
- **nesting** and **keyctl** features enabled: Docker needs to create containers inside the container and use the kernel keyring.

> Proxmox officially recommends running Docker inside a VM. Running it in an LXC saves resources and works fine for a homelab.

## Deployment

```bash
# On the host
apt install -y prometheus-node-exporter

# On the monitoring LXC
mkdir -p /opt/monitoring && cd /opt/monitoring
# copy monitoring/docker-compose.yml and monitoring/prometheus.yml from this repo
docker compose up -d
```

Files: [`monitoring/docker-compose.yml`](../monitoring/docker-compose.yml), [`monitoring/prometheus.yml`](../monitoring/prometheus.yml).

Check: at `http://192.168.0.51:9090/targets` both targets should be **UP**.

## Current scope

Host metrics and availability only. No alerting, application metrics or tracing yet.
