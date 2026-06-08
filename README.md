# Prometheus & Grafana Monitoring Stack 📊

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F27830?style=flat&logo=grafana&logoColor=white)

Complete **monitoring stack** setup with Prometheus for metrics collection, Grafana for visualization, and AlertManager for incident management. Includes pre-built dashboards and alert rules.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Dashboards](#dashboards)
- [Alerting](#alerting)
- [Troubleshooting](#troubleshooting)

## 🎯 Overview

Enterprise-grade monitoring setup providing:
- **Metrics Collection** - Prometheus scraping
- **Visualization** - Grafana dashboards
- **Alerting** - Prometheus AlertManager
- **Service Discovery** - Kubernetes & Consul support
- **Pre-built Dashboards** - Kubernetes, Application, Infrastructure

## ✨ Features

✅ **High Availability** - Replicated Prometheus instances  
✅ **Service Discovery** - Automatic target discovery  
✅ **Custom Dashboards** - Pre-built & ready-to-use  
✅ **Alert Rules** - Production-ready alert definitions  
✅ **RBAC** - Team-based access control  
✅ **Data Retention** - Configurable storage policies  
✅ **Backup & Restore** - Data protection strategy  
✅ **Multi-Cluster** - Monitor multiple K8s clusters  
✅ **Docker & Kubernetes** - Full container support  
✅ **Notification System** - Slack, PagerDuty, Email  

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│          Monitoring Stack                           │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌─ Data Collection Layer ──────────────────────┐  │
│  │                                               │  │
│  │ ┌───────────────┐  ┌────────────────────┐   │  │
│  │ │ Prometheus 1  │  │ Prometheus 2       │   │  │
│  │ │ (Primary)     │  │ (Secondary)        │   │  │
│  │ └───────────────┘  └────────────────────┘   │  │
│  │                                               │  │
│  └───────────────────────────────────────────────┘  │
│                   │                                  │
│  ┌────────────────▼───────────────────────────┐    │
│  │  Remote Storage (Long-term retention)      │    │
│  │  - Thanos                                  │    │
│  │  - S3 Compatible Storage                   │    │
│  └───────────────────────────────────────────┘    │
│                   │                                │
│  ┌────────────────▼───────────────────────────┐   │
│  │  Visualization & Analysis                  │   │
│  │  ┌──────────────────────────────────────┐ │   │
│  │  │     Grafana Dashboard                │ │   │
│  │  │  - Kubernetes Metrics                │ │   │
│  │  │  - Application Performance           │ │   │
│  │  │  - Infrastructure Health             │ │   │
│  │  └──────────────────────────────────────┘ │   │
│  └───────────────────────────────────────────┘   │
│                   │                               │
│  ┌────────────────▼───────────────────────────┐  │
│  │  Alerting Layer                            │  │
│  │  ┌──────────────────────────────────────┐ │  │
│  │  │   AlertManager                       │ │  │
│  │  │  - Alert Routing                     │ │  │
│  │  │  - Slack Integration                 │ │  │
│  │  │  - PagerDuty Integration            │ │  │
│  │  └──────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────┘  │
│                                                  │
└─────────────────────────────────────────────────┘
```

## 🚀 Quick Start

### Docker Compose Setup

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alerts.yml:/etc/prometheus/alerts.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
      GF_USERS_ALLOW_SIGN_UP: false
    volumes:
      - grafana_data:/var/lib/grafana
      - ./provisioning/:/etc/grafana/provisioning/
    depends_on:
      - prometheus
    networks:
      - monitoring

  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
      - alertmanager_data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    networks:
      - monitoring

volumes:
  prometheus_data:
  grafana_data:
  alertmanager_data:

networks:
  monitoring:
```

### Start Stack

```bash
# Launch services
docker-compose up -d

# Verify services
docker-compose ps

# Access services
# Prometheus: http://localhost:9090
# Grafana: http://localhost:3000 (admin/admin)
# AlertManager: http://localhost:9093
```

## 🔧 Configuration

### Prometheus Configuration (`prometheus.yml`)

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'production'
    environment: 'prod'

# AlertManager Configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']

# Alert Rules Files
rule_files:
  - 'alerts.yml'

scrape_configs:
  # Prometheus Self-Monitoring
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Kubernetes Metrics
  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
      - role: endpoints
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https

  # Node Exporter
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']

  # Docker Daemon
  - job_name: 'docker'
    static_configs:
      - targets: ['localhost:9323']

  # Application Metrics (Optional)
  - job_name: 'application'
    static_configs:
      - targets: ['localhost:8080']
```

### Alert Rules (`alerts.yml`)

```yaml
groups:
  - name: kubernetes
    interval: 30s
    rules:
      - alert: KubernetesNodeNotReady
        expr: kube_node_status_condition{condition="Ready",status="true"} == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Kubernetes node not ready (instance {{ $labels.node }})"
          description: "Node {{ $labels.node }} has been unready for more than 5 minutes."

      - alert: KubernetesPodCrashLooping
        expr: rate(kube_pod_container_status_restarts_total[15m]) > 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Kubernetes pod crash looping (instance {{ $labels.pod }})"
          description: "Pod {{ $labels.namespace }}/{{ $labels.pod }} is crash looping."

  - name: system
    interval: 30s
    rules:
      - alert: HostHighCpuLoad
        expr: (100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Host high CPU load (instance {{ $labels.instance }})"
          description: "CPU load is > 80%"

      - alert: HostMemoryIsUnusuallyHigh
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Host memory is unusually high (instance {{ $labels.instance }})"
          description: "Memory usage is > 80%"

      - alert: HostDiskSpaceRunningOut
        expr: (node_filesystem_avail_bytes{fstype!~"tmpfs"} / node_filesystem_size_bytes) < 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Host disk space running out (instance {{ $labels.instance }})"
          description: "Disk space is running out (< 10%)"

  - name: application
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate (instance {{ $labels.instance }})"
          description: "Error rate is above 5%"

      - alert: HighLatency
        expr: histogram_quantile(0.95, http_request_duration_seconds_bucket) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency (instance {{ $labels.instance }})"
          description: "95th percentile latency is above 1 second"
```

### AlertManager Configuration (`alertmanager.yml`)

```yaml
global:
  resolve_timeout: 5m
  slack_api_url: 'YOUR_SLACK_WEBHOOK_URL'

route:
  receiver: 'default'
  group_by: ['alertname', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  
  routes:
    - receiver: 'critical'
      group_wait: 0s
      match:
        severity: critical
    
    - receiver: 'warning'
      match:
        severity: warning

receivers:
  - name: 'default'
    slack_configs:
      - channel: '#alerts'
        title: '⚠️ {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'critical'
    slack_configs:
      - channel: '#critical-alerts'
        title: '🚨 CRITICAL: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'

  - name: 'warning'
    slack_configs:
      - channel: '#warning-alerts'
        title: '⚠️ WARNING: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
```

## 📊 Pre-built Dashboards

### Kubernetes Cluster Health
- Node Status & Capacity
- Pod Resource Usage
- Cluster CPU & Memory
- Namespace Isolation
- Network Traffic

### Application Performance
- Request Latency (P50, P95, P99)
- Error Rates by Endpoint
- Throughput & QPS
- Database Query Performance
- Cache Hit Rates

### Infrastructure Metrics
- CPU & Memory Usage per Host
- Disk I/O Performance
- Network Traffic Analysis
- System Load Average
- Uptime & Availability

## 🔑 PromQL Query Examples

### Kubernetes Queries

```promql
# Pod CPU usage
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod_name)

# Pod memory usage
sum(container_memory_usage_bytes) by (pod_name)

# Node status
kube_node_status_condition{condition="Ready", status="true"}

# Pod restart count
rate(kube_pod_container_status_restarts_total[15m])
```

### Application Queries

```promql
# Request rate
rate(http_requests_total[5m])

# Error rate
rate(http_requests_total{status=~"5.."}[5m])

# Latency (95th percentile)
histogram_quantile(0.95, http_request_duration_seconds_bucket)

# Active connections
rate(tcp_connections_total[1m])
```

### System Queries

```promql
# CPU usage percentage
(1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m]))) * 100

# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage percentage
(1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100

# Disk I/O
rate(node_disk_io_time_ms[5m])
```

## 📈 Dashboard Best Practices

✅ **Organize by Topic** - Separate dashboards for different concerns  
✅ **Use Variables** - Filter by environment, namespace, pod  
✅ **Enable Annotations** - Mark deployments and events  
✅ **Set Appropriate Intervals** - Balance precision vs. performance  
✅ **Use Panels** - Graphs, tables, gauges, heat maps  
✅ **Add Descriptions** - Explain metrics and thresholds  
✅ **Share Dashboards** - Team collaboration  
✅ **Version Control** - Store dashboard definitions in Git  

## 🔍 Troubleshooting

### Issue: Prometheus targets down

```bash
# Check Prometheus targets
curl http://localhost:9090/api/v1/targets

# Verify scrape configs
curl http://localhost:9090/api/v1/query_range?query=up

# Check logs
docker logs prometheus
```

### Issue: No data in Grafana

```bash
# Verify Prometheus datasource
# Grafana UI → Configuration → Data Sources → Test

# Check metrics in Prometheus
http://localhost:9090/graph

# Verify scrape interval
# Check prometheus.yml scrape_interval
```

### Issue: AlertManager not sending notifications

```bash
# Verify AlertManager config
docker logs alertmanager

# Check alert status
curl http://localhost:9090/api/v1/alerts

# Test Slack webhook
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Test"}' \
  YOUR_SLACK_WEBHOOK_URL
```

## 📚 Resources

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [AlertManager Guide](https://prometheus.io/docs/alerting/latest/alertmanager/)
- [PromQL Documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Node Exporter](https://github.com/prometheus/node_exporter)
- [Kubernetes Metrics](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/)

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Add/improve dashboards and alerts
4. Submit pull request

## 📄 License

MIT License

## 👤 Author

**Dinesh Kumar Gudla**  
DevOps Engineer | Monitoring & Observability Specialist  
📧 [dinesh.kumar2442000@gmail.com](mailto:dinesh.kumar2442000@gmail.com)  
🔗 [LinkedIn](https://www.linkedin.com/in/dineshgudla)  
🐙 [GitHub](https://github.com/DjDinesh1234)

## 📞 Support

For issues, questions, or suggestions:
- Open an [Issue](https://github.com/DjDinesh1234/prometheus-grafana-monitoring-stack/issues)
- Check [Discussions](https://github.com/DjDinesh1234/prometheus-grafana-monitoring-stack/discussions)
- Read [Prometheus Docs](https://prometheus.io/docs/)

---

**Last Updated:** 2026-06-08  
**Status:** ✅ Production Ready
