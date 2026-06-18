# Specs — 系统行为真相源

本目录是项目规范的 **source of truth（唯一真相源）**，描述系统当前应当如何运作。

> 核心原则：**规范是 source of truth，代码是规范的派生产物。**

## 目录组织

按 **领域（domain）** 划分，每个领域一个子目录，内含 `spec.md`：

```
specs/
├── auth/
│   └── spec.md           # 认证与会话管理
├── payments/
│   └── spec.md           # 支付处理
└── ui/
    └── spec.md           # UI 行为与主题
```

常见划分方式：
- **按功能区域**：`auth/`、`payments/`、`search/`
- **按组件**：`api/`、`frontend/`、`workers/`
- **按限界上下文**：`ordering/`、`fulfillment/`、`inventory/`

## 规范格式

每个 spec 包含 **需求（Requirements）**，每个需求包含 **场景（Scenarios）**：

```markdown
# Auth Specification

## Purpose
认证与会话管理。

## Requirements

### Requirement: User Authentication
系统 SHALL 在成功登录后签发 JWT 令牌。

#### Scenario: Valid credentials
- GIVEN 一个拥有有效凭证的用户
- WHEN 用户提交登录表单
- THEN 返回一个 JWT 令牌
- AND 用户被重定向到仪表盘

#### Scenario: Invalid credentials
- GIVEN 无效凭证
- WHEN 用户提交登录表单
- THEN 显示错误信息
- AND 不签发任何令牌
```

## 关键元素

| 元素 | 用途 |
|---------|---------|
| `## Purpose` | 该 spec 领域的高层描述 |
| `### Requirement:` | 系统必须具备的具体行为 |
| `#### Scenario:` | 需求的具体示例 |
| SHALL/MUST/SHOULD | RFC 2119 关键词，表示需求强度 |

## 规范是什么（以及不是什么）

spec 是 **行为契约（behavior contract）**，而非实现计划。

**应包含：**
- 用户或下游系统依赖的可观察行为
- 输入、输出和错误条件
- 外部约束（安全、隐私、可靠性、兼容性）

**应避免：**
- 内部类名/函数名
- 库或框架选择
- 分步实现细节（这些属于 `design.md` 或 `tasks.md`）

**快速检验：** 如果实现方式可以改变而不改变对外可见的行为，那它很可能不属于 spec。

## 如何更新规范

不要直接编辑此目录的 spec 来"加功能"。正确做法：
1. 运行 `/opsx:propose <你的想法>` 创建变更提案
2. 在变更的 `specs/` 目录中编写 **delta spec**（增量规范）
3. 实现完成后运行 `/opsx:archive`，delta 会自动合并到此处

> 参阅 OpenSpec concepts: https://github.com/Fission-AI/OpenSpec/blob/master/docs/concepts.md
