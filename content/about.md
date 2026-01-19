---
title: 关于 ModelGate
type: about
---

# 关于 ModelGate

ModelGate 是一个企业级大模型 API 网关和管理服务，旨在为企业和开发者提供统一的接口来管理和转发到多个大模型服务。

## 项目愿景

随着人工智能技术的快速发展，各种大语言模型层出不穷。企业和开发者在使用这些模型时面临以下挑战：

- **接口不统一**：不同模型提供商的 API 接口各不相同，增加了集成成本
- **密钥管理困难**：多个服务的 API 密钥分散管理，存在安全隐患
- **成本控制**：难以追踪和控制各个服务的使用成本
- **权限管理**：无法精细控制不同用户对不同模型的访问权限

ModelGate 通过统一的 API 网关解决了这些问题，让企业能够轻松管理和使用多个大模型服务。

## 核心价值

### 统一接口

兼容 OpenAI API 格式，只需更改 base URL 即可使用，迁移成本低。

### 多模型支持

支持 OpenAI、Anthropic、DeepSeek、智谱 AI、Ollama 等多个主流大模型提供商。

### 完善的管理功能

提供用户管理、API 密钥管理、模型管理、计费系统等完整的管理功能。

### 安全可靠

采用 JWT 认证 + RBAC 权限控制，确保 API 调用的安全性。

### 灵活部署

支持 Docker、Kubernetes 等多种部署方式，适应不同规模的部署需求。

## 技术架构

ModelGate 采用前后端分离架构：

- **后端服务**：使用 Go 语言开发，提供高性能的 API 转发服务
- **前端界面**：基于 Vue 3 + NaiveUI 构建，提供现代化的管理界面
- **通信协议**：使用 gRPC-Web (Connect RPC) 进行前后端通信

## 适用场景

### 企业内部使用

为企业内部提供统一的 AI 模型访问入口，便于管理和控制成本。

### SaaS 服务集成

为 SaaS 产品提供 AI 能力，支持多模型切换和成本控制。

### 开发者工具

为开发者提供便捷的 AI 模型调用和管理工具。

### 教育研究

为教育机构和研究机构提供 AI 模型管理和使用平台。

## 开源协议

ModelGate 采用 MIT 开源协议，允许自由使用、修改和分发。

## 联系我们

- **GitHub**: [https://github.com/modelgate/modelgate](https://github.com/modelgate/modelgate)
- **文档**: [https://modelgate.github.io](https://modelgate.github.io)
- **问题反馈**: [提交 Issue](https://github.com/modelgate/modelgate/issues)

## 贡献指南

我们欢迎任何形式的贡献，包括但不限于：

- 报告 Bug
- 提出新功能建议
- 提交代码改进
- 完善文档

请阅读我们的[贡献指南](/docs/development/contributing)了解更多详情。

---

Made with ❤️ by ModelGate Team
