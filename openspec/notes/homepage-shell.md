# Homepage Shell · 模块规格说明

> **文档版本**：v1.0
> **所属变更**：[build-homepage](../changes/build-homepage/proposal.md)
> **关联 delta spec**：[homepage-shell/spec.md](../changes/build-homepage/specs/homepage-shell/spec.md)
> **优先级**：P0（页面骨架，所有区块的组装容器）

---

## 0. 技术决策记录

| # | 决策项 | 结论 | 说明 |
|---|--------|------|------|
| TD1 | **页面渲染模式** | Server Component（默认） | Shell 本身为 Server Component，在 SSR 阶段组装各子区块；数据区块（TrendingPosts / PopularAttractions）内部自行处理 CSR 加载 |
| TD2 | **错误隔离策略** | 每个数据区块包裹独立 React Error Boundary | 单区块 API 失败不影响整页，其余区块正常渲染 |
| TD3 | **区块加载策略** | 并行非阻塞 | 静态区块（Hero / CityQuickAccess / FeatureNav）即时渲染；数据区块各自异步加载，带 skeleton 占位 |
| TD4 | **SEO 注入方式** | Next.js `generateMetadata` | 在页面级别导出 `generateMetadata` 函数，动态生成 title / description / Open Graph |

---

## 1. 模块边界

### 1.1 In Scope（包含的功能）

| # | 功能点 | 说明 |
|---|--------|------|
| 1 | **区块顺序组装** | 按固定顺序渲染 5 个内容区块：Hero → TrendingPosts → PopularAttractions → CityQuickAccess → FeatureNav |
| 2 | **加载态管理** | 数据区块加载时显示 skeleton 占位，不阻塞其余区块渲染 |
| 3 | **空态兜底** | 数据区块返回空结果时显示友好提示，不显示空白区域 |
| 4 | **错误隔离** | 单个数据区块出错时仅显示该区块的错误兜底 + 重试按钮，其余区块不受影响 |
| 5 | **SEO metadata** | 页面级 title、meta description、Open Graph 标签注入 |
| 6 | **公开访问** | 首页游客可访问（Sa-Token 白名单），无需登录 |

### 1.2 Out of Scope（不包含的功能）

- ❌ **数据聚合 API**：Shell 不新建首页聚合接口，各数据区块直连对应模块接口
- ❌ **区块顺序自定义**：区块顺序固定，不支持用户拖拽或个性化排列
- ❌ **A/B 测试框架**：所有用户看到相同的区块组成与顺序
- ❌ **区块懒加载（Intersection Observer）**：本期所有区块随页面一起渲染（含 skeleton），不做滚动触发的延迟加载
- ❌ **SSR 数据预取**：Shell 不在 Server Component 阶段预取数据区块的数据，各区块自行管理数据获取

---

## 2. 核心场景

### 2.1 正常路径

**SC-Shell-01：页面组装渲染**
- **WHEN** 访客（含未登录游客）打开首页 `/`
- **THEN** 页面按固定顺序渲染 5 个区块：Hero → TrendingPosts → PopularAttractions → CityQuickAccess → FeatureNav
- **AND** 静态区块（Hero / CityQuickAccess / FeatureNav）即时渲染，无加载延迟
- **AND** 数据区块（TrendingPosts / PopularAttractions）先显示 skeleton，数据到达后替换

**SC-Shell-02：SEO metadata 输出**
- **WHEN** 首页 HTML 被搜索引擎或社交平台抓取
- **THEN** `<head>` 中包含 `<title>WanderChina — Unlock the Real China</title>`
- **AND** 包含 `<meta name="description" content="...">` 描述平台核心价值
- **AND** 包含 `og:title`、`og:description`、`og:image` Open Graph 标签

**SC-Shell-03：游客公开访问**
- **WHEN** 未登录访客导航至首页
- **THEN** 页面正常加载，无登录墙、无重定向
- **AND** 首页 API 端点在 Sa-Token 白名单内（后端不返回 401）

