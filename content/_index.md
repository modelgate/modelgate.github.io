---
title: ModelGate
toc: false
---

# ModelGate

企业级大模型 API 网关和管理服务

一个功能完整的大模型 API 网关，提供统一的接口来管理和转发到多个大模型服务。支持用户管理、API 密钥管理、计费系统、请求记录等完整功能。

## 特性

{{< cards >}}
  {{< card link="docs/getting-started/introduction" title="多模型支持" icon="cube" >}}
  {{< card link="docs/getting-started/introduction" title="统一接口" icon="link" >}}
  {{< card link="docs/getting-started/introduction" title="用户管理" icon="users" >}}
  {{< card link="docs/getting-started/introduction" title="计费系统" icon="credit-card" >}}
{{< /cards >}}

## 核心功能

- **多模型支持** - 支持 OpenAI、Anthropic、智谱等多个大模型提供商
- **接口转发** - 提供统一的 API 接口，智能转发到不同的大模型服务
- **用户管理** - 完整的用户注册、登录、认证授权系统
- **API 密钥管理** - 用户 API 密钥的创建、管理和权限控制
- **供应商管理** - 管理不同的大模型供应商及其配置
- **模型管理** - 管理不同供应商的 AI 模型配置和定价
- **计费系统** - 基于 Token 使用量的计费和账户流水管理
- **请求记录** - 完整的 API 请求历史记录和查询
- **速率限制** - 内置请求速率限制功能
- **流式响应** - 支持流式和非流式响应模式
- **Redis 缓存** - 支持 Redis 缓存提升性能

## 项目结构

ModelGate 由三个子项目组成：

- **[modelgate](https://github.com/modelgate/modelgate)** - Go 后端服务
- **[modelgate-web](https://github.com/modelgate/modelgate-web)** - Vue3 前端管理界面
- **[modelgate.github.io](https://github.com/modelgate/modelgate.github.io)** - 文档网站

## 技术栈

### 后端 (modelgate)

- **语言**: Go 1.25.5
- **Web 框架**: Gin (REST API) + Connect RPC (gRPC-Web)
- **数据库 ORM**: GORM
- **认证授权**: JWT
- **依赖注入**: samber/do
- **配置管理**: Viper
- **日志**: Logrus
- **缓存**: Redis
- **命令行**: Cobra
- **协议**: Protocol Buffers

### 前端 (modelgate-web)

- **框架**: Vue 3.5.15
- **语言**: TypeScript
- **构建工具**: Vite 6.3.5
- **组件库**: NaiveUI
- **样式**: UnoCSS
- **状态管理**: Pinia
- **模板**: 基于 SoybeanAdmin

## 快速开始

### 后端服务

```bash
# 克隆仓库
git clone https://github.com/modelgate/modelgate.git
cd modelgate

# 安装依赖
go mod download

# 安装 buf (用于 Protobuf 代码生成)
go install github.com/bufbuild/buf/cmd/buf@latest

# 生成 Protobuf 代码
buf generate

# 配置环境变量
cp configs/.env.example configs/.env

# 运行数据库迁移
go run cmd/main.go migrate

# 启动服务
go run cmd/main.go all
```

### 前端界面

```bash
# 克隆仓库
git clone https://github.com/modelgate/modelgate-web.git
cd modelgate-web

# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev
```

## 文档

{{< cards >}}
  {{< card link="docs/getting-started/introduction" title="快速开始" icon="book-open" >}}
  {{< card link="docs/backend/architecture" title="后端架构" icon="server" >}}
  {{< card link="docs/frontend/overview" title="前端开发" icon="view-grid" >}}
  {{< card link="docs/api/overview" title="API 文档" icon="code" >}}
  {{< card link="docs/deployment/overview" title="部署指南" icon="paper-airplane" >}}
  {{< card link="docs/development/contributing" title="开发贡献" icon="code" >}}
{{< /cards >}}

## 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](https://github.com/modelgate/modelgate/blob/main/LICENSE) 文件

## 联系方式

如有问题或建议，请提交 [Issue](https://github.com/modelgate/modelgate/issues)
