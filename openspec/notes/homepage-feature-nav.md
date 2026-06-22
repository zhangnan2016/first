# Homepage Feature Nav · 模块规格说明

> **文档版本**：v1.0
> **所属变更**：[build-homepage](../changes/build-homepage/proposal.md)
> **关联 delta spec**：[homepage-feature-nav/spec.md](../changes/build-homepage/specs/homepage-feature-nav/spec.md)
> **优先级**：P0（静态区块，零 API 依赖，可独立先行交付）

---

## 0. 技术决策记录

| # | 决策项 | 结论 | 说明 |
|---|--------|------|------|
| TD1 | **数据来源** | 前端静态常量 | 3 个导航卡片内容硬编码为 TypeScript 常量，不依赖 API |
| TD2 | **卡片样式** | shadcn/ui 风格卡片 | 图标区 + 标题 + 描述，hover 有 elevation 效果 |
| TD3 | **组件渲染模式** | Server Component | 纯静态内容（链接 + 文案），无客户端交互逻辑 |
| TD4 | **AI 助手卡片行为** | 与悬浮按钮一致 | 点击 AI 助手卡片的行为与 AiAssistantEntry 悬浮按钮一致（登录态判断） |

---

## 1. 模块边界

### 1.1 In Scope（包含的功能）

| # | 功能点 | 说明 |
|---|--------|------|
| 1 | **三大入口卡片** | 展示旅游社区、景点攻略、AI 助手三张卡片 |
| 2 | **卡片内容** | 每张卡片：图标 + 标题 + 一句话描述 |
| 3 | **点击跳转** | 社区 → `/community`；攻略 → `/guides`（占位）；AI → `/assistant`（占位）或打开 AI 对话 |
| 4 | **hover 交互** | 鼠标悬停时卡片有视觉反馈（阴影增强 / 上移） |
| 5 | **响应式布局** | 桌面 3 列横排，移动端竖排堆叠 |

### 1.2 Out of Scope（不包含的功能）

- ❌ **动态导航项**：不从配置或 API 获取导航项，固定 3 个
- ❌ **卡片内嵌功能预览**：卡片不含社区最新帖子预览、景点缩略图等
- ❌ **卡片排序自定义**：固定顺序（社区 → 攻略 → AI）
- ❌ **卡片 Badge / 角标**：不展示 "New" / "Beta" 等标签
- ❌ **暗黑模式图标适配**：本期使用固定配色图标

---

## 2. 核心场景

### 2.1 正常路径

**SC-Nav-01：功能导航渲染**
- **WHEN** 首页加载，FeatureNav 区块渲染
- **THEN** 区块标题 "Start Your Journey" 显示
- **AND** 三张卡片水平排列（桌面端）：Travel Community → Attraction Guides → AI Assistant
- **AND** 每张卡片显示图标（lucide-react）+ 标题 + 一句话描述
- **AND** 无网络请求（纯静态渲染）

**SC-Nav-02：卡片内容**
- **WHEN** FeatureNav 区块渲染完成
- **THEN** Travel Community 卡片显示：`Users` 图标 + "Travel Community" + "Share experiences and get real advice from fellow travelers"
- **AND** Attraction Guides 卡片显示：`MapPin` 图标 + "Attraction Guides" + "Explore curated guides for China's top destinations"
- **AND** AI Assistant 卡片显示：`Sparkles` 图标 + "AI Assistant" + "Get instant answers about traveling in China, anytime"

**SC-Nav-03：点击跳转 - 社区**
- **WHEN** 用户点击 Travel Community 卡片
- **THEN** 浏览器导航至 `/community`
- **AND** 社区模块本期为 P0 功能，正常进入

**SC-Nav-04：点击跳转 - 攻略**
- **WHEN** 用户点击 Attraction Guides 卡片
- **THEN** 浏览器导航至 `/guides`
- **AND** 目标页面为 Coming Soon 占位页（P1 模块），返回 HTTP 200

