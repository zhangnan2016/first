## 1. 静态区块（无 API 依赖，可并行开发）

- [ ] 1.1 创建首页页面路由 `src/app/(dashboard)/page.tsx`，搭建 HomepageShell 容器组件骨架（按序组装各区块的占位结构）
- [ ] 1.2 实现 HeroSection 组件：全宽背景图 + 标语 "Unlock the Real China" + 副标题 + 搜索框占位 UI（提交时显示 "Coming soon" 提示，无 API 调用）
- [ ] 1.3 实现 CityQuickAccess 组件：定义 8 城市 MVP_CITIES 静态常量（Beijing/Shanghai/Chengdu/Xi'an/Guangzhou/Shenzhen/Hangzhou/Guilin），渲染图标网格，点击跳转 `/guides/<city>` 占位页
- [ ] 1.4 实现 FeatureNav 组件：渲染三大入口卡片（Travel Community / Attraction Guides / AI Assistant），每张卡片含图标+标题+一句话描述，点击跳转对应路由
- [ ] 1.5 实现各静态区块的响应式布局（移动端/平板/桌面端断点适配）

## 2. 数据依赖区块（需后端接口支持）

- [ ] 2.1 实现 TrendingPosts 组件：SSR 首次请求社区帖子接口（7 天范围 + upvote_count 降序 + limit=10），客户端负责加载态与重试
- [ ] 2.2 TrendingPosts 降级逻辑：当接口不支持 upvote_count 排序时，降级为 create_time 降序，隐藏 upvote 指标
- [ ] 2.3 TrendingPosts 空态兜底：接口返回空列表时显示友好提示文案
- [ ] 2.4 实现 PopularAttractions 组件：SSR 首次请求景点接口（view_count 降序 + limit=8），客户端负责加载态与重试
- [ ] 2.5 PopularAttractions 降级逻辑：当接口不支持 view_count 排序时，降级为 popularity 降序，隐藏浏览量数字
- [ ] 2.6 PopularAttractions 空态兜底：接口返回空列表时显示友好提示文案

## 3. AI 助手悬浮入口

- [ ] 3.1 实现 AiAssistantEntry 悬浮按钮组件（固定定位 + AI 图标 + hover tooltip + aria-label）
- [ ] 3.2 将 AiAssistantEntry 挂载到 `(dashboard)/layout.tsx` 全局 Layout（非 homepage-shell 内）
- [ ] 3.3 实现点击行为：已登录用户打开 AI 对话界面（或跳转 `/assistant`）；未登录用户跳转登录页

## 4. 骨架组装与错误隔离

- [ ] 4.1 在 HomepageShell 中按序组装：Hero → TrendingPosts → PopularAttractions → CityQuickAccess → FeatureNav
- [ ] 4.2 为 TrendingPosts 和 PopularAttractions 包裹独立 Error Boundary，实现单区块错误不影响整页
- [ ] 4.3 实现各区块的加载态 skeleton 占位

## 5. SEO 与公开访问

- [ ] 5.1 在首页页面设置 SEO metadata（title、meta description、Open Graph tags），使用 Next.js `generateMetadata` 或 `export const metadata`
- [ ] 5.2 确认首页路由与相关只读 API 在 Sa-Token 白名单内（游客可访问，无需登录）

## 6. 联调与验证

- [ ] 6.1 端到端验证：游客（未登录）打开首页，所有区块正常渲染，无登录墙
- [ ] 6.2 验证 SEO：查看页面源码确认 title/description/og 标签正确输出（SSR）
- [ ] 6.3 验证错误隔离：模拟某个数据接口超时/报错，确认仅该区块显示错误兜底，其余区块正常
- [ ] 6.4 验证响应式：在移动端/平板/桌面端检查所有区块布局正确
- [ ] 6.5 验证 AI 悬浮按钮：在首页任意滚动位置可见，点击行为符合预期

## 7. 跨模块前置任务（非首页代码，但阻塞 #2、#5 实现）

- [ ] 7.1 [R-T2] 社区 `post` 表新增 `upvote_count` 字段（INT, NOT NULL DEFAULT 0）+ 帖子列表接口支持 `sort=upvote_count` 参数
- [ ] 7.2 [R-T3] 景点 `attraction` 表新增 `view_count` 字段（INT, NOT NULL DEFAULT 0）+ 景点列表接口支持 `sort=view_count` 参数
