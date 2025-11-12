# Pico-Observability to Pico-Local: Feature Comparison & Recommendations

**Date**: 2025-11-12
**Purpose**: Identify components from pico-observability that can enhance pico-local for RPi5 homelab use

---

## Executive Summary

**pico-observability** is a full SaaS multi-tenant observability platform (still in planning/early dev)
**pico-local** is a local monitoring solution for homelabs, deployed on RPi5 with touchscreen

### Key Findings
- **9 Grafana dashboards** are identical (already in pico-local)
- **Loki log aggregation** is missing from pico-local - HIGH VALUE ADD
- **Docker configs** from pico-observability provide better local dev experience
- **Promtail** for log shipping would complement existing monitoring
- **Simplified UI** concepts could enhance RPi5 touchscreen experience

---

## Feature Comparison Matrix

| Feature | pico-observability | pico-local | Recommendation |
|---------|-------------------|------------|----------------|
| **Metrics Collection** | ✅ Prometheus + Node Exporter | ✅ Prometheus + Node Exporter | ✅ Already in sync |
| **Log Aggregation** | ✅ Loki + Promtail | ❌ Not present | 🔥 **HIGH PRIORITY** - Add Loki/Promtail |
| **Grafana Dashboards** | ✅ 9 dashboards | ✅ Same 9 dashboards | ✅ Already in sync |
| **Alert Rules** | 🚧 Planning stage | ✅ 31 alert rules | ⬅️ pico-local is ahead |
| **Container Metrics** | ✅ cAdvisor | ✅ cAdvisor | ✅ Already in sync |
| **Backend API** | ✅ FastAPI (multi-tenant) | ❌ N/A (local only) | ❌ Not applicable for local |
| **Frontend Web UI** | ✅ Next.js dashboard | ❌ Only Grafana | 💡 **CONSIDER** - Lightweight local UI |
| **Authentication** | ✅ JWT + 2FA + Billing | ❌ Basic Grafana auth | ❌ Not needed for homelab |
| **Ansible Deployment** | 🚧 Detailed planning | ✅ Full implementation | ⬅️ pico-local is ahead |
| **Docker Compose Setup** | ✅ Full stack | ❌ Manual deployment | 🔥 **HIGH PRIORITY** - Local dev |
| **Cluster Management** | ❌ Not planned | ✅ Health checks, backups | ⬅️ pico-local is ahead |
| **Multi-tenancy** | ✅ Core feature | ❌ N/A | ❌ Not applicable |

---

## Recommendations: What to Pull from pico-observability

### 🔥 HIGH PRIORITY - Quick Wins

#### 1. **Loki Log Aggregation** (HIGHEST VALUE)
**Why**: pico-local has metrics but no log aggregation. Loki is lightweight and perfect for RPi5.

**What to copy**:
```
pico-observability/docker/loki/loki-config.yml  → pico-local/monitoring/config/loki/
```

**Adapt for local use**:
- Remove multi-tenancy (`auth_enabled: false`)
- Simplify retention (7-15 days for homelab)
- Local filesystem storage (no S3)

**Value**: Complete observability stack (metrics + logs)

---

#### 2. **Promtail Log Shipper**
**Why**: Collect logs from cluster nodes and ship to Loki

**What to copy**:
- Promtail configuration from `planning/11_ANSIBLE_SCRIPTS.md`
- Adapt the Ansible role for local deployment

**New Ansible playbook needed**:
```
pico-local/monitoring/metrics_collection/install_promtail.ansible
```

**Features to include**:
- System logs (`/var/log/syslog`)
- Docker container logs
- Systemd journal logs
- Application-specific logs

**Value**: Centralized log viewing in Grafana

---

#### 3. **Docker Compose Development Setup**
**Why**: pico-local currently lacks a local dev/test environment

**What to copy**:
```
pico-observability/docker-compose.yml → pico-local/docker-compose.yml
```

**Adapt**:
- Remove backend API service (FastAPI)
- Remove frontend service (Next.js)
- Remove PostgreSQL and Redis (SaaS-only)
- Keep: Prometheus, Loki, Grafana
- Add: Pre-load dashboards into Grafana volume

**Value**: Quick local testing before deploying to RPi5

**Usage**:
```bash
cd pico-local
docker-compose up -d
# Test at http://localhost:3000 (Grafana)
```

---

### 💡 MEDIUM PRIORITY - Enhanced Features

#### 4. **Grafana Provisioning Configuration**
**Why**: Auto-provision datasources and dashboards on startup

