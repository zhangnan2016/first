# Homepage Hero · 模块规格说明

> **文档版本**：v1.0
> **所属变更**：[build-homepage](../openspec/changes/build-homepage/proposal.md)
> **关联 delta spec**：[homepage-hero/spec.md](../openspec/changes/build-homepage/specs/homepage-hero/spec.md)
> **优先级**：P0（静态区块，无 API 依赖，可独立先行交付）

---

## 0. 技术决策记录

| # | 决策项 | 结论 | 说明 |
|---|--------|------|------|
| TD1 | **UI 技术栈** | Tailwind CSS + shadcn/ui | ⚠️ **本决策偏离 AGENTS.md 现有约束**（原约束为 Ant Design 6）。需同步：① 安装 Tailwind + shadcn/ui ② 更新 AGENTS.md「前端技术选型约束」③ 处理与现有 Ant Design 代码（AppLayout 等）的共存/迁移策略 |
| TD2 | **组件渲染模式** | Server Component（默认） | Hero 主体为静态内容，SSR 渲染利于 SEO；仅搜索框交互部分拆为 Client Component |
| TD3 | **背景图方案** | `next/image` (`fill` + `priority`) | 使用 Next.js 内置图片优化，绝对定位作为背景层，避免 CSS `background-image`（后者无法获得优化） |
| TD4 | **响应式策略** | Mobile-first（Tailwind 断点） | 基础样式面向移动端，`sm:` / `md:` / `lg:` 递进增强 |

---

## 1. 模块边界

### 1.1 In Scope（包含的功能）

| # | 功能点 | 说明 |
|---|--------|------|
| 1 | **全宽品牌背景图** | 使用 `next/image` 渲染全屏宽度背景图，覆盖 Hero 区域，带半透明遮罩以保证文字可读性 |
| 2 | **品牌标语** | 固定文案 "Unlock the Real China"，醒目大字号展示 |
| 3 | **副标题** | 固定文案 "Curated English guides, instant AI answers, and a traveler community — everything you need to explore China with confidence." |
| 4 | **搜索框（占位 UI）** | 视觉搜索输入框 + 提交按钮，placeholder 为 "Search destinations, guides, or ask anything…" |
| 5 | **搜索提交反馈** | 提交时显示非阻塞提示 "Search is coming soon!"，**不调用任何后端 API** |
| 6 | **响应式布局** | Mobile-first，移动端 / 平板 / 桌面端自适应高度与字号 |

### 1.2 Out of Scope（不包含的功能）

- ❌ **全站搜索功能**：搜索框仅为视觉占位，本期不做任何搜索逻辑（明确约束）
- ❌ **搜索结果页**：无搜索结果展示页面
- ❌ **Banner 轮播 / 广告位**：Hero 为静态单图，不做轮播
- ❌ **个性化标语**：所有用户看到相同文案，不做 A/B 测试或个性化
- ❌ **视频背景**：仅静态图片
- ❌ **Hero 内嵌 CTA 按钮**（除搜索框外）：本期搜索框是唯一交互元素

---

## 2. 核心场景

### 2.1 正常路径

**SC-Hero-01：首页加载渲染 Hero**
- **WHEN** 访客（含未登录游客）打开首页
- **THEN** Hero 区域渲染在全宽顶部，从上至下依次为：背景图（带遮罩）→ 标语 → 副标题 → 搜索框
- **AND** 背景图通过 `next/image` 加载，带有 `priority` 属性（首屏 LCP 优先）
- **AND** 标语 "Unlock the Real China" 以最大字号居中显示
- **AND** 副标题以次级字号居中显示，宽度受限以保证可读性

**SC-Hero-02：搜索框占位展示**
- **WHEN** 用户看到 Hero 区域的搜索框
- **THEN** 输入框显示 placeholder 文本 "Search destinations, guides, or ask anything…"
- **AND** 输入框右侧（或下方移动端）显示一个搜索按钮，带搜索图标

**SC-Hero-03：搜索提交反馈**
- **WHEN** 用户在搜索框输入任意文本并提交（回车或点击搜索按钮）
- **THEN** 系统显示非阻塞提示消息 "Search is coming soon!"（使用 toast / inline message）
- **AND** **不发起任何 API 请求**（无网络调用）
- **AND** 输入框内容清空或保留（保留更友好）

### 2.2 异常路径

