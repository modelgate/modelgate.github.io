---
title: API 概述
weight: 1
---

# API 概述

ModelGate 提供两组 API：

- **API 转发服务** (端口 8888) - 提供统一的大模型调用接口
- **管理后台 API** (端口 8889) - 提供系统管理功能

## 认证方式

### API 密钥认证

API 转发服务使用 API 密钥认证：

```bash
curl -X POST http://localhost:8888/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

### JWT 认证

管理后台 API 使用 JWT 认证：

```bash
# 登录获取 Token
curl -X POST http://localhost:8889/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "password"}'

# 使用 Token 访问 API
curl -X GET http://localhost:8889/api/v1/users \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## API 转发服务

### 基础 URL

```
http://localhost:8888
```

### 兼容性

API 转发服务兼容 OpenAI API 格式，可以无缝替换 OpenAI 的 base URL。

### 支持的端点

#### 聊天完成 (Chat Completions)

```http
POST /v1/chat/completions
```

**请求示例**：

```json
{
  "model": "gpt-4",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"}
  ],
  "temperature": 0.7,
  "max_tokens": 1000,
  "stream": false
}
```

**响应示例**：

```json
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1677652288,
  "model": "gpt-4",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "Hello! How can I help you today?"
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 9,
    "completion_tokens": 12,
    "total_tokens": 21
  }
}
```

#### 流式响应

设置 `"stream": true` 启用流式响应：

```http
POST /v1/chat/completions
```

```json
{
  "model": "gpt-4",
  "messages": [
    {"role": "user", "content": "Write a poem"}
  ],
  "stream": true
}
```

流式响应使用 Server-Sent Events (SSE) 格式。

#### 其他端点

ModelGate 还支持以下端点（兼容 OpenAI API）：

- `POST /v1/completions` - 文本补全
- `POST /v1/embeddings` - 向量嵌入
- `GET /v1/models` - 模型列表

## 管理后台 API

### 基础 URL

```
http://localhost:8889
```

### 认证端点

#### 登录

```http
POST /api/v1/auth/login
```

**请求**：

```json
{
  "username": "admin",
  "password": "password"
}
```

