# Homepage Trending Posts · 模块规格说明

> **文档版本**：v1.0
> **所属变更**：[build-homepage](../openspec/changes/build-homepage/proposal.md)
> **关联 delta spec**：[homepage-trending-posts/spec.md](../openspec/changes/build-homepage/specs/homepage-trending-posts/spec.md)
> **优先级**：P0（数据区块，依赖社区模块 API）

---

## 0. 技术决策记录

| # | 决策项 | 结论 | 说明 |
|---|--------|------|------|
| TD1 | **数据获取方式** | 客户端获取（CSR） | 使用 shadcn/ui Skeleton 做加载态；SSR 阶段不预取（降低后端首屏并发压力） |
| TD2 | **排序降级策略** | `upvote_count` → `create_time` | 当社区接口不支持 `upvote_count` 排序时（R-T2 未完成），降级为 `create_time` 降序，隐藏 upvote 指标 |
| TD3 | **列表渲染** | 垂直列表（非横向滚动） | 移动端单列、桌面端双列；每条帖子为可点击 Card |
| TD4 | **7 天范围计算** | 客户端计算时间戳 | 请求时计算 `Date.now() - 7 * 24 * 60 * 60 * 1000` 作为 `startDate` 参数 |

---

## 1. 模块边界

### 1.1 In Scope（包含的功能）

| # | 功能点 | 说明 |
|---|--------|------|
| 1 | **热门帖子列表** | 展示过去 7 天按 `upvote_count` 降序排列的 Top 10 帖子 |
| 2 | **帖子卡片信息** | 每张卡片展示：标题、作者名、城市标签、Upvote 数、评论数 |
| 3 | **点击跳转详情** | 点击卡片跳转至帖子详情页 `/community/posts/<id>` |
| 4 | **加载态 skeleton** | 数据加载期间显示 10 个骨架卡片占位 |
| 5 | **空态提示** | 无帖子时显示友好引导文案 |
| 6 | **降级排序** | `upvote_count` 字段不可用时降级为 `create_time` 排序 |

### 1.2 Out of Scope（不包含的功能）

- ❌ **分页 / 加载更多**：固定 Top 10，不做分页或无限滚动
- ❌ **排序切换**：用户不可手动切换排序方式（固定热度排序）
- ❌ **帖子内容预览**：卡片不展示帖子正文摘要（仅标题 + 元信息）
- ❌ **点赞 / 评论交互**：卡片仅展示数据，不可直接点赞或评论（需进详情页）
- ❌ **实时更新**：不做 WebSocket 推送，数据在页面加载时获取一次

---

## 2. 核心场景

### 2.1 正常路径

**SC-Posts-01：加载并展示热门帖子**
- **WHEN** 首页加载，TrendingPosts 区块挂载
- **THEN** 区块标题 "Trending Posts" 显示
- **AND** 发起 GET 请求至社区帖子接口，参数包含 `startDate=<7天前时间戳>` + `sort=upvote_count,desc` + `size=10`
- **AND** 加载期间显示 10 个 skeleton 卡片占位
- **AND** 数据返回后，skeleton 替换为实际帖子卡片，按 `upvote_count` 降序排列

**SC-Posts-02：帖子卡片信息展示**
- **WHEN** 一条帖子卡片渲染完成
- **THEN** 卡片显示帖子标题（最多 2 行截断）
- **AND** 显示作者名（如 "By John D."）
- **AND** 显示城市标签（如 "Beijing"，使用 Tailwind badge 样式）
- **AND** 显示 Upvote 数（如 "↑ 42"，带向上箭头图标）
- **AND** 显示评论数（如 "💬 8"，带消息图标）

**SC-Posts-03：点击跳转帖子详情**
- **WHEN** 用户点击某张帖子卡片
- **THEN** 浏览器导航至 `/community/posts/<post.id>`
- **AND** 卡片有 hover 效果（阴影增强 / 边框高亮）

