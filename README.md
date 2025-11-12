# PicoCluster Local Monitoring & Management

This repository contains monitoring, observability, and cluster management tools for PicoCluster.

## Contents

### Monitoring Directory
Complete monitoring infrastructure for PicoCluster clusters:

- **Prometheus & Grafana Installation**
  - RPI5 Ubuntu setup: `monitoring/rpi5_ubuntu/install_prometheus_grafana.ansible`
  - RPI5 Raspbian setup: `monitoring/rpi5_raspbian/install_prometheus_grafana.ansible`
  - Universal Node Exporter: `monitoring/metrics_collection/install_node_exporter.ansible`
  - Container metrics: `monitoring/metrics_collection/install_container_metrics.ansible`
  - Cluster deployment: `monitoring/metrics_collection/deploy_metrics_to_cluster.ansible`

- **Loki Log Aggregation (Optional)**
  - RPI5 Ubuntu: `monitoring/rpi5_ubuntu/install_loki.ansible`
  - RPI5 Raspbian: `monitoring/rpi5_raspbian/install_loki.ansible`
  - Promtail log shipper: `monitoring/metrics_collection/install_promtail.ansible`
  - Log dashboard included

- **Grafana Dashboards** (10 pre-configured)
  - Cluster Overview
  - System Metrics
  - Docker Metrics
  - Kubernetes Metrics
  - Cluster Inventory
  - Service Health
  - Network Performance
  - Capacity Planning
  - Workload Performance
  - Logs Overview (with Loki)

- **Prometheus Configuration**
  - 31 alert rules with Slack/email notifications
  - AlertManager setup
  - Prometheus scrape configuration

- **Documentation**
  - `MONITORING_SETUP_GUIDE.md`: Complete monitoring setup
  - `LOKI_SETUP_GUIDE.md`: Optional log aggregation setup
  - `ALERT_RULES_GUIDE.md`: Alert rules reference
  - `DASHBOARDS_README.md`: Dashboard overview (10 dashboards)

### Cluster Management Directory
Tools for managing and maintaining PicoCluster:

- **Health & Validation**
  - Cluster health check: `cluster-management/cluster_health_check.sh`
  - Setup validation: `cluster-management/validate_cluster_setup.sh`

- **Backup & Restore**
  - Automated backups: `cluster-management/backup_cluster_state.ansible`
  - 7-day rolling retention

- **Alerting**
  - 31 Prometheus alert rules
  - Deployment playbook: `cluster-management/deploy_alert_rules.ansible`
  - Slack/email integration

- **Documentation**
  - `CLUSTER_MANAGEMENT_GUIDE.md`: All tools and usage
  - `ALERT_RULES_GUIDE.md`: Alert rules reference

## Quick Start

### Enable Monitoring

```bash
# Install Prometheus & Grafana on RPI5
ansible-playbook monitoring/rpi5_ubuntu/install_prometheus_grafana.ansible

# Deploy Node Exporter to all cluster nodes
ansible-playbook monitoring/metrics_collection/install_node_exporter.ansible

# Deploy container metrics collection
ansible-playbook monitoring/metrics_collection/install_container_metrics.ansible

# Deploy all metrics collectors to cluster
ansible-playbook monitoring/metrics_collection/deploy_metrics_to_cluster.ansible

# Optional: Add log aggregation (see LOKI_SETUP_GUIDE.md)
ansible-playbook monitoring/rpi5_ubuntu/install_loki.ansible
ansible-playbook monitoring/metrics_collection/install_promtail.ansible
```

### Manage Cluster

```bash
# Check cluster health
./cluster-management/cluster_health_check.sh

# Validate cluster setup
./cluster-management/validate_cluster_setup.sh

# Deploy alert rules
ansible-playbook cluster-management/deploy_alert_rules.ansible

# Backup cluster state
ansible-playbook cluster-management/backup_cluster_state.ansible
```

## Documentation Files

| File | Contents |
|------|----------|
| `monitoring/MONITORING_SETUP_GUIDE.md` | Complete monitoring setup guide (300+ lines) |
| `monitoring/LOKI_SETUP_GUIDE.md` | Optional log aggregation setup (350+ lines) |
| `monitoring/config/grafana/DASHBOARDS_README.md` | 10 dashboard overview (500+ lines) |
| `cluster-management/CLUSTER_MANAGEMENT_GUIDE.md` | All management tools guide (555 lines) |
| `cluster-management/ALERT_RULES_GUIDE.md` | 31 alert rules reference (400+ lines) |
| `PICO_OBSERVABILITY_COMPARISON.md` | Feature comparison with pico-observability SaaS |

## Key Features

### Monitoring
- Real-time metrics collection (Prometheus)
- Beautiful dashboards (Grafana with 10 pre-configured)
- Optional log aggregation (Loki + Promtail)
- Multi-architecture support (ARM64, x86-64)
- Container metrics (Docker, Containerd, Kubernetes)
- Automatic service discovery
- Alert rules (31 rules for various scenarios)

### Cluster Management
- Real-time health monitoring
- Pre-deployment validation
- Automated backup with retention policies
- Alert rule deployment
- Color-coded status output
- Comprehensive logging

## Architecture

### Metrics Stack (Core)
```
Cluster Nodes
    ↓
Node Exporter (metrics)
    ↓
Container Metrics (cAdvisor, kube-state-metrics)
    ↓
Prometheus (scrapes metrics)
    ↓
Grafana (visualizes)
    ↓
10 Pre-configured Dashboards
```

### Optional: Logs Stack
```
Cluster Nodes
    ↓
Promtail (log shipper)
    ↓
Loki (log aggregation)
    ↓
Grafana (visualizes logs)
```

## Alert Rules

31 alert rules covering:
- Node health (CPU, Memory, Disk)
- Service status
- Networking
- Kubernetes health
- Cluster metrics
- Certificate expiration
- And more...

## Utility Scripts

- `cluster_health_check.sh`: Real-time cluster health (430+ lines)
- `validate_cluster_setup.sh`: Pre-deployment validation (520+ lines)

## Related Repositories

- **ApplicationSets**: Infrastructure as code, all automation tools
  - GitOps (Flux)
  - CI/CD (GitLab Runner)
  - Networking (Linkerd, Calico, WireGuard)
  - Observability (Jaeger, Loki)
  - And more...

## Support

For issues or questions:
1. Check the relevant SETUP_GUIDE.md
2. Review the ALERT_RULES_GUIDE.md or CLUSTER_MANAGEMENT_GUIDE.md
3. Check cluster health: `./cluster-management/cluster_health_check.sh`
4. Validate setup: `./cluster-management/validate_cluster_setup.sh`

---

**Part of PicoCluster**: Complete infrastructure-as-code for heterogeneous single-board computer clusters
