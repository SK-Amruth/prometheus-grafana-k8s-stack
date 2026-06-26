# 📊 Prometheus + Grafana Monitoring Stack

A production-ready monitoring stack using **Prometheus**, **Grafana**, **Node Exporter**, and **Alertmanager** — deployed via Docker Compose. Includes auto-provisioned dashboards, alerting rules, and a clean folder structure ready for extension.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Docker Network                    │
│                                                     │
│  ┌──────────────┐       ┌──────────────────────┐   │
│  │ Node Exporter│──────▶│      Prometheus       │   │
│  │  (port 9100) │       │      (port 9090)      │   │
│  └──────────────┘       └──────────┬───────────┘   │
│                                    │                │
│                         ┌──────────▼───────────┐   │
│                         │     Alertmanager      │   │
│                         │     (port 9093)       │   │
│                         └──────────────────────-┘   │
│                                    │                │
│                         ┌──────────▼───────────┐   │
│                         │       Grafana         │   │
│                         │     (port 3000)       │   │
│                         └──────────────────────-┘   │
└─────────────────────────────────────────────────────┘
```

## 🧩 Stack Components

| Component | Version | Purpose |
|---|---|---|
| Prometheus | v2.51.0 | Metrics collection & storage |
| Grafana | v10.4.0 | Visualization & dashboards |
| Node Exporter | v1.7.0 | Host system metrics |
| Alertmanager | v0.27.0 | Alert routing & notifications |

---

## 🚀 Quick Start

### Prerequisites
- Docker >= 20.x
- Docker Compose >= 2.x

### 1. Clone the repository
```bash
git clone https://github.com/SK-Amruth/prometheus-grafana-k8s-stack.git
cd prometheus-grafana-k8s-stack
```

### 2. Start the stack
```bash
docker compose up -d
```

### 3. Verify all services are running
```bash
docker compose ps
```

### 4. Access the UIs

| Service | URL | Credentials |
|---|---|---|
| Grafana | http://localhost:3000 | admin / admin123 |
| Prometheus | http://localhost:9090 | — |
| Alertmanager | http://localhost:9093 | — |
| Node Exporter | http://localhost:9100/metrics | — |

> ⚠️ **Change the default Grafana password** after first login in production.

---

## 📁 Project Structure

```
prometheus-grafana-k8s-stack/
├── docker-compose.yml                        # Main stack definition
├── prometheus/
│   ├── prometheus.yml                        # Scrape configs & targets
│   ├── alert_rules.yml                       # Alerting rules (CPU, Memory, Disk)
│   └── alertmanager.yml                      # Alert routing config
└── grafana/
    ├── provisioning/
    │   ├── datasources/
    │   │   └── prometheus.yml                # Auto-provision Prometheus datasource
    │   └── dashboards/
    │       └── dashboard.yml                 # Dashboard loader config
    └── dashboards/
        └── node-exporter.json                # Node Exporter dashboard
```

---

## 📈 Pre-built Dashboard

The stack ships with a **Node Exporter - System Overview** dashboard that auto-loads on startup:

- ✅ CPU Usage % (stat + timeseries)
- ✅ Memory Usage % (stat + timeseries)
- ✅ Disk Usage % (stat)
- ✅ System Uptime
- ✅ Network Traffic (RX/TX)
- ✅ Disk I/O (Read/Write)

---

## 🚨 Alerting Rules

Pre-configured alert rules in `prometheus/alert_rules.yml`:

| Alert | Condition | Severity |
|---|---|---|
| `InstanceDown` | Target unreachable > 1 min | 🔴 Critical |
| `HighCPUUsage` | CPU > 80% for 5 min | 🟡 Warning |
| `HighMemoryUsage` | Memory > 85% for 5 min | 🟡 Warning |
| `DiskSpaceLow` | Disk > 90% for 5 min | 🔴 Critical |

### Enable Slack Notifications
Edit `prometheus/alertmanager.yml` and uncomment the Slack section:
```yaml
receivers:
  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
        channel: '#alerts'
```

---

## ➕ Adding a New Scrape Target

Edit `prometheus/prometheus.yml` and add a new job:
```yaml
scrape_configs:
  - job_name: 'my-app'
    static_configs:
      - targets: ['my-app-host:8080']
    metrics_path: /metrics
```

Then reload Prometheus config (no restart needed):
```bash
curl -X POST http://localhost:9090/-/reload
```

---

## ☸️ Kubernetes Extension (Helm)

To deploy on Kubernetes using the kube-prometheus-stack Helm chart:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.adminPassword=admin123
```

> See the `k8s/` branch for full Kubernetes manifests and values overrides.

---

## 🛑 Tear Down

```bash
# Stop and remove containers
docker compose down

# Also remove volumes (clears all data)
docker compose down -v
```

---

## 🔧 Useful Prometheus Queries (PromQL)

```promql
# CPU usage percentage
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage percentage
(1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100

# Network receive rate (bytes/s)
rate(node_network_receive_bytes_total{device!~"lo"}[5m])
```

---

## 📌 Roadmap

- [ ] Add cAdvisor for container-level metrics
- [ ] Add Loki for log aggregation
- [ ] Kubernetes Helm values file
- [ ] GitHub Actions CI for config validation
- [ ] Multi-host scraping via service discovery

---

## 🤝 Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you'd like to change.

---

## 📄 License

[MIT](LICENSE)

---

> Built by [Amrutha S](https://linkedin.com/in/amrutha-devops) — DevOps & Cloud Engineer
