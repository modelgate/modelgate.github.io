---
title: 部署概述
weight: 1
---

# 部署概述

ModelGate 支持多种部署方式，本文介绍常见的部署方案。

## 部署架构

```
                      ┌─────────────────┐
                      │   负载均衡器    │
                      │   (Nginx/ELB)   │
                      └────────┬────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
      ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
      │  前端容器 1  │ │  前端容器 2  │ │  前端容器 N  │
      │   (Nginx)    │ │   (Nginx)    │ │   (Nginx)    │
      └──────────────┘ └──────────────┘ └──────────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
      ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
      │  后端实例 1  │ │  后端实例 2  │ │  后端实例 N  │
      │  (8888+8889) │ │  (8888+8889) │ │  (8888+8889) │
      └──────────────┘ └──────────────┘ └──────────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
      ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
      │ 数据库主节点 │ │ 数据库从节点 │ │ 数据库从节点 │
      │  (MySQL/PG)  │ │  (MySQL/PG)  │ │  (MySQL/PG)  │
      └──────────────┘ └──────────────┘ └──────────────┘
```

## 部署方式

### 1. Docker Compose 部署

适合小规模部署或测试环境。

### 2. Kubernetes 部署

适合大规模生产环境，支持自动扩缩容。

### 3. 传统部署

直接在服务器上运行二进制文件。

## 部署前准备

### 1. 服务器要求

**最低配置**：

- CPU: 2 核
- 内存: 4 GB
- 磁盘: 20 GB

**推荐配置**：

- CPU: 4 核+
- 内存: 8 GB+
- 磁盘: 50 GB+

### 2. 软件要求

- Docker 20.10+
- Docker Compose 2.0+
- (可选) Kubernetes 1.20+

### 3. 网络要求

- 开放端口: 80, 443, 8888, 8889
- 数据库连接
- 访问外部 API (OpenAI, Anthropic 等)

### 4. 域名配置

准备以下域名：

- `api.modelgate.com` - API 转发服务
- `admin.modelgate.com` - 管理后台（可选）
- `modelgate.com` - 前端界面

## 数据库选择

### MySQL

推荐用于生产环境：

```env
MG_DATABASE_TYPE=mysql
MG_DATABASE_DSN=user:password@tcp(hostname:3306)/modelgate?charset=utf8mb4&parseTime=True&loc=Local
```

### PostgreSQL

```env
MG_DATABASE_TYPE=postgres
MG_DATABASE_DSN=host=hostname user=user password=password dbname=modelgate port=5432 sslmode=disable
```

### SQLite

适合开发和小规模部署：

```env
MG_DATABASE_TYPE=sqlite
MG_DATABASE_DSN=file:modelgate.db
```

## 安全建议

### 1. 环境变量管理

- 使用 `.env` 文件存储敏感信息
- 设置文件权限: `chmod 600 configs/.env`
- 不要将 `.env` 提交到版本控制

### 2. JWT 密钥

使用强随机字符串：

```bash
# 生成随机密钥
openssl rand -base64 32
```

### 3. 数据库安全

- 使用强密码
- 限制数据库访问 IP
- 定期备份数据

### 4. HTTPS

生产环境必须启用 HTTPS：

```nginx
server {
    listen 443 ssl http2;
    server_name api.modelgate.com;

    ssl_certificate /etc/ssl/certs/modelgate.crt;
    ssl_certificate_key /etc/ssl/private/modelgate.key;

    location / {
        proxy_pass http://localhost:8888;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 5. 防火墙配置

```bash
# 只允许必要端口
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 22/tcp
ufw enable
```

### 6. API 密钥轮换

定期轮换供应商 API 密钥和用户 API 密钥。

## 监控和日志

### 1. 日志管理

配置日志输出到文件：

```toml
[log]
level = "info"
format = "json"
output = "file"
filename = "logs/modelgate.log"
max_size = 100
max_age = 30
compress = true
```

### 2. 监控指标

监控以下指标：

- 请求量和延迟
- 错误率
- Token 使用量
- 费用统计
- 服务器资源使用

### 3. 告警配置

配置以下告警：

- 服务不可用
- 错误率超过阈值
- API 配额耗尽
- 数据库连接失败

## 备份策略

### 1. 数据库备份

```bash
# MySQL 备份
mysqldump -u user -p modelgate > backup_$(date +%Y%m%d).sql

# PostgreSQL 备份
pg_dump modelgate > backup_$(date +%Y%m%d).sql
```

### 2. 配置备份

```bash
# 备份配置文件
tar -czf config_backup_$(date +%Y%m%d).tar.gz configs/
```

### 3. 自动备份

使用 cron 定时备份：

```bash
# 每天凌晨 2 点备份
0 2 * * * /path/to/backup_script.sh
```

## 性能优化

### 1. 数据库优化

- 添加适当的索引
- 配置连接池
- 使用读写分离

### 2. 缓存策略

- 使用 Redis 缓存热点数据
- 配置合理的缓存过期时间

### 3. 负载均衡

使用 Nginx 或云服务商的负载均衡器：

```nginx
upstream modelgate_backend {
    server backend1:8889;
    server backend2:8889;
    server backend3:8889;
}

server {
    location / {
        proxy_pass http://modelgate_backend;
    }
}
```

## 升级策略

### 1. 零停机升级

使用滚动更新：

1. 部署新版本到部分实例
2. 验证新版本正常工作
3. 逐步替换旧实例

### 2. 数据库迁移

```bash
# 运行迁移
go run cmd/main.go migrate

# 回滚迁移
go run cmd/main.go migrate down
```

### 3. 备份验证

升级前验证备份可用性。

## 故障排查

### 1. 服务无法启动

```bash
# 查看日志
tail -f logs/modelgate.log

# 检查配置
go run cmd/main.go --config-check
```

### 2. 数据库连接失败

```bash
# 测试数据库连接
mysql -h hostname -u user -p modelgate
```

### 3. API 调用失败

- 检查 API 密钥是否正确
- 检查供应商服务是否正常
- 查看请求日志获取详细信息
