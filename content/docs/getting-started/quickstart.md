---
title: 快速上手
weight: 3
---

# 快速上手

本指南将帮助你在 10 分钟内快速上手 ModelGate。

## 创建管理员账户

首先创建一个管理员账户：

```bash
go run cmd/main.go create-admin --username admin --email admin@example.com --password your_password
```

## 登录管理后台

1. 打开浏览器访问 `http://localhost:5173`
2. 使用刚创建的管理员账户登录

## 配置供应商

登录后，首先配置你要使用的大模型供应商：

1. 进入「供应商管理」页面
2. 点击「添加供应商」
3. 填写供应商信息：
   - 名称：如 "OpenAI"
   - 类型：选择对应的供应商类型
   - API 密钥：填入你的 API 密钥
   - API 基础 URL：使用默认值或自定义

## 配置模型

供应商配置完成后，添加对应的模型：

1. 进入「模型管理」页面
2. 点击「添加模型」
3. 填写模型信息：
   - 名称：如 "gpt-4"
   - 供应商：选择刚添加的供应商
   - 输入价格：每 1K tokens 的价格
   - 输出价格：每 1K tokens 的价格

## 创建 API 密钥

为你的应用创建 API 密钥：

1. 进入「API 密钥」页面
2. 点击「创建密钥」
3. 填写密钥信息：
   - 名称：如 "My App Key"
   - 过期时间：可选
4. 保存生成的密钥（只显示一次）

## 调用 API

使用你的 API 密钥调用 ModelGate API：

### curl 示例

```bash
curl -X POST http://localhost:8888/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "gpt-4",
    "messages": [
      {"role": "user", "content": "Hello!"}
    ]
  }'
```

### Python 示例

```python
import openai

openai.api_base = "http://localhost:8888/v1"
openai.api_key = "YOUR_API_KEY"

response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[
        {"role": "user", "content": "Hello!"}
    ]
)

print(response.choices[0].message.content)
```

### Node.js 示例

```javascript
import OpenAI from 'openai';

const openai = new OpenAI({
  baseURL: 'http://localhost:8888/v1',
  apiKey: 'YOUR_API_KEY',
});

const response = await openai.chat.completions.create({
  model: 'gpt-4',
  messages: [
    { role: 'user', content: 'Hello!' },
  ],
});

console.log(response.choices[0].message.content);
```

## 查看使用记录

在管理后台的「使用记录」页面，你可以查看所有的 API 调用记录，包括：

- 请求时间
- 使用的模型
- Token 使用量
- 产生的费用
- 请求状态

## 流式响应

ModelGate 支持流式响应（Server-Sent Events）：

```bash
curl -X POST http://localhost:8888/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "gpt-4",
    "messages": [
      {"role": "user", "content": "写一首诗"}
    ],
    "stream": true
  }'
```

## 用户管理

如果需要为其他用户创建账户：

1. 进入「用户管理」页面
2. 点击「添加用户」
3. 填写用户信息
4. 为用户分配角色和权限

用户登录后可以创建自己的 API 密钥，查看自己的使用记录。

## 下一步

恭喜！你已经成功上手 ModelGate。继续探索：

- [API 文档](/docs/api/overview) - 查看完整的 API 参考
- [部署指南](/docs/deployment/overview) - 了解生产环境部署
- [开发指南](/docs/development/contributing) - 参与项目开发