**SC-Nav-05：点击跳转 - AI 助手**
- **WHEN** 已登录用户点击 AI Assistant 卡片
- **THEN** 浏览器跳转至 `/assistant`（行为与悬浮按钮一致）
- **AND** 本期 `/assistant` 为 Coming Soon 占位页

### 2.2 异常路径

**SC-Nav-06：未登录用户点击 AI 卡片**
- **WHEN** 未登录用户点击 AI Assistant 卡片
- **THEN** 浏览器跳转至 `/login?redirect=<当前路径>`
- **AND** 行为与悬浮按钮一致（AI 为需登录功能）

**SC-Nav-07：移动端布局**
- **WHEN** 首页在移动端查看
- **THEN** 三张卡片竖向堆叠（单列，grid-cols-1）
- **AND** 卡片宽度撑满容器，高度自适应

---

## 3. 数据结构

### 3.1 组件树

```
FeatureNav (Server Component)
├── <h2> "Start Your Journey"
└── <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
    └── FeatureCard × 3
        ├── Icon (lucide-react)
        ├── <h3> Title
        ├── <p> Description
        └── <Link href="{route}">
```

### 3.2 组件 Props 定义

#### `FeatureNav`（主组件 · Server Component）

```typescript
interface FeatureNavProps {
  /**
   * 区块标题文案。
   * @default "Start Your Journey"
   */
  title?: string;

  /**
   * 自定义导航项列表。
   * 不传时使用 FEATURE_NAV_ITEMS 默认 3 项。
   */
  items?: FeatureNavItem[];
}
```

#### `FeatureCard`（导航卡片 · Server Component）

```typescript
interface FeatureCardProps {
  /** 导航项数据 */
  item: FeatureNavItem;
}

interface FeatureNavItem {
  /** 唯一标识 */
  id: string;
  /** 卡片标题 */
  title: string;
  /** 一句话描述 */
  description: string;
  /** lucide-react 图标组件 */
  icon: LucideIcon;
  /** Tailwind 图标背景色类（如 "bg-blue-50"） */
  iconBgClass: string;
  /** Tailwind 图标颜色类（如 "text-blue-600"） */
  iconColorClass: string;
  /** 跳转路由 */
  route: string;
  /**
   * 是否需要登录才能访问。
   * 需要登录的卡片由 Client Component 子组件处理认证跳转。
   */
  requiresAuth?: boolean;
}
```

### 3.3 静态导航项常量

```typescript
// src/components/homepage/feature-nav/nav-constants.ts

import { Users, MapPin, Sparkles } from "lucide-react";

export const FEATURE_NAV_ITEMS: FeatureNavItem[] = [
  {
    id: "community",
    title: "Travel Community",
    description: "Share experiences and get real advice from fellow travelers",
    icon: Users,
    iconBgClass: "bg-blue-50",
    iconColorClass: "text-blue-600",
    route: "/community",
    requiresAuth: false,
  },
  {
    id: "guides",
    title: "Attraction Guides",
    description: "Explore curated guides for China's top destinations",
    icon: MapPin,
    iconBgClass: "bg-green-50",
    iconColorClass: "text-green-600",
    route: "/guides",
    requiresAuth: false,
  },
  {
    id: "assistant",
    title: "AI Assistant",
    description: "Get instant answers about traveling in China, anytime",
    icon: Sparkles,
    iconBgClass: "bg-purple-50",
    iconColorClass: "text-purple-600",
    route: "/assistant",
    requiresAuth: true,
  },
];

export const NAV_DEFAULTS = {
  title: "Start Your Journey",
} as const;
```

### 3.4 AI 卡片的认证处理

由于 FeatureNav 主体为 Server Component，但 AI 卡片需客户端认证判断，采用以下方案：

```typescript
// FeatureCard 在 requiresAuth=true 时渲染一个 Client Component 包装器
// FeatureCardAuthWrapper (Client Component) 负责：
// 1. 读取 useUserStore 判断登录态
// 2. 已登录 → router.push("/assistant")
// 3. 未登录 → router.push("/login?redirect=...")
```

### 3.5 文件结构

