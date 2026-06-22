## Why

项目前端当前使用 Ant Design 6 作为 UI 组件库，但产品定位面向英语母语游客，需要高度定制化的视觉设计与 mobile-first 响应式布局。Ant Design 的设计语言偏「企业后台」，与面向 C 端游客的产品调性不匹配；同时其 SSR 方案（`AntdRegistry` + `transpilePackages`）增加了构建复杂度。切换到 Tailwind CSS + shadcn/ui 可获得：完全的样式控制力、更小的打包体积、更优的 SSR 兼容性、以及 mobile-first 的设计工作流。

此变更作为首页开发（`build-homepage`）的前置依赖，确保后续所有页面基于统一的新技术栈开发。

## What Changes

- **BREAKING**：移除 Ant Design 6 全家桶（`antd`、`@ant-design/icons`、`@ant-design/nextjs-registry`），切换到 Tailwind CSS v4 + shadcn/ui
- **新增** Tailwind CSS v4 初始化（`postcss.config.mjs` + `globals.css` Tailwind 指令 + CSS 变量主题）
- **新增** shadcn/ui 初始化（`components.json` + `lib/utils.ts` + 基础组件集）
- **新增** `lucide-react` 图标库（替代 `@ant-design/icons`）
- **新增** `sonner` toast 库（替代 Ant Design `message` / `Modal.confirm`）
- **重写** `app/layout.tsx`：移除 `AntdRegistry` / `ConfigProvider`，改为 Tailwind + shadcn/ui 主题注入
- **重写** `components/layout/AppLayout.tsx`：用 shadcn/ui（Sheet / DropdownMenu / Breadcrumb）替代 Ant Design（Layout / Menu / Dropdown / Tag）
- **重写** `app/(auth)/login/page.tsx`：用 shadcn/ui（Card / Input / Button / react-hook-form）替代 Ant Design（Form / Input / Button / Card）
- **重构** `lib/client-request.ts`：将 `message.error()` 替换为 `sonner` toast 或回调注入机制
- **更新** `next.config.ts`：移除 `transpilePackages: ["antd", ...]`
- **更新** `openspec/specs/frontend-scaffold/spec.md`：将 UI 库约束从 Ant Design 6 改为 Tailwind CSS + shadcn/ui
- **更新** `AGENTS.md`：同步前端技术选型约束

## Capabilities

### New Capabilities

<!-- 无新增 capability，本次为现有 frontend-scaffold 的技术栈替换 -->

### Modified Capabilities

- `frontend-scaffold`: 将 UI 组件库从 Ant Design 6 替换为 Tailwind CSS v4 + shadcn/ui；布局模板与运行时行为保持等价

## Impact

- **依赖变更**（package.json）：
  - 移除：`antd`、`@ant-design/icons`、`@ant-design/nextjs-registry`
  - 新增：`tailwindcss`、`@tailwindcss/postcss`、`class-variance-authority`、`clsx`、`tailwind-merge`、`lucide-react`、`sonner`、`react-hook-form`、`@hookform/resolvers`、`zod`
- **配置文件**：`next.config.ts`、`postcss.config.mjs`（新增）、`tsconfig.json`（路径别名）、`globals.css`、`components.json`（新增）
- **受影响代码**（4 个文件重写）：
  - `src/app/layout.tsx`
  - `src/components/layout/AppLayout.tsx`
  - `src/app/(auth)/login/page.tsx`
  - `src/lib/client-request.ts`
- **规范文档**：`AGENTS.md` 前端技术选型约束表、`openspec/specs/frontend-scaffold/spec.md`
- **阻断关系**：本变更 MUST 在 `build-homepage` 的所有前端实现任务之前完成
