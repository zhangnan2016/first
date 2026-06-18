## Why

当前前端基于 Vite + Vue 3 构建，采用纯 SPA 模式。随着项目需要 SEO 能力、服务端首屏渲染以及 Vercel 原生部署体验，Vue SPA 方案已无法满足。迁移到 React 19 + Next.js (App Router) 可获得 SSR/SEO 能力、文件路由约定与 Vercel 零配置部署，同时 Next.js 作为薄 BFF 解决 SSR 场景下的认证态管理问题。

## What Changes

- **BREAKING**: 前端框架从 Vite + Vue 3 迁移到 Next.js 15 (App Router) + React 19
- **BREAKING**: UI 组件库从 Element Plus 切换为 Ant Design
- **BREAKING**: 状态管理从 Pinia 切换为 Zustand
- **BREAKING**: 认证 token 存储从 `localStorage` 迁移到 `httpOnly cookie`，适配 SSR 服务端渲染
- 新增薄 BFF 层：Next.js Route Handlers / Server Actions 负责 cookie 读写与认证请求转发，不承担业务逻辑
- 路由守卫从 `vue-router.beforeEach` 迁移到 Next.js `middleware.ts`
- 环境变量从 `VITE_*` 前缀切换为 `NEXT_PUBLIC_*` 前缀
- 路由从配置式 (`vue-router`) 切换为文件路由约定 (App Router `app/` 目录)
- 部署方式从静态产物托管切换为 Vercel 原生部署
- 保留 axios 请求层，复用 `R<T>` 信封解包与统一错误处理逻辑

## Capabilities

### New Capabilities

- `nextjs-bff`: 薄 BFF 层规范 —— Next.js Route Handlers / Server Actions 负责 httpOnly cookie 写入（登录/登出）、cookie 读取与服务端请求 token 注入、客户端请求代理转发。严格不承担业务逻辑，业务逻辑全部由后端 Spring Boot 处理。

### Modified Capabilities

- `frontend-scaffold`: 技术栈从 Vite + Vue 3 切换为 Next.js 15 (App Router) + React 19 + TypeScript；环境变量从 `VITE_API_BASE_URL` 改为 `NEXT_PUBLIC_API_BASE_URL`；新增 SSR 渲染与 Vercel 部署能力；布局组件从 Vue SFC 改为 React Server/Client Components。
- `frontend-request-layer`: token 存储从 `localStorage` + Pinia 迁移到 `httpOnly cookie`；token 注入在服务端通过 `cookies()` 读取、在客户端通过 cookie 读取注入；路由权限守卫从 `vue-router.beforeEach` 改为 Next.js `middleware.ts`。

## Impact

- **代码**: `frontend/` 子模块整体重写（Vite/Vue 项目删除，Next.js 项目初始化）
- **依赖**: 移除 vue/vue-router/pinia/element-plus/@vitejs/plugin-vue，新增 next/react/react-dom/antd/zustand
- **部署**: 从静态文件托管切换为 Vercel 原生部署（`vercel.json` 或零配置）
- **后端**: 无影响，后端 API 契约 (`R<T>` 信封) 与认证接口保持不变
- **规范**: `frontend-scaffold`、`frontend-request-layer` 的 delta spec 需合并更新；新增 `nextjs-bff` spec
