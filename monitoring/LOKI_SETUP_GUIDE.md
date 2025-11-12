# Loki Log Aggregation Setup Guide

**Optional Feature**: Add centralized log aggregation to your pico-local monitoring stack.

## Overview

This guide helps you add **Grafana Loki** log aggregation to your existing pico-local setup. Loki complements Prometheus metrics with centralized log collection and viewing.

### What You'll Get

- **Centralized logs** from all cluster nodes
- **Log visualization** in Grafana alongside metrics
- **Log-based alerting** for error patterns
- **Lightweight** - optimized for RPi5 and homelab use
- **7-day retention** by default (configurable)

### Architecture

```
Cluster Nodes                RPI5 Monitoring Server
    │
    ├─ Promtail ──────────┐
    ├─ Promtail ──────────┼──> Loki ───> Grafana
    └─ Promtail ──────────┘
       (log shipper)
```

---

## Prerequisites

- ✅ Existing pico-local setup with Prometheus + Grafana
- ✅ RPI5 with at least 4GB RAM
- ✅ 20-50 GB free storage (for logs)
- ✅ Ansible installed
- ✅ SSH access to all cluster nodes

---

## Storage Requirements

### Estimated Log Volume

| Cluster Size | Logs/Day | 7-Day Retention | Recommendation |
|--------------|----------|-----------------|----------------|
| 5 nodes | 2-5 GB | 14-35 GB | SD Card OK |
| 10 nodes | 4-10 GB | 28-70 GB | External SSD recommended |
| 20+ nodes | 8-20 GB | 56-140 GB | External SSD required |

**Note**: Actual volume depends on application verbosity and log levels.

---

## Installation Steps

### Step 1: Install Loki on RPI5

Choose based on your OS:

#### For Ubuntu:
```bash
cd monitoring/rpi5_ubuntu
ansible-playbook -i inventory.ini install_loki.ansible
```

#### For Raspbian:
```bash
cd monitoring/rpi5_raspbian
ansible-playbook -i inventory.ini install_loki.ansible
```

**What this does**:
- Downloads and installs Loki binary
- Creates systemd service
- Configures local single-tenant storage
- Starts Loki on port 3100

**Verify installation**:
```bash
# Check service status
sudo systemctl status loki

# Test API
curl http://localhost:3100/ready

# View logs
sudo journalctl -u loki -f
```

---

### Step 2: Install Promtail on Cluster Nodes

Create inventory file if you don't have one:

**inventory.ini**:
```ini
[monitoring_server]
rpi5-monitor ansible_host=192.168.1.100

[cluster_nodes]
node-01 ansible_host=192.168.1.11
node-02 ansible_host=192.168.1.12
node-03 ansible_host=192.168.1.13

[cluster_nodes:vars]
ansible_user=pi
ansible_become=yes
```

Install Promtail on all nodes:
```bash
cd monitoring/metrics_collection
ansible-playbook -i ../../inventory.ini install_promtail.ansible
```

**What this does**:
- Installs Promtail on each node
- Configures log collection from:
  - System logs (`/var/log/*.log`)
  - Syslog
  - Auth logs
  - Systemd journal
  - Docker container logs (if Docker present)
- Ships logs to Loki server

**Verify installation**:
```bash
# On each node
ssh node-01 "sudo systemctl status promtail"

# Check metrics endpoint
ssh node-01 "curl -s http://localhost:9080/metrics | grep promtail"
```

---

### Step 3: Configure Grafana Datasource

The Loki datasource is automatically configured if you deploy with the updated provisioning config.

**Manual configuration** (if needed):

1. Open Grafana: `http://<rpi5-ip>:3000`
2. Go to **Configuration** → **Data Sources**
3. Click **Add data source**
4. Select **Loki**
5. Configure:
   - **Name**: Loki
   - **URL**: `http://localhost:3100`
   - Click **Save & Test**

---

### Step 4: Import Log Dashboard

Import the pre-configured log dashboard:

1. In Grafana, go to **Dashboards** → **Import**
2. Upload: `monitoring/config/grafana/dashboard_logs_overview.json`
3. Select **Loki** datasource
4. Click **Import**

**Dashboard features**:
- Log volume by host
- Error and warning counts
- Recent errors/warnings
- All logs with filtering by host and job
- Log volume by source

---

