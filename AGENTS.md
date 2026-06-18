# AGENTS.md — 项目智能体编排入口

> 本文件是整个项目的大脑，融合三大理念，为 AI 编码智能体提供统一的行为准则。

## 项目定位

这是一个 **语言无关** 的 AI 辅助编码工作区（AIWorkspace），融合三大理念构建一套完整的"从想法到交付"工作流。应用代码（前端、后端）以 **git submodule** 形式独立托管，根仓库仅承载规范、技能与编排配置。

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

## 应用代码层（frontend / backend）

前端与后端是实际的应用代码，以 **git submodule** 形式纳入根仓库，各自拥有独立仓库与提交历史。根仓库通过 `.gitmodules` 记录子模块的仓库地址与锁定的提交版本。

| 子模块 | 路径 | 仓库 | 技术栈 | 职责 |
|--------|------|------|--------|------|
| **frontend** | `frontend/` | `git@github.com:zhangnan2016/first_frontend.git` | Next.js 16 (App Router) + React 19 + TypeScript + Ant Design 6 | 用户界面、SSR、薄 BFF、路由守卫、登录态管理 |
| **backend** | `backend/` | `git@github.com:zhangnan2016/first_backend.git` | Spring Boot + Spring Data JPA/Hibernate（Maven 多模块） | 业务逻辑、数据访问、统一响应封装 |

### 后端模块结构（backend/）

Spring Boot 多模块工程，模块间单向依赖（web → service → dal → model → common）：

| 模块 | 职责 |
|------|------|
| `imooc-common` | 通用 API（`R`、`PageResult`、`ResultCode`）、`BaseEntity`、`BizException` |
| `imooc-model` | 实体、DTO、VO、Query 等数据模型（如 `SysUser` 全家桶） |
| `imooc-dal` | 数据访问层：Spring Data JPA Repository、`JpaConfig`、逻辑删除（`@SQLDelete`+`@SQLRestriction`） |
| `imooc-service` | 业务服务层：`ISysUserService` 及其实现 |
| `imooc-web` | Web 启动入口（`ImoocApplication`）、控制器、全局异常处理、多环境配置 |

### 技术选型约束

后端持久层技术选型已确定，**禁止混用或回退**：

| 维度 | 选型 | 约束说明 |
|------|------|----------|
| **ORM 框架** | Spring Data JPA + Hibernate | 禁止引入 MyBatis / MyBatis-Plus |
| **主键生成** | 雪花算法（自定义 `SnowflakeIdGenerator`） | Hibernate `@GenericGenerator` 策略，等价原 `IdType.ASSIGN_ID` |
| **逻辑删除** | `@SQLDelete` + `@SQLRestriction` | 实体类声明，非全局配置；remove → UPDATE deleted=1，查询自动过滤 |
| **审计字段** | JPA 生命周期回调 `@PrePersist`/`@PreUpdate` | 替代 MyBatis-Plus `MetaObjectHandler` |
| **动态查询** | JPA `Specification` + `CriteriaBuilder` | 替代 MyBatis-Plus `LambdaQueryWrapper` |
| **DDL 策略** | `hibernate.ddl-auto: validate` | 仅校验表结构，禁止自动建表/改表 |
| **命名策略** | `CamelCaseToUnderscoresNamingStrategy` | 驼峰自动转下划线，实体字段无需手写 `@Column` |

> 新增实体或 Repository 时，遵循上述约束。如需引入其他持久化方案（如 Redis、MongoDB），通过独立的 starter 引入，不得替换或混入 JPA 管理的实体。

### 前端技术选型约束

前端技术选型已确定，**禁止混用或回退**：

