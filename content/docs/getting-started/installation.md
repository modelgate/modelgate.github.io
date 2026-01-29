---
title: 安装部署
weight: 2
---

# 安装部署

本文介绍如何快速安装和部署 ModelGate。

## 前置要求

### 后端服务

- Go 1.25.5+
- MySQL / PostgreSQL / SQLite
- Redis
- (可选) Docker 和 Docker Compose

### 前端界面

- Node.js 18.12.0+
- pnpm 8.7.0+

## 后端安装

### 1. 克隆仓库

```bash
git clone https://github.com/modelgate/modelgate.git
cd modelgate
```

### 2. 安装依赖

```bash
go mod download
```

### 3. 安装 buf 工具

buf 用于 Protocol Buffers 代码生成：

```bash
go install github.com/bufbuild/buf/cmd/buf@latest
```

### 4. 生成 Protobuf 代码

```bash
buf generate
```

### 5. 配置环境变量

复制环境变量模板并编辑：

```bash
cp configs/.env.example configs/.env
```

编辑 `configs/.env` 文件，配置必要的环境变量：

```env
MG_DATABASE_TYPE=mysql
MG_DATABASE_HOST=you_db_host
MG_DATABASE_PORT=3306
MG_DATABASE_USER=your_db_user
MG_JWT_SECRET=your_jwt_secret
MG_REDIS_HOST=localhost:6379
# ... 其他配置
```

### 6. 数据库迁移

运行数据库迁移，创建必要的表结构：

```bash
go run cmd/main.go migrate
```

### 7. 启动服务

```bash
# 同时启动 API 转发服务和管理后台
go run cmd/main.go all

# 或者分别启动
go run cmd/main.go api    # API 转发服务 (端口 8888)
go run cmd/main.go admin  # 管理后台服务 (端口 8889)
```

## 前端安装

### 1. 克隆仓库

```bash
git clone https://github.com/modelgate/modelgate-web.git
cd modelgate-web
```

### 2. 安装依赖

```bash
pnpm install
```

### 3. 配置后端地址

编辑 `.env.development` 文件，配置后端服务地址：

```env
VITE_API_URL=http://localhost:8889
```

### 4. 启动开发服务器

```bash
pnpm dev
```

访问 `http://localhost:5173` 即可看到管理界面。

## Docker 部署

### Docker Compose (推荐)

项目提供了 Docker Compose 配置文件，可以一键启动所有服务：

```bash
docker-compose -f deployments/docker-compose.yaml up -d
```

### 手动 Docker 部署

#### 构建后端镜像

```bash
cd modelgate
docker build -t modelgate:latest .
```

#### 运行后端容器

```bash
docker run -d \
  -p 8888:8888 \
  -p 8889:8889 \
  -e MG_DATABASE_DSN=your_database_dsn \
  -e MG_JWT_SECRET=your_jwt_secret \
  --name modelgate \
  modelgate:latest
```

#### 构建前端镜像

```bash
cd modelgate-web
docker build -t modelgate-web:latest .
```

#### 运行前端容器

```bash
docker run -d \
  -p 80:80 \
  --name modelgate-web \
  modelgate-web:latest
```

## 验证安装

### 检查后端服务

```bash
# 检查 API 转发服务
curl http://localhost:8888/health

# 检查管理后台服务
curl http://localhost:8889/health
```

### 检查前端界面

访问 `http://localhost:5173`（开发模式）或配置的域名（生产模式），应该能看到登录界面。

## 下一步

安装完成后，请继续阅读：

- [后端架构](/docs/backend/architecture) - 了解后端系统架构
- [前端开发](/docs/frontend/overview) - 了解前端开发指南
- [API 文档](/docs/api/overview) - 查看 API 接口文档