## Configuration

### Adjust Log Retention

Default retention is **7 days**. To change:

Edit `/etc/loki/loki-local.yml`:
```yaml
table_manager:
  retention_deletes_enabled: true
  retention_period: 336h  # 14 days (instead of 168h)
```

Restart Loki:
```bash
sudo systemctl restart loki
```

### Change Log Storage Location

To use external SSD:

1. Mount SSD to `/mnt/loki-data`
2. Edit `/etc/loki/loki-local.yml`:
   ```yaml
   common:
     path_prefix: /mnt/loki-data
   ```
3. Create directories:
   ```bash
   sudo mkdir -p /mnt/loki-data/{chunks,rules,tsdb-index,tsdb-cache,compactor}
   sudo chown -R loki:loki /mnt/loki-data
   ```
4. Restart Loki:
   ```bash
   sudo systemctl restart loki
   ```

### Add Custom Log Sources

Edit Promtail config on any node: `/etc/promtail/config.yml`

Example - add Nginx logs:
```yaml
scrape_configs:
  # ... existing configs ...

  - job_name: nginx
    static_configs:
      - targets:
          - localhost
        labels:
          job: nginx
          host: {{ ansible_hostname }}
          __path__: /var/log/nginx/*.log
```

Restart Promtail:
```bash
sudo systemctl restart promtail
```

---

## Usage

### Viewing Logs in Grafana

1. Open the **Logs Overview** dashboard
2. Use filters:
   - **Host**: Select specific nodes
   - **Job**: Filter by log source (syslog, docker, etc.)
   - **Time Range**: Adjust in top-right
3. Click any log line to see details

### Searching Logs

Use **Explore** feature in Grafana:

1. Click **Explore** (compass icon)
2. Select **Loki** datasource
3. Use LogQL queries:

```logql
# All logs from a specific host
{host="node-01"}

# Errors from all hosts
{job=~".+"} |~ "(?i)error"

# Docker container logs
{job="docker"} |= "container_name"

# Auth failures
{job="auth"} |~ "Failed password"

# Logs from last hour with "nginx"
{job="syslog"} |~ "nginx" [1h]
```

### Common Queries

**Show all errors**:
```logql
{job=~".+"} |~ "(?i)error|fail|exception"
```

**Failed SSH attempts**:
```logql
{job="auth"} |~ "Failed password"
```

**Docker container errors**:
```logql
{job="docker"} |~ "(?i)error"
```

**High CPU warnings from syslog**:
```logql
{job="syslog"} |~ "high CPU|cpu threshold"
```

---

## Troubleshooting

### Loki Not Starting

Check logs:
```bash
sudo journalctl -u loki -n 100
```

Common issues:
- **Port conflict**: Check if port 3100 is in use
  ```bash
  sudo lsof -i :3100
  ```
- **Storage permission**: Ensure `/var/lib/loki` is owned by `loki` user
  ```bash
  sudo chown -R loki:loki /var/lib/loki
  ```
- **Disk space**: Check available space
  ```bash
  df -h /var/lib/loki
  ```

### Promtail Not Shipping Logs

Check Promtail status:
```bash
sudo systemctl status promtail
sudo journalctl -u promtail -f
```

Common issues:
- **Can't reach Loki**: Verify network connectivity
  ```bash
  curl http://<rpi5-ip>:3100/ready
  ```
- **Permission denied**: Promtail user needs to read logs
  ```bash
  sudo usermod -aG adm promtail
  sudo systemctl restart promtail
  ```
- **Wrong config**: Check `/etc/promtail/config.yml` has correct Loki URL

### No Logs in Grafana

1. **Check Loki has data**:
   ```bash
   curl -G -s "http://localhost:3100/loki/api/v1/query" \
     --data-urlencode 'query={job=~".+"}' | jq
   ```

2. **Verify Grafana datasource**:
   - Go to Configuration → Data Sources → Loki
   - Click "Test" - should see "Data source is working"

3. **Check time range**: Logs might be outside selected time range

4. **Verify Promtail is running** on nodes:
   ```bash
   ansible cluster_nodes -i inventory.ini -m shell -a "systemctl is-active promtail"
   ```

### High Disk Usage

Check log volume:
```bash
du -sh /var/lib/loki/*
```

