## Context

前端当前使用 Ant Design 6 作为唯一 UI 库，涉及 4 个文件、3 个 npm 依赖、1 个 Next.js 配置项。本次变更是全量技术栈替换：Ant Design 6 → Tailwind CSS v4 + shadcn/ui。

迁移范围经代码扫描确认（`grep "from \"antd\"|from \"@ant-design"`），受影响文件清单：

| 文件 | Ant Design 用法 | 迁移目标 |
|------|----------------|---------|
| `src/app/layout.tsx` | `AntdRegistry` / `ConfigProvider` / `App` / `zhCN` | Tailwind CSS 导入 + shadcn 主题变量 + `<Toaster />` |
| `src/components/layout/AppLayout.tsx` | `Layout` / `Sider` / `Header` / `Content` / `Menu` / `Breadcrumb` / `Dropdown` / `Tag` / `Modal.confirm` / `App.useApp().message` / 6 个 `@ant-design/icons` | shadcn/ui Sheet + DropdownMenu + Breadcrumb + Dialog + `lucide-react` 图标 |
| `src/app/(auth)/login/page.tsx` | `Form` / `Input` / `Button` / `Card` / `Typography` / `App.useApp().message` | shadcn/ui Card + Input + Button + Label + `react-hook-form` + `zod` + `sonner` |
| `src/lib/client-request.ts` | `message.error()` (2 处) | `sonner` 的 `toast.error()` 或回调注入 |

## Goals / Non-Goals

**Goals:**
- 完全移除 Ant Design 依赖（`package.json` 中 0 个 antd 相关包）
- 所有现有功能在新技术栈下行为等价（登录、登出、布局、路由守卫、错误提示）
- Tailwind CSS v4 正确配置，shadcn/ui CLI 可用（`npx shadcn@latest add <component>` 可正常添加组件）
- 现有 SSR 行为不受影响（游客可访问公开页面，登录态通过 httpOnly cookie 管理）

**Non-Goals:**
- 不改变路由结构、BFF 边界、Zustand store、httpOnly cookie 认证机制（这些与 UI 库无关）
- 不新增功能（纯替换，不增加新页面或新交互）
- 不做视觉设计的全面升级（保持功能等价，视觉优化留给后续首页开发）
- 不迁移 `src/proxy.ts`（路由守卫，无 UI 依赖）

## Decisions

### D1: Tailwind CSS v4（非 v3）
**选择**：Tailwind CSS v4（`@tailwindcss/postcss` 插件方式），适配 Next.js 16。

**理由**：Tailwind v4 简化了配置（零配置 CSS-first），直接在 CSS 中用 `@import "tailwindcss"` 引入，不再需要 `tailwind.config.js`。shadcn/ui 已支持 Tailwind v4 的 `@theme inline` 方案。

**备选**：Tailwind v3（`tailwind.config.ts` + `postcss.config.js`）——否决，v3 配置更繁琐且 v4 是当前稳定方向。

### D2: shadcn/ui 组件策略（本地拷贝非 npm 包）
**选择**：通过 `npx shadcn@latest init` 初始化，组件文件拷贝到 `src/components/ui/`。不是 npm 依赖，而是源码级集成。

**理由**：shadcn/ui 的核心理念是"copy-paste"而非依赖，开发者完全拥有组件源码，可自由定制。

### D3: 表单方案：react-hook-form + zod
**选择**：登录页表单从 Ant Design `Form` 迁移到 `react-hook-form` + `@hookform/resolvers` + `zod`。

**理由**：shadcn/ui 表单组件官方推荐 react-hook-form。zod 提供类型安全的 schema 校验，与 TypeScript strict 模式契合。

**备选**：纯手写表单状态管理——否决，缺乏校验库支持，代码量更大且容易出错。

### D4: Toast 方案：sonner
**选择**：使用 `sonner` 替代 Ant Design `message` API。`<Toaster />` 挂载在 `app/layout.tsx`。

**理由**：shadcn/ui 官方推荐 sonner。API 简洁（`toast.success("msg")`），SSR 友好，体积小（~5KB）。

### D5: client-request.ts 去耦
**选择**：`lib/client-request.ts` 中直接导入 `sonner` 的 `toast` 调用 `toast.error()`，替代 `message.error()`。

**备选**：引入回调注入机制（`onError` 回调）让调用方决定如何展示错误——否决为过度设计（YAGNI），直接用 sonner 更简单。保留回调注入作为注释中的未来扩展点。

### D6: AppLayout 迁移映射

| Ant Design 组件 | shadcn/ui 替代 | 说明 |
|----------------|---------------|------|
| `Layout` / `Sider` / `Header` / `Content` | 原生 `<div>` + Tailwind flexbox | 布局结构用 CSS 而非组件库 |
| `Menu` (侧边栏) | 原生 `<nav>` + Tailwind + `Link` | 菜单项本身就是路由链接 |
| `Dropdown` (用户菜单) | shadcn `DropdownMenu` | |
| `Modal.confirm` (登出确认) | shadcn `Dialog` + `useState` | |
| `App.useApp().message` | `sonner` `toast` | |
| `Tag` (标签页) | 原生 `<span>` + Tailwind badge 类 | |
| `Breadcrumb` | shadcn `Breadcrumb` | |
| `theme.useToken()` | CSS 变量直接引用 | |

### D7: 图标映射

| `@ant-design/icons` | `lucide-react` |
|---------------------|----------------|
| `HomeOutlined` | `Home` |
| `UserOutlined` | `User` |
| `LogoutOutlined` | `LogOut` |
| `MenuFoldOutlined` | `PanelLeftClose` |
| `MenuUnfoldOutlined` | `PanelLeftOpen` |

### D8: globals.css 设计令牌
采用 shadcn/ui 默认的 HSL 色彩变量方案：

```css
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  --primary: 222.2 47.4% 11.2%;
  /* ... shadcn/ui 完整变量集 */
}

.dark {
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;
  /* ... */
}
```

## Risks / Trade-offs

- **[SSR 闪烁] shadcn/ui 组件 SSR 首次渲染可能无样式** → Tailwind v4 + `@tailwindcss/postcss` 在构建时处理，不含运行时 CSS-in-JS，SSR 输出直接包含 class，无闪烁风险
- **[登录页表单校验行为差异] react-hook-form vs Ant Design Form** → 校验时机和错误展示方式可能略有不同；用 zod schema 保证校验规则等价，UI 错误提示位置保持一致
- **[Toaster SSR] sonner `<Toaster>` 需在 Client Component 中渲染** → 放在 `app/layout.tsx` 的 `<body>` 内，作为客户端岛；不影响其余 SSR 内容
- **[回滚成本] 移除 antd 后回滚需重装依赖** → 迁移在独立 git 分支进行，回滚即 `git checkout main`；迁移完成后在 submodule 内提交
- **[AGENTS.md 约束更新滞后] 规范文档与代码不一致** → 迁移完成、验证通过后立即更新 AGENTS.md，作为 tasks 的最后一步