**What to copy**:
```
pico-observability/docker/grafana/provisioning/ → pico-local/monitoring/config/grafana/provisioning/
```

**Adapt**:
- Add Loki as datasource
- Auto-load all 9 dashboards
- Set up default home dashboard

**Value**: Zero-config Grafana on RPi5 boot

---

#### 5. **Loki Log Dashboard for Grafana**
**Why**: Currently no dashboard for log visualization

**Create new dashboard**:
```
pico-local/monitoring/config/grafana/dashboard_logs_overview.json
```

**Features**:
- Recent logs from all nodes
- Log volume over time
- Error/warning detection
- Log search interface
- Per-service log filtering

**Value**: Unified logs + metrics viewing

---

#### 6. **Health Check API (Lightweight)**
**Why**: RPi5 touchscreen could show simple health status

**Create minimal Python/FastAPI service**:
```python
# pico-local/monitoring/healthcheck/api.py
from fastapi import FastAPI
import requests

app = FastAPI()

@app.get("/health")
async def cluster_health():
    """Query Prometheus for cluster status"""
    # Check if Prometheus is up
    # Check if all nodes are reporting
    # Return simple JSON for touchscreen display
    return {
        "status": "healthy",
        "nodes_up": 5,
        "nodes_total": 5,
        "alerts_firing": 0
    }
```

**Value**: Simple API for RPi5 touchscreen dashboard

---

### 🎨 LOW PRIORITY - Nice to Have

#### 7. **Simplified Touchscreen UI**
**Why**: Grafana works but may not be ideal for small touchscreen

**Create**:
```
pico-local/ui/touchscreen-dashboard/
```

**Tech stack**:
- Simple HTML/CSS/JS (no framework needed)
- Query Prometheus API directly
- Show: Cluster health, resource usage, alerts
- Optimized for small screen (7-10 inch)

**Alternative**: Use Grafana Kiosk mode with custom dashboard

**Value**: Better UX on RPi5 touchscreen

---

#### 8. **Email Alert Configuration Templates**
**Why**: pico-observability has email templates; pico-local has Slack/email alerts

**What to copy**:
```
pico-observability/backend/app/templates/emails/ → pico-local/cluster-management/email-templates/
```

**Adapt**:
- Remove SaaS-specific branding
- Add homelab-friendly language
- Use with AlertManager email notifications

**Value**: Professional-looking alert emails

---

## What NOT to Pull (SaaS-Specific)

### ❌ Skip These Components

1. **Backend API (FastAPI)** - Multi-tenant, billing, user management (not needed for local)
2. **Frontend Web UI (Next.js)** - User registration, subscription management (not needed)
3. **Database (PostgreSQL)** - User accounts, billing data (not needed)
4. **Redis Cache** - API caching (not needed)
5. **Stripe Integration** - Payments (not needed)
6. **JWT Authentication** - Multi-user auth (Grafana auth is sufficient)
7. **Multi-tenancy Configuration** - Tenant isolation (single homelab)

---

## Architecture Comparison

### pico-observability (SaaS)
```
Customer → Next.js Frontend → FastAPI Backend → PostgreSQL
                                ↓
                    Prometheus (multi-tenant)
                    Loki (multi-tenant)
                    Grafana (multi-org)
                                ↓
                            S3 Storage
```

### pico-local (Current)
```
Cluster Nodes → Node Exporter → Prometheus → Grafana
                  ↓
              Alert Rules → AlertManager → Slack/Email
```

### pico-local (Recommended with Loki)
```
Cluster Nodes → Node Exporter → Prometheus ┐
                                            ├→ Grafana → RPi5 Touchscreen
Cluster Nodes → Promtail → Loki ───────────┘
                  ↓
              Alert Rules → AlertManager → Slack/Email
```

---

## Implementation Roadmap

### Phase 1: Loki Integration (2-3 hours)
- [ ] Copy Loki configuration from pico-observability
- [ ] Adapt for single-tenant local use
- [ ] Create Ansible playbook for RPi5 Loki installation
- [ ] Create Promtail Ansible role for cluster nodes
- [ ] Add Loki datasource to Grafana provisioning
- [ ] Test log ingestion

### Phase 2: Docker Compose Dev Environment (1 hour)
- [ ] Adapt docker-compose.yml for local dev
- [ ] Remove SaaS components
- [ ] Add dashboard provisioning
- [ ] Document usage in README

### Phase 3: Log Dashboard (2 hours)
- [ ] Create Grafana dashboard for log visualization
- [ ] Add log volume panels
- [ ] Add error/warning filtering
- [ ] Integrate with existing dashboards

