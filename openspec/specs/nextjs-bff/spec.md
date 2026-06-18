# Next.js BFF

## Purpose

Defines the thin Backend-for-Frontend layer using Next.js Route Handlers and Server Actions for httpOnly cookie management, server-side token injection, and client request proxying. The BFF layer SHALL NOT contain business logic — all business logic resides in the backend Spring Boot services.

## Requirements

### Requirement: 登录后写入 httpOnly cookie
登录成功后，Next.js Route Handler / Server Action SHALL 将后端返回的 token 写入 `httpOnly`、`secure`、`sameSite=lax` 的 cookie，使服务端与客户端均可读取而 JavaScript 无法直接访问。

#### Scenario: 登录成功设置 cookie
- **WHEN** 用户提交有效凭据，后端返回 token 与 userId
- **THEN** Next.js 将 token 写入名为 `imooc_token` 的 httpOnly cookie，并将 userId 写入 `imooc_user_id` cookie

#### Scenario: cookie 防 XSS 窃取
- **WHEN** token 已写入 httpOnly cookie
- **THEN** 客户端 JavaScript 无法通过 `document.cookie` 读取 token 值

### Requirement: 登出时清除 cookie
登出操作 SHALL 通过 Server Action / Route Handler 清除认证相关 cookie，并调用后端登出接口。

#### Scenario: 登出清除认证态
- **WHEN** 用户执行登出
- **THEN** `imooc_token` 与 `imooc_user_id` cookie 被清除，并重定向到登录页

### Requirement: 服务端请求 token 注入
Server Components 发起的后端请求 SHALL 从请求上下文中读取 cookie 中的 token，注入到 `Authorization: Bearer <token>` 请求头。

#### Scenario: 服务端请求携带 token
- **WHEN** 一个 Server Component 在已登录状态下发起后端请求
- **THEN** 服务端 axios 实例从 `cookies()` 读取 token 并注入 `Authorization` 请求头

#### Scenario: 无 token 时服务端请求省略请求头
- **WHEN** 一个 Server Component 在未登录状态下发起后端请求
- **THEN** 请求不携带 `Authorization` 请求头

### Requirement: 客户端请求 token 代理
客户端请求 SHALL 通过 Next.js Route Handler 代理转发，由 Route Handler 读取 cookie 中的 token 注入后端请求，使客户端无需直接持有 token。

#### Scenario: 客户端请求经代理转发
- **WHEN** 客户端组件发起需要认证的 API 请求
- **THEN** 请求发送到 Next.js Route Handler，由其读取 cookie 注入 token 后转发到后端

### Requirement: 业务逻辑隔离
薄 BFF 层 SHALL 仅负责认证 cookie 管理、token 注入与请求转发，SHALL NOT 承担任何业务逻辑（如数据转换、业务校验、状态聚合）。

#### Scenario: BFF 不包含业务逻辑
- **WHEN** 审查 Route Handler / Server Action 代码
- **THEN** 其中不包含数据转换、业务校验或状态聚合逻辑，仅做 cookie 读写与请求透传
