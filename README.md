# Kubernetes Add-ons on Huawei Cloud

[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28+-blue.svg)](https://kubernetes.io)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-2.8+-green.svg)](https://argoproj.github.io)

[中文版](./README.cn.md)

---

## Introduction

A Kubernetes Add-ons configuration repository based on Huawei Cloud CCE, using ArgoCD + Helm pattern to manage various operational components.

## Table of Contents

- [Architecture](#architecture)
- [Key Features](#key-features)
- [Cloud Services & Costs](#cloud-services--costs)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Usage](#usage)
- [Cleanup](#cleanup)
- [Details](#details)
- [License](#license)
- [Contact](#contact)

## Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│                   Git Repository (OBS / CodeHub)                   │
│               Stores Helm Charts Configuration                     │
└──────────────────────────────────┬─────────────────────────────────┘
                                   │ Pull
                                   ▼
┌────────────────────────────────────────────────────────────────────┐
│                        ArgoCD (App of Apps)                        │
│                    Declarative GitOps Continuous Delivery         │
└──────────────────────────────────┬─────────────────────────────────┘
                                   │ Sync
                                   ▼
┌────────────────────────────────────────────────────────────────────┐
│                            Helm Charts                              │
│                        Add-ons Package Management                  │
└───────┬─────────────────┬─────────────────┬─────────────────┬──────┘
        │                 │                 │                 │
        ▼                 ▼                 ▼                 ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Monitoring  │  │   Logging    │  │   Network    │  │   Operations │
│Prometheus   │  │Fluent Bit   │  │ELB Ingress  │  │metrics-server│
│   Grafana   │  │   + LTS     │  │cert-manager │  │              │
└───────┴─────┴────────┬───────┴────────┬───────┴─────┴───────┘
                       │                │              │
                       └────────┬───────┴──────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│                         Huawei CCE Cluster                          │
│                Kubernetes Container Engine (Managed)               │
└────────────────────────────────────────────────────────────────────┘
```

## Key Features

- **GitOps Pattern**: Declarative deployment using ArgoCD
- **Helm Management**: Based on Helm Charts packaging
- **Open Source Compatible**: Most Add-ons can be directly reused
- **Cloud Native**: Fully compatible with Kubernetes standards

## Cloud Services

| Service | Settlement |
|---------|------------|
| CCE Cluster | By spec |
| CCE Nodes | On-demand/Annual |
| OBS | By storage |
| ELB | By spec |

## Prerequisites

1. Huawei Cloud account with CCE, OBS, ELB enabled
2. kubectl installed
3. Helm installed
4. ArgoCD CLI installed

## Quick Start

```bash
# 1. Create CCE cluster (via CCE console)

# 2. Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 3. Configure cluster credentials
argocd cluster add <context-name>

# 4. Deploy Add-ons
kubectl apply -f application.yaml
```

## Usage

### View Add-ons Status

```bash
kubectl get applications -n argocd
```

### Deploy New Add-on

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: prometheus
  namespace: argocd
spec:
  project: default
  source:
    chart: prometheus
    repoURL: https://prometheus-community.github.io/helm-charts
    targetRevision: 15.0.0
  destination:
    server: https://kubernetes.default.svc
    namespace: monitoring
```

## Cleanup

```bash
# Delete ArgoCD applications
kubectl delete Application <app-name> -n argocd

# Delete CCE cluster
oc cce cluster delete <cluster-name>
```

## Details

### Supported Add-ons

| Add-on | Description | Status |
|--------|-------------|--------|
| metrics-server | Resource metrics | ✅ |
| prometheus | Monitoring | ✅ |
| grafana | Visualization | ✅ |
| fluent-bit | Log collection | ✅ |
| elb-ingress-controller | Load balancing | ✅ |
| cert-manager | Certificate management | ✅ |
| ingress-nginx | Ingress controller | ✅ |

### Helm Charts Adaptation

Most Helm Charts can directly use public repositories:

```bash
# Add Prometheus repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Add Grafana repository
helm repo add grafana https://grafana.github.io/helm-charts
```

## License

MIT License - See [LICENSE](LICENSE) file for details.

## Contact

- Maintainer: Huawei Cloud Architecture Team
- Email: cloud-architecture@huaweicloud.com

---

*Based on EKS Blueprints Add-ons architecture*
