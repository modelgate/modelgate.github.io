---
title: 配置说明
weight: 2
---

# 配置说明

ModelGate 后端使用配置文件和环境变量来管理配置。

## 配置文件位置

配置文件位于 `configs/` 目录：

```
configs/
├── config.toml       # 主配置文件
├── .env.example      # 环境变量模板
├── .env              # 环境变量（需自行创建）
└── rbac_model.conf   # RBAC 权限模型
```

## config.toml 说明

### 服务器配置

```toml
[apiServer]
# API 转发服务名称
name = "api"

# 监听地址（留空表示监听所有接口）
host = ""

# API 转发服务端口
port = 8888

[adminServer]
# 管理后台服务名称
name = "admin"

# 监听地址（留空表示监听所有接口）
host = ""

# 管理后台服务端口
port = 8889

[server]
# 运行模式: debug, release
mode = "release"

# 读取超时（秒）
read_timeout = 60

# 写入超时（秒）
write_timeout = 60
```

### 数据库配置

```toml
[database]
# 数据库类型: mysql, postgres, sqlite
type = "mysql"

# 连接字符串（也可以通过环境变量 MG_DATABASE_DSN 设置）
# dsn = "user:password@tcp(localhost:3306)/modelgate?charset=utf8mb4&parseTime=True"

# 最大空闲连接数
max_idle_conns = 10

# 最大打开连接数
max_open_conns = 100

# 连接最大生命周期（秒）
conn_max_lifetime = 3600
```

### JWT 配置

```toml
[jwt]
# JWT 签名密钥（也可以通过环境变量 MG_JWT_SECRET 设置）
# secret = "your-secret-key"

# Token 有效期（小时）
expire_hours = 168

# Issuer
issuer = "modelgate"
```

### 日志配置

```toml
[log]
# 日志级别: debug, info, warn, error
level = "info"

# 日志格式: json, text
format = "json"

# 输出目标: stdout, file
output = "stdout"

# 日志文件路径（output=file 时生效）
# filename = "logs/modelgate.log"

# 日志文件最大大小（MB）
# max_size = 100

# 日志文件最大保存天数
# max_age = 30

# 是否压缩日志文件
# compress = true
```

### CORS 配置

```toml
[cors]
# 是否启用 CORS
enable = true

# 允许的源（* 表示允许所有）
allow_origins = ["*"]

# 允许的 HTTP 方法
allow_methods = ["GET", "POST", "PUT", "DELETE", "OPTIONS"]

# 允许的请求头
allow_headers = ["Origin", "Content-Type", "Authorization"]

# 允许凭证
allow_credentials = true
```

### Redis 配置

```toml
[redis]
# Redis 服务器地址
host = "localhost"

# Redis 端口
port = 6379

# Redis 密码（留空表示无密码）
password = ""

# Redis 数据库编号
db = 0

# 连接池大小
pool_size = 10
```

### 速率限制配置

```toml
[rateLimit]
# 是否启用速率限制
enable = true

# 每个 API 密钥每分钟请求数限制
requests_per_minute = 60

# 每个 IP 每分钟请求数限制
ip_requests_per_minute = 120
```

## 环境变量说明

环境变量前缀为 `MG_`，在 `.env` 文件中配置：

### 数据库

```env
# 数据库连接字符串
MG_DATABASE_DSN=user:password@tcp(localhost:3306)/modelgate?charset=utf8mb4&parseTime=True

# 数据库类型
MG_DATABASE_TYPE=mysql

# 数据库主机
MG_DATABASE_HOST=localhost

# 数据库端口
MG_DATABASE_PORT=3306

# 数据库用户名
MG_DATABASE_USER=your_db_user

# 数据库密码
MG_DATABASE_PASSWORD=your_db_password

# 数据库名称
MG_DATABASE_NAME=modelgate
```

### Redis

```env
# Redis 主机
MG_REDIS_HOST=localhost:6379

# Redis 密码（如果有）
MG_REDIS_PASSWORD=your_redis_password

# Redis 数据库编号
MG_REDIS_DB=0
```

### JWT

```env
# JWT 签名密钥（必填）
MG_JWT_SECRET=your-random-secret-key-at-least-32-characters
```

### 供应商 API 密钥

```env
# OpenAI
MG_OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx

# Anthropic
MG_ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxx

# DeepSeek
MG_DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxx

# 智谱 AI
MG_ZHIPU_API_KEY=xxxxxxxxxxxxxxxx

# Ollama（本地部署，通常不需要密钥）
# MG_OLLAMA_API_KEY=
```

### S3 存储（可选）

```env
# S3 访问密钥（用于存储文件）
MG_S3_ACCESS_KEY=your-access-key

# S3 密钥
MG_S3_SECRET_KEY=your-secret-key

# S3 端点
MG_S3_ENDPOINT=https://s3.amazonaws.com

# S3 区域
MG_S3_REGION=us-east-1

# S3 存储桶
MG_S3_BUCKET=modelgate
```

## RBAC 权限模型

`rbac_model.conf` 定义了 Casbin 的权限模型：

```ini
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[role_definition]
g = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act
```

### 权限策略格式

权限策略在数据库中存储，格式为：

```
用户ID, 资源, 操作
角色ID, 资源, 操作
```

例如：

```
# admin 用户拥有所有权限
admin, /, *

# user 角色可以查看模型
user, /models, GET

# user 角色可以创建 API 密钥
user, /api-keys, POST
```

## 配置优先级

配置的优先级从高到低：

1. 环境变量
2. config.toml
3. 代码中的默认值

## 生产环境建议

### 安全配置

```toml
[server]
mode = "release"

[log]
level = "info"
format = "json"
```

### 性能配置

```toml
[database]
max_idle_conns = 20
max_open_conns = 200
conn_max_lifetime = 7200
```

### 环境变量

生产环境**必须**通过环境变量设置敏感信息：

```env
MG_JWT_SECRET=强随机字符串（至少32位）
MG_DATABASE_DSN=生产数据库连接
MG_OPENAI_API_KEY=生产 API 密钥
```

### CORS 限制

```toml
[cors]
allow_origins = ["https://your-domain.com"]
allow_credentials = true
```

## 配置验证

启动时会自动验证配置：

- 检查必需的环境变量
- 验证数据库连接
- 检查 JWT 密钥长度

如果配置无效，服务将拒绝启动并显示错误信息。
