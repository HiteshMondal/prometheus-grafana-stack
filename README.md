# Production Prometheus & Grafana Monitoring Stack

A production-ready, enterprise-grade monitoring solution using Prometheus and Grafana with comprehensive alerting, service discovery, and visualization capabilities.

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Directory Structure](#directory-structure)
- [Configuration](#configuration)
- [Dashboard Import](#dashboard-import)
- [Security Best Practices](#security-best-practices)
- [Scaling & High Availability](#scaling--high-availability)
- [Monitoring Best Practices](#monitoring-best-practices)
- [Troubleshooting](#troubleshooting)

## 🏗 Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Monitoring Stack                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐ │
│  │   Grafana    │◄───│  Prometheus  │◄───│  Exporters   │ │
│  │  (Port 3000) │    │  (Port 9090) │    │   Various    │ │
│  └──────────────┘    └──────┬───────┘    └──────────────┘ │
│                              │                              │
│                              ▼                              │
│                     ┌──────────────┐                        │
│                     │ AlertManager │                        │
│                     │  (Port 9093) │                        │
│                     └──────────────┘                        │
│                              │                              │
│              ┌───────────────┼───────────────┐             │
│              ▼               ▼               ▼             │
│          Email          Slack         PagerDuty            │
└─────────────────────────────────────────────────────────────┘
```

## ✨ Features

### Core Components
- **Prometheus 2.47.0**: Time-series database and monitoring engine
- **Grafana 10.1.5**: Advanced visualization and dashboards
- **AlertManager 0.26.0**: Alert routing and management
- **Node Exporter**: Hardware and OS metrics
- **cAdvisor**: Container metrics
- **Blackbox Exporter**: Endpoint monitoring (HTTP/HTTPS/TCP/ICMP)
- **Pushgateway**: Batch job metrics collection

### Advanced Features
- ✅ Multi-level alert routing (Critical, Warning, Info)
- ✅ Team-based alert routing (Infrastructure, Database, Security, etc.)
- ✅ Business hours alert suppression
- ✅ Alert inhibition rules to reduce noise
- ✅ SSL certificate monitoring
- ✅ Service discovery support
- ✅ High-performance TSDB with 30-day retention
- ✅ Remote write capability for long-term storage
- ✅ Pre-configured production-grade alerts
- ✅ Multi-channel notifications (Email, Slack, PagerDuty)

## 📦 Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- 4GB RAM minimum (8GB recommended)
- 50GB disk space minimum
- Linux/macOS/Windows with WSL2

## 🚀 Quick Start

### 1. Clone and Setup

```bash
# Create project directory
mkdir prometheus-grafana-stack && cd prometheus-grafana-stack

# Create directory structure
mkdir -p prometheus/{alerts,rules,targets}
mkdir -p grafana/{provisioning/{datasources,dashboards},dashboards}
mkdir -p alertmanager
mkdir -p blackbox

# Download configurations (save the YAML files provided above)
```

### 2. Configure AlertManager

Edit `alertmanager/alertmanager.yml` and update:

```yaml
global:
  smtp_auth_username: 'your_email@gmail.com'
  smtp_auth_password: 'your_app_password'  # Use Gmail App Password
  slack_api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
```

**Gmail App Password Setup:**
1. Enable 2FA on your Google account
2. Go to Google Account > Security > App passwords
3. Generate a new app password for "Mail"
4. Use this password in the configuration

### 3. Launch the Stack

```bash
# Start all services
docker-compose up -d

# Verify all containers are running
docker-compose ps

# Check logs
docker-compose logs -f
```

### 4. Access Services

| Service | URL | Default Credentials |
|---------|-----|---------------------|
| Grafana | http://localhost:3000 | admin / admin_change_me |
| Prometheus | http://localhost:9090 | None |
| AlertManager | http://localhost:9093 | None |
| Node Exporter | http://localhost:9100 | None |
| cAdvisor | http://localhost:8080 | None |

## 📁 Directory Structure

```
.
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml          # Main Prometheus config
│   ├── alerts/
│   │   └── alerts.yml         # Alert rules
│   ├── rules/
│   │   └── recording_rules.yml # Recording rules
│   └── targets/
│       └── services.json      # File-based service discovery
├── grafana/
│   ├── provisioning/
│   │   ├── datasources/
│   │   │   └── datasource.yml
│   │   └── dashboards/
│   │       └── default.yml
│   └── dashboards/            # JSON dashboard files
├── alertmanager/
│   └── alertmanager.yml       # AlertManager config
└── blackbox/
    └── blackbox.yml           # Blackbox exporter config
```

## ⚙️ Configuration

### Add Custom Targets

Create `prometheus/targets/services.json`:

```json
[
  {
    "targets": ["api-server-1:8080", "api-server-2:8080"],
    "labels": {
      "job": "api-server",
      "environment": "production",
      "team": "backend"
    }
  },
  {
    "targets": ["db-server-1:9187"],
    "labels": {
      "job": "postgres",
      "environment": "production",
      "team": "database"
    }
  }
]
```

### Reload Configuration

```bash
# Reload Prometheus configuration
curl -X POST http://localhost:9090/-/reload

# Reload AlertManager configuration
curl -X POST http://localhost:9093/-/reload
```

## 📊 Dashboard Import

### Recommended Grafana Dashboards

Import these community dashboards via Grafana UI (Dashboard > Import):

1. **Node Exporter Full** - ID: `1860`
   - Comprehensive system metrics

2. **Docker Container & Host Metrics** - ID: `11600`
   - Container and host monitoring

3. **Blackbox Exporter** - ID: `7587`
   - Endpoint monitoring

4. **Prometheus Stats** - ID: `2`
   - Prometheus performance metrics

5. **AlertManager** - ID: `9578`
   - AlertManager overview

### Import Steps:
1. Login to Grafana (http://localhost:3000)
2. Navigate to Dashboards > Import
3. Enter dashboard ID
4. Select "Prometheus" as data source
5. Click Import

## 🔒 Security Best Practices

### 1. Change Default Credentials

```bash
# Update in docker-compose.yml
GF_SECURITY_ADMIN_PASSWORD=<strong-password>
```

### 2. Enable HTTPS

Use a reverse proxy (Nginx/Traefik) with SSL certificates:

```nginx
server {
    listen 443 ssl http2;
    server_name grafana.example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 3. Network Isolation

```yaml
# In docker-compose.yml, restrict external access
services:
  prometheus:
    ports:
      - "127.0.0.1:9090:9090"  # Only localhost
```

### 4. Authentication

Enable Prometheus basic auth:

```yaml
# prometheus.yml
global:
  external_labels:
    cluster: 'production'

basic_auth_users:
  admin: <bcrypt-hashed-password>
```

### 5. Backup Strategy

```bash
# Backup Prometheus data
tar -czf prometheus-backup-$(date +%Y%m%d).tar.gz \
  /var/lib/docker/volumes/prometheus_data

# Backup Grafana dashboards
tar -czf grafana-backup-$(date +%Y%m%d).tar.gz \
  /var/lib/docker/volumes/grafana_data
```

## 📈 Scaling & High Availability

### Prometheus High Availability

Deploy multiple Prometheus instances with identical configurations:

```yaml
# docker-compose.ha.yml
services:
  prometheus-1:
    image: prom/prometheus:v2.47.0
    # ... configuration
  
  prometheus-2:
    image: prom/prometheus:v2.47.0
    # ... configuration (identical)
```

Use Thanos or Cortex for:
- Global query view
- Long-term storage
- Horizontal scalability

### Grafana High Availability

1. Use external database (PostgreSQL/MySQL):

```yaml
services:
  grafana:
    environment:
      - GF_DATABASE_TYPE=postgres
      - GF_DATABASE_HOST=postgres:5432
      - GF_DATABASE_NAME=grafana
      - GF_DATABASE_USER=grafana
      - GF_DATABASE_PASSWORD=secret
```

2. Deploy multiple Grafana instances behind a load balancer

### Resource Allocation

**Recommended resources per service:**

| Service | CPU | Memory | Storage |
|---------|-----|--------|---------|
| Prometheus | 2 cores | 4GB | 50GB SSD |
| Grafana | 1 core | 2GB | 10GB |
| AlertManager | 0.5 core | 512MB | 5GB |
| Exporters | 0.25 core | 256MB | 1GB |

## 📚 Monitoring Best Practices

### 1. Alert Design Principles

- **Alert on symptoms, not causes**: Monitor user-facing issues
- **Use multiple severity levels**: Critical, Warning, Info
- **Set appropriate thresholds**: Based on SLOs/SLAs
- **Include context**: Clear descriptions and runbook links
- **Reduce noise**: Use inhibition rules

### 2. Metric Naming Convention

```
<namespace>_<subsystem>_<metric_name>_<unit>

Examples:
- http_requests_total
- node_cpu_seconds_total
- app_response_time_seconds
```

### 3. Label Best Practices

```yaml
# Good labels (low cardinality)
environment: production
region: us-east-1
team: backend

# Avoid high cardinality labels
user_id: 12345  # ❌ Too many unique values
request_id: abc123  # ❌ Unique per request
```

### 4. Recording Rules

Create recording rules for expensive queries:

```yaml
# prometheus/rules/recording_rules.yml
groups:
  - name: api_metrics
    interval: 30s
    rules:
      - record: job:api_requests_per_second:rate5m
        expr: rate(http_requests_total[5m])
      
      - record: job:api_error_rate:rate5m
        expr: |
          rate(http_requests_total{status=~"5.."}[5m])
          /
          rate(http_requests_total[5m])
```

### 5. Dashboard Design

- **Use template variables**: For dynamic filtering
- **Show time ranges**: 1h, 6h, 24h, 7d options
- **Include SLO/SLA indicators**: Visual thresholds
- **Group related metrics**: Logical panel organization
- **Add annotations**: For deployments and incidents

## 🔧 Troubleshooting

### Prometheus Not Scraping Targets

```bash
# Check target status
curl http://localhost:9090/api/v1/targets

# Verify network connectivity
docker exec -it prometheus wget -O- http://node-exporter:9100/metrics

# Check Prometheus logs
docker-compose logs prometheus
```

### High Memory Usage

```yaml
# Adjust retention and chunk size in prometheus.yml
command:
  - '--storage.tsdb.retention.time=15d'
  - '--storage.tsdb.retention.size=5GB'
```

### AlertManager Not Sending Alerts

```bash
# Test AlertManager configuration
docker exec -it alertmanager amtool config routes test

# Check alert status
curl http://localhost:9093/api/v2/alerts

# Verify SMTP settings
docker-compose logs alertmanager | grep -i smtp
```

### Grafana Dashboard Not Loading

```bash
# Check Grafana logs
docker-compose logs grafana

# Verify datasource connection
curl -u admin:admin http://localhost:3000/api/datasources

# Test Prometheus connectivity from Grafana container
docker exec -it grafana wget -O- http://prometheus:9090/api/v1/status/config
```

### Performance Optimization

```yaml
# Increase query timeout
global:
  query_timeout: 2m

# Adjust scrape intervals for less critical targets
scrape_configs:
  - job_name: 'low-priority'
    scrape_interval: 60s  # Instead of 15s
```

## 🔄 Maintenance Tasks

### Daily
- Monitor alert volume and adjust thresholds
- Review critical alerts and root causes

### Weekly
- Check disk space usage
- Review dashboard usage metrics
- Update alert runbooks

### Monthly
- Update Docker images to latest stable versions
- Review and optimize recording rules
- Clean up unused dashboards
- Review retention policies

### Quarterly
- Audit alert effectiveness
- Capacity planning review
- Disaster recovery testing

## 📞 Production Checklist

Before going to production:

- [ ] Changed default passwords
- [ ] Configured SSL/TLS
- [ ] Set up proper authentication
- [ ] Configured backup strategy
- [ ] Tested alert routing
- [ ] Set up log aggregation
- [ ] Configured retention policies
- [ ] Documented runbooks
- [ ] Tested disaster recovery
- [ ] Set up monitoring for monitoring (meta-monitoring)
- [ ] Configured rate limiting
- [ ] Set up network policies
- [ ] Reviewed security settings
- [ ] Load tested the stack
- [ ] Set up automated health checks

## 📖 Additional Resources

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [AlertManager Documentation](https://prometheus.io/docs/alerting/latest/alertmanager/)
- [PromQL Tutorial](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Grafana Dashboard Best Practices](https://grafana.com/docs/grafana/latest/best-practices/)

## 🤝 Support & Contributing

For issues and questions:
- Check logs: `docker-compose logs -f`
- Review Prometheus targets: http://localhost:9090/targets
- Check AlertManager status: http://localhost:9093

## 📄 License

This monitoring stack configuration is provided as-is for production use.

---

**Made with ❤️ for reliable infrastructure monitoring**