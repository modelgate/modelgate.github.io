---
title: Kubernetes 部署
weight: 3
---

# Kubernetes 部署

使用 Kubernetes 可以实现 ModelGate 的高可用部署和自动扩缩容。

## 准备工作

### 1. 安装 kubectl

```bash
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# 验证安装
kubectl version --client
```

### 2. 配置 kubeconfig

```bash
mkdir -p ~/.kube
cp your-kubeconfig ~/.kube/config
```

### 3. 验证集群连接

```bash
kubectl cluster-info
kubectl get nodes
```

## 部署步骤

### 1. 创建 Namespace

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: modelgate
```

```bash
kubectl apply -f namespace.yaml
```

### 2. 创建 ConfigMap

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: modelgate-config
  namespace: modelgate
data:
  config.toml: |
    [server]
    api_port = 8888
    admin_port = 8889
    mode = "release"

    [database]
    type = "mysql"
    max_idle_conns = 10
    max_open_conns = 100

    [log]
    level = "info"
    format = "json"
    output = "stdout"

    [cors]
    enable = true
    allow_origins = ["*"]
    allow_methods = ["GET", "POST", "PUT", "DELETE", "OPTIONS"]
```

```bash
kubectl apply -f configmap.yaml
```

### 3. 创建 Secret

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: modelgate-secret
  namespace: modelgate
type: Opaque
stringData:
  MG_DATABASE_DSN: "user:password@tcp(mysql-service:3306)/modelgate?charset=utf8mb4&parseTime=True"
  MG_JWT_SECRET: "your-jwt-secret-at-least-32-characters"
  MG_OPENAI_API_KEY: "sk-your-openai-api-key"
  MG_ANTHROPIC_API_KEY: "sk-ant-your-anthropic-api-key"
```

```bash
kubectl apply -f secret.yaml
```

### 4. 部署 MySQL

```yaml
# mysql-deployment.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: modelgate
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: modelgate
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "root_password"
        - name: MYSQL_DATABASE
          value: "modelgate"
        - name: MYSQL_USER
          value: "modelgate"
        - name: MYSQL_PASSWORD
          value: "password"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  namespace: modelgate
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
```

```bash
kubectl apply -f mysql-deployment.yaml
```

### 5. 部署后端服务

```yaml
# backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: modelgate-backend
  namespace: modelgate
spec:
  replicas: 3
  selector:
    matchLabels:
      app: modelgate-backend
  template:
    metadata:
      labels:
        app: modelgate-backend
    spec:
      containers:
      - name: backend
        image: modelgate/backend:latest
        imagePullPolicy: Always
        ports:
        - name: api
          containerPort: 8888
        - name: admin
          containerPort: 8889
        env:
        - name: MG_DATABASE_DSN
          valueFrom:
            secretKeyRef:
              name: modelgate-secret
              key: MG_DATABASE_DSN
        - name: MG_JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: modelgate-secret
              key: MG_JWT_SECRET
        - name: MG_OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: modelgate-secret
              key: MG_OPENAI_API_KEY
        volumeMounts:
        - name: config
          mountPath: /app/configs
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8889
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8889
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: config
        configMap:
          name: modelgate-config

---
apiVersion: v1
kind: Service
metadata:
  name: modelgate-backend
  namespace: modelgate
spec:
  selector:
    app: modelgate-backend
  ports:
  - name: api
    port: 8888
    targetPort: 8888
  - name: admin
    port: 8889
    targetPort: 8889
  type: ClusterIP
```

```bash
kubectl apply -f backend-deployment.yaml
```

### 6. 部署前端服务

```yaml
# frontend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: modelgate-frontend
  namespace: modelgate
spec:
  replicas: 2
  selector:
    matchLabels:
      app: modelgate-frontend
  template:
    metadata:
      labels:
        app: modelgate-frontend
    spec:
      containers:
      - name: frontend
        image: modelgate/frontend:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: modelgate-frontend
  namespace: modelgate
spec:
  selector:
    app: modelgate-frontend
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

```bash
kubectl apply -f frontend-deployment.yaml
```

### 7. 配置 Ingress

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: modelgate-ingress
  namespace: modelgate
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - api.modelgate.com
    - admin.modelgate.com
    - modelgate.com
    secretName: modelgate-tls
  rules:
  - host: api.modelgate.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: modelgate-backend
            port:
              number: 8888
  - host: admin.modelgate.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: modelgate-backend
            port:
              number: 8889
  - host: modelgate.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: modelgate-frontend
            port:
              number: 80
```

```bash
kubectl apply -f ingress.yaml
```

### 8. 配置 HPA (自动扩缩容)

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: modelgate-backend-hpa
  namespace: modelgate
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: modelgate-backend
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: modelgate-frontend-hpa
  namespace: modelgate
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: modelgate-frontend
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

```bash
kubectl apply -f hpa.yaml
```

## 初始化数据库

```bash
# 获取后端 Pod
kubectl get pods -n modelgate -l app=modelgate-backend

# 进入 Pod 并运行迁移
kubectl exec -it <pod-name> -n modelgate -- /main migrate

# 创建管理员账户
kubectl exec -it <pod-name> -n modelgate -- /main create-admin \
  --username admin \
  --email admin@example.com \
  --password your_password
```

## 管理命令

### 查看 Pod 状态

```bash
kubectl get pods -n modelgate
kubectl get pods -n modelgate -w
```

### 查看日志

```bash
# 查看所有 Pod 日志
kubectl logs -n modelgate -l app=modelgate-backend

# 查看特定 Pod 日志
kubectl logs -n modelgate <pod-name>

# 实时查看日志
kubectl logs -n modelgate <pod-name> -f
```

### 扩缩容

```bash
# 手动扩容
kubectl scale deployment modelgate-backend -n modelgate --replicas=5

# 手动缩容
kubectl scale deployment modelgate-backend -n modelgate --replicas=3
```

### 更新部署

```bash
# 更新镜像
kubectl set image deployment/modelgate-backend \
  backend=modelgate/backend:v1.0.1 \
  -n modelgate

# 查看更新状态
kubectl rollout status deployment/modelgate-backend -n modelgate

# 回滚更新
kubectl rollout undo deployment/modelgate-backend -n modelgate
```

### 进入 Pod

```bash
kubectl exec -it <pod-name> -n modelgate -- sh
```

## 监控和日志

### 安装 Prometheus

```bash
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml
```

### 配置 ServiceMonitor

```yaml
# servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: modelgate-backend
  namespace: modelgate
spec:
  selector:
    matchLabels:
      app: modelgate-backend
  endpoints:
  - port: admin
    path: /metrics
```

### 日志聚合

使用 EFK (Elasticsearch, Fluentd, Kibana) 或 Loki 进行日志聚合。

## 备份和恢复

### 备份数据库

```bash
# 创建备份
kubectl exec -n modelgate mysql-service -- mysqldump -u modelgate -p modelgate > backup.sql

# 从备份恢复
kubectl exec -i -n modelgate mysql-service -- mysql -u modelgate -p modelgate < backup.sql
```

### 持久化备份

```bash
# 使用 Velero 进行集群备份
velero backup create modelgate-backup --include-namespaces modelgate
```

## 故障排查

### Pod 无法启动

```bash
# 查看 Pod 详情
kubectl describe pod <pod-name> -n modelgate

# 查看事件
kubectl get events -n modelgate
```

### 服务无法访问

```bash
# 检查 Service
kubectl get svc -n modelgate

# 检查 Endpoints
kubectl get endpoints -n modelgate

# 测试服务连接
kubectl run test-pod --rm -it --image=busybox -n modelgate -- wget -O- http://modelgate-backend:8889/health
```

### Ingress 无法访问

```bash
# 检查 Ingress
kubectl get ingress -n modelgate

# 检查 Ingress Controller
kubectl get pods -n ingress-nginx
```

## 生产环境建议

### 1. 使用分布式追踪

集成 Jaeger 或 Zipkin 进行分布式追踪。

### 2. 配置告警

使用 Prometheus Alertmanager 配置告警规则。

### 3. 使用 Pod Disruption Budgets

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: modelgate-backend-pdb
  namespace: modelgate
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: modelgate-backend
```

### 4. 配置资源配额

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: modelgate-quota
  namespace: modelgate
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
```
