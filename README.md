# Demo CRM GitOps Repository

> **Production-ready GitOps repository** for managing Demo CRM infrastructure and applications using ArgoCD

[![ArgoCD](https://img.shields.io/badge/ArgoCD-2.8+-EF4444?logo=argo)](https://argo-cd.readthedocs.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.24+-326CE5?logo=kubernetes)](https://kubernetes.io/)
[![Helm](https://img.shields.io/badge/Helm-3.8+-0F1689?logo=helm)](https://helm.sh/)

## 🎯 Overview

This repository implements a **GitOps-based infrastructure and application management** system using ArgoCD. It follows the **App of Apps pattern** with a clean hierarchy for managing infrastructure components and applications.

### Key Features

- ✅ **Single Root Application** (`root-apps`) managing the entire stack
- ✅ **Infrastructure as Code** - All infra components versioned and managed
- ✅ **Automated Sync** with self-healing capabilities
- ✅ **Production-ready** infrastructure components
- ✅ **Clean separation** between infrastructure and applications

## 📁 Repository Structure

```
demo-crm-gitops/
├── apps/                          # ArgoCD Applications
│   ├── app-of-apps.yaml          # Root application (root-apps)
│   ├── infra-root.yaml           # Infrastructure root application
│   ├── demo-crm-helm.yaml        # Demo CRM application
│   └── infra/                    # Infrastructure applications
│       ├── infra-cert-manager.yaml
│       ├── infra-clusterissuer.yaml
│       ├── infra-ingress-nginx-classic.yaml
│       ├── infra-mongodb-operator.yaml
│       ├── infra-mongodb-community.yaml
│       └── namespaces-app.yaml
├── infra-apps/                   # Infrastructure charts & manifests
│   ├── charts/                   # Helm charts for infrastructure
│   │   ├── cert-manager/
│   │   ├── ingress-nginx-classic/
│   │   └── community-operator/
│   └── manifests/                # Kubernetes manifests
│       ├── cert-manager/
│       ├── mongodb/
│       └── namespaces/
└── argocd-install/                # ArgoCD installation manifests
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
│   └── infra-mongodb-community
└── demo-crm-helm (Application)
```

### Infrastructure Components

| Component | Purpose | Namespace |
|-----------|---------|-----------|
| **ingress-nginx-classic** | Ingress controller (Kubernetes community) | `ingress-nginx-classic` |
| **cert-manager** | TLS certificate management | `cert-manager` |
| **ClusterIssuer** | Let's Encrypt certificate issuer | `cert-manager` |
| **MongoDB Operator** | MongoDB Community Operator | `demo-crm` |
| **MongoDB Community** | MongoDB replica set | `demo-crm` |

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
- **Includes**: `infra-root.yaml`, `demo-crm-helm.yaml`
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

## 📊 Application Status

Monitor your applications:

```bash
# List all applications
kubectl -n argocd get applications

# View application tree
argocd app get root-apps --show-tree

# Check sync status
argocd app list
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

## 📝 License

This project is part of a learning course and is provided as-is.

---

**Maintained by**: [tziyon31](https://github.com/tziyon31)  
**Repository**: [demo-crm-gitops](https://github.com/tziyon31/demo-crm-gitops)
