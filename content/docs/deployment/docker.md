---
title: Docker 部署
weight: 2
---

# Docker 部署

使用 Docker Compose 可以快速部署 ModelGate。

## 准备工作

### 1. 安装 Docker

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# macOS
brew install --cask docker

# 启动 Docker 服务
sudo systemctl start docker
sudo systemctl enable docker
```

### 2. 安装 Docker Compose

```bash
# Linux
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 验证安装
docker-compose --version
```

## 使用 Docker Compose 部署

### 1. 克隆项目

```bash
git clone https://github.com/modelgate/modelgate.git
cd modelgate
```

### 2. 配置环境变量

创建 `.env` 文件：

```bash
cp configs/.env.example configs/.env
```

编辑 `configs/.env`：

```env
# 数据库配置
MG_DATABASE_TYPE=mysql
MG_DATABASE_DSN=modelgate:password@tcp(mysql:3306)/modelgate?charset=utf8mb4&parseTime=True&loc=Local

# Redis 配置
MG_REDIS_HOST=redis:6379
MG_REDIS_DB=0

# JWT 密钥（请修改为随机字符串）
MG_JWT_SECRET=your-random-secret-key-at-least-32-characters

# 供应商 API 密钥
MG_OPENAI_API_KEY=sk-your-openai-api-key
MG_ANTHROPIC_API_KEY=sk-ant-your-anthropic-api-key

# 其他配置...
```

### 3. 创建 docker-compose.yml

```yaml
version: '3.8'

services:
  # MySQL 数据库
  mysql:
    image: mysql:8.0
    container_name: modelgate-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: modelgate
      MYSQL_USER: modelgate
      MYSQL_PASSWORD: password
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"
    networks:
      - modelgate-network

  # Redis 缓存
  redis:
    image: redis:7-alpine
    container_name: modelgate-redis
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - modelgate-network

  # ModelGate 后端
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: modelgate-backend
    restart: always
    ports:
      - "8888:8888"
      - "8889:8889"
    env_file:
      - configs/.env
    depends_on:
      - mysql
      - redis
    volumes:
      - ./configs:/app/configs
      - ./logs:/app/logs
    networks:
      - modelgate-network
    command: ./main all

  # ModelGate 前端
  frontend:
    build:
      context: ../modelgate-web
      dockerfile: Dockerfile
    container_name: modelgate-frontend
    restart: always
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - modelgate-network

  # Nginx 反向代理
  nginx:
    image: nginx:alpine
    container_name: modelgate-nginx
    restart: always
    ports:
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - backend
      - frontend
    networks:
      - modelgate-network

volumes:
  mysql_data:
  redis_data:

networks:
  modelgate-network:
    driver: bridge
```

### 4. 创建 Dockerfile

#### 后端 Dockerfile

```dockerfile
# Dockerfile
FROM golang:1.25.5-alpine AS builder

WORKDIR /app

# 安装必要工具
RUN apk add --no-cache git make buf

# 复制依赖文件
COPY go.mod go.sum ./
RUN go mod download

# 复制源代码
COPY . .

# 生成 Protobuf 代码
RUN buf generate

# 编译
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main cmd/main.go

# 运行阶段
FROM alpine:latest

RUN apk --no-cache add ca-certificates tzdata

WORKDIR /app

# 复制编译产物
COPY --from=builder /app/main .
COPY --from=builder /app/configs ./configs

# 创建日志目录
RUN mkdir -p logs

EXPOSE 8888 8889

CMD ["./main", "all"]
```

#### 前端 Dockerfile

在 `modelgate-web` 目录创建：

```dockerfile
# Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# 安装 pnpm
RUN npm install -g pnpm

# 复制依赖文件
COPY package.json pnpm-lock.yaml ./

# 安装依赖
RUN pnpm install --frozen-lockfile

# 复制源代码
COPY . .

# 构建生产版本
RUN pnpm build

# 运行阶段
FROM nginx:alpine

# 复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html

# 复制 nginx 配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### 5. 配置 Nginx

创建 `nginx/nginx.conf`：

```nginx
events {
    worker_connections 1024;
}

http {
    upstream backend_api {
        server backend:8888;
    }

    upstream backend_admin {
        server backend:8889;
    }

    upstream frontend {
        server frontend:80;
    }

    # HTTP 重定向到 HTTPS
    server {
        listen 80;
        server_name api.modelgate.com admin.modelgate.com modelgate.com;
        return 301 https://$server_name$request_uri;
    }

    # HTTPS 配置
    server {
        listen 443 ssl http2;
        server_name api.modelgate.com;

        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;

        location / {
            proxy_pass http://backend_api;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    server {
        listen 443 ssl http2;
        server_name admin.modelgate.com;

        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;

        location / {
            proxy_pass http://backend_admin;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    server {
        listen 443 ssl http2;
        server_name modelgate.com;

        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;

        location / {
            proxy_pass http://frontend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

### 6. 启动服务

```bash
# 构建并启动
docker-compose up -d

# 查看日志
docker-compose logs -f

# 查看服务状态
docker-compose ps
```

### 7. 初始化数据库

```bash
# 进入后端容器
docker-compose exec backend sh

# 运行数据库迁移
./main migrate
```

### 8. 创建管理员账户

```bash
docker-compose exec backend ./main create-admin \
  --username admin \
  --email admin@example.com \
  --password your_password
```

## 管理命令

### 查看日志

```bash
# 查看所有服务日志
docker-compose logs -f

# 查看特定服务日志
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f mysql
```

### 重启服务

```bash
# 重启所有服务
docker-compose restart

# 重启特定服务
docker-compose restart backend
```

### 停止服务

```bash
# 停止所有服务
docker-compose down

# 停止并删除数据卷
docker-compose down -v
```

### 更新服务

```bash
# 拉取最新代码
git pull

# 重新构建并启动
docker-compose up -d --build
```

### 进入容器

```bash
# 进入后端容器
docker-compose exec backend sh

# 进入数据库容器
docker-compose exec mysql bash
```

## 数据持久化

### 备份数据

```bash
# 备份数据库
docker-compose exec mysql mysqldump -u modelgate -p modelgate > backup.sql

# 备份配置
tar -czf config_backup.tar.gz configs/
```

### 恢复数据

```bash
# 恢复数据库
docker-compose exec -T mysql mysql -u modelgate -p modelgate < backup.sql
```

## 监控和健康检查

### 添加健康检查

在 `docker-compose.yml` 中添加：

```yaml
services:
  backend:
    healthcheck:
      test: ["CMD", "wget", "--spider", "http://localhost:8889/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### 查看容器资源使用

```bash
docker stats
```

## 生产环境建议

### 1. 使用外部数据库

对于生产环境，建议使用云数据库服务。

### 2. 配置资源限制

```yaml
services:
  backend:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
        reservations:
          cpus: '1'
          memory: 2G
```

### 3. 使用 Docker Swarm 或 Kubernetes

对于大规模部署，考虑使用 Docker Swarm 或 Kubernetes。

### 4. 配置日志轮转

```yaml
services:
  backend:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### 5. 使用 secrets 管理敏感信息

```bash
echo "your_secret_key" | docker secret create jwt_secret -
```

## 故障排查

### 容器无法启动

```bash
# 查看详细日志
docker-compose logs backend

# 检查容器状态
docker-compose ps
```

### 数据库连接失败

- 检查数据库容器是否运行
- 检查 `MG_DATABASE_DSN` 配置
- 确认网络连接正常

### 服务无法访问

- 检查端口映射
- 检查防火墙配置
- 验证 Nginx 配置
