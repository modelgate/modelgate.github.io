---
title: 前端开发指南
weight: 2
---

# 前端开发指南

本文介绍如何参与 ModelGate 前端开发。

## 开发环境准备

### 环境要求

- Node.js >= 18.12.0
- pnpm >= 8.7.0

### 安装依赖

```bash
cd modelgate-web
pnpm install
```

### 配置环境变量

创建 `.env.development` 文件：

```env
# 后端 API 地址
VITE_API_URL=http://localhost:8889

# 其他配置...
```

### 启动开发服务器

```bash
pnpm dev
```

访问 `http://localhost:5173`

## 开发命令

```bash
# 启动开发服务器
pnpm dev

# 构建生产版本
pnpm build

# 预览生产构建
pnpm preview

# 代码检查
pnpm lint

# 代码格式化
pnpm format

# 类型检查
pnpm typecheck
```

## 添加新页面

### 1. 创建页面组件

在 `src/views/` 下创建页面：

```vue
<!-- src/views/new-page/index.vue -->
<template>
  <div class="new-page">
    <h1>{{ t('newPage.title') }}</h1>
    <!-- 页面内容 -->
  </div>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n'

const { t } = useI18n()
</script>

<style scoped>
.new-page {
  padding: 20px;
}
</style>
```

### 2. 添加路由配置

在 `src/router/routes/` 下添加路由：

```typescript
// src/router/routes/modules/new-page.ts
import type { RouteRecordRaw } from 'vue-router'

const route: RouteRecordRaw = {
  path: '/new-page',
  name: 'new_page',
  component: () => import('@/views/new-page/index.vue'),
  meta: {
    title: '新页面',
    i18nKey: 'route.newPage'
  }
}

export default route
```

### 3. 添加国际化文本

```typescript
// src/locales/zh-cn.ts
export default {
  route: {
    newPage: '新页面'
  },
  newPage: {
    title: '新页面标题'
  }
}
```

## 添加新组件

### 1. 创建组件

```vue
<!-- src/components/custom/MyComponent.vue -->
<template>
  <div class="my-component">
    <slot />
  </div>
</template>

<script setup lang="ts">
interface Props {
  title?: string
}

withDefaults(defineProps<Props>(), {
  title: 'Default Title'
})
</script>

<style scoped>
.my-component {
  /* 样式 */
}
</style>
```

### 2. 使用组件

```vue
<template>
  <MyComponent title="Custom Title">
    内容
  </MyComponent>
</template>

<script setup lang="ts">
import MyComponent from '@/components/custom/MyComponent.vue'
</script>
```

## API 调用

### 定义 API 接口

```typescript
// src/api/user.ts
import { client } from '@/service/connect'
import type { User } from '@/types/user'

export async function fetchUsers(): Promise<User[]> {
  const response = await client.getUsers({})
  return response.users || []
}

export async function createUser(data: CreateUserRequest): Promise<User> {
  const response = await client.createUser(data)
  return response.user || null
}

export async function updateUser(id: string, data: UpdateUserRequest): Promise<User> {
  const response = await client.updateUser({ id, ...data })
  return response.user || null
}

export async function deleteUser(id: string): Promise<void> {
  await client.deleteUser({ id })
}
```

### 在组件中使用

```vue
<template>
  <div>
    <n-button @click="loadUsers">加载用户</n-button>
    <n-list v-if="users.length">
      <n-list-item v-for="user in users" :key="user.id">
        {{ user.username }}
      </n-list-item>
    </n-list>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { fetchUsers } from '@/api/user'
import type { User } from '@/types/user'

const users = ref<User[]>([])
const loading = ref(false)

async function loadUsers() {
  loading.value = true
  try {
    users.value = await fetchUsers()
  } catch (error) {
    console.error('Failed to fetch users:', error)
  } finally {
    loading.value = false
  }
}
</script>
```

## 状态管理

### 创建 Store

```typescript
// src/stores/my-store.ts
import { defineStore } from 'pinia'

export const useMyStore = defineStore('myStore', {
  state: () => ({
    data: [],
    loading: false
  }),

  getters: {
    dataCount: (state) => state.data.length
  },

  actions: {
    async fetchData() {
      this.loading = true
      try {
        const response = await api.fetchData()
        this.data = response
      } finally {
        this.loading = false
      }
    }
  }
})
```

### 使用 Store

```vue
<script setup lang="ts">
import { useMyStore } from '@/stores/my-store'

const myStore = useMyStore()

// 访问 state
console.log(myStore.data)

// 访问 getters
console.log(myStore.dataCount)

// 调用 actions
myStore.fetchData()
</script>
```

