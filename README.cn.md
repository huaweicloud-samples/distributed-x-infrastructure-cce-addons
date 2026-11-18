# Kubernetes Add-ons on 华为云

[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28+-blue.svg)](https://kubernetes.io)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-2.8+-green.svg)](https://argoproj.github.io)

[English](./README.md)

## 简介

基于华为云 CCE 的 Kubernetes Add-ons 配置仓库，使用 ArgoCD + Helm 模式管理各类运维组件。

## 目录

- [架构图](#架构图)
- [方案亮点](#方案亮点)
- [涉及云服务与费用](#涉及云服务与费用)
- [前置条件](#前置条件)
- [快速开始](#快速开始)
- [使用方法](#使用方法)
- [清理资源](#清理资源)
- [详细说明](#详细说明)
- [许可证](#许可证)
- [联系方式](#联系方式)

## 架构图

```
┌────────────────────────────────────────────────────────────────────┐
│                      Git 仓库 (OBS / CodeHub)                      │
│                     存放 Helm Charts 配置文件                      │
└──────────────────────────────────┬─────────────────────────────────┘
                                   │ Pull
                                   ▼
┌────────────────────────────────────────────────────────────────────┐
│                        ArgoCD (App of Apps)                        │
│                       声明式 GitOps 持续交付                       │
└──────────────────────────────────┬─────────────────────────────────┘
                                   │ Sync
                                   ▼
┌────────────────────────────────────────────────────────────────────┐
│                            Helm Charts                             │
│                         Add-ons 包管理工具                         │
└───────┬─────────────────┬─────────────────┬─────────────────┬──────┘
        │                 │                 │                 │
        ▼                 ▼                 ▼                 ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   监控组件   │  │   日志组件   │  │   网络组件   │  │   运维组件   │
│Prometheus   │  │Fluent Bit   │  │ELB Ingress  │  │metrics-server│
│   Grafana   │  │   + LTS     │  │cert-manager │  │              │
└───────┴─────┴────────┬───────┴────────┬───────┴─────┴───────┘
                       │                │              │
                       └────────┬───────┴──────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│                          Huawei CCE 集群                           │
│                    Kubernetes 容器引擎托管服务                     │
└────────────────────────────────────────────────────────────────────┘
```

## 方案亮点

- **GitOps 模式**：使用 ArgoCD 实现声明式部署
- **Helm 化管理**：基于 Helm Charts 封装
- **开源兼容**：大部分 Add-ons 可直接复用
- **云原生**：完全兼容 Kubernetes 标准

## 涉及云服务

| 云服务 | 结算方式 |
|--------|----------|
| CCE 集群 | 按规格 |
| CCE 节点 | 按需/包年 |
| OBS | 按存储量 |
| ELB | 按规格 |

## 前置条件

1. 华为云账号已开通 CCE、OBS、ELB
2. 安装 kubectl
3. 安装 Helm
4. 安装 ArgoCD CLI

## 快速开始

```bash
# 1. 创建 CCE 集群（参考 CCE 控制台）

# 2. 安装 ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 3. 配置集群凭据
argocd cluster add <context-name>

# 4. 部署 Add-ons
kubectl apply -f application.yaml
```

## 使用方法

### 查看 Add-ons 状态

```bash
kubectl get applications -n argocd
```

### 部署新的 Add-on

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

## 清理资源

```bash
# 删除 ArgoCD 应用
kubectl delete Application <app-name> -n argocd

# 删除 CCE 集群
oc cce cluster delete <cluster-name>
```

## 详细说明

### 支持的 Add-ons

| Add-on | 说明 | 状态 |
|--------|------|------|
| metrics-server | 资源指标 | ✅ |
| prometheus | 监控 | ✅ |
| grafana | 可视化 | ✅ |
| fluent-bit | 日志收集 | ✅ |
| elb-ingress-controller | 负载均衡 | ✅ |
| cert-manager | 证书管理 | ✅ |
| ingress-nginx | Ingress 控制器 | ✅ |

### Helm Charts 适配

大部分 Helm Charts 可直接使用公共仓库：

```bash
# 添加 Prometheus 仓库
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# 添加 Grafana 仓库
helm repo add grafana https://grafana.github.io/helm-charts
```

## 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件

## 联系方式

- 维护者：华为云架构团队
- 邮箱：cloud-architecture@huaweicloud.com

---
*基于 EKS Blueprints Add-ons 架构改编*
