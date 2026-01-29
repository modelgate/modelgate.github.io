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
├── configs/               # 配置文件目录
│   ├── config.toml        # 主配置文件
│   └── .env               # 环境变量文件
├── deployments/           # 部署相关
├── docs/                  # 文档
├── internal/              # 内部代码
│   ├── app/              # 应用层 (HTTP/RPC 处理器)
│   │   ├── admin/        # 管理后台服务
│   │   └── api/          # API 转发服务
│   ├── config/           # 配置管理
│   ├── relay/            # 业务模块
│   │   ├── dao/          # 数据访问层
│   │   ├── model/        # 数据模型
│   │   └── service/      # 业务逻辑层
│   ├── runtime/          # 运行时 (转发执行引擎)
│   │   ├── core/         # 核心转发逻辑
│   │   ├── hooks/        # 转发钩子
│   │   └── provider/     # 供应商适配器
│   │       ├── openai/
│   │       ├── anthropic/
│   │       └── zhipu/
│   ├── server/           # 服务器 (路由、中间件)
│   │   └── middleware/   # 中间件
│   └── system/           # 系统模块 (用户、角色、菜单)
├── pkg/                  # 公共包
│   ├── db/               # 数据库工具
│   ├── proto/            # Protobuf 生成代码
│   └── utils/            # 通用工具
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
internal/relay/
├── dao/           # 数据访问层
├── model/         # 数据模型
└── service/       # 业务逻辑层
```

### 2. Runtime 模块

Runtime 模块是转发的核心引擎，负责实际的请求转发和响应处理。

**结构**：

```
internal/runtime/
├── core/         # 核心转发逻辑
├── hooks/        # 转发钩子，用于扩展功能
└── provider/     # 供应商适配器
    ├── openai/
    ├── anthropic/
    └── zhipu/
```

**支持的提供商**：
- OpenAI
- Anthropic
- 智谱 (Zhipu AI)

### 3. System 模块

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
