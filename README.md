# Demo CRM GitOps Repository

> **Production-ready GitOps repository** for managing Demo CRM infrastructure and applications using ArgoCD

[![ArgoCD](https://img.shields.io/badge/ArgoCD-2.8+-EF4444?logo=argo)](https://argo-cd.readthedocs.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.24+-326CE5?logo=kubernetes)](https://kubernetes.io/)
[![Helm](https://img.shields.io/badge/Helm-3.8+-0F1689?logo=helm)](https://helm.sh/)

## 🎯 Overview

This repository implements a **GitOps-based infrastructure and application management** system using ArgoCD. It follows the **App of Apps pattern** with a clean hierarchy for managing infrastructure components and applications.

### Key Features

- ✅ **Production-ready deployment configuration**
- ✅ **High availability** with configurable replicas
- ✅ **Automatic TLS** via cert-manager integration
- ✅ **MongoDB integration** with connection management
- ✅ **Configurable resources and scaling**
- ✅ **GitOps-ready** for ArgoCD deployment

## 📁 Repository Structure

```
demo-crm-gitops/
├── apps/                              # ArgoCD Applications
│   ├── app-of-apps.yaml              # Root application (root-apps)
│   ├── infra-root.yaml               # Infrastructure root application
│   ├── demo-crm-helm.yaml            # Demo CRM application
│   ├── rando.yaml                    # Rando application
│   └── infra/                        # Infrastructure applications
│       ├── infra-cert-manager.yaml
│       ├── infra-clusterissuer.yaml
│       ├── infra-elasticsearch.yaml
│       ├── infra-fluent-bit.yaml
│       ├── infra-kibana.yaml
│       ├── infra-mongodb-operator.yaml
│       ├── infra-mongodb-community.yaml
│       ├── ingress-nginx-classic-app.yaml
│       └── namespaces-app.yaml
├── infra-apps/                       # Infrastructure charts & manifests
│   ├── charts/                       # Helm charts for infrastructure
│   │   ├── cert-manager/
│   │   ├── community-operator/
│   │   ├── community-operator-crds/
│   │   ├── elasticsearch/
│   │   ├── fluent-bit/
│   │   ├── ingress-nginx-classic/
│   │   └── kibana/
│   └── manifests/                    # Kubernetes manifests
│       ├── cert-manager/
│       ├── logging/
│       ├── mongodb/
│       └── namespaces/
└── argocd-install/                    # ArgoCD installation manifests
    └── argo-cd/
```

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
│   ├── infra-elasticsearch
│   ├── infra-fluent-bit
│   └── infra-kibana
├── demo-crm-helm (Application)
└── rando (Application)
```

### Infrastructure Components

| Component | Purpose | Namespace |
|-----------|---------|-----------|
| **ingress-nginx-classic** | Ingress controller (Kubernetes community) | `ingress-nginx-classic` |
| **cert-manager** | TLS certificate management | `cert-manager` |
| **ClusterIssuer** | Let's Encrypt certificate issuer | `cert-manager` |
| **MongoDB Operator** | MongoDB Community Operator | `demo-crm` |
| **MongoDB Community** | MongoDB replica set | `demo-crm` |
| **Elasticsearch** | Search & analytics engine (logging backend) | `logging` |
| **Fluent Bit** | Lightweight log forwarder (DaemonSet) | `logging` |
| **Kibana** | Log visualization & dashboards | `logging` |

### EFK Logging Stack

The EFK (Elasticsearch, Fluent Bit, Kibana) stack is deployed in the `logging` namespace:

- **Elasticsearch** - Single-node cluster with TLS enabled, stores all cluster logs
- **Fluent Bit** - DaemonSet collecting container and systemd logs, using polling mode (`Inotify_Watcher Off`) for broad platform compatibility
- **Kibana** - Connects to Elasticsearch via service account token, with a PostSync hook Job that automatically:
  1. Creates an Elasticsearch service account token
  2. Stores it in a Kubernetes secret
  3. Configures role mappings
  4. Restarts Kibana to pick up the token

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

### Sync Operations

```bash
# Sync all applications
argocd app sync root-apps

# Sync with prune (use with caution)
argocd app sync root-apps --prune

# Check application status
argocd app get root-apps
```

## ⚙️ Configuration

### Root Application (`root-apps`)

- **Path**: `apps/`
- **Recurse**: `false` (explicit include pattern)
- **Includes**: `infra-root.yaml`, `demo-crm-helm.yaml`, `rando.yaml`
- **Sync Policy**: Automated with self-healing (prune disabled for safety)

### Infrastructure Root (`infra-root`)

- **Path**: `apps/infra/`
- **Recurse**: `true` (discovers all infra applications)
- **Sync Policy**: Automated with prune and self-healing

## 🔒 Security & Best Practices

- **Prune Policy**: Only enabled on root applications (`root-apps`, `infra-root`)
- **Self-Healing**: Enabled on all applications for automatic recovery
- **Namespace Isolation**: Each component deployed to dedicated namespaces
- **TLS/HTTPS**: Automatic certificate management via cert-manager
- **Elasticsearch TLS**: Internal communication secured with auto-generated certificates
- **Kibana Encryption**: `xpack.encryptedSavedObjects.encryptionKey` set for alerting and saved object encryption

## 📊 Observability & Dashboards

Kibana dashboards are used to analyze cluster and application logs collected via the EFK stack.
The dashboard was created following the **Tiered Dashboard Design** approach from the course.

### Dashboard Structure

#### Tier 1 – High-Level Metrics

- Total log volume over time
- Log activity trends across namespaces
- Quick visibility into cluster-wide logging behavior

#### Tier 2 – Detailed Analysis Panels

- Color distribution of Rando events
- Logs grouped over time
- Namespace activity comparison
- Discovery table showing latest log entries

#### Error Investigation Panel

A **Discover-based panel** is embedded in the dashboard, allowing:

- Viewing the latest logs directly
- Sorting by timestamp
- Quickly identifying recent errors
- Correlating events across namespaces and components

This panel is used as a **fast troubleshooting entry point** instead of navigating manually to Discover.

#### Red Event Percentage Metric

A metric visualization calculates the percentage of logs containing `rando_details.color = "red"` using:

```
count(kql='rando_details.color:"red"') / count()
```

The metric:
- Displays the percentage value
- Uses **color thresholds**
- Turns red when exceeding alert thresholds
- Provides an immediate signal of abnormal red-event rates

#### Pending Dashboard Enhancements

The following panels are planned but not yet implemented:

- Traffic split visualization between Demo CRM and Kibana via ingress logs
- Unique MongoDB log counting visualization

## 📈 Application Status

Monitor your applications:

```bash
# List all applications
kubectl -n argocd get applications

# View application tree
argocd app get root-apps --show-tree

# Check sync status
argocd app list

# Check EFK stack
kubectl get pods -n logging
```

## 🔄 GitOps Workflow

1. **Make changes** to manifests/charts in this repository
2. **Commit and push** to the main branch
3. **ArgoCD detects** changes automatically
4. **Applications sync** based on sync policy
5. **Monitor** via ArgoCD UI or CLI

## 📚 Related Repositories

- **[demo-crm-helm](https://github.com/tziyon31/demo-crm-helm)** - Helm chart for Demo CRM application

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

### Infrastructure Issues

```bash
# Check infra applications
kubectl -n argocd get applications | grep infra

# Verify resources
kubectl get all -n <namespace>
```

### EFK Stack Issues

```bash
# Check Elasticsearch health
kubectl exec -n logging elasticsearch-master-0 -- curl -sk https://localhost:9200/_cluster/health?pretty

# Check Fluent Bit pods
kubectl get pods -n logging -l app.kubernetes.io/name=fluent-bit

# Check Kibana logs
kubectl logs -n logging -l app=kibana --tail=50

# Check Kibana setup hook job
kubectl get jobs -n logging
```

## 📝 License

This project is part of a learning course and is provided as-is.

---

**Maintained by**: [tziyon31](https://github.com/tziyon31)  
**Repository**: [demo-crm-gitops](https://github.com/tziyon31/demo-crm-gitops)
