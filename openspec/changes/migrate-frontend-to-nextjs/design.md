## Context

当前前端是一个 Vue 3 + Vite + TypeScript 的中后台管理系统，以 git submodule 形式托管在 `git@github.com:zhangnan2016/first_frontend.git`。现有功能包括：vue-router 路由守卫、Pinia 管理 localStorage token、axios 请求层（`R<T>` 解包 + 401 跳转）、Element Plus 组件库、登录页 + 布局骨架 + 首页。

后端 Spring Boot 提供统一响应信封 `R<T>`（`code`/`msg`/`data`，`code === 0` 为成功）与 Sa-Token 认证。后端 API 契约在本次变更中保持不变。

迁移目标：React 19 + Next.js 15 (App Router) + TypeScript，启用 SSR/SEO，部署到 Vercel，Next.js 作为薄 BFF 处理认证态。

## Goals / Non-Goals

**Goals:**
- 将前端框架从 Vite + Vue 3 迁移到 Next.js 15 (App Router) + React 19
- 启用 SSR，使服务端可渲染首屏并拉取认证数据
- 认证 token 从 localStorage 迁移到 httpOnly cookie，兼容 SSR 服务端读取
- Next.js 薄 BFF 仅做 cookie 管理 + 请求转发，业务逻辑全在后端
- 复用现有 axios 请求层的 `R<T>` 解包逻辑
- 支持环境变量配置（`NEXT_PUBLIC_*` 客户端 + `API_BASE_URL` 服务端）
- 支持 Vercel 原生部署

**Non-Goals:**
- 不修改后端任何代码或 API 契约
- 不新增业务功能（仅做框架对等迁移）
- 不实现完整的菜单/权限动态路由（仅保留守卫骨架，动态路由留后续迭代）
- 不承担 SEO 内容优化（仅启用 SSR 能力，中后台页面内容不变）

## Decisions

### D1: App Router + React Server Components

**选择**: Next.js 15 App Router，利用 React 19 的 Server Components。

**理由**: App Router 是 Next.js 官方推荐的路由方案，支持文件路由约定、Server Components、流式渲染。用户明确要求 SSR/SEO 能力与文件路由约定。

**替代方案**: Pages Router（旧版路由，不支持 RSC，已不推荐用于新项目）。

### D2: httpOnly cookie 认证 + 薄 BFF

**选择**: token 存 httpOnly cookie，通过 Next.js Route Handlers / Server Actions 读写。

**理由**:
- SSR 服务端渲染时无法访问 `localStorage`，必须用 cookie
- httpOnly 防 XSS 窃取，比 localStorage 更安全
- Server Components 可通过 `cookies()` 读取 token，服务端请求带认证

**替代方案**:
- 保留 localStorage + 纯客户端认证（丢失 SSR 认证数据能力，且 SSR 无意义）
- 使用 NextAuth.js（过重，后端已有 Sa-Token 认证体系，无需引入第三方认证框架）

**架构**:
```
客户端浏览器
  ├── httpOnly cookie: imooc_token / imooc_user_id
  │
  ▼
Next.js (薄 BFF)
  ├── middleware.ts          → 路由守卫（检查 cookie）
  ├── Server Components      → cookies() 读 token，服务端 axios 请求后端
  ├── Route Handlers (/api)  → 客户端请求代理（读 cookie 注入 token）
  └── Server Actions         → 登录/登出（写/清 cookie + 调后端）
  │
  ▼
Spring Boot 后端 (业务逻辑 + Sa-Token)
```

### D3: 客户端请求通过 Route Handler 代理

**选择**: 客户端组件的 API 请求发送到 Next.js Route Handler（`/api/*`），由 Route Handler 读取 cookie 中的 token 注入后转发到后端。

**理由**: 客户端 JavaScript 无法读取 httpOnly cookie，需通过同源 Route Handler 代理。这也实现了"客户端不直接持有 token"的安全目标。

**替代方案**: 在客户端直接读 cookie（需放弃 httpOnly，降低安全性）。

### D4: 保留 axios 请求层

**选择**: 保留 axios，创建两个实例：服务端实例（从 `cookies()` 注入 token）和客户端实例（请求发到 `/api/*` Route Handler）。

**理由**: 迁移成本最低，`R<T>` 解包与统一错误处理逻辑可直接复用。

**替代方案**: 改用 fetch 封装（需重写全部拦截器逻辑，无收益）。

### D5: UI 组件库 Ant Design + 状态管理 Zustand

**选择**: Ant Design（组件库）+ Zustand（客户端状态）。

**理由**: Ant Design 与 Element Plus 功能完全对等（Form 表单校验、message 提示、Layout/Menu/Breadcrumb/Dropdown/Tag），迁移成本最低。Zustand API 风格接近 Pinia，迁移 user store 简单。

### D6: 双环境变量策略

**选择**:
- `NEXT_PUBLIC_API_BASE_URL` — 客户端可见，指向 Next.js Route Handler 前缀（如 `/api`）
- `API_BASE_URL` — 仅服务端可见，指向后端真实地址（如 `http://localhost:8080`）

**理由**: 客户端请求发到同源 Route Handler（`/api/*`），由 Route Handler 用服务端 `API_BASE_URL` 转发到后端，隐藏后端真实地址。

## Risks / Trade-offs

| 风险 | 缓解 |
|------|------|
| Ant Design 与 React 19 / Next.js 15 兼容性 | 使用 antd@5 最新版，已官方支持 React 19；Next.js 15 配置 `transpilePackages: ['antd']` |
| SSR 下 Ant Design 样式闪烁 (FOUC) | 使用 `@ant-design/nextjs-registry` 的 `AntdRegistry` 包裹根布局 |
| httpOnly cookie 跨域问题 | 开发环境通过 Next.js rewrites 代理；生产环境 Vercel 同源部署 |
| 子模块整体重写丢失 git 历史 | 在子模块仓库内创建新分支，保留旧代码历史；旧 Vue 代码可通过 git tag 标记 |
| Route Handler 代理增加一层延迟 | Route Handler 与后端同区域部署（Vercel + 后端云服务器），延迟可控；后续可优化为直连 |

## Migration Plan

1. **准备**: 在 `frontend/` 子模块仓库内创建 `nextjs` 分支
2. **清理**: 删除 Vue/Vite 相关文件（src/、vite.config.ts、index.html 等）
3. **初始化**: `npx create-next-app@latest` 初始化 Next.js 项目（TypeScript + App Router + ESLint）
4. **依赖安装**: antd、@ant-design/nextjs-registry、@ant-design/icons、axios、zustand
5. **基础配置**: tsconfig 路径别名（`@/*`）、环境变量文件、AntdRegistry 根布局
6. **薄 BFF 层**: Route Handlers (`/api/[...path]`) 代理转发、Server Actions 登录/登出
7. **请求层**: 服务端/客户端双 axios 实例、`R<T>` 解包
8. **状态管理**: Zustand user store（从 cookie 初始化）
9. **路由守卫**: middleware.ts 检查 cookie
10. **页面迁移**: 登录页（Client Component）、布局骨架（Server + Client）、首页（Server Component）
11. **验证**: `npm run dev` + `npm run build` 通过
12. **回滚策略**: 子模块切回 main 分支即可完全回滚