### 2.2 异常路径

**SC-Shell-04：单个数据区块 API 失败**
- **WHEN** TrendingPosts 区块请求社区接口超时或返回错误
- **THEN** 仅 TrendingPosts 区块显示错误兜底 UI（错误图标 + "Failed to load" + Retry 按钮）
- **AND** PopularAttractions、CityQuickAccess、FeatureNav、Hero 全部不受影响，正常渲染
- **AND** 浏览器控制台记录错误日志

**SC-Shell-05：数据区块返回空列表**
- **WHEN** TrendingPosts 请求成功但返回空数组（过去 7 天无帖子）
- **THEN** TrendingPosts 区块显示空态提示 "Be the first to share your China travel story!"
- **AND** 区块高度不坍塌，保持合理间距
- **AND** 其余区块正常渲染

**SC-Shell-06：后端服务完全不可用**
- **WHEN** 所有后端 API 均不可达
- **THEN** 两个数据区块分别显示各自的错误兜底 + Retry 按钮
- **AND** 三个静态区块（Hero / CityQuickAccess / FeatureNav）仍正常渲染
- **AND** 页面不白屏、不崩溃

---

## 3. 数据结构

### 3.1 组件树

```
HomePage (Server Component, src/app/(dashboard)/page.tsx)
└── HomepageShell (Server Component)
    ├── <HeroSection />                           # 静态，即时渲染
    ├── <SectionErrorBoundary>                    # Error Boundary 包裹
    │   └── <TrendingPosts />                     # 数据区块，含 skeleton
    ├── <SectionErrorBoundary>                    # Error Boundary 包裹
    │   └── <PopularAttractions />                # 数据区块，含 skeleton
    ├── <CityQuickAccess />                       # 静态，即时渲染
    └── <FeatureNav />                            # 静态，即时渲染
```

### 3.2 组件 Props 定义

#### `HomepageShell`（主容器 · Server Component）

```typescript
interface HomepageShellProps {
  /**
   * 子区块节点。由 page.tsx 组装后传入。
   * Shell 仅负责布局与间距，不关心子区块内部实现。
   */
  children: React.ReactNode;
}
```

**约束**：
- Shell 不接收任何数据 props（数据由各子区块自行获取）
- Shell 不标记 `"use client"`（Server Component）

#### `SectionErrorBoundary`（错误边界 · Client Component）

```typescript
"use client";

interface SectionErrorBoundaryProps {
  children: React.ReactNode;

  /**
   * 自定义错误兜底 UI。
   * @default <DefaultErrorFallback />
   */
  fallback?: React.ReactNode;

  /**
   * 错误发生时的回调（用于日志上报等）。
   */
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
}
```

**约束**：
- MUST 标记 `"use client"`（React Error Boundary 仅在 Client Component 中可用）
- 捕获子组件渲染错误、生命周期错误，但不捕获事件处理函数中的错误
- `fallback` 默认渲染错误图标 + 文案 + Retry 按钮

#### `DefaultErrorFallback`（默认错误兜底 · Client Component）

```typescript
"use client";

interface DefaultErrorFallbackProps {
  /** 错误提示文案 */
  message?: string;
  /** 重试回调，由 Error Boundary 的 reset 机制提供 */
  onRetry?: () => void;
}
```

### 3.3 页面级 SEO Metadata

```typescript
// src/app/(dashboard)/page.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "WanderChina — Unlock the Real China",
  description:
    "Curated English guides, instant AI answers, and a traveler community — everything you need to explore China with confidence.",
  openGraph: {
    title: "WanderChina — Unlock the Real China",
    description:
      "Curated English guides, instant AI answers, and a traveler community.",
    type: "website",
    images: ["/og/homepage-og.png"],
  },
};
```

### 3.4 文件结构