**响应**：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": 1,
      "username": "admin",
      "email": "admin@example.com"
    }
  }
}
```

#### 登出

```http
POST /api/v1/auth/logout
```

#### 刷新 Token

```http
POST /api/v1/auth/refresh
```

### 用户管理端点

#### 获取用户列表

```http
GET /api/v1/users?page=1&page_size=20
```

#### 获取用户详情

```http
GET /api/v1/users/:id
```

#### 创建用户

```http
POST /api/v1/users
```

**请求**：

```json
{
  "username": "newuser",
  "email": "user@example.com",
  "password": "password123",
  "role_id": 2
}
```

#### 更新用户

```http
PUT /api/v1/users/:id
```

#### 删除用户

```http
DELETE /api/v1/users/:id
```

### 供应商管理端点

#### 获取供应商列表

```http
GET /api/v1/providers
```

**响应**：

```json
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "name": "OpenAI",
      "type": "openai",
      "base_url": "https://api.openai.com/v1",
      "enabled": true
    }
  ]
}
```

#### 创建供应商

```http
POST /api/v1/providers
```

**请求**：

```json
{
  "name": "OpenAI",
  "type": "openai",
  "api_key": "sk-...",
  "base_url": "https://api.openai.com/v1"
}
```

#### 更新供应商

```http
PUT /api/v1/providers/:id
```

#### 删除供应商

```http
DELETE /api/v1/providers/:id
```

#### 测试供应商连接

```http
POST /api/v1/providers/:id/test
```

### 模型管理端点

#### 获取模型列表

```http
GET /api/v1/models?provider_id=1
```

#### 创建模型

```http
POST /api/v1/models
```

**请求**：

```json
{
  "name": "gpt-4",
  "provider_id": 1,
  "input_price": 0.03,
  "output_price": 0.06,
  "enabled": true
}
```

#### 更新模型

```http
PUT /api/v1/models/:id
```

#### 删除模型

```http
DELETE /api/v1/models/:id
```

### API 密钥管理端点

#### 获取密钥列表

```http
GET /api/v1/api-keys
```

**响应**：

```json
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "name": "My App Key",
      "key": "sk-...xxxx",
      "created_at": "2024-01-01T00:00:00Z",
      "expires_at": null,
      "is_active": true
    }
  ]
}
```

#### 创建密钥

```http
POST /api/v1/api-keys
```

**请求**：

```json
{
  "name": "My App Key",
  "expires_at": "2024-12-31T23:59:59Z"
}
```

**响应**：

```json
{
  "code": 0,
  "data": {
    "id": 1,
    "key": "mg-sk-xxxxxxxxxxxx"  // 只显示一次
  }
}
```

#### 删除密钥

```http
DELETE /api/v1/api-keys/:id
```

### 使用统计端点

#### 获取请求记录

```http
GET /api/v1/requests?page=1&page_size=20&start_date=2024-01-01&end_date=2024-01-31
```

**响应**：

```json
{
  "code": 0,
  "data": {
    "total": 100,
    "items": [
      {
        "id": 1,
        "user_id": 1,
        "model": "gpt-4",
        "prompt_tokens": 100,
        "completion_tokens": 200,
        "total_tokens": 300,
        "cost": 0.015,
        "status": "success",
        "created_at": "2024-01-01T00:00:00Z"
      }
    ]
  }
}
```

#### 获取账户流水

```http
GET /api/v1/accounts?page=1&page_size=20
```

#### 获取使用统计

```http
GET /api/v1/statistics?period=7d
```

**响应**：

```json
{
  "code": 0,
  "data": {
    "total_requests": 1000,
    "total_tokens": 500000,
    "total_cost": 15.5,
    "by_model": [
      {"model": "gpt-4", "requests": 500, "tokens": 300000, "cost": 12.0},
      {"model": "gpt-3.5-turbo", "requests": 500, "tokens": 200000, "cost": 3.5}
    ]
  }
}
```

## 错误响应

所有 API 在发生错误时返回统一格式：

```json
{
  "code": 40001,
  "message": "Invalid API key",
  "details": "The provided API key is invalid or expired"
}
```

### 常见错误码

| 错误码 | 说明 |
|--------|------|
| 40001 | API 密钥无效 |
| 40002 | Token 余额不足 |
| 40003 | 模型不存在或已禁用 |
| 40004 | 请求参数错误 |
| 40005 | 供应商 API 错误 |
| 40101 | 未认证 |
| 40102 | Token 过期 |
| 40301 | 权限不足 |
| 50000 | 服务器内部错误 |

## 速率限制

API 转发服务默认速率限制：

- 每分钟请求数：60
- 每分钟 Token 数：100,000

管理后台 API 默认速率限制：

- 每分钟请求数：120

## SDK 支持

由于 ModelGate 兼容 OpenAI API，你可以使用 OpenAI 官方 SDK：

### Python

```python
import openai

openai.api_base = "http://localhost:8888/v1"
openai.api_key = "YOUR_API_KEY"

response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

### Node.js

```javascript
import OpenAI from 'openai';

const openai = new OpenAI({
  baseURL: 'http://localhost:8888/v1',
  apiKey: 'YOUR_API_KEY',
});

const response = await openai.chat.completions.create({
  model: 'gpt-4',
  messages: [{ role: 'user', content: 'Hello!' }],
});
```

### Go

```go
package main

import (
    "github.com/sashabaranov/go-openai"
)

func main() {
    config := openai.DefaultConfig("YOUR_API_KEY")
    config.BaseURL = "http://localhost:8888/v1"

    client := openai.NewClientWithConfig(config)

    resp, err := client.CreateChatCompletion(
        context.Background(),
        openai.ChatCompletionRequest{
            Model: openai.GPT4,
            Messages: []openai.ChatCompletionMessage{
                {Role: "user", Content: "Hello!"},
            },
        },
    )
}
```