| 维度 | 选型 | 约束说明 |
|------|------|----------|
| **框架** | Next.js 16 (App Router) | 禁止使用 Pages Router 或回退到 Vue |
| **UI 库** | React 19 + Ant Design 6 | 禁止引入其他 UI 组件库（如 MUI、Chakra）；SSR 需用 `AntdRegistry` |
| **语言** | TypeScript（strict 模式） | 禁止使用纯 JavaScript；`tsconfig.json` 的 `strict: true` 不可关闭 |
| **状态管理** | Zustand（仅客户端） | 禁止引入 Redux/Pinia；token 不入 store，由 httpOnly cookie 管理 |
| **HTTP 客户端** | Axios（双实例架构） | 服务端 `server-request.ts`（直连后端）+ 客户端 `client-request.ts`（走 BFF 代理） |
| **认证机制** | httpOnly cookie + Sa-Token | token 存 httpOnly cookie，服务端经 `cookies()` 读取注入；禁止 localStorage 存 token |
| **路由守卫** | `proxy.ts`（Edge Middleware） | 检查 `imooc_token` cookie，未认证重定向登录页；禁止用客户端 JS 守卫 |
| **BFF 边界** | 薄 BFF（仅 cookie/token 代理） | Route Handler 仅做 cookie 读写与请求转发，**禁止包含业务逻辑** |
| **环境配置** | `.env.development` / `.env.production` | `NEXT_PUBLIC_*` 暴露给客户端，无前缀变量仅服务端可用 |
| **构建工具** | Next.js 内置（webpack/turbopack） | 禁止引入独立 Vite/Rollup 配置 |

> 新增页面或组件时，遵循上述约束。服务端渲染优先（Server Components），仅在需要交互时使用 `'use client'` 声明 Client Components。

### 前端结构（frontend/）

| 目录/文件 | 职责 |
|-----------|------|
| `src/app/` | Next.js App Router 页面与布局（`layout.tsx`、`(auth)/login`、`(dashboard)`） |
| `src/lib/server-request.ts` | 服务端 axios 实例（`cookies()` 读 token，注入 `Authorization`） |
| `src/lib/client-request.ts` | 客户端 axios 实例（走 `/api` Route Handler 代理，401 重定向） |
| `src/lib/auth.ts` | 服务端认证辅助（`getToken`、`getUserId`、`isLoggedIn`） |
| `src/stores/user.ts` | Zustand 客户端用户状态（userId、permissions；token 不入 store） |
| `src/proxy.ts` | Next.js 路由权限守卫（检查 `imooc_token` cookie，重定向逻辑） |
| `src/components/layout/AppLayout.tsx` | 整体布局骨架（侧边栏、头部、面包屑、标签页） |
| `src/app/api/[...path]/` | Route Handler 代理层（客户端请求经此转发到后端） |
| `src/app/api/auth/` | 认证 Route Handler / Server Action（登录写 cookie、登出清 cookie） |

### Submodule 工作流

```bash
# 首次克隆含子模块的完整工作区
git clone --recurse-submodules <root-repo>

# 已克隆但未拉子模块时初始化
git submodule update --init --recursive

# 拉取子模块远程最新提交并合并
git submodule update --remote --merge

# 在子模块内开发：先进子模块提交推送，再回根仓库记录指针
cd frontend
git add . && git commit -m "feat(ui): ..." && git push
cd ..
git add frontend && git commit -m "chore: 升级 frontend 子模块指针"
```

> **关键约束**：子模块变更必须先在子模块仓库内提交并推送，再在根仓库提交 gitlink 指针。切勿在根仓库直接修改子模块内文件而不进入子模块目录，否则会产生游离的脏状态。

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
my-first-project/  (AIWorkspace 根仓库 → first.git)
├── AGENTS.md                     # 本文件 — 统一编排入口
├── README.md                     # 项目说明与快速上手
├── .gitignore
├── .gitmodules                   # 子模块注册表（frontend / backend）
├── frontend/                     # [git submodule] Next.js 16 + React 19 前端 → first_frontend.git
├── backend/                      # [git submodule] Spring Boot 后端 → first_backend.git
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
- frontend 子模块: https://github.com/zhangnan2016/first_frontend
- backend 子模块: https://github.com/zhangnan2016/first_backend
