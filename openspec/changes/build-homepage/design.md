## Context

本项目是一个面向英语母语游客的来华旅行平台（WanderChina），前端为 Next.js 16 (App Router) + React 19 + Ant Design 6，后端为 Spring Boot 多模块。当前已有前端骨架（layout、登录页）与后端分层架构，但尚无首页页面。

本设计文档覆盖首页的 7 个独立开发单元的技术实现方案。首页是访客落地第一眼，需兼顾 SEO 友好（SSR 渲染）与交互体验（加载态、错误隔离）。

## Goals / Non-Goals

**Goals:**
- 按序组装 5 个内容区块 + 1 个全局悬浮入口，实现独立加载与错误隔离
- 充分利用 SSR 保证首屏内容可被搜索引擎抓取
- 每个 spec 单元可作为独立组件交付，支持并行开发
- 所有静态区块（Hero、城市入口、功能导航）零 API 依赖，可即时渲染

**Non-Goals:**
- 不实现全站搜索（Hero 搜索框仅为占位 UI）
- 不实现个性化推荐算法（所有用户看到相同内容）
- 不实现 Banner 轮播广告位
- 不在本期实现 AI 助手对话的完整 UI（仅做悬浮入口按钮）
- 不新建首页聚合 API（各数据区块直连对应模块接口）

## Decisions

### D1: 路由与页面结构
首页使用 Next.js App Router，在现有 `(dashboard)` 路由组下创建首页页面。

**选择**：复用 `(dashboard)` 路由组而非新建独立路由组，因为首页需要复用已有的全局 Layout（侧边栏、头部），且 AI 助手悬浮按钮挂载于该 Layout。

**备选方案**：创建独立的 `(public)` 路由组供游客访问——否决，因为本期首页与 dashboard 共享同一布局骨架，且游客访问通过 Sa-Token 白名单实现而非路由隔离。

### D2: Server Component vs Client Component 划分
- **Server Component（默认）**：homepage-shell、homepage-hero、homepage-city-quick-access、homepage-feature-nav — 这些区块为静态内容或 SSR 数据获取，无需客户端交互
- **Client Component（`'use client'`）**：homepage-trending-posts、homepage-popular-attractions（需客户端加载态/重试）、homepage-ai-assistant-entry（悬浮按钮交互）、homepage-hero 的搜索框部分（客户端提交反馈）

**数据获取策略**：热门帖子和热门景点在 Server Component 阶段发起首次请求（SSR），传递初始数据给 Client Component；客户端负责重试与刷新。

### D3: 组件目录结构
```
src/components/homepage/
  ├── HomepageShell.tsx        # 骨架容器（组装各区块）
  ├── HeroSection.tsx          # Hero 区
  ├── TrendingPosts.tsx        # 热门帖子
  ├── PopularAttractions.tsx   # 热门景点
  ├── CityQuickAccess.tsx      # 城市快速入口
  ├── FeatureNav.tsx           # 功能导航栏
  └── AiAssistantEntry.tsx     # AI 助手悬浮入口（也可放 layout 级别）
```

### D4: 错误隔离策略
每个数据依赖区块（TrendingPosts、PopularAttractions）包裹独立的 React Error Boundary。某个区块请求失败时，仅该区块显示错误兜底 UI + 重试按钮，不影响其他区块渲染。

### D5: 跨模块数据依赖（R-T2 / R-T3）
- **R-T2**：社区 `post` 表需新增 `upvote_count` 字段（INT, NOT NULL DEFAULT 0）。此字段为首页热门帖子的排序依据。在字段就位前，前端降级为按 `create_time` 降序排序。
- **R-T3**：景点 `attraction` 表需新增 `view_count` 字段（INT, NOT NULL DEFAULT 0）。此字段为首页热门景点的排序依据。在字段就位前，前端降级为按 `popularity` 降序排序。

这两个字段的新增属于社区/景点模块的前置任务，需在首页 #2、#3 单元实现前完成。首页侧仅需适配接口的排序参数。

### D6: 8 城市静态配置
城市快速入口的 8 个 MVP 城市定义为前端静态常量（TypeScript），不依赖 API：

```typescript
const MVP_CITIES = [
  { code: 'BJS', name: 'Beijing', icon: '...' },
  { code: 'SHA', name: 'Shanghai', icon: '...' },
  { code: 'CTU', name: 'Chengdu', icon: '...' },
  { code: 'SIA', name: "Xi'an", icon: '...' },
  { code: 'CAN', name: 'Guangzhou', icon: '...' },
  { code: 'SZX', name: 'Shenzhen', icon: '...' },
  { code: 'HGH', name: 'Hangzhou', icon: '...' },
  { code: 'KWL', name: 'Guilin', icon: '...' },
];
```

### D7: AI 助手悬浮按钮放置层级
悬浮按钮组件放在 `(dashboard)/layout.tsx`（全局 Layout）中，而非 homepage-shell 内。这样它在 dashboard 下所有页面都可见，符合"随时唤起"的需求。

## Risks / Trade-offs

- **[R-T2 未完成] 首页热门帖子降级排序** → 降级为 `create_time` 降序；前端通过 API 返回字段判断是否有 `upvote_count`，有则显示"Upvoted X"，无则隐藏该指标
- **[R-T3 未完成] 首页热门景点降级排序** → 降级为 `popularity` 降序；前端不显示浏览量数字
- **[SEO 与 CSR 数据获取] Server Component 请求可能阻塞首屏** → 设置后端请求超时（3s），超时后该区块降级为客户端异步加载 + skeleton 占位
- **[跨模块依赖] 社区/景点接口可能尚未提供首页所需排序参数** → 前端容错：接口不支持 `sort=upvote_count` 时降级为默认排序，不报错
- **[AI 助手未完成] 悬浮按钮点击后无法打开对话** → 本期点击跳转 `/assistant` 占位页或显示 "Coming Soon" 提示
