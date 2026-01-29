---
title: 开发指南
weight: 3
---

# 后端开发指南

本文介绍如何参与 ModelGate 后端开发。

## 开发环境准备

### 安装 Go

确保安装了 Go 1.25.5 或更高版本：

```bash
go version
```

### 安装工具

```bash
# buf - Protobuf 代码生成工具
go install github.com/bufbuild/buf/cmd/buf@latest

# gowatch - 开发热重载工具
go install github.com/silenceper/gowatch@latest

# golangci-lint - 代码检查工具
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

### 克隆项目

```bash
git clone https://github.com/modelgate/modelgate.git
cd modelgate
```

### 安装依赖

```bash
go mod download
```

### 生成 Protobuf 代码

```bash
buf generate
```

### 配置环境

```bash
cp configs/.env.example configs/.env
# 编辑 configs/.env 填入必要的配置
```

### 初始化数据库

```bash
go run cmd/main.go migrate
```

## 开发流程

### 使用 gowatch 热重载

```bash
gowatch
```

代码修改后自动重新编译和重启。

### 手动运行

```bash
# 仅启动 API 转发服务 (端口 8888)
go run cmd/main.go api

# 仅启动管理后台服务 (端口 8889)
go run cmd/main.go admin

# 同时启动两个服务
go run cmd/main.go all

# 数据库迁移
go run cmd/main.go migrate

# 查看帮助
go run cmd/main.go --help
```

## 项目结构说明

### 添加新的 API 端点

1. 在 `internal/app/admin/router/` 中添加路由：

```go
// internal/app/admin/router/router.go
func RegisterRoutes(r *gin.Engine, handlers *handler.Handlers) {
    v1 := r.Group("/api/v1")
    {
        // 新增路由
        v1.GET("/new-endpoint", handlers.NewHandler.Handle)
    }
}
```

2. 在 `internal/app/admin/handler/` 中创建处理器：

```go
// internal/app/admin/handler/new_handler.go
package handler

type NewHandler struct {
    service *new_service.Service
}

func NewNewHandler(service *new_service.Service) *NewHandler {
    return &NewHandler{service: service}
}

func (h *NewHandler) Handle(c *gin.Context) {
    // 处理逻辑
    c.JSON(200, gin.H{"message": "success"})
}
```

3. 在 `internal/module/new_module/service/` 中创建服务层：

```go
// internal/module/new_module/service/new_service.go
package service

type Service struct {
    dao *dao.NewDAO
}

func New(dao *dao.NewDAO) *Service {
    return &Service{dao: dao}
}

func (s *Service) DoSomething() error {
    // 业务逻辑
    return nil
}
```

4. 在 `internal/app/inject.go` 中注册依赖注入：

```go
provider.Provide(injector, new_dao.New)
provider.Provide(injector, new_service.New)
provider.Provide(injector, handler.NewNewHandler)
```

### 添加新的数据模型

1. 在 `internal/module/new_module/model/` 中定义模型：

```go
// internal/module/new_module/model/new_model.go
package model

import "time"

type NewModel struct {
    ID        uint      `gorm:"primarykey"`
    Name      string    `gorm:"uniqueIndex"`
    CreatedAt time.Time
    UpdatedAt time.Time
}
```

2. 在 `internal/module/new_module/dao/` 中创建 DAO：

```go
// internal/module/new_module/dao/new_dao.go
package dao

type NewDAO struct {
    db *gorm.DB
}

func New(db *gorm.DB) *NewDAO {
    return &NewDAO{db: db}
}

func (d *NewDAO) Create(model *model.NewModel) error {
    return d.db.Create(model).Error
}

func (d *NewDAO) GetByID(id uint) (*model.NewModel, error) {
    var m model.NewModel
    err := d.db.First(&m, id).Error
    return &m, err
}
```

### 修改 Protobuf 定义

1. 编辑 `proto/` 目录下的 `.proto` 文件

2. 重新生成代码：

```bash
buf generate
```

3. 验证 Protobuf 文件：

```bash
buf lint
```

## 添加新的供应商支持

要添加新的大模型供应商，需要实现以下接口：

1. 在 `internal/pkg/relay/types.go` 中定义供应商类型：

```go
const (
    ProviderOpenAI    = "openai"
    ProviderNewVendor = "newvendor" // 新供应商
)
```

2. 在 `internal/module/relay/runtime/` 中创建转发实现：

```go
// internal/module/relay/runtime/newvendor.go
package runtime

type NewVendorHandler struct {
    client *http.Client
}

func NewNewVendorHandler() *NewVendorHandler {
    return &NewVendorHandler{
        client: &http.Client{Timeout: 60 * time.Second},
    }
}

func (h *NewVendorHandler) Handle(req *relay.Request, provider *model.Provider) (*relay.Response, error) {
    // 实现转发逻辑
    return &relay.Response{}, nil
}
```

3. 在转发工厂中注册：

```go
// internal/module/relay/runtime/factory.go
func GetHandler(providerType string) Handler {
    switch providerType {
    case model.ProviderOpenAI:
        return NewOpenAIHandler()
    case model.ProviderNewVendor:
        return NewNewVendorHandler() // 新增
    default:
        return nil
    }
}
```

## 测试

### 运行测试

```bash
# 运行所有测试
go test ./...

# 运行测试并查看覆盖率
go test -cover ./...

# 生成覆盖率报告
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

### 编写测试

```go
// internal/module/user/service/user_service_test.go
package service_test

import (
    "testing"
    // ...
)

func TestUserService_Create(t *testing.T) {
    // 测试逻辑
}
```

## 代码规范

### 命名规范

- 包名使用小写单词
- 导出函数/变量使用大驼峰
- 私有函数/变量使用小驼峰

### 错误处理

```go
// 好的做法
result, err := someFunc()
if err != nil {
    return fmt.Errorf("failed to do something: %w", err)
}

// 不好的做法
result, _ := someFunc()
```

### 日志记录

```go
import "github.com/sirupsen/logrus"

log.WithFields(logrus.Fields{
    "user_id": userID,
    "action":  "create",
}).Info("User created successfully")
```

## 调试

### 使用 Delve 调试器

```bash
# 安装 Delve
go install github.com/go-delve/delve/cmd/dlv@latest

# 调试运行
dlv debug cmd/main.go
```

### 日志级别

在开发时使用 debug 级别：

```toml
[log]
level = "debug"
```

## 提交代码

### 代码检查

```bash
golangci-lint run
```

### 提交前检查

- [ ] 所有测试通过
- [ ] 代码检查通过
- [ ] 添加了必要的测试
- [ ] 更新了相关文档

### Pull Request 流程

1. Fork 项目
2. 创建特性分支：`git checkout -b feature/your-feature`
3. 提交更改：`git commit -m "Add some feature"`
4. 推送分支：`git push origin feature/your-feature`
5. 创建 Pull Request

## 性能优化

### 数据库查询

- 使用索引优化查询
- 避免 N+1 查询
- 使用预加载（Preload）

### 缓存

对于频繁访问的数据，考虑添加缓存层。

### 连接池

合理配置数据库连接池：

```toml
[database]
max_idle_conns = 20
max_open_conns = 200
```

## 常见问题

### Protobuf 生成失败

确保安装了 buf 工具：

```bash
go install github.com/bufbuild/buf/cmd/buf@latest
```

### 数据库连接失败

检查 `configs/.env` 中的 `MG_DATABASE_DSN` 配置。

### JWT 验证失败

确保 `MG_JWT_SECRET` 环境变量已设置且长度至少 32 字符。
