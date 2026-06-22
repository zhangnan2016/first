# Homepage Popular Attractions · 模块规格说明

> **文档版本**：v1.0
> **所属变更**：[build-homepage](../changes/build-homepage/proposal.md)
> **关联 delta spec**：[homepage-popular-attractions/spec.md](../changes/build-homepage/specs/homepage-popular-attractions/spec.md)
> **优先级**：P0（数据区块，依赖景点模块 API）

---

## 0. 技术决策记录

| # | 决策项 | 结论 | 说明 |
|---|--------|------|------|
| TD1 | **数据获取方式** | 客户端获取（CSR） | 使用 shadcn/ui Skeleton 做加载态；与 TrendingPosts 一致 |
| TD2 | **排序降级策略** | `view_count` → `popularity` | 当景点接口不支持 `view_count` 排序时（R-T3 未完成），降级为 `popularity` 降序，隐藏浏览量数字 |
| TD3 | **卡片布局** | 网格卡片（图片 + 文字） | 移动端 2 列、桌面端 4 列；每张卡片含封面图、名称、城市名、简介 |
| TD4 | **封面图优化** | `next/image`（width + height 模式） | 使用固定宽高比（16:9），利用 Next.js 图片优化自动生成 srcset |

---

## 1. 模块边界

### 1.1 In Scope（包含的功能）

| # | 功能点 | 说明 |
|---|--------|------|
| 1 | **热门景点卡片网格** | 按浏览量（`view_count`）降序展示 6–8 个景点卡片 |
| 2 | **卡片信息** | 每张卡片展示：封面图、英文名、所属城市、简短描述 |
| 3 | **点击跳转** | 点击卡片跳转至景点详情页 `/guides/attractions/<id>`（本期跳占位页） |
| 4 | **加载态 skeleton** | 数据加载期间显示 8 个骨架卡片 |
| 5 | **空态提示** | 无景点数据时显示友好引导 |
| 6 | **降级排序** | `view_count` 不可用时降级为 `popularity` 排序 |

### 1.2 Out of Scope（不包含的功能）

- ❌ **景点评分 / 评论**：卡片不展示评分星级或评论数
- ❌ **城市筛选**：本期所有用户看到相同景点列表，不支持按城市过滤
- ❌ **地图视图**：不做地图模式展示
- ❌ **分页 / 加载更多**：固定 6–8 张卡片
- ❌ **实时浏览量**：卡片不显示实时浏览数（降级模式下隐藏），仅用于排序

---

## 2. 核心场景

### 2.1 正常路径

**SC-Attractions-01：加载并展示热门景点**
- **WHEN** 首页加载，PopularAttractions 区块挂载
- **THEN** 区块标题 "Popular Destinations" 显示
- **AND** 发起 GET 请求至景点接口，参数包含 `sort=view_count,desc` + `size=8`
- **AND** 加载期间显示 8 个 skeleton 卡片（含图片占位 + 文字占位）
- **AND** 数据返回后，skeleton 替换为实际景点卡片，按 `view_count` 降序排列

**SC-Attractions-02：景点卡片信息展示**
- **WHEN** 一张景点卡片渲染完成
- **THEN** 卡片顶部显示封面图（16:9 宽高比，`next/image` 优化）
- **AND** 图片下方显示景点英文名（加粗，1 行截断）
- **AND** 显示所属城市名（次要文字色，如 "Beijing"）
- **AND** 显示简短描述（2 行截断，次要文字色）

**SC-Attractions-03：点击跳转景点详情**
- **WHEN** 用户点击某张景点卡片
- **THEN** 浏览器导航至 `/guides/attractions/<attraction.id>`
- **AND** 本期景点详情为占位页（Coming Soon），返回 HTTP 200
- **AND** 卡片有 hover 效果（图片缩放 / 阴影增强）

### 2.2 异常路径

**SC-Attractions-04：空态（无景点数据）**
- **WHEN** 景点接口返回空列表（种子数据未导入）
- **THEN** 区块显示空态提示 "Attractions are coming soon!"
- **AND** 空态区域居中显示，高度不低于 200px

**SC-Attractions-05：降级排序（view_count 不可用）**
- **WHEN** 景点接口不支持 `sort=view_count` 参数（R-T3 字段未完成）
- **THEN** 请求降级为 `sort=popularity,desc`
- **AND** 卡片信息正常显示（图片、名称、城市、描述）
- **AND** 开发环境控制台输出降级警告日志

**SC-Attractions-06：封面图加载失败**
- **WHEN** 某景点卡片的封面图 URL 无效或加载超时
- **THEN** 该卡片图片区域显示降级占位图（`/placeholder-attraction.svg`）
- **AND** 其余卡片不受影响
- **AND** 卡片文字信息正常显示

**SC-Attractions-07：API 请求失败**
- **WHEN** 景点接口返回 500 错误或网络超时
- **THEN** 区块显示错误兜底 UI（由 HomepageShell 的 Error Boundary 捕获）
- **AND** 显示 "Failed to load popular attractions" + Retry 按钮

---

## 3. 数据结构

### 3.1 组件树

```
PopularAttractions (Client Component)
├── <h2> "Popular Destinations"
├── Loading State:
│   └── SkeletonCard × 8 (shadcn/ui Skeleton, 含图片占位)
├── Data State:
│   └── AttractionCard × N (N ≤ 8)
│       ├── CoverImage (next/image, 16:9)
│       ├── Name (font-bold, truncate)
│       ├── CityName (text-muted-foreground)
│       └── Description (line-clamp-2)
│       └── <Link href="/guides/attractions/{id}">
└── Empty State:
    └── EmptyMessage "Attractions are coming soon!"
```