### 2.2 异常路径

**SC-Posts-04：空态（无帖子）**
- **WHEN** 社区接口返回空列表（过去 7 天无帖子）
- **THEN** 区块显示空态提示 "Be the first to share your China travel story!"
- **AND** 空态区域居中显示，高度不低于 200px
- **AND** 不显示 skeleton 或错误

**SC-Posts-05：降级排序（upvote_count 不可用）**
- **WHEN** 社区接口不支持 `sort=upvote_count` 参数（R-T2 字段未完成）
- **THEN** 请求降级为 `sort=create_time,desc`
- **AND** 帖子卡片隐藏 Upvote 数指标（不显示 "↑ X"）
- **AND** 其余信息（标题、作者、城市、评论数）正常显示
- **AND** 开发环境控制台输出降级警告日志

**SC-Posts-06：API 请求失败**
- **WHEN** 社区接口返回 500 错误或网络超时
- **THEN** 区块显示错误兜底 UI（由 HomepageShell 的 Error Boundary 捕获）
- **AND** 显示 "Failed to load trending posts" + Retry 按钮
- **AND** 不影响其他区块渲染

---

## 3. 数据结构

### 3.1 组件树

```
TrendingPosts (Client Component)
├── <h2> "Trending Posts"
├── Loading State:
│   └── SkeletonCard × 10 (shadcn/ui Skeleton)
├── Data State:
│   └── PostCard × N (N ≤ 10)
│       ├── Title (2-line clamp)
│       ├── Author + CityTag + UpvoteCount + CommentCount
│       └── <Link href="/community/posts/{id}">
└── Empty State:
    └── EmptyMessage "Be the first to share your China travel story!"
```

### 3.2 组件 Props 定义

#### `TrendingPosts`（主组件 · Client Component）

```typescript
"use client";

interface TrendingPostsProps {
  /**
   * 展示帖子数量上限。
   * @default 10
   */
  limit?: number;

  /**
   * 时间范围（天数），筛选最近 N 天的帖子。
   * @default 7
   */
  daysRange?: number;

  /**
   * 区块标题文案。
   * @default "Trending Posts"
   */
  title?: string;
}
```

**约束**：
- MUST 标记 `"use client"`（需客户端 useState / useEffect 管理加载状态）
- `limit` 范围限定 1–50（后端 PageResult 最大 size=200，但首页推荐 ≤10）
- `daysRange` 范围限定 1–30

#### `PostCard`（帖子卡片 · Client Component）

```typescript
interface PostCardProps {
  /** 帖子数据 */
  post: TrendingPost;
  /** 是否支持 upvote 展示（降级模式下为 false） */
  showUpvote?: boolean;
}

interface TrendingPost {
  /** 帖子 ID（Snowflake，序列化为 string） */
  id: string;
  /** 帖子标题 */
  title: string;
  /** 作者昵称 */
  authorName: string;
  /** 城市标签（英文城市名） */
  cityName: string;
  /** Upvote 数（降级模式下可能不存在） */
  upvoteCount?: number;
  /** 评论数 */
  commentCount: number;
  /** 创建时间（ISO 8601） */
  createTime: string;
}
```

### 3.3 API 请求契约

```typescript
// GET /api/posts/trending
// Query Parameters:
//   startDate: number  // 7 天前的时间戳（ms）
//   sort: string       // "upvote_count,desc" 或 "create_time,desc"（降级）
//   size: number       // 10
//   current: number    // 1
//
// Response: R<PageResult<PostVO>>
// PageResult<T> = { records: T[], total: number, current: number, size: number }

interface TrendingPostsResponse {
  code: number;        // 0 = success
  msg: string;
  data: {
    records: TrendingPost[];
    total: number;
    current: number;
    size: number;
  };
}
```

### 3.4 降级逻辑常量