```
src/components/homepage/
├── HomepageShell.tsx              # 主容器（Server Component）
├── SectionErrorBoundary.tsx       # 错误边界（Client Component）
├── DefaultErrorFallback.tsx       # 默认错误兜底（Client Component）
└── index.ts                       # 模块导出

src/app/(dashboard)/page.tsx       # 首页页面入口（组装 Shell + 子区块 + metadata）
```

---

## 4. 验收标准（Acceptance Checklist）

### 4.1 功能验收

- [ ] **AC-01** 游客打开首页，页面按固定顺序渲染 5 个区块（Hero → TrendingPosts → PopularAttractions → CityQuickAccess → FeatureNav）
- [ ] **AC-02** 静态区块（Hero / CityQuickAccess / FeatureNav）在页面加载时立即可见，无 skeleton 等待
- [ ] **AC-03** 数据区块（TrendingPosts / PopularAttractions）加载期间显示 skeleton 占位符

### 4.2 错误隔离验收

- [ ] **AC-04** 将 TrendingPosts 的 API 模拟为 500 错误，仅该区块显示错误兜底 + Retry 按钮，其余 4 个区块正常
- [ ] **AC-05** 将 PopularAttractions 的 API 模拟为超时，仅该区块显示错误兜底，TrendingPosts 正常显示数据
- [ ] **AC-06** 点击 Retry 按钮后，该区块重新发起请求（skeleton → 数据/错误）
- [ ] **AC-07** 模拟后端完全不可用，两个数据区块均显示错误兜底，三个静态区块正常渲染，页面不白屏

### 4.3 空态验收

- [ ] **AC-08** TrendingPosts 返回空数组时显示 "Be the first to share your China travel story!" 提示
- [ ] **AC-09** PopularAttractions 返回空数组时显示 "Attractions are coming soon!" 提示
- [ ] **AC-10** 空态区块保持合理高度，不出现布局坍塌

### 4.4 SEO 验收

- [ ] **AC-11** 页面源码（view-source）中 `<title>` 为 "WanderChina — Unlock the Real China"
- [ ] **AC-12** 页面源码中包含 `<meta name="description">` 标签
- [ ] **AC-13** 页面源码中包含 `og:title`、`og:description`、`og:image` 标签

### 4.5 公开访问验收

- [ ] **AC-14** 未登录用户（清除 cookie）打开首页，页面正常加载，无 401 跳转
- [ ] **AC-15** 首页 API 请求不携带 `Authorization` 头（或后端在白名单内不校验）

### 4.6 技术约束验收

- [ ] **AC-16** HomepageShell 为 Server Component（无 `"use client"`）
- [ ] **AC-17** SectionErrorBoundary 为 Client Component（有 `"use client"`）
- [ ] **AC-18** 样式使用 Tailwind CSS 类名（无 inline style）
- [ ] **AC-19** 区块间距使用 Tailwind 间距类（如 `space-y-16` 或 `gap-16`）
- [ ] **AC-20** `npm run build` 通过 TypeScript strict 检查，无错误

---

## 5. 依赖关系

| 维度 | 说明 |
|------|------|
| **数据依赖** | 无（Shell 不直接获取数据） |
| **组件依赖** | 组装 Hero / TrendingPosts / PopularAttractions / CityQuickAccess / FeatureNav 五个子区块 |
| **被依赖** | 无（首页是终端展示层） |
| **外部依赖** | React Error Boundary（内置）、Tailwind CSS、Next.js `Metadata` |
| **前置任务** | ⚠️ Tailwind CSS + shadcn/ui 技术栈迁移（`migrate-ui-to-shadcn`）须先完成 |

---

## 6. 开放问题

1. **OG 图片素材**：`/og/homepage-og.png` 需要设计制作，尺寸建议 1200×630px，内容为品牌主视觉。
2. **区块间距规范**：是否统一使用 `space-y-16`（64px）作为区块间距，还是各区块自定义上下 margin？
3. **Error Boundary 复用**：SectionErrorBoundary 是否提取为通用组件供其他页面复用？
