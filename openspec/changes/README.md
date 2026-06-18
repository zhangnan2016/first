# Changes — 变更管理

本目录存放 **正在进行的变更（proposed modifications）**。每个变更是一个独立文件夹。

## 工作流

```
/opsx:propose → /opsx:apply → /opsx:archive
   (提案)          (实现)         (归档)
```

## 变更结构

每个变更（change）是一个自包含的文件夹：

```
changes/add-dark-mode/
├── proposal.md           # 为什么 + 做什么（why & what）
├── design.md             # 怎么做（技术方案）
├── tasks.md              # 实现清单（步骤）
├── .openspec.yaml        # 变更元数据（可选）
└── specs/                # Delta spec（增量规范）
    └── ui/
        └── spec.md       # ui/spec.md 中将要改变的内容
```

## 工件流（Artifact Flow）

工件之间存在依赖关系，但依赖是"使能"而非"门控"：

```
proposal ──────► specs ──────► design ──────► tasks ──────► implement
   why             what          how          steps
 + scope        changes       approach      to take
```

- **proposal** — 意图、范围和方法
- **specs (delta)** — 相对当前规范的变更（ADDED / MODIFIED / REMOVED）
- **design** — 技术方案与架构决策
- **tasks** — 带层级编号的实现清单

## Delta Spec（增量规范）

变更不是重写整个规范，而是描述 **差异（delta）**：

| 段落 | 含义 | 归档时发生什么 |
|---------|---------|------------------------|
| `## ADDED Requirements` | 新增行为 | 追加到主 spec |
| `## MODIFIED Requirements` | 变更行为 | 替换现有需求 |
| `## REMOVED Requirements` | 废弃行为 | 从主 spec 删除 |

## 为什么用文件夹而非单个文件

1. **一切集中** — proposal、design、tasks、specs 同处一地
2. **并行工作** — 多个变更可同时存在互不冲突
3. **干净历史** — 归档后完整上下文保留在 `archive/`
4. **便于审查** — 打开文件夹即可理解变更全貌

## 归档（Archive）

实现完成后运行 `/opsx:archive`：
1. Delta spec 合并到 `specs/`（真相源）
2. 变更文件夹移至 `changes/archive/`（加日期前缀）
3. 所有工件保留，便于审计

```
changes/
├── add-dark-mode/          # 进行中
└── archive/                # 已完成
    └── 2026-01-23-add-2fa/
```

## 命名建议

使用清晰的 kebab-case 名称，让 `openspec list` 更有用：
- `add-dark-mode`、`fix-auth-timeout`、`refactor-payment-flow`
