# 部署指南

## 前置条件

1. 华为云账号
2. 安装 kubectl
3. 安装 Helm
4. 安装 ArgoCD CLI

## 部署步骤

### 1. 创建 CCE 集群

```bash
# 通过控制台创建 CCE 集群
# 选择版本：Kubernetes 1.28+
# 配置节点池
```

### 2. 安装 ArgoCD

```bash
# 创建命名空间
kubectl create namespace argocd

# 安装 ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 获取初始密码
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 3. 配置集群

```bash
# 登录 ArgoCD
argocd login <argocd-server>

# 添加集群
argocd cluster add <context-name>
```

### 4. 部署 Add-ons

```bash
# 部署根应用
kubectl apply -f chart/application.yaml
```

## 资源清单

| 资源类型 | 说明 |
|----------|------|
| CCE 集群 | 1 个 |
| 节点池 | 根据需求 |
| ArgoCD | 系统组件 |
| Add-ons | 按需部署 |

## 清理

```bash
# 删除应用
kubectl delete -f chart/application.yaml

# 删除 CCE 集群（通过控制台）
```