**SC-Hero-04：背景图加载失败**
- **WHEN** 背景图资源加载失败（网络错误、404 等）
- **THEN** Hero 区域显示降级背景（Tailwind 渐变色 `bg-gradient-to-b from-slate-800 to-slate-900`）
- **AND** 标语与副标题仍正常显示（文字颜色适配深色降级背景）
- **AND** 控制台输出警告日志（开发环境）

**SC-Hero-05：移动端视口适配**
- **WHEN** 首页在移动端（宽度 < 640px）打开
- **THEN** Hero 区域高度缩小为视口高度的 60%–70%（桌面端为 80%–90%）
- **AND** 标语字号缩小为桌面端的约 50%（如桌面 `text-6xl`，移动 `text-4xl`）
- **AND** 搜索框宽度撑满容器（`w-full`），搜索按钮移至输入框下方
- **AND** 背景图按 `cover` 模式缩放，不变形

**SC-Hero-06：大屏超宽视口**
- **WHEN** 首页在超宽桌面（宽度 ≥ 1280px）打开
- **THEN** Hero 区域内容居中，最大宽度受限（`max-w-7xl`），两侧留白
- **AND** 背景图仍全宽覆盖

---

## 3. 数据结构

### 3.1 组件树

```
HeroSection (Server Component)
├── HeroBackground (next/image, fill + priority)
├── HeroOverlay (半透明遮罩层)
└── HeroContent (内容容器，居中)
    ├── <h1> 标语
    ├── <p> 副标题
    └── HeroSearch (Client Component)
        ├── <Input> (shadcn/ui)
        └── <Button> (shadcn/ui)
```

### 3.2 组件 Props 定义

#### `HeroSection`（主组件 · Server Component）

```typescript
interface HeroSectionProps {
  /**
   * 背景图配置。
   * 使用 next/image 的 fill 模式，图片域名须在 next.config.ts 的 images.remotePatterns 中注册。
   */
  backgroundImage: HeroImage;

  /**
   * 品牌标语。固定为 "Unlock the Real China"。
   * 设为可选属性以支持未来覆盖，本期有默认值。
   * @default "Unlock the Real China"
   */
  tagline?: string;

  /**
   * 副标题文案。
   * @default "Curated English guides, instant AI answers, and a traveler community — everything you need to explore China with confidence."
   */
  subtitle?: string;

  /**
   * 搜索框 placeholder 文本。
   * @default "Search destinations, guides, or ask anything…"
   */
  searchPlaceholder?: string;
}

interface HeroImage {
  /** 图片源地址（相对路径或已注册的远程域名 URL） */
  src: string;
  /** 无障碍描述，MUST 不能为空字符串 */
  alt: string;
  /** 移动端专用图片（可选，更小尺寸优化移动流量） */
  mobileSrc?: string;
}
```

**约束**：
- `backgroundImage.alt` MUST 非空（无障碍要求，WCAG 2.1 AA）
- 当 `backgroundImage.src` 为远程 URL 时，域名 MUST 在 `next.config.ts` → `images.remotePatterns` 中注册
- `tagline` / `subtitle` / `searchPlaceholder` 均有默认值，组件可不传任何 props 直接渲染

#### `HeroSearch`（搜索框 · Client Component）

```typescript
"use client";

interface HeroSearchProps {
  /**
   * 搜索框 placeholder 文本。
   * @default "Search destinations, guides, or ask anything…"
   */
  placeholder?: string;

  /**
   * 提交搜索时的回调。
   * 本期固定行为：显示 "coming soon" 提示。
   * 暴露为 prop 以支持未来接入真实搜索。
   */
  onSubmit?: (query: string) => void;
}
```

**约束**：
- 此组件 MUST 标记 `"use client"`（含表单交互与 toast 状态）
- `onSubmit` 默认实现为显示提示消息，不发起网络请求
- 输入框禁止空提交（空字符串不触发反馈）

### 3.3 静态默认值常量

```typescript
// src/components/homepage/hero-constants.ts

export const HERO_DEFAULTS = {
  tagline: "Unlock the Real China",
  subtitle:
    "Curated English guides, instant AI answers, and a traveler community — everything you need to explore China with confidence.",
  searchPlaceholder: "Search destinations, guides, or ask anything…",
  comingSoonMessage: "Search is coming soon!",
} as const;

export const HERO_BACKGROUND_FALLBACK_CLASS =
  "bg-gradient-to-b from-slate-800 to-slate-900";
```

### 3.4 文件结构