### Phase 4: Health Check API (Optional, 3-4 hours)
- [ ] Create lightweight FastAPI service
- [ ] Query Prometheus for cluster status
- [ ] Create simple touchscreen UI
- [ ] Deploy to RPi5

---

## File Mapping: What to Copy

| Source (pico-observability) | Destination (pico-local) | Notes |
|----------------------------|--------------------------|-------|
| `docker/loki/loki-config.yml` | `monitoring/config/loki/loki-local.yml` | Remove auth, simplify |
| `docker/prometheus/prometheus.yml` | Reference only | Already have better version |
| `docker-compose.yml` | `docker-compose.yml` | Strip SaaS components |
| `planning/11_ANSIBLE_SCRIPTS.md` (Promtail sections) | `monitoring/metrics_collection/install_promtail.ansible` | Create new playbook |
| `dashboards/templates/*.json` | Already identical | No action needed |

---

## Storage Requirements

### Current (pico-local)
- Prometheus data: ~5-10 GB (15-day retention)
- Grafana config: ~100 MB

### With Loki (Estimated)
- Loki logs: ~2-5 GB per day (depends on log volume)
- Recommended retention: 7 days = ~14-35 GB
- **Total**: ~20-45 GB for full stack

### RPi5 Capacity
- Typical setup: 256 GB+ SD card or SSD
- **Recommendation**: Use external SSD for log storage

---

## Benefits Summary

### Adding Loki to pico-local:
1. ✅ Complete observability (metrics + logs)
2. ✅ Troubleshoot issues with log context
3. ✅ Correlate logs with metrics in single Grafana view
4. ✅ Log-based alerting (error patterns)
5. ✅ Centralized log search across cluster
6. ✅ Still lightweight for RPi5

### Adding Docker Compose:
1. ✅ Quick local development/testing
2. ✅ Test changes before deploying to cluster
3. ✅ Onboard new contributors faster

### Optional Health Check API:
1. ✅ Touchscreen-friendly status display
2. ✅ Simple REST API for integrations
3. ✅ Custom dashboards beyond Grafana

---

## Cost-Benefit Analysis

| Component | Implementation Effort | Value to Homelab | Priority |
|-----------|----------------------|------------------|----------|
| Loki + Promtail | 3-4 hours | 🔥🔥🔥🔥🔥 Very High | 1 |
| Docker Compose | 1 hour | 🔥🔥🔥 Medium-High | 2 |
| Grafana Provisioning | 30 min | 🔥🔥🔥 Medium-High | 3 |
| Log Dashboard | 2 hours | 🔥🔥🔥 Medium-High | 4 |
| Health Check API | 4 hours | 🔥🔥 Medium | 5 |
| Touchscreen UI | 8+ hours | 🔥 Low-Medium | 6 |

---

## Questions to Consider

1. **Storage**: Do you have enough storage on RPi5 for logs? (Recommend external SSD)
2. **Use Case**: Do you need log aggregation, or are metrics enough?
3. **Touchscreen**: What size touchscreen? (affects UI design)
4. **Complexity**: Want to keep it simple, or add advanced features?
5. **Maintenance**: Who will maintain this? (Docker Compose is easier for updates)

---

## Next Steps

### Recommended Action Plan:

1. **Start with Loki** (highest value, relatively simple)
   - Copy configuration
   - Deploy to RPi5
   - Create Promtail Ansible playbook
   - Test with one node

2. **Add Docker Compose** (fast win)
   - Adapt docker-compose.yml
   - Document in README
   - Use for future testing

3. **Create Log Dashboard** (complete the stack)
   - Design in Grafana
   - Export JSON
   - Add to provisioning

4. **Evaluate touchscreen needs** (based on user feedback)
   - Test Grafana on touchscreen first
   - If inadequate, build custom UI

---

## Conclusion

**Key Takeaway**: The biggest gap in pico-local is **log aggregation**. Adding Loki + Promtail from pico-observability would complete the observability stack while staying lightweight for RPi5.

**Recommended First Step**: Copy Loki configuration and create Promtail deployment playbook. This gives you the most value with minimal complexity.

**Long-term Vision**: pico-local becomes a complete homelab monitoring solution with:
- ✅ Metrics (Prometheus)
- ✅ Logs (Loki) ← **Add this**
- ✅ Visualization (Grafana)
- ✅ Alerting (AlertManager)
- ✅ Easy deployment (Ansible)
- ✅ Local dev environment (Docker Compose) ← **Add this**

---

**Document prepared by**: Claude
**Review Status**: Draft for discussion
**Version**: 1.0
