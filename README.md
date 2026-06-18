# my-first-project

基于 **harness + openspec + superpowers** 三大理念的 AI 辅助编码项目。

融合规范驱动开发、技能方法论和智能体运行时，构建一套完整的"从想法到交付"工作流。

## 三大理念

| 理念 | 解决什么 | 来源 |
|------|---------|------|
| **OpenSpec** | **做什么** — 规范驱动，先对齐再编码 | [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec) |
| **Superpowers** | **怎么做** — 技能方法论，系统化流程 | [obra/superpowers](https://github.com/obra/superpowers) |
| **Harness (Qoder)** | **在哪跑** — AI 智能体运行时 | 当前平台 |

## 快速上手

### 环境要求

- Node.js 20.19.0+
- OpenSpec CLI：`npm install -g @fission-ai/openspec@latest`

### 开始开发

**1. 头脑风暴 + 创建规范**

在 Qoder 中直接输入：

```
/opsx:propose add-dark-mode
```

这会创建一个变更文件夹，包含 `proposal.md`、`design.md`、`tasks.md` 和 delta spec。

**2. 实现任务**

```
/opsx:apply
```

按 `tasks.md` 逐项实现，遵循 TDD（RED-GREEN-REFACTOR）。

**3. 归档变更**

```
/opsx:archive
```

Delta spec 合并到 `openspec/specs/`，变更移至 `changes/archive/`。

### 其他命令

| 命令 | 作用 |
|------|------|
| `/opsx:propose <想法>` | 创建变更并生成全部工件 |
| `/opsx:apply` | 实现任务清单 |
| `/opsx:archive` | 归档变更，合并规范 |
| `/opsx:explore` | 浏览现有规范和变更 |
| `openspec list` | 终端查看活跃变更 |

## 目录结构

```
my-first-project/
├── AGENTS.md                     # 统一编排入口 — 融合三大理念的大脑
├── README.md                     # 本文件
├── .gitignore
├── openspec/                     # [OpenSpec] 规范驱动开发层
│   ├── config.yaml               # OpenSpec 配置
│   ├── specs/                    # 系统行为真相源（source of truth）
│   │   └── README.md             # 规范编写指南
│   └── changes/                  # 变更管理
│       └── README.md             # 变更流程指南
├── .qoder/                       # [Qoder 平台配置]
│   ├── commands/opsx/            # 斜杠命令（propose/apply/archive/explore/sync）
│   ├── skills/                   # openspec-* 技能
│   ├── rules/                    # 项目规范约束
│   │   ├── coding-standards.md   # 编码规范（始终生效）
│   │   ├── git-commit.md         # Git 提交规范（模型决策）
│   │   ├── spec-driven.md        # 规范驱动开发规则（始终生效）
│   │   └── tdd-requirements.md   # TDD 测试要求（模型决策）
│   └── agents/                   # 自定义子智能体
│       ├── code-reviewer.md      # 代码审查智能体
│       ├── test-writer.md        # 测试编写智能体
│       └── spec-reviewer.md      # 规范审查智能体
└── superpowers/                  # [Superpowers] 技能方法论
    ├── README.md                 # 技能体系说明
    └── skills/                   # 核心技能库
        ├── using-superpowers/    # 技能系统入口
        ├── brainstorming/        # 苏格拉底式设计提炼
        ├── writing-plans/        # 设计拆解为任务清单
        ├── executing-plans/      # 分批执行 + 检查点
        ├── test-driven-development/  # RED-GREEN-REFACTOR
        ├── systematic-debugging/ # 4 阶段根因定位
        ├── verification-before-completion/  # 完成前验证
        └── requesting-code-review/   # 代码审查请求
```

## 协同工作流

```
想法 → brainstorming → /opsx:propose → writing-plans → /opsx:apply(TDD) → 验证审查 → /opsx:archive
```

详见 [AGENTS.md](AGENTS.md)。

## 编码准则

1. **先测试，后代码** — RED-GREEN-REFACTOR
2. **YAGNI** — 别写用不到的代码
3. **DRY** — 不要重复自己
4. **小步提交** — 每个逻辑增量一次提交
5. **证据优于声明** — 完成前验证
6. **系统化优于即兴** — 流程优于猜测

## 许可证

MIT
