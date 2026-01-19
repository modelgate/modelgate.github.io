---
title: 前端概述
weight: 1
---

# 前端概述

ModelGate 前端基于 [SoybeanAdmin](https://github.com/soybeanjs/soybean-admin) 模板开发，使用 Vue 3 + TypeScript 构建。

## 技术栈

| 技术 | 版本 | 说明 |
|------|------|------|
| Vue | 3.5.15 | 渐进式 JavaScript 框架 |
| TypeScript | 5.x | JavaScript 的超集 |
| Vite | 6.3.5 | 下一代前端构建工具 |
| NaiveUI | - | Vue 3 组件库 |
| UnoCSS | - | 原子化 CSS 引擎 |
| Pinia | - | Vue 状态管理库 |
| VueRouter | - | Vue 官方路由 |
| Connect RPC | - | gRPC-Web 通信 |

## 项目结构

```
modelgate-web/
├── src/
│   ├── api/              # API 接口定义
│   │   ├── auth.ts       # 认证相关 API
│   │   ├── user.ts       # 用户相关 API
│   │   ├── relay.ts      # 转发相关 API
│   │   └── ...
│   ├── assets/           # 静态资源
│   │   ├── images/       # 图片
│   │   └── styles/       # 全局样式
│   ├── components/       # 公共组件
│   │   └── custom/       # 自定义组件
│   ├── layouts/          # 布局组件
│   │   ├── basic.ts      # 基础布局
│   │   └── blank.ts      # 空白布局
│   ├── locales/          # 国际化
│   │   ├── zh-cn.ts      # 简体中文
│   │   └── en-us.ts      # English
│   ├── router/           # 路由配置
│   │   ├── routes/       # 路由定义
│   │   └── index.ts      # 路由入口
│   ├── service/          # RPC 服务
│   │   └── connect.ts    # Connect RPC 客户端
│   ├── stores/           # Pinia 状态管理
│   │   ├── auth.ts       # 认证状态
│   │   ├── user.ts       # 用户状态
│   │   └── ...
│   ├── views/            # 页面视图
│   │   ├── home/         # 首页
│   │   ├── relay/        # API 转发管理
│   │   ├── manage/       # 系统管理
│   │   ├── usage/        # 使用统计
│   │   └── user-center/  # 用户中心
│   ├── utils/            # 工具函数
│   ├── types/            # TypeScript 类型定义
│   ├── App.vue           # 根组件
│   └── main.ts           # 应用入口
├── public/               # 公共资源
├── .env.development      # 开发环境变量
├── .env.production       # 生产环境变量
├── index.html            # HTML 模板
├── package.json          # 项目依赖
├── tsconfig.json         # TypeScript 配置
├── vite.config.ts        # Vite 配置
└── uno.config.ts         # UnoCSS 配置
```

## 核心模块

### 1. 认证模块

处理用户登录、登出、Token 刷新等。

### 2. 用户管理

- 用户列表、创建、编辑、删除
- 角色分配
- 权限管理

### 3. 供应商管理

- 供应商配置
- API 密钥管理
- 连接测试

### 4. 模型管理

- 模型列表
- 价格配置
- 模型启用/禁用

### 5. API 密钥管理

- 密钥创建
- 密钥列表
- 权限配置

### 6. 使用统计

- 请求记录
- Token 使用统计
- 费用统计
- 图表展示

## 路由结构

```
/                          # 首页
/home                      # 首页
/relay                     # API 转发管理
  /relay/providers         # 供应商管理
  /relay/models            # 模型管理
  /relay/api-keys          # API 密钥
/manage                    # 系统管理
  /manage/users            # 用户管理
  /manage/roles            # 角色管理
  /manage/menus            # 菜单管理
/usage                     # 使用统计
  /usage/requests          # 请求记录
  /usage/accounts          # 账户流水
/user-center               # 用户中心
  /user-center/profile     # 个人资料
  /user-center/security    # 安全设置
/auth                      # 认证
  /auth/login              # 登录
```

## 状态管理

使用 Pinia 进行状态管理：

### auth store

```typescript
// src/stores/auth.ts
export const useAuthStore = defineStore('auth', {
  state: () => ({
    token: '',
    isLoggedIn: false,
    userInfo: null
  }),
  actions: {
    async login(username: string, password: string) {
      // 登录逻辑
    },
    logout() {
      // 登出逻辑
    }
  }
})
```

### user store

```typescript
// src/stores/user.ts
export const useUserStore = defineStore('user', {
  state: () => ({
    users: [],
    currentUser: null
  }),
  actions: {
    async fetchUsers() {
      // 获取用户列表
    }
  }
})
```

## 与后端通信

使用 Connect RPC (gRPC-Web) 与后端通信：

```typescript
// src/service/connect.ts
import { createPromiseClient } from '@connectrpc/connect-web'
import { createConnectTransport } from '@connectrpc/connect-web'

const transport = createConnectTransport({
  baseUrl: import.meta.env.VITE_API_URL,
  useFetch: true
})

export const client = createPromiseClient( /* ... */ , { transport })
```

## 样式系统

使用 UnoCSS 实现原子化 CSS：

```html
<!-- 示例：使用原子类 -->
<div class="flex items-center justify-between p-4 bg-white rounded-lg">
  <span class="text-lg font-semibold">标题</span>
  <button class="px-4 py-2 bg-blue-500 text-white rounded">按钮</button>
</div>
```

## 国际化

使用 Vue I18n 实现多语言：

```typescript
// src/locales/zh-cn.ts
export default {
  common: {
    confirm: '确认',
    cancel: '取消'
  },
  auth: {
    login: '登录',
    logout: '登出'
  }
}
```

## 开发规范

### 命名规范

- 组件文件使用 PascalCase：`UserList.vue`
- 工具文件使用 camelCase：`formatDate.ts`
- 样式类名使用 kebab-case：`user-list-container`

### 组件开发

```vue
<template>
  <div class="my-component">
    <!-- 模板内容 -->
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

// 响应式数据
const count = ref(0)

// 计算属性
const doubleCount = computed(() => count.value * 2)

// 方法
function increment() {
  count.value++
}
</script>

<style scoped>
.my-component {
  /* 样式 */
}
</style>
```

### API 调用

```typescript
// src/api/user.ts
import { client } from '@/service/connect'

export async function fetchUsers() {
  const response = await client.getUsers({})
  return response.users
}

export async function createUser(data: CreateUserRequest) {
  return await client.createUser(data)
}
```