```
src/components/homepage/feature-nav/
├── FeatureNav.tsx                # 主组件（Server Component）
├── FeatureCard.tsx               # 导航卡片（Server Component）
├── FeatureCardAuthWrapper.tsx    # 认证包装器（Client Component，仅 requiresAuth 卡片用）
├── nav-constants.ts              # FEATURE_NAV_ITEMS 静态常量
└── index.ts                      # 模块导出
```

---

## 4. 验收标准（Acceptance Checklist）

### 4.1 功能验收

- [ ] **AC-01** 区块标题 "Start Your Journey" 可见
- [ ] **AC-02** 三张卡片按顺序展示：Travel Community → Attraction Guides → AI Assistant
- [ ] **AC-03** 每张卡片显示图标 + 标题 + 一句话描述（内容与 §2.1 SC-Nav-02 一致）
- [ ] **AC-04** 点击 Travel Community 卡片跳转 `/community`
- [ ] **AC-05** 点击 Attraction Guides 卡片跳转 `/guides`（占位页，HTTP 200）
- [ ] **AC-06** 已登录用户点击 AI Assistant 卡片跳转 `/assistant`
- [ ] **AC-07** 页面加载时 Network 面板无导航相关 API 请求（纯静态）

### 4.2 认证验收

- [ ] **AC-08** 未登录用户点击 AI Assistant 卡片，跳转 `/login?redirect=<当前路径>`
- [ ] **AC-09** 登录成功后自动跳回原页面

### 4.3 交互验收

- [ ] **AC-10** 鼠标悬停卡片时有视觉反馈（阴影增强 `hover:shadow-lg` / 上移 `hover:-translate-y-1`）
- [ ] **AC-11** 卡片支持键盘导航（Tab 可聚焦，Enter 可跳转）
- [ ] **AC-12** 卡片有 `focus-visible:ring` 聚焦样式

### 4.4 响应式验收

- [ ] **AC-13** 移动端（< 768px）：三张卡片竖向堆叠（grid-cols-1）
- [ ] **AC-14** 桌面端（≥ 768px）：三张卡片水平排列（md:grid-cols-3）
- [ ] **AC-15** 卡片间距在各断点下自然（gap-4 或 gap-6）

### 4.5 无障碍验收

- [ ] **AC-16** 每张卡片链接有描述性 `aria-label`（如 `aria-label="Go to Travel Community"`）
- [ ] **AC-17** 图标有 `aria-hidden="true"`（装饰性图标）
- [ ] **AC-18** 卡片标题使用 `<h3>` 语义化标签

### 4.6 技术约束验收

- [ ] **AC-19** FeatureNav 主体为 Server Component（无 `"use client"`）
- [ ] **AC-20** FeatureCardAuthWrapper 为 Client Component（有 `"use client"`）
- [ ] **AC-21** 样式使用 Tailwind CSS（无 inline style）
- [ ] **AC-22** 图标使用 `lucide-react`（非 `@ant-design/icons`）
- [ ] **AC-23** `npm run build` 通过 TypeScript strict 检查

---

## 5. 依赖关系

| 维度 | 说明 |
|------|------|
| **数据依赖** | 无（3 个导航项为前端静态常量） |
| **组件依赖** | 被 `HomepageShell` 组装挂载；AI 卡片行为与 `AiAssistantEntry` 悬浮按钮一致 |
| **外部依赖** | `lucide-react`（Users / MapPin / Sparkles）、Tailwind CSS、Next.js `Link` / `useRouter`、Zustand `useUserStore` |
| **前置任务** | ⚠️ Tailwind CSS + shadcn/ui 迁移；`/community`、`/guides`、`/assistant` 路由须存在（含占位页） |

---

## 6. 开放问题

1. **社区模块状态**：本期社区为 P0，`/community` 应正常进入而非占位。需确认社区页面开发计划是否与首页同步。
2. **卡片图标色系**：当前社区蓝、攻略绿、AI 紫，是否需要与品牌色规范对齐？
3. **卡片描述文案**：当前为英文文案，是否需要考虑未来 i18n（多语言切换）的预留？