## 表格组件使用

ModelGate 使用 NaiveUI 的表格组件：

```vue
<template>
  <n-data-table
    :columns="columns"
    :data="dataSource"
    :loading="loading"
    :pagination="pagination"
    @update:page="handlePageChange"
  />
</template>

<script setup lang="ts">
import { ref, h } from 'vue'
import { NButton, NSpace, NTag } from 'naive-ui'
import type { DataTableColumns } from 'naive-ui'

interface User {
  id: number
  username: string
  email: string
  status: 'active' | 'inactive'
}

const dataSource = ref<User[]>([])
const loading = ref(false)

const columns: DataTableColumns<User> = [
  { title: 'ID', key: 'id' },
  { title: '用户名', key: 'username' },
  { title: '邮箱', key: 'email' },
  {
    title: '状态',
    key: 'status',
    render: (row) => {
      return h(NTag, {
        type: row.status === 'active' ? 'success' : 'default'
      }, { default: () => row.status })
    }
  },
  {
    title: '操作',
    key: 'actions',
    render: (row) => {
      return h(NSpace, {}, {
        default: () => [
          h(NButton, {
            size: 'small',
            onClick: () => handleEdit(row)
          }, { default: () => '编辑' }),
          h(NButton, {
            size: 'small',
            type: 'error',
            onClick: () => handleDelete(row)
          }, { default: () => '删除' })
        ]
      })
    }
  }
]

function handleEdit(row: User) {
  // 编辑逻辑
}

function handleDelete(row: User) {
  // 删除逻辑
}

const pagination = ref({
  page: 1,
  pageSize: 10,
  itemCount: 0
})

function handlePageChange(page: number) {
  pagination.value.page = page
  // 加载数据
}
</script>
```

## 表单组件

### 表单验证

```vue
<template>
  <n-form
    ref="formRef"
    :model="formData"
    :rules="rules"
    label-placement="left"
    label-width="100"
  >
    <n-form-item label="用户名" path="username">
      <n-input v-model:value="formData.username" placeholder="请输入用户名" />
    </n-form-item>

    <n-form-item label="邮箱" path="email">
      <n-input v-model:value="formData.email" placeholder="请输入邮箱" />
    </n-form-item>

    <n-form-item>
      <n-space>
        <n-button type="primary" @click="handleSubmit">提交</n-button>
        <n-button @click="handleReset">重置</n-button>
      </n-space>
    </n-form-item>
  </n-form>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import type { FormInst, FormRules } from 'naive-ui'

interface FormData {
  username: string
  email: string
}

const formRef = ref<FormInst | null>(null)
const formData = ref<FormData>({
  username: '',
  email: ''
})

const rules: FormRules = {
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 3, max: 20, message: '用户名长度为 3-20 个字符', trigger: 'blur' }
  ],
  email: [
    { required: true, message: '请输入邮箱', trigger: 'blur' },
    { type: 'email', message: '请输入有效的邮箱地址', trigger: 'blur' }
  ]
}

async function handleSubmit() {
  try {
    await formRef.value?.validate()
    // 提交逻辑
  } catch (error) {
    console.log('Validation failed:', error)
  }
}

function handleReset() {
  formRef.value?.restoreValidation()
  formData.value = {
    username: '',
    email: ''
  }
}
</script>
```

## 构建和部署

### 构建

```bash
pnpm build
```

构建产物在 `dist/` 目录。

### Docker 构建

```dockerfile
# Dockerfile
FROM node:18-alpine as builder

WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install

COPY . .
RUN pnpm build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 环境变量

生产环境变量在 `.env.production` 中配置：

```env
VITE_API_URL=https://api.modelgate.com
```

## 代码规范

### TypeScript 规范

- 使用 `interface` 定义对象类型
- 使用 `type` 定义联合类型
- 避免使用 `any`，使用 `unknown` 代替

### Vue 规范

- 使用 `<script setup>` 语法
- 组件名使用 PascalCase
- Props 定义使用 TypeScript 类型

### 样式规范

- 优先使用 UnoCSS 原子类
- 复杂样式使用 `<style scoped>`
- 全局样式放在 `src/assets/styles/`

## 常见问题

### 开发服务器启动失败

检查 Node.js 版本是否符合要求，清理缓存：

```bash
rm -rf node_modules .vite
pnpm install
```

### API 调用失败

检查 `.env.development` 中的 `VITE_API_URL` 是否正确。

### 类型检查失败

运行 `pnpm typecheck` 查看具体的类型错误。
