---
title: 后端架构
weight: 1
---

# 后端架构

ModelGate 后端采用清晰的分层架构设计，使用 Go 语言开发。

## 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                        应用层 (App)                          │
│  ┌─────────────────┐         ┌─────────────────┐           │
│  │   Admin API     │         │   Relay API     │           │
│  │   (Gin Router)  │         │   (Gin Router)  │           │
│  └─────────────────┘         └─────────────────┘           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      业务层 (Module)                         │
│  ┌─────────────────┐         ┌─────────────────┐           │
│  │   System 模块   │         │   Relay 模块    │           │
│  │  - 用户         │         │  - 供应商        │           │
│  │  - 角色         │         │  - 模型          │           │
│  │  - 菜单         │         │  - 转发引擎      │           │
│  │  - 权限         │         │  - 计费          │           │
│  └─────────────────┘         └─────────────────┘           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       数据层 (DAO)                           │
│  ┌─────────────────┐         ┌─────────────────┐           │
│  │    GORM ORM     │         │   数据库模型     │           │
│  └─────────────────┘         └─────────────────┘           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    数据库 (MySQL/PG/SQLite)                  │
└─────────────────────────────────────────────────────────────┘
```

## 项目结构

```
modelgate/
├── cmd/                    # 命令行入口
│   └── main.go            # 主程序入口
├── configs/               # 配置文件目录
│   ├── config.toml        # 主配置文件
│   ├── .env.example       # 环境变量模板
│   └── rbac_model.conf    # RBAC 权限模型
├── deployments/           # 部署相关
│   └── docker-compose.yaml
├── docs/                  # 文档
├── internal/              # 内部代码
│   ├── app/              # 应用层 (HTTP/RPC 处理器)
│   │   ├── admin/        # 管理后台服务
│   │   │   ├── handler/  # HTTP 处理器
│   │   │   └── router/   # 路由配置
│   │   └── api/          # API 转发服务
│   │       ├── handler/  # HTTP 处理器
│   │       └── router/   # 路由配置
│   ├── config/           # 配置管理
│   ├── module/           # 业务模块
│   │   ├── relay/        # 模型转发模块
│   │   │   ├── dao/      # 数据访问层
│   │   │   ├── model/    # 数据模型
│   │   │   ├── service/  # 业务逻辑层
│   │   │   └── runtime/  # 运行时 (转发执行引擎)
│   │   └── system/       # 系统模块
│   │       ├── user/     # 用户模块
│   │       ├── role/     # 角色模块
│   │       └── menu/     # 菜单模块
│   ├── pkg/              # 工具包
│   │   ├── db/           # 数据库工具
│   │   ├── relay/        # 转发接口定义
│   │   ├── rbac/         # 权限控制
│   │   └── utils/        # 通用工具
│   ├── proto/            # Protobuf 生成代码
│   └── server/           # 服务器
│       ├── middleware/   # 中间件
│       └── interceptor/  # RPC 拦截器
├── proto/                # Protobuf 定义
├── go.mod
├── go.sum
├── buf.yaml
└── LICENSE
```

## 核心模块

### 1. Relay 模块

Relay 模块是 ModelGate 的核心，负责处理大模型 API 转发。

**结构**：

```
internal/module/relay/
├── dao/           # 数据访问层
│   ├── provider_dao.go
│   ├── model_dao.go
│   ├── request_dao.go
│   └── account_dao.go
├── model/         # 数据模型
│   ├── provider.go
│   ├── model.go
│   ├── request.go
│   └── account.go
├── service/       # 业务逻辑层
│   ├── provider_service.go
│   ├── model_service.go
│   └── account_service.go
└── runtime/       # 转发执行引擎
    ├── openai.go
    ├── anthropic.go
    ├── deepseek.go
    ├── zhipu.go
    └── ollama.go
```

**工作流程**：

1. 接收客户端请求
2. 验证 API 密钥
3. 根据模型名称获取供应商配置
4. 通过 runtime 转发到对应供应商
5. 记录请求日志和计费信息
6. 返回结果给客户端

### 2. System 模块

System 模块提供基础系统功能。

**结构**：

```
internal/module/system/
├── user/          # 用户模块
├── role/          # 角色模块
└── menu/          # 菜单模块
```

**功能**：

- 用户注册、登录、认证
- 角色和权限管理
- 菜单和路由配置

### 3. 通信协议

ModelGate 使用两种协议：

**HTTP/REST** (Gin)：

- 管理后台 API
- API 转发接口
- 健康检查等

**gRPC-Web** (Connect RPC)：

- 前端与后端通信
- Protobuf 定义在 `proto/` 目录

## 依赖注入

使用 [samber/do](https://github.com/samber/do) 实现依赖注入：

```go
// 在 internal/app/inject.go 中
func NewInjector(cfg *config.Config) (*injector, error) {
    injector := &injector{}

    // 数据库
    provider.ProvideValue(injector, cfg)
    provider.Provide(injector, db.NewDB)

    // DAO 层
    provider.Provide(injector, user_dao.New)
    provider.Provide(injector, provider_dao.New)

    // Service 层
    provider.Provide(injector, user_service.New)
    provider.Provide(injector, provider_service.New)

    // Handler 层
    provider.Provide(injector, handler.NewUserHandler)
    provider.Provide(injector, handler.NewProviderHandler)

    return injector, nil
}
```

## 中间件

位于 `internal/server/middleware/`：

- **AuthMiddleware** - JWT 认证
- **RBACMiddleware** - 权限控制
- **RequestLogMiddleware** - 请求日志
- **CORS Middleware** - 跨域支持

## 数据库模型

使用 GORM 定义数据模型：

```go
// 用户模型
type User struct {
    ID        uint   `gorm:"primarykey"`
    Username  string `gorm:"uniqueIndex"`
    Email     string `gorm:"uniqueIndex"`
    Password  string
    RoleID    uint
    CreatedAt time.Time
    UpdatedAt time.Time
}

// 供应商模型
type Provider struct {
    ID          uint   `gorm:"primarykey"`
    Name        string `gorm:"uniqueIndex"`
    Type        string
    APIKey      string
    BaseURL     string
    CreatedAt   time.Time
    UpdatedAt   time.Time
}

// 模型模型
type Model struct {
    ID          uint   `gorm:"primarykey"`
    Name        string `gorm:"uniqueIndex"`
    ProviderID  uint
    InputPrice  float64
    OutputPrice float64
    CreatedAt   time.Time
    UpdatedAt   time.Time
}
```

## 配置管理

使用 Viper 管理配置：

- **config.toml** - 主配置文件
- **.env** - 环境变量（敏感信息）

配置项包括：

- 服务器配置（端口、模式）
- 数据库配置
- JWT 配置
- 日志配置
- 各供应商 API 配置

## 安全设计

- **密码加密**：使用 bcrypt 加密存储
- **JWT 认证**：无状态认证机制
- **RBAC 权限**：基于 Casbin 的角色权限控制
- **API 密钥**：独立的 API 密钥管理系统
- **CORS 控制**：可配置的跨域策略