```
src/components/homepage/hero/
├── HeroSection.tsx        # 主组件（Server Component）
├── HeroSearch.tsx         # 搜索框（Client Component）
├── hero-constants.ts      # 默认文案常量
└── index.ts               # 模块导出
```

---

## 4. 验收标准（Acceptance Checklist）

### 4.1 功能验收

- [ ] **AC-01** 游客（未登录）打开首页，Hero 区域渲染背景图 + 标语 "Unlock the Real China" + 副标题 + 搜索框
- [ ] **AC-02** 背景图使用 `next/image` 的 `fill` 模式渲染，带有 `priority` 属性（可通过 DevTools 确认 `<img>` 带 `fetchpriority="high"`）
- [ ] **AC-03** 搜索框显示 placeholder "Search destinations, guides, or ask anything…"
- [ ] **AC-04** 在搜索框输入文本并提交（回车 or 点击按钮），显示 "Search is coming soon!" 提示
- [ ] **AC-05** 搜索提交时 Network 面板无任何 API 请求（确认占位行为）
- [ ] **AC-06** 空输入提交时不触发任何反馈（防抖校验）

### 4.2 异常与降级验收

- [ ] **AC-07** 将背景图 `src` 改为无效地址，Hero 显示降级渐变背景（`from-slate-800 to-slate-900`），标语副标题正常可见
- [ ] **AC-08** 背景图加载失败时，文字颜色与降级背景对比度满足 WCAG AA 标准（对比度 ≥ 4.5:1）

### 4.3 响应式验收

- [ ] **AC-09** 移动端（< 640px）：Hero 高度约为视口 60%–70%，搜索框 `w-full`，按钮在输入框下方
- [ ] **AC-10** 平板（640px–1024px）：布局过渡自然，无内容溢出
- [ ] **AC-11** 桌面端（≥ 1024px）：Hero 高度约为视口 80%–90%，内容居中，最大宽度受限
- [ ] **AC-12** 超宽屏（≥ 1280px）：内容居中且 `max-w-7xl`，两侧留白，背景图仍全宽

### 4.4 无障碍验收

- [ ] **AC-13** 背景图 `next/image` 带有非空 `alt` 属性
- [ ] **AC-14** 搜索框 `<input>` 带有 `aria-label` 或关联 `<label>`（屏幕阅读器可识别）
- [ ] **AC-15** 搜索按钮带有 `aria-label="Search"` 或等效无障碍标签
- [ ] **AC-16** Hero 区域语义化：标语使用 `<h1>`，副标题使用 `<p>`

### 4.5 SEO 验收

- [ ] **AC-17** 页面源码（view-source）中包含 `<h1>Unlock the Real China</h1>`（SSR 输出，非客户端渲染）
- [ ] **AC-18** 副标题在页面源码中可见（SSR 输出）

### 4.6 技术约束验收

- [ ] **AC-19** 样式使用 Tailwind CSS 类名（无 inline style、无 CSS Modules）
- [ ] **AC-20** 搜索框组件使用 shadcn/ui 的 `Input` 和 `Button`（非原生 HTML 标签）
- [ ] **AC-21** 组件类型声明通过 TypeScript strict 模式检查（`npm run build` 无类型错误）
- [ ] **AC-22** HeroSection 为 Server Component（无 `"use client"` 声明）；HeroSearch 为 Client Component（有 `"use client"` 声明）

---

## 5. 依赖关系

| 维度 | 说明 |
|------|------|
| **数据依赖** | 无（纯静态内容，所有文案为前端常量） |
| **组件依赖** | 无（不依赖其他首页区块） |
| **被依赖** | 被 `homepage-shell` 组装挂载 |
| **外部依赖** | `next/image`（Next.js 内置）、shadcn/ui `Input` + `Button`、Tailwind CSS |
| **前置任务** | ⚠️ 安装并配置 Tailwind CSS + shadcn/ui（技术栈切换，见 TD1） |

---

## 6. 开放问题

1. **背景图素材**：需要确定具体的背景图资源（是放在 `public/` 本地，还是使用远程图床 URL？）。如远程，需在 `next.config.ts` 注册域名。
2. **shadcn/ui 迁移范围**：本次仅 Hero 用 shadcn/ui，还是全站逐步迁移？影响 AGENTS.md 约束更新范围与现有 AppLayout（Ant Design）的共存策略。
3. **toast 组件选型**：搜索提交反馈用 shadcn/ui 的 `sonner` / `toast`，还是其他方案？
