## 1. 子模块准备与项目初始化

- [x] 1.1 进入 `frontend/` 子模块，创建 `nextjs` 迁移分支
- [x] 1.2 删除 Vue/Vite 相关文件（`src/`、`vite.config.ts`、`index.html`、`tsconfig.*`、`auto-imports.d.ts`、`components.d.ts`）
- [x] 1.3 使用 `npx create-next-app@latest .` 初始化 Next.js 项目（TypeScript + App Router + ESLint + src/ 目录）
- [x] 1.4 验证 `npm run dev` 启动成功，默认页面正常渲染

## 2. 依赖安装与基础配置

- [x] 2.1 安装运行时依赖：`antd`、`@ant-design/icons`、`@ant-design/nextjs-registry`、`axios`、`zustand`
- [x] 2.2 配置 `tsconfig.json` 路径别名 `@/*` → `./src/*`
- [x] 2.3 配置 `next.config.ts` 添加 `transpilePackages: ['antd']`
- [x] 2.4 创建 `.env.development`（`NEXT_PUBLIC_API_BASE_URL=/api`、`API_BASE_URL=http://localhost:8080`）
- [x] 2.5 创建 `.env.production`（`NEXT_PUBLIC_API_BASE_URL=/api`、`API_BASE_URL=` 留空由部署平台注入）
- [x] 2.6 更新 `app/layout.tsx`，用 `AntdRegistry` 包裹 children，配置 Antd `App` 组件与 `ConfigProvider`

## 3. 薄 BFF 层（Route Handlers + Server Actions）

- [x] 3.1 创建 `src/app/api/[...path]/route.ts`，实现 catch-all Route Handler 代理：读取 cookie 中 token 注入 `Authorization` 头，转发到后端 `API_BASE_URL`
- [x] 3.2 创建 `src/app/api/auth/login/route.ts`，调用后端登录接口，成功后通过 `Set-Cookie` 写入 `imooc_token` 与 `imooc_user_id`（httpOnly + secure + sameSite=lax）
- [x] 3.3 创建 `src/app/api/auth/logout/route.ts`，调用后端登出接口并清除认证 cookie
- [x] 3.4 验证 Route Handler 代理转发正常请求且注入 token 头
- [x] 3.5 验证 Route Handler 不包含业务逻辑（仅 cookie 读写 + 请求透传）

## 4. 请求层（双 axios 实例 + R<T> 解包）

- [x] 4.1 创建 `src/lib/api-result.ts`，定义 `ApiResult<T>` 类型与 `R<T>` 信封结构
- [x] 4.2 创建 `src/lib/server-request.ts`，服务端 axios 实例：baseURL 读 `API_BASE_URL`，请求拦截器从 `cookies()` 读 token 注入，响应拦截器解包 `R<T>` + 401 处理
- [x] 4.3 创建 `src/lib/client-request.ts`，客户端 axios 实例：baseURL 读 `NEXT_PUBLIC_API_BASE_URL`（指向 `/api`），响应拦截器解包 `R<T>` + 401 跳转
- [x] 4.4 验证服务端请求成功注入 token 并解包 `R<T>`
- [x] 4.5 验证客户端请求经 Route Handler 代理并解包 `R<T>`

## 5. 状态管理（Zustand）

- [x] 5.1 创建 `src/stores/user.ts`，Zustand user store：管理 `userId`、`permissions`，提供 `setUser`、`clearUser` 方法（token 由 cookie 管理，不存 store）
- [x] 5.2 创建 `src/lib/auth.ts`，服务端辅助函数：从 `cookies()` 读取 token 与 userId，提供 `getToken()`、`getUserId()`、`isLoggedIn()`

## 6. 路由守卫（middleware.ts）

- [x] 6.1 创建 `src/middleware.ts`，检查 `imooc_token` cookie：未认证访问受保护路由 → 重定向 `/login?redirect=<原路径>`
- [x] 6.2 配置 matcher 排除静态资源与 `/login`、`/api/auth/login`
- [x] 6.3 验证未认证访问受保护路由被重定向到登录页
- [x] 6.4 验证已认证用户访问 `/login` 被重定向到首页

## 7. 页面与组件迁移

- [x] 7.1 创建 `src/api/auth.ts`，认证 API 调用函数：`login`（POST `/api/auth/login`）、`logout`（POST `/api/auth/logout`）、`getUserInfo`
- [x] 7.2 创建 `src/app/(auth)/login/page.tsx`，登录页 Client Component：Ant Design Form 表单校验 + 提交调登录接口 + 成功后跳转
- [x] 7.3 创建 `src/components/layout/AppLayout.tsx`，布局骨架：Ant Design Layout + Sider（Menu）+ Header（Breadcrumb + Dropdown 用户菜单）+ TagsView + Content 区域
- [x] 7.4 创建 `src/app/(dashboard)/layout.tsx`，受保护布局：Server Component，用 `AppLayout` 包裹子路由
- [x] 7.5 创建 `src/app/(dashboard)/page.tsx`，首页：展示欢迎信息，Server Component 渲染
- [x] 7.6 迁移登录页样式与布局骨架样式（侧边栏、顶栏、面包屑、标签视图）
- [x] 7.7 验证登录 → 跳转首页 → 布局渲染完整（侧边栏/顶栏/面包屑/标签视图）

## 8. 构建验证与部署配置

- [x] 8.1 运行 `npm run build` 验证生产构建无错误
- [x] 8.2 创建 `vercel.json`（如需自定义配置）或确认零配置部署可用
- [x] 8.3 验证 `.gitignore` 排除 `.next/`、`node_modules/`
- [ ] 8.4 在子模块仓库提交并推送 `nextjs` 分支
- [ ] 8.5 回到根仓库更新 `frontend` 子模块指针（`git add frontend && git commit`）
