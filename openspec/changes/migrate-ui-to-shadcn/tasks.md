## 1. 安装与初始化 Tailwind CSS v4 + shadcn/ui

- [ ] 1.1 安装 Tailwind CSS v4 依赖：`npm install tailwindcss @tailwindcss/postcss`，创建 `postcss.config.mjs` 配置 `@tailwindcss/postcss` 插件
- [ ] 1.2 重写 `src/app/globals.css`：添加 `@import "tailwindcss"` 指令 + shadcn/ui CSS 变量（`:root` + `.dark`）+ `@theme inline` 映射
- [ ] 1.3 运行 `npx shadcn@latest init` 初始化 shadcn/ui（生成 `components.json` + `lib/utils.ts`），确认 style=new-york、base color=slate、css variables=yes
- [ ] 1.4 安装基础依赖：`npm install class-variance-authority clsx tailwind-merge lucide-react sonner react-hook-form @hookform/resolvers zod`
- [ ] 1.5 通过 shadcn CLI 添加基础组件集：`npx shadcn@latest add button input card label dropdown-menu dialog sheet breadcrumb sonner`

## 2. 移除 Ant Design 依赖与配置

- [ ] 2.1 卸载 Ant Design 包：`npm uninstall antd @ant-design/icons @ant-design/nextjs-registry`
- [ ] 2.2 更新 `next.config.ts`：移除 `transpilePackages: ["antd", "@ant-design/icons", "@ant-design/cssinjs"]`，清理为空配置或仅保留必要项
- [ ] 2.3 全局搜索确认无残留 antd 导入：`grep -r "from \"antd\"\|from \"@ant-design" src/` 返回空结果

## 3. 重写 app/layout.tsx（根布局）

- [ ] 3.1 移除 `AntdRegistry` / `ConfigProvider` / `App as AntdApp` / `zhCN` 导入
- [ ] 3.2 重写为：`<html>` + `<body className="...">` + Tailwind 全局样式 + shadcn/ui `<Toaster />` 挂载
- [ ] 3.3 更新 `metadata`：将 title/description 更新为 WanderChina 品牌信息
- [ ] 3.4 验证：`npm run dev` 启动后页面无样式错误，Toaster 正常挂载

## 4. 重写 lib/client-request.ts（HTTP 拦截器去耦）

- [ ] 4.1 将 `import { message } from "antd"` 替换为 `import { toast } from "sonner"`
- [ ] 4.2 将响应拦截器中 `message.error(res.msg || "请求失败")` 替换为 `toast.error(res.msg || "请求失败")`
- [ ] 4.3 将错误拦截器中 `message.error(msg)` 替换为 `toast.error(msg)`
- [ ] 4.4 验证：模拟 API 错误时 toast 正确弹出

## 5. 重写 AppLayout.tsx（全局布局）

- [ ] 5.1 将布局结构从 Ant Design `Layout` / `Sider` / `Header` / `Content` 改为原生 `<div>` + Tailwind flexbox 布局
- [ ] 5.2 将侧边栏 `Menu` 改为原生 `<nav>` + `<Link>` + Tailwind 样式（active 状态用 `usePathname` 判断）
- [ ] 5.3 将用户菜单 `Dropdown` 替换为 shadcn `DropdownMenu` 组件
- [ ] 5.4 将登出确认 `Modal.confirm` 替换为 shadcn `Dialog` + `useState` 控制开关
- [ ] 5.5 将 `App.useApp().message` 替换为 `sonner` 的 `toast`
- [ ] 5.6 将 `Breadcrumb` 替换为 shadcn `Breadcrumb` 组件
- [ ] 5.7 将 `Tag`（标签页）替换为 Tailwind badge 样式的 `<span>`
- [ ] 5.8 将 `theme.useToken()` 替换为 CSS 变量直接引用（如 `bg-background`、`text-foreground`）
- [ ] 5.9 替换全部图标：`@ant-design/icons` → `lucide-react`（Home/User/LogOut/PanelLeftClose/PanelLeftOpen）
- [ ] 5.10 验证：侧边栏菜单、折叠、面包屑、用户下拉、登出确认全部功能正常

## 6. 重写 login/page.tsx（登录页）

- [ ] 6.1 将 Ant Design `Card` 替换为 shadcn `Card`（CardHeader / CardContent）
- [ ] 6.2 将 Ant Design `Form` + `Form.Item` 替换为 `react-hook-form` + `zod` schema 校验 + shadcn `Form` 封装
- [ ] 6.3 将 Ant Design `Input` / `Input.Password` 替换为 shadcn `Input`
- [ ] 6.4 将 Ant Design `Button` 替换为 shadcn `Button`
- [ ] 6.5 将 `App.useApp().message.success("登录成功")` 替换为 `toast.success("登录成功")`
- [ ] 6.6 将页面容器 inline style 替换为 Tailwind 类（`flex items-center justify-center min-h-screen bg-muted`）
- [ ] 6.7 验证：登录表单校验（空值拦截）、提交 loading 态、成功跳转、错误 toast 全部正常

## 7. 构建验证与端到端测试

- [ ] 7.1 `npm run build` 成功，无 TypeScript 错误、无 ESLint 错误
- [ ] 7.2 `npm run dev` 启动后页面正常渲染（首页占位 + 登录页）
- [ ] 7.3 登录流程端到端验证：登录 → 跳转首页 → 登出 → 跳转登录页
- [ ] 7.4 API 错误提示验证：模拟后端不可用，toast 正确显示错误消息
- [ ] 7.5 SSR 验证：`view-source` 确认页面 HTML 包含布局结构（非空 body）
- [ ] 7.6 移动端验证：Chrome DevTools 移动端视口下布局正常

## 8. 更新规范文档

- [ ] 8.1 更新 `openspec/specs/frontend-scaffold/spec.md`：将所有 "Ant Design 6" 替换为 "Tailwind CSS v4 + shadcn/ui"，更新对应 scenario
- [ ] 8.2 更新 `AGENTS.md` 前端技术选型约束表：UI 库从 "Ant Design 6" 改为 "shadcn/ui + Tailwind CSS v4"，更新所有相关约束说明
- [ ] 8.3 更新 `frontend/AGENTS.md`（如存在）：同步技术栈描述