### 3.2 组件 Props 定义

#### `PopularAttractions`（主组件 · Client Component）

```typescript
"use client";

interface PopularAttractionsProps {
  /** 展示景点数量上限 */
  limit?: number;

  /** 区块标题文案 */
  title?: string;
}
```

**约束**：
- MUST 标记 `"use client"`
- `limit` 范围限定 4–12（默认 8）

#### `AttractionCard`（景点卡片 · Client Component）

```typescript
interface AttractionCardProps {
  /** 景点数据 */
  attraction: PopularAttraction;
}

interface PopularAttraction {
  /** 景点 ID（Snowflake，序列化为 string） */
  id: string;
  /** 英文名 */
  nameEn: string;
  /** 中文名（可选，hover tooltip 展示） */
  nameCn?: string;
  /** 所属城市英文名 */
  cityName: string;
  /** 封面图 URL */
  coverImageUrl: string;
  /** 英文描述（RAG 语料摘要） */
  description: string;
  /** 浏览量（降级模式下可能不存在） */
  viewCount?: number;
  /** 预设热度（降级排序使用） */
  popularity?: number;
}
```

### 3.3 API 请求契约

```typescript
// GET /api/attractions/popular
// Query Parameters:
//   sort: string    // "view_count,desc" 或 "popularity,desc"（降级）
//   size: number    // 8
//   current: number // 1
//
// Response: R<PageResult<AttractionVO>>

interface PopularAttractionsResponse {
  code: number;
  msg: string;
  data: {
    records: PopularAttraction[];
    total: number;
    current: number;
    size: number;
  };
}
```

### 3.4 降级与常量

```typescript
// src/components/homepage/popular-attractions/attractions-constants.ts

export const ATTRACTIONS_DEFAULTS = {
  title: "Popular Destinations",
  limit: 8,
  emptyMessage: "Attractions are coming soon!",
  errorMessage: "Failed to load popular attractions",
  placeholderImage: "/placeholder-attraction.svg",
} as const;

export const FALLBACK_SORT = "popularity,desc";
export const PREFERRED_SORT = "view_count,desc";
```

### 3.5 文件结构

```
src/components/homepage/popular-attractions/
├── PopularAttractions.tsx       # 主组件（Client Component）
├── AttractionCard.tsx           # 景点卡片
├── attractions-constants.ts     # 默认值与降级常量
├── use-popular-attractions.ts   # 数据获取 hook
└── index.ts                     # 模块导出
```

---

## 4. 验收标准（Acceptance Checklist）

### 4.1 功能验收

- [ ] **AC-01** 区块标题 "Popular Destinations" 可见
- [ ] **AC-02** 加载期间显示 8 个 skeleton 卡片（含图片占位区 + 文字占位行）
- [ ] **AC-03** 数据返回后显示景点卡片，按 `view_count` 降序排列
- [ ] **AC-04** 每张卡片显示封面图（16:9）、英文名、城市名、描述（2 行截断）
- [ ] **AC-05** 封面图使用 `next/image` 渲染（DevTools 确认 `<img>` 带 srcset）
- [ ] **AC-06** 点击卡片跳转至 `/guides/attractions/<id>`
- [ ] **AC-07** 卡片 hover 效果（图片缩放或阴影增强）

### 4.2 降级验收

- [ ] **AC-08** 接口不支持 `view_count` 排序时，降级为 `popularity` 排序
- [ ] **AC-09** 降级模式下卡片信息正常显示，开发环境控制台输出警告

### 4.3 空态与错误验收

- [ ] **AC-10** 接口返回空列表时显示 "Attractions are coming soon!"
- [ ] **AC-11** 封面图加载失败时显示降级占位图，卡片文字正常
- [ ] **AC-12** 接口 500 / 超时时显示错误兜底 + Retry 按钮

### 4.4 响应式验收

- [ ] **AC-13** 移动端（< 640px）：卡片 2 列（grid-cols-2）
- [ ] **AC-14** 平板（640px–1024px）：卡片 3 列（grid-cols-3）
- [ ] **AC-15** 桌面端（≥ 1024px）：卡片 4 列（grid-cols-4）

### 4.5 技术约束验收

- [ ] **AC-16** PopularAttractions 为 Client Component（有 `"use client"`）
- [ ] **AC-17** 样式使用 Tailwind CSS（无 inline style）
- [ ] **AC-18** `npm run build` 通过 TypeScript strict 检查
- [ ] **AC-19** API 请求通过 `clientRequest`（BFF 代理）

---

## 5. 依赖关系

| 维度 | 说明 |
|------|------|
| **数据依赖** | 景点列表接口 `GET /api/attractions/popular`（需支持 `view_count` 排序） |
| **组件依赖** | 被 `HomepageShell` 包裹在 `SectionErrorBoundary` 内 |
| **外部依赖** | `next/image`、shadcn/ui `Skeleton`、Tailwind CSS |
| **前置任务** | ⚠️ R-T3：景点 `attraction` 表新增 `view_count` 字段 + 种子数据导入 |

---

## 6. 开放问题

1. **景点 API 端点**：用专用端点 `/api/attractions/popular` 还是复用列表端点 `/api/attractions` + 排序参数？
2. **卡片描述来源**：`description` 字段为完整英文描述（可能很长），卡片需要截取前 N 字符还是接口返回专用 `summary` 字段？
3. **占位图素材**：`/placeholder-attraction.svg` 需要设计，内容建议为灰色山脉/建筑轮廓。