```typescript
// src/components/homepage/trending-posts/trending-constants.ts

export const TRENDING_DEFAULTS = {
  title: "Trending Posts",
  limit: 10,
  daysRange: 7,
  emptyMessage: "Be the first to share your China travel story!",
  errorMessage: "Failed to load trending posts",
} as const;

/** 降级排序字段（当 upvote_count 不可用时） */
export const FALLBACK_SORT = "create_time,desc";
/** 首选排序字段 */
export const PREFERRED_SORT = "upvote_count,desc";
```

### 3.5 文件结构

```
src/components/homepage/trending-posts/
├── TrendingPosts.tsx          # 主组件（Client Component）
├── PostCard.tsx               # 帖子卡片
├── trending-constants.ts      # 默认值与降级常量
├── use-trending-posts.ts      # 数据获取 hook
└── index.ts                   # 模块导出
```

---

## 4. 验收标准（Acceptance Checklist）

### 4.1 功能验收

- [ ] **AC-01** 区块标题 "Trending Posts" 可见
- [ ] **AC-02** 加载期间显示 10 个 skeleton 卡片占位
- [ ] **AC-03** 数据返回后显示帖子卡片，按 `upvote_count` 降序排列
- [ ] **AC-04** 每张卡片显示标题、作者名、城市标签、Upvote 数、评论数
- [ ] **AC-05** 帖子标题超过 2 行时正确截断（CSS line-clamp-2）
- [ ] **AC-06** 点击卡片跳转至 `/community/posts/<id>`
- [ ] **AC-07** 卡片有 hover 视觉反馈（阴影或边框高亮）

### 4.2 降级验收

- [ ] **AC-08** 接口不支持 `upvote_count` 排序时，降级为 `create_time` 排序
- [ ] **AC-09** 降级模式下卡片隐藏 Upvote 数（不显示 "↑ X"），其余信息正常
- [ ] **AC-10** 降级模式开发环境控制台输出警告日志

### 4.3 空态与错误验收

- [ ] **AC-11** 接口返回空列表时显示 "Be the first to share your China travel story!"
- [ ] **AC-12** 接口 500 / 超时时显示错误兜底 + Retry 按钮
- [ ] **AC-13** 点击 Retry 后重新发起请求

### 4.4 响应式验收

- [ ] **AC-14** 移动端（< 640px）：卡片单列排列
- [ ] **AC-15** 桌面端（≥ 1024px）：卡片双列排列（grid-cols-2）

### 4.5 技术约束验收

- [ ] **AC-16** TrendingPosts 为 Client Component（有 `"use client"`）
- [ ] **AC-17** 样式使用 Tailwind CSS（skeleton 用 shadcn/ui Skeleton 组件）
- [ ] **AC-18** `npm run build` 通过 TypeScript strict 检查
- [ ] **AC-19** API 请求通过 `clientRequest`（BFF 代理），不直连后端

---

## 5. 依赖关系

| 维度 | 说明 |
|------|------|
| **数据依赖** | 社区帖子接口 `GET /api/posts/trending`（需支持 `upvote_count` 排序 + 7 天范围） |
| **组件依赖** | 被 `HomepageShell` 包裹在 `SectionErrorBoundary` 内 |
| **外部依赖** | shadcn/ui `Skeleton`、`lucide-react`（ArrowBigUp / MessageCircle 图标）、Tailwind CSS |
| **前置任务** | ⚠️ R-T2：社区 `post` 表新增 `upvote_count` 字段（阻塞完整功能） |

---

## 6. 开放问题

1. **API 端点路径**：热门帖子用专用端点 `/api/posts/trending` 还是复用列表端点 `/api/posts` + 排序参数？
2. **作者头像**：卡片是否需要展示作者头像？当前 `TrendingPost` 未包含 `avatarUrl`，需确认社区接口是否返回。
3. **帖子缩略图**：卡片是否需要封面图？当前设计为纯文字卡片，如有图需接口返回 `coverImageUrl`。
