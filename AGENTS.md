# AGENTS.md — 项目智能体编排入口

> 本文件是整个项目的大脑，融合三大理念，为 AI 编码智能体提供统一的行为准则。

## 项目定位

这是一个 **语言无关** 的 AI 辅助编码项目，融合三大理念构建一套完整的"从想法到交付"工作流。

## 三大理念分工

| 理念 | 解决的问题 | 在本项目中的位置 |
|------|-----------|-----------------|
| **OpenSpec** | **做什么** — 规范驱动，先对齐再编码 | `openspec/` + `.qoder/` 斜杠命令 |
| **Superpowers** | **怎么做** — 技能方法论，系统化流程 | `superpowers/skills/` |
| **Harness (Qoder)** | **在哪跑** — AI 智能体运行时 | 当前平台 |

### OpenSpec（规范层）

OpenSpec 由 Fission-AI 开源，核心理念是 **"规范是 source of truth，代码是规范的派生产物"**。

- `openspec/specs/` — 系统行为真相源（当前应当如何运作）
- `openspec/changes/` — 变更管理（正在进行的修改）
- 斜杠命令：`/opsx:propose` → `/opsx:apply` → `/opsx:archive`

### Superpowers（方法论层）

Superpowers 由 obra 开源，是一套为 AI 编码智能体设计的完整软件开发方法论。

- 流程技能：brainstorming、systematic-debugging、writing-plans、executing-plans
- 实现技能：test-driven-development、verification-before-completion、requesting-code-review
- 原则：TDD 优先、系统化优于即兴、降低复杂度、证据优于声明

## 指令优先级

**严格遵循以下优先级顺序：**

1. **用户的显式指令**（本文件、直接请求、对话中的明确要求）— 最高优先级
2. **Superpowers 技能**（`superpowers/skills/` 中的方法论）— 覆盖默认行为
3. **OpenSpec 规范**（`openspec/specs/` 中的真相源）— 约束实现范围
4. **默认系统提示** — 最低优先级

> 如果本文件说"不要用 TDD"而某个技能说"总是用 TDD"，遵循本文件。用户始终拥有控制权。

## 技能触发规则

**在任何任务或响应之前，先检查 `superpowers/skills/` 中是否有适用技能。**

即使只有 1% 的可能某个技能适用，也要读取它。技能优先级：

1. **流程技能优先**（brainstorming、systematic-debugging）— 决定"如何"着手
2. **实现技能其次**（TDD、plans）— 指导执行

## 协同工作流

三大理念协同的完整开发循环：

```
┌──────────────────────────────────────────────────────────────┐
│                    理念协同开发循环                            │
│                                                              │
│  1. brainstorming          (Superpowers)                     │
│     提炼想法，探索方案，确认设计                               │
│           │                                                  │
│           ▼                                                  │
│  2. /opsx:propose          (OpenSpec)                        │
│     创建变更，生成 proposal → specs → design → tasks          │
│           │                                                  │
│           ▼                                                  │
│  3. writing-plans          (Superpowers)                     │
│     将 tasks.md 细化为可执行级任务清单                         │
│           │                                                  │
│           ▼                                                  │
│  4. /opsx:apply            (OpenSpec + Superpowers)          │
│     实现任务，遵循 TDD (RED-GREEN-REFACTOR)                   │
│     遇 bug → systematic-debugging                            │
│           │                                                  │
│           ▼                                                  │
│  5. verification + review  (Superpowers)                     │
│     完成前验证 → 代码审查                                     │
│           │                                                  │
│           ▼                                                  │
│  6. /opsx:archive          (OpenSpec)                        │
│     归档变更，delta 合并到 specs/，成为新真相源               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 简化版

```
想法 → brainstorming → /opsx:propose → writing-plans → /opsx:apply(TDD) → 验证审查 → /opsx:archive
```

## Qoder 平台配置说明

`.qoder/` 目录包含四类配置，覆盖从快捷指令到子智能体的完整能力链：

| 目录 | 用途 | 优先级 |
|------|------|--------|
| `.qoder/rules/` | **规范约束** — 编码规范、Git 提交规范、TDD 要求、规范驱动规则 | 始终生效/模型决策 |
| `.qoder/commands/` | **斜杠命令** — OpenSpec 的 /opsx:* 命令 | 手动触发 |
| `.qoder/skills/` | **技能包** — OpenSpec 工作流技能 | 模型自动识别 |
| `.qoder/agents/` | **子智能体** — 代码审查、测试编写、规范审查 | 委派调用 |

### 规则文件（rules/）

规则将团队标准编码为 AI 可遵循的约束。当规则与 AGENTS.md 冲突时，**规则优先**。

| 文件 | 类型 | 作用 |
|------|------|------|
| `coding-standards.md` | 始终生效 | 编码通用原则、风格、错误处理、安全 |
| `spec-driven.md` | 始终生效 | 规范驱动开发工作流（propose→apply→archive）|
| `tdd-requirements.md` | 模型决策 | 实现代码时强制 RED-GREEN-REFACTOR |
| `git-commit.md` | 模型决策 | Conventional Commits 格式 |

### 子智能体（agents/）

子智能体拥有独立上下文和工具权限，适合特定领域的深度任务：

| 智能体 | 触发时机 | 作用 |
|--------|---------|------|
| `code-reviewer` | 任务完成后、提交前 | 对照规范和计划审查代码，按严重程度报告问题 |
| `test-writer` | 实现功能前 | 编写符合 TDD 的测试用例，覆盖正常/边界/错误路径 |
| `spec-reviewer` | /opsx:propose 后 | 审查变更工件质量，确保规范达标再实现 |

## OpenSpec 斜杠命令速查

| 命令 | 作用 | 对应技能 |
|------|------|---------|
| `/opsx:propose <想法>` | 创建变更并生成全部工件 | brainstorming |
| `/opsx:apply` | 实现任务清单 | executing-plans + TDD |
| `/opsx:archive` | 归档变更，合并规范 | verification + review |
| `/opsx:explore` | 浏览现有规范和变更 | — |
| `/opsx:sync` | 同步规范状态 | — |

## 编码准则

无论使用什么语言或框架：

1. **先测试，后代码** — 遵循 RED-GREEN-REFACTOR
2. **YAGNI** — 你不会需要它，别写用不到的代码
3. **DRY** — 不要重复自己
4. **小步提交** — 每个逻辑增量一次提交
5. **证据优于声明** — 完成前验证，用证据证明而非声称
6. **系统化优于即兴** — 流程优于猜测

## 目录结构

```
my-first-project/
├── AGENTS.md                     # 本文件 — 统一编排入口
├── README.md                     # 项目说明与快速上手
├── .gitignore
├── openspec/                     # [OpenSpec] 规范驱动开发层
│   ├── config.yaml
│   ├── specs/                    # 系统行为真相源
│   └── changes/                  # 变更管理
├── .qoder/                       # [Qoder 平台配置] 斜杠命令、技能、规则、子智能体
│   ├── commands/opsx/            # /opsx:propose, apply, archive 等
│   ├── skills/                   # openspec-* 技能
│   ├── rules/                    # 项目规范约束（编码规范、提交规范、TDD、规范驱动）
│   └── agents/                   # 自定义子智能体（代码审查、测试编写、规范审查）
└── superpowers/                  # [Superpowers] 技能方法论
    ├── README.md
    └── skills/                   # 8 个核心技能
```

## 参考

- OpenSpec: https://github.com/Fission-AI/OpenSpec
- Superpowers: https://github.com/obra/superpowers