Solutions:
- Reduce retention period (see Configuration above)
- Move to external storage
- Reduce log verbosity on applications
- Add log filtering in Promtail

---

## Performance Tuning

### For Small RPi5 (4GB RAM)

Keep default settings - optimized for low resource use.

### For Larger Setup (8GB RAM, many nodes)

Edit `/etc/loki/loki-local.yml`:

```yaml
limits_config:
  ingestion_rate_mb: 30  # Increase from 20
  ingestion_burst_size_mb: 50  # Increase from 30
  max_streams_per_user: 20000  # Increase from 10000
```

Restart Loki after changes.

---

## Log-Based Alerting

### Add Loki Ruler (Optional)

Loki can generate alerts from log patterns.

**Example alert rule** (`/var/lib/loki/rules/alerts.yml`):

```yaml
groups:
  - name: error_alerts
    interval: 1m
    rules:
      - alert: HighErrorRate
        expr: |
          sum by (host) (
            rate({job=~".+"} |~ "(?i)error" [5m])
          ) > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High error rate on {{ $labels.host }}"
          description: "{{ $labels.host }} is logging errors at {{ $value }} per second"
```

Reload Loki ruler:
```bash
curl -X POST http://localhost:3100/loki/api/v1/rules
```

---

## Backup and Restore

### Backup Loki Data

```bash
# Stop Loki
sudo systemctl stop loki

# Backup data directory
sudo tar -czf loki-backup-$(date +%Y%m%d).tar.gz /var/lib/loki

# Start Loki
sudo systemctl start loki
```

### Restore from Backup

```bash
# Stop Loki
sudo systemctl stop loki

# Restore data
sudo tar -xzf loki-backup-YYYYMMDD.tar.gz -C /

# Fix permissions
sudo chown -R loki:loki /var/lib/loki

# Start Loki
sudo systemctl start loki
```

---

## Uninstall

### Remove Loki from RPI5

```bash
# Stop and disable service
sudo systemctl stop loki
sudo systemctl disable loki

# Remove files
sudo rm /etc/systemd/system/loki.service
sudo rm -rf /etc/loki
sudo rm -rf /var/lib/loki
sudo rm /usr/local/bin/loki

# Remove user
sudo userdel loki

# Reload systemd
sudo systemctl daemon-reload
```

### Remove Promtail from Nodes

```bash
# On each node
sudo systemctl stop promtail
sudo systemctl disable promtail
sudo rm /etc/systemd/system/promtail.service
sudo rm -rf /etc/promtail
sudo rm -rf /var/lib/promtail
sudo rm /usr/local/bin/promtail
sudo userdel promtail
sudo systemctl daemon-reload
```

---

## FAQ

**Q: Does this replace Prometheus?**
A: No! Loki is for **logs**, Prometheus is for **metrics**. They complement each other.

**Q: Can I use Loki without Promtail?**
A: Technically yes, but Promtail is the recommended way to ship logs to Loki.

**Q: Will this slow down my RPi5?**
A: Loki is lightweight. With default settings, it uses ~200-500 MB RAM and minimal CPU.

**Q: Can I query logs older than 7 days?**
A: Only if you increase retention. Older logs are automatically deleted.

**Q: Can I ship logs from Windows/macOS?**
A: Promtail supports Windows and macOS, but this guide focuses on Linux clusters.

**Q: Is this production-ready?**
A: This is optimized for homelab use. For production, consider Grafana Cloud or enterprise Loki.

---

## Additional Resources

- **Loki Documentation**: https://grafana.com/docs/loki/latest/
- **LogQL Query Language**: https://grafana.com/docs/loki/latest/logql/
- **Promtail Configuration**: https://grafana.com/docs/loki/latest/clients/promtail/
- **Grafana Explore**: https://grafana.com/docs/grafana/latest/explore/

---

## Summary

After following this guide, you'll have:

- ✅ Loki running on RPI5 for log aggregation
- ✅ Promtail on all cluster nodes shipping logs
- ✅ Grafana datasource configured
- ✅ Log dashboard for visualization
- ✅ Centralized log search and filtering
- ✅ Foundation for log-based alerting

**Result**: Complete observability with both metrics (Prometheus) and logs (Loki)!

---

**Last Updated**: 2025-11-12
**Version**: 1.0
**Tested On**: RPI5 Ubuntu 22.04, Raspbian Bookworm
