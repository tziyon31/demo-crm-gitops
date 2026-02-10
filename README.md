# Demo CRM GitOps Repository

> **Production-ready GitOps repository** for managing Demo CRM infrastructure, applications, monitoring, and observability using ArgoCD

[![ArgoCD](https://img.shields.io/badge/ArgoCD-2.8+-EF4444?logo=argo)](https://argo-cd.readthedocs.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.24+-326CE5?logo=kubernetes)](https://kubernetes.io/)
[![Helm](https://img.shields.io/badge/Helm-3.8+-0F1689?logo=helm)](https://helm.sh/)
[![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-E6522C?logo=prometheus)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-Dashboards-F46800?logo=grafana)](https://grafana.com/)
[![Elasticsearch](https://img.shields.io/badge/Elasticsearch-Logging-005571?logo=elasticsearch)](https://www.elastic.co/elasticsearch/)
[![Fluent Bit](https://img.shields.io/badge/Fluent_Bit-Log_Forwarder-49BDA5?logo=fluentbit)](https://fluentbit.io/)
[![Kibana](https://img.shields.io/badge/Kibana-Dashboards-E8478B?logo=kibana)](https://www.elastic.co/kibana/)

---

## 🎯 Overview

This repository implements a **GitOps-based infrastructure, application, logging, and monitoring management system** using ArgoCD.

It follows the **App of Apps pattern** with a clean hierarchy for managing:

- Infrastructure components (ingress, TLS, database)
- Applications (Demo CRM, Rando, Juice Shop)
- Logging stack (EFK — Elasticsearch, Fluent Bit, Kibana)
- Monitoring stack (Prometheus & Grafana)
- Dashboards and alerting

### Key Features

- ✅ Production-ready deployment configuration
- ✅ High availability with configurable replicas
- ✅ Automatic TLS via cert-manager + Let's Encrypt
- ✅ MongoDB replica set via Community Operator
- ✅ GitOps-ready ArgoCD deployment with self-healing
- ✅ Centralized logging via EFK stack
- ✅ Full cluster monitoring via Prometheus + Grafana
- ✅ Tiered dashboard design for operations visibility
- ✅ Configurable resources and scaling

---

## 📁 Repository Structure

```
demo-crm-gitops/
├── apps/                                    # ArgoCD Applications
│   ├── app-of-apps.yaml                    # Root application (root-apps)
│   ├── infra-root.yaml                     # Infrastructure root application
│   ├── demo-crm-helm.yaml                 # Demo CRM application
│   ├── demo-crm-juice-shop.yaml           # Juice Shop application
│   ├── rando.yaml                          # Rando application
│   └── infra/                              # Infrastructure applications
│       ├── namespaces-app.yaml
│       ├── ingress-nginx-classic-app.yaml
│       ├── infra-cert-manager.yaml
│       ├── infra-clusterissuer.yaml
│       ├── infra-mongodb-operator.yaml
│       ├── infra-mongodb-community.yaml
│       ├── infra-kube-prometheus-stack.yaml # Monitoring (Prometheus + Grafana)
│       ├── infra-juiceshop-servicemonitor.yaml # ServiceMonitor for Juice Shop
│       ├── infra-elasticsearch.yaml         # Logging backend
│       ├── infra-fluent-bit.yaml            # Log forwarder
│       └── infra-kibana.yaml                # Log visualization
├── infra-apps/                              # Infrastructure charts & manifests
│   ├── charts/                             # Helm charts for infrastructure
│   │   ├── cert-manager/
│   │   ├── community-operator/
│   │   ├── community-operator-crds/
│   │   ├── elasticsearch/
│   │   ├── fluent-bit/
│   │   ├── ingress-nginx-classic/
│   │   └── kibana/
│   ├── manifests/                          # Raw Kubernetes manifests
│   │   ├── cert-manager/
│   │   ├── logging/
│   │   ├── monitoring/                     # ServiceMonitor for Juice Shop
│   │   ├── mongodb/
│   │   └── namespaces/
│   └── values/                             # External Helm chart values
│       ├── juice-shop/
│       └── kube-prometheus-stack/
└── argocd-install/                          # ArgoCD installation manifests
    └── argo-cd/
```

---

## 🏗️ Architecture

### Application Hierarchy

```
root-apps (Root Application)
├── infra-root (Infrastructure Root)
│   ├── infra-namespaces
│   ├── infra-ingress-nginx-classic
│   ├── infra-cert-manager
│   ├── infra-clusterissuer
│   ├── infra-mongodb-operator
│   ├── infra-mongodb-community
│   ├── infra-kube-prometheus-stack    ← Monitoring
│   ├── infra-juiceshop-servicemonitor ← Monitoring
│   ├── infra-elasticsearch            ← Logging
│   ├── infra-fluent-bit               ← Logging
│   └── infra-kibana                   ← Logging
├── demo-crm-helm (Application)
├── demo-crm-juice-shop (Application)
└── rando (Application)
```

### Infrastructure Components

| Component | Purpose | Namespace |
|---|---|---|
| **ingress-nginx-classic** | Ingress controller (Kubernetes community) | `ingress-nginx-classic` |
| **cert-manager** | TLS certificate management | `cert-manager` |
| **ClusterIssuer** | Let's Encrypt certificate issuer | `cert-manager` |
| **MongoDB Operator** | MongoDB Community Operator | `demo-crm` |
| **MongoDB Community** | MongoDB replica set | `demo-crm` |
| **Prometheus** | Metrics collection & alerting rules | `monitoring` |
| **Grafana** | Dashboards, visualization & alerting | `monitoring` |
| **Juice Shop ServiceMonitor** | Exposes Juice Shop metrics to Prometheus | `monitoring` |
| **Elasticsearch** | Search & analytics engine (logging backend) | `logging` |
| **Fluent Bit** | Lightweight log forwarder (DaemonSet) | `logging` |
| **Kibana** | Log visualization & dashboards | `logging` |

---

## 📊 Monitoring Stack (Prometheus & Grafana)

The monitoring stack is deployed using **kube-prometheus-stack** (v65.1.0) in the `monitoring` namespace.

### Prometheus

- Collects metrics from Kubernetes nodes, pods, services, ingress, and applications
- Enables cluster-wide visibility into resource usage and health
- Evaluates alerting rules for proactive incident detection

### Grafana

- Pre-configured dashboards for cluster and application monitoring
- Alerting rules with severity-based routing
- Tiered dashboard design for operations visibility (see [Observability](#-observability--dashboards))

### Service Monitoring

Applications that expose a `/metrics` endpoint (e.g. Juice Shop) are scraped automatically via **ServiceMonitor** resources registered with Prometheus.

---

## 📝 EFK Logging Stack

The EFK (Elasticsearch, Fluent Bit, Kibana) stack is deployed in the `logging` namespace.

### Elasticsearch

- Single-node cluster with TLS enabled
- Stores all cluster logs collected by Fluent Bit

### Fluent Bit

- DaemonSet collecting container and systemd logs from every node
- Uses polling mode (`Inotify_Watcher Off`) for broad platform compatibility

### Kibana

- Connects to Elasticsearch via service account token
- **PostSync hook Job** automatically:
  1. Creates an Elasticsearch service account token
  2. Stores it in a Kubernetes secret
  3. Configures role mappings
  4. Restarts Kibana to pick up the token

---

## 🚀 Quick Start

### Prerequisites

- Kubernetes cluster (1.24+)
- ArgoCD installed and configured
- Access to the cluster with `kubectl`

### Installation

1. **Bootstrap ArgoCD** (if not already installed):

   ```bash
   kubectl apply -f argocd-install/
   ```

2. **Create root-apps Application**:

   ```bash
   kubectl apply -f apps/app-of-apps.yaml
   ```

3. **Verify deployment**:

   ```bash
   kubectl -n argocd get applications
   ```

4. **Verify monitoring stack**:

   ```bash
   kubectl get pods -n monitoring
   ```

5. **Verify logging stack**:

   ```bash
   kubectl get pods -n logging
   ```

### Sync Operations

```bash
# Sync all applications
argocd app sync root-apps

# Sync with prune (use with caution)
argocd app sync root-apps --prune

# Check application status
argocd app get root-apps
```

---

## ⚙️ Configuration

### Root Application (`root-apps`)

| Setting | Value |
|---|---|
| Path | `apps/` |
| Recurse | `false` (explicit include pattern) |
| Includes | `infra-root.yaml`, `demo-crm-helm.yaml`, `demo-crm-juice-shop.yaml`, `rando.yaml` |
| Sync Policy | Automated with self-healing (prune disabled for safety) |

### Infrastructure Root (`infra-root`)

| Setting | Value |
|---|---|
| Path | `apps/infra/` |
| Recurse | `true` (discovers all infra applications) |
| Sync Policy | Automated with prune and self-healing |

---

## 🔒 Security & Best Practices

- **Prune Policy** — Only enabled on infrastructure root (`infra-root`); disabled on `root-apps` for safety
- **Self-Healing** — Enabled on all applications for automatic drift recovery
- **Namespace Isolation** — Each component deployed to a dedicated namespace
- **TLS/HTTPS** — Automatic certificate management via cert-manager + Let's Encrypt
- **Elasticsearch TLS** — Internal communication secured with auto-generated certificates
- **Kibana Encryption** — `xpack.encryptedSavedObjects.encryptionKey` set for alerting and saved object encryption
- **ServerSideApply** — Enabled on Prometheus stack and Juice Shop to avoid field-manager conflicts

---

## 📊 Observability & Dashboards

The system provides both **metrics monitoring** (Prometheus → Grafana) and **centralized logging** (Fluent Bit → Elasticsearch → Kibana).

Dashboards follow the **Tiered Dashboard Design** approach.

### Tier 1 — High-Level Overview

- Cluster health and node status
- Total log volume over time
- Traffic trends across namespaces
- Quick visibility into cluster-wide behavior

### Tier 2 — Operations Dashboard

Used by DevOps / on-call engineers for deeper analysis:

- CPU saturation and memory usage
- Disk space utilization
- Request rate, error rate, latency (RED metrics)
- Log activity comparison across namespaces
- Color distribution of Rando events

### Alerting Dashboard

Dedicated dashboard with metrics that trigger alerts:

| Severity | Example Condition |
|---|---|
| **Critical** | CPU > 95% |
| **Warning** | Disk free < 20% |
| **Info** | Error rate > 1% |

Alerts are evaluated periodically by Grafana and routed via configured contact points.

### Error Investigation Panels

**Discover-based panels** embedded in Kibana dashboards allow:

- Viewing the latest logs directly
- Sorting by timestamp
- Quickly identifying recent errors
- Correlating events across namespaces and components

Used as a **fast troubleshooting entry point** instead of navigating manually to Discover.

### Red Event Percentage Metric

A custom metric visualization calculates the percentage of logs containing `rando_details.color = "red"`:

```
count(kql='rando_details.color:"red"') / count()
```

- Displays the percentage value with color thresholds
- Turns red when exceeding alert thresholds
- Provides an immediate signal of abnormal red-event rates

### Pending Dashboard Enhancements

- Traffic split visualization between Demo CRM and Kibana via ingress logs
- Unique MongoDB log counting visualization

---

## 📈 Application Status

```bash
# List all applications
kubectl -n argocd get applications

# View application tree
argocd app get root-apps --show-tree

# Check sync status
argocd app list

# Check monitoring stack
kubectl get pods -n monitoring

# Check logging stack
kubectl get pods -n logging
```

---

## 🔄 GitOps Workflow

1. **Make changes** to manifests/charts in this repository
2. **Commit and push** to the main branch
3. **ArgoCD detects** changes automatically
4. **Applications sync** based on sync policy
5. **Monitor** via ArgoCD UI, Grafana dashboards, and Kibana logs

---

## 🛠️ Troubleshooting

### Application Not Syncing

```bash
# Check application status
argocd app get <app-name>

# Force refresh
argocd app get <app-name> --refresh

# View application logs
argocd app logs <app-name>
```

### Monitoring Issues

```bash
# Check monitoring pods
kubectl get pods -n monitoring

# Prometheus logs
kubectl logs -n monitoring -l app.kubernetes.io/name=prometheus --tail=50

# Grafana logs
kubectl logs -n monitoring -l app.kubernetes.io/name=grafana --tail=50
```

### Logging Issues

```bash
# Elasticsearch health
kubectl exec -n logging elasticsearch-master-0 -- curl -sk https://localhost:9200/_cluster/health?pretty

# Fluent Bit pods
kubectl get pods -n logging -l app.kubernetes.io/name=fluent-bit

# Kibana logs
kubectl logs -n logging -l app=kibana --tail=50

# Kibana setup hook job
kubectl get jobs -n logging
```

### General Infrastructure

```bash
# Check infra applications
kubectl -n argocd get applications | grep infra

# Verify resources in a namespace
kubectl get all -n <namespace>
```

---

## 📚 Related Repositories

- **[demo-crm-helm](https://github.com/tziyon31/demo-crm-helm)** — Helm chart for the Demo CRM application

---

**Maintained by**: [tziyon31](https://github.com/tziyon31)  
**Repository**: [demo-crm-gitops](https://github.com/tziyon31/demo-crm-gitops)
