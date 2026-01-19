---
title: 贡献指南
weight: 1
---

# 贡献指南

感谢你有兴趣为 ModelGate 做出贡献！我们欢迎任何形式的贡献。

## 如何贡献

### 报告 Bug

如果你发现了 Bug，请：

1. 检查 [Issues](https://github.com/modelgate/modelgate/issues) 中是否已有相同问题
2. 如果没有，创建新的 Issue，包含：
   - 清晰的标题
   - 详细的 Bug 描述
   - 复现步骤
   - 预期行为
   - 实际行为
   - 环境信息（操作系统、版本等）
   - 相关日志或截图

### 提出新功能

如果你有新功能的想法：

1. 先在 [Discussions](https://github.com/modelgate/modelgate/discussions) 中讨论
2. 创建 Issue 描述你的想法
3. 说明用例和预期效果
4. 等待社区反馈

### 提交代码

1. Fork 项目
2. 创建特性分支：`git checkout -b feature/your-feature`
3. 提交更改：`git commit -m "Add some feature"`
4. 推送分支：`git push origin feature/your-feature`
5. 创建 Pull Request

### 改进文档

文档是项目的重要组成部分，你可以：

- 修正错误
- 补充遗漏的内容
- 添加示例
- 翻译文档

## 代码规范

### Go 代码规范

- 遵循 [Effective Go](https://go.dev/doc/effective_go) 指南
- 使用 `gofmt` 格式化代码
- 使用 `golangci-lint` 进行代码检查

```bash
gofmt -w .
golangci-lint run
```

### Vue/TypeScript 代码规范

- 遵循 Vue 3 [风格指南](https://vuejs.org/style-guide/)
- 使用 ESLint 和 Prettier

```bash
pnpm lint
pnpm format
```

### 提交信息规范

使用 [Conventional Commits](https://www.conventionalcommits.org/) 规范：

```
<type>(<scope>): <subject>

<body>

<footer>
```

**类型 (type)**：

- `feat`: 新功能
- `fix`: Bug 修复
- `docs`: 文档更新
- `style`: 代码格式（不影响代码运行）
- `refactor`: 重构
- `perf`: 性能优化
- `test`: 测试相关
- `chore`: 构建或辅助工具的变动
- `ci`: CI 配置文件和脚本的变动

**示例**：

```
feat(relay): add support for new provider

Add support for XYZ provider including:
- Provider configuration
- API key management
- Request forwarding

Closes #123
```

## Pull Request 指南

### PR 标题

使用清晰的标题，格式：`[类型] 简短描述`

例如：
- `[Feature] Add support for streaming responses`
- `[Bugfix] Fix token counting error`
- `[Docs] Update installation guide`

### PR 描述

包含以下信息：

- 变更说明
- 相关 Issue
- 测试情况
- 截图（如适用）

### 代码审查

- 保持 PR 小而聚焦
- 确保代码通过所有测试
- 响应审查意见
- 及时更新代码

## 测试

### 后端测试

```bash
# 运行所有测试
go test ./...

# 运行特定测试
go test ./internal/module/relay/...

# 查看覆盖率
go test -cover ./...
```

### 前端测试

```bash
# 运行单元测试
pnpm test

# 运行 E2E 测试
pnpm test:e2e
```

## 发布流程

### 版本号

遵循 [语义化版本](https://semver.org/lang/zh-CN/)：

- 主版本号：不兼容的 API 变更
- 次版本号：向下兼容的功能新增
- 修订号：向下兼容的 Bug 修复

### 发布步骤

1. 更新版本号
2. 更新 CHANGELOG
3. 创建 Git 标签
4. 发布 Release

## 社区准则

- 尊重他人
- 保持友好和专业
- 接受建设性批评
- 关注对社区最有利的事情

## 获取帮助

如果你需要帮助：

- 查看 [文档](https://modelgate.github.io)
- 在 [Discussions](https://github.com/modelgate/modelgate/discussions) 中提问
- 加入我们的交流群（如有）

## 许可证

提交代码即表示你同意将代码以 [MIT 许可证](https://github.com/modelgate/modelgate/blob/main/LICENSE) 发布。

---

再次感谢你的贡献！
