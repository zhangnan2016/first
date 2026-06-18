---
trigger: always_on
---
# Git 提交规范

模型决策：当用户请求提交代码、生成 commit message 或执行 git commit 时应用。

## Conventional Commits 格式

所有提交消息使用 Conventional Commits 规范：

```
<type>(<scope>): <subject>

<body>

<footer>
```

## Type 取值

| Type | 含义 |
|------|------|
| `feat` | 新功能 |
| `fix` | Bug 修复 |
| `docs` | 文档变更 |
| `style` | 代码格式（不影响功能） |
| `refactor` | 重构（非新功能、非修复） |
| `test` | 新增或修改测试 |
| `chore` | 构建、依赖、配置等杂项 |
| `ci` | CI/CD 配置变更 |
| `perf` | 性能优化 |

## 规则

- **Subject**：使用祈使句，首字母不大写，结尾不加句号，不超过 50 字符
- **Scope**（可选）：标识影响的模块，如 `auth`、`api`、`ui`
- **Body**（可选）：说明"为什么"做这个改动，每行不超过 72 字符
- **Footer**（可选）：关联 issue，如 `Closes #123`
- 小步提交：每个逻辑变更一次提交，不要把无关改动混在一起

## 示例

```
feat(auth): 添加 JWT 令牌刷新机制

会话过期前 5 分钟自动刷新令牌，避免用户操作中断。
使用 silent refresh 策略，兼容旧版令牌。

Closes #42
```

```
fix(payment): 修复金额为 0 时支付失败的问题

支付网关拒绝 amount=0 的请求，改为在金额为 0 时
跳过网关直接标记为已支付。
```
