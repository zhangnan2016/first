## Why

WanderChina 首页是访客落地第一眼，需要用「品牌主视觉 + 热门内容 + 快速入口」聚合平台核心价值，降低首次决策成本并承载 SEO 获客。当前项目已有前端骨架与后端分层架构，但尚无首页页面。本次变更将首页拆分为 7 个可独立开发、独立测试、独立交付的 spec 单元，以支持并行开发与增量交付。

## What Changes

- **新增首页页面骨架**：按序组装各内容区块（Hero → 热门帖子 → 热门景点 → 城市入口 → 功能导航），统一处理加载态/空态/错误兜底与 SEO meta
- **新增 Hero 区**：品牌主视觉背景图 + 标语"Unlock the Real China" + 副标题 + 搜索框（占位 UI，本期不做全站搜索）
- **新增热门帖子区**：过去 7 天按 Upvote 数降序取 Top 10，直连社区帖子列表接口
- **新增热门景点区**：按浏览量降序展示 6–8 个城市/景点卡片，直连景点列表接口
- **新增城市快速入口**：8 个 MVP 城市（北京/上海/成都/西安/广州/深圳/杭州/桂林）图标网格，点击跳转城市攻略占位页
- **新增 AI 助手入口**：全局悬浮按钮，随时唤起 AI 对话
- **新增功能导航栏**：旅游社区 / 景点攻略 / AI 助手 三大入口卡片（图标+标题+一句话描述）

## Capabilities

### New Capabilities

- `homepage-shell`: 首页页面骨架——组装各内容区块的容器，统一处理加载态/空态/错误兜底与 SEO meta
- `homepage-hero`: Hero 区——品牌主视觉背景 + 标语 + 副标题 + 搜索框占位 UI
- `homepage-trending-posts`: 热门帖子区——过去 7 天按 Upvote 数降序 Top 10，依赖社区帖子接口
- `homepage-popular-attractions`: 热门景点区——按浏览量降序 6–8 个城市/景点卡片，依赖景点列表接口
- `homepage-city-quick-access`: 城市快速入口——8 个 MVP 城市图标网格，点击跳转城市攻略占位页
- `homepage-ai-assistant-entry`: AI 助手入口——全局悬浮按钮，随时唤起 AI 对话
- `homepage-feature-nav`: 功能导航栏——旅游社区 / 景点攻略 / AI 助手 三大入口卡片

### Modified Capabilities

<!-- 无现有 spec 需修改 -->

## Impact

- **前端代码**：新增首页页面（`src/app/` 下路由）、7 个组件区块、对应数据获取逻辑
- **后端接口**：社区帖子列表接口需支持按 `upvote_count` 排序与 7 天范围筛选（依赖社区模块新增 `upvote_count` 字段）；景点列表接口需支持按 `view_count` 排序（依赖景点模块新增 `view_count` 字段）
- **跨模块前置依赖**：社区 `post` 表新增 `upvote_count` 字段（R-T2）；景点 `attraction` 表新增 `view_count` 字段（R-T3）
- **SEO**：首页为 SSR 渲染，需正确设置 meta 标签（title / description / og tags）
- **鉴权**：首页为公开页面，游客可访问（需在 Sa-Token 白名单内）
