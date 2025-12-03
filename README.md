# Mobile MPC Project Helm Chart

这是一个基于 SecretFlow 的移动多方安全计算项目的 Kubernetes Helm Chart。

## 简介

此 Chart 用于在 Kubernetes 集群中部署多方安全计算（MPC）应用，支持：
- Company 节点（甲方/服务器）
- Partner 节点（乙方/客户端）
- Coordinator 节点（可选协调者）
- Web UI 管理界面

## 前置要求

- Kubernetes 1.19+
- Helm 3.2.0+
- PV provisioner（如果启用持久化存储）
- Docker registry（用于存储镜像）

## 安装 Chart

### 1. 构建 Docker 镜像

```bash
cd /path/to/mobile_project3_new
docker build -t your-registry.com/mpc/mobile-mpc-project:v1.0.0 .
docker push your-registry.com/mpc/mobile-mpc-project:v1.0.0
```

### 2. 更新 values.yaml

编辑 `values.yaml` 文件，修改镜像仓库地址：

```yaml
global:
  imageRegistry: "your-registry.com/mpc/"
```

### 3. 安装 Chart

```bash
# 创建命名空间
kubectl create namespace mpc-project

# 安装 Chart
helm install my-mpc ./helm-chart/mobile-mpc-project -n mpc-project

# 或使用自定义配置
helm install my-mpc ./helm-chart/mobile-mpc-project -n mpc-project -f custom-values.yaml
```

## 配置说明

### 主要配置项

| 参数 | 描述 | 默认值 |
|------|------|--------|
| `company.enabled` | 是否启用 Company 节点 | `true` |
| `company.replicaCount` | Company 节点副本数 | `1` |
| `company.image.repository` | Company 镜像名称 | `mobile-mpc-project` |
| `company.image.tag` | Company 镜像标签 | `v1.0.0` |
| `partner.enabled` | 是否启用 Partner 节点 | `true` |
| `partner.replicaCount` | Partner 节点副本数 | `1` |
| `webui.enabled` | 是否启用 Web UI | `true` |
| `webui.service.type` | Web UI 服务类型 | `LoadBalancer` |
| `persistence.enabled` | 是否启用持久化存储 | `false` |

### 示例配置

#### 生产环境配置

```yaml
global:
  imageRegistry: "registry.example.com/mpc/"
  resourcesEnabled: true
  affinityEnabled: true

company:
  enabled: true
  replicaCount: 1
  resources:
    limits:
      cpu: "4"
      memory: 8Gi
    requests:
      cpu: "2"
      memory: 4Gi
  persistence:
    enabled: true
    size: 50Gi

partner:
  enabled: true
  replicaCount: 1
  resources:
    limits:
      cpu: "4"
      memory: 8Gi
    requests:
      cpu: "2"
      memory: 4Gi
  persistence:
    enabled: true
    size: 50Gi

webui:
  enabled: true
  ingress:
    enabled: true
    hosts:
      - host: mpc.example.com
        paths:
          - path: /
            pathType: Prefix
```

## 卸载 Chart

```bash
helm uninstall my-mpc -n mpc-project
```

## 常用命令

### 查看状态

```bash
# 查看 Helm 发布状态
helm status my-mpc -n mpc-project

# 查看 Pod 状态
kubectl get pods -n mpc-project

# 查看服务
kubectl get svc -n mpc-project
```

### 查看日志

```bash
# Company 节点日志
kubectl logs -n mpc-project -l app.kubernetes.io/component=company -f

# Partner 节点日志
kubectl logs -n mpc-project -l app.kubernetes.io/component=partner -f
```

### 升级 Chart

```bash
helm upgrade my-mpc ./helm-chart/mobile-mpc-project -n mpc-project
```

### 回滚

```bash
# 查看历史版本
helm history my-mpc -n mpc-project

# 回滚到上一版本
helm rollback my-mpc -n mpc-project

# 回滚到指定版本
helm rollback my-mpc 1 -n mpc-project
```

## 故障排查

### Pod 启动失败

```bash
# 查看 Pod 详情
kubectl describe pod <pod-name> -n mpc-project

# 查看事件
kubectl get events -n mpc-project --sort-by='.lastTimestamp'
```

### 网络连接问题

```bash
# 测试服务连通性
kubectl run -it --rm debug --image=busybox --restart=Never -n mpc-project -- sh
# 在容器内执行
nslookup my-mpc-company-svc
telnet my-mpc-company-svc 9394
```

