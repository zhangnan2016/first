# Homepage City Quick Access · 模块规格说明

> **文档版本**：v1.0
> **所属变更**：[build-homepage](../openspec/changes/build-homepage/proposal.md)
> **关联 delta spec**：[homepage-city-quick-access/spec.md](../openspec/changes/build-homepage/specs/homepage-city-quick-access/spec.md)
> **优先级**：P0（静态区块，零 API 依赖，可独立先行交付）

---

## 0. 技术决策记录

| # | 决策项 | 结论 | 说明 |
|---|--------|------|------|
| TD1 | **数据来源** | 前端静态常量 | 8 个 MVP 城市硬编码为 TypeScript 常量，不依赖 API。保证零网络请求、即时渲染 |
| TD2 | **城市图标方案** | `lucide-react` 图标 + Tailwind 背景色 | 每个城市使用一个代表性图标（如 Beijing 用 `Landmark`），配差异化背景色 |
| TD3 | **跳转目标** | `/guides/<city-code>` | 小写城市编码作为路由参数（如 `/guides/bjs`），目标页面为 Coming Soon 占位页 |
| TD4 | **组件渲染模式** | Server Component | 纯静态内容，无交互逻辑，SSR 渲染利于 SEO |

---

## 1. 模块边界

### 1.1 In Scope（包含的功能）

| # | 功能点 | 说明 |
|---|--------|------|
| 1 | **8 城图标网格** | 展示 8 个 MVP 城市的图标 + 城市英文名 |
| 2 | **点击跳转** | 点击城市图标跳转至 `/guides/<city-code>` 城市攻略占位页 |
| 3 | **响应式网格** | 桌面 4 列、平板 3 列、移动端 2 列 |
| 4 | **hover 交互** | 鼠标悬停时图标卡片有视觉反馈 |

### 1.2 Out of Scope（不包含的功能）

- ❌ **动态城市列表**：不从 API 获取城市列表，固定 8 城
- ❌ **城市搜索 / 筛选**：不支持在城市网格内搜索
- ❌ **城市天气 / 实时信息**：卡片不展示天气或实时数据
- ❌ **城市照片背景**：使用图标而非照片（照片留给 PopularAttractions）
- ❌ **用户收藏城市**：不支持收藏/标记功能

---

## 2. 核心场景

### 2.1 正常路径

**SC-City-01：城市网格渲染**
- **WHEN** 首页加载，CityQuickAccess 区块渲染
- **THEN** 区块标题 "Explore by City" 显示
- **AND** 8 个城市图标按固定顺序排列在网格中：Beijing → Shanghai → Chengdu → Xi'an → Guangzhou → Shenzhen → Hangzhou → Guilin
- **AND** 每个城市显示代表性图标（lucide-react）+ 英文名
- **AND** 无网络请求（纯静态渲染）

**SC-City-02：点击城市跳转**
- **WHEN** 用户点击城市图标（如 Beijing）
- **THEN** 浏览器导航至 `/guides/bjs`
- **AND** 目标页面为 Coming Soon 占位页，返回 HTTP 200
- **AND** 占位页提供 "Back to Home" 链接

**SC-City-03：hover 交互**
- **WHEN** 用户鼠标悬停在城市图标上
- **THEN** 图标卡片背景色加深或边框高亮
- **AND** 光标变为 pointer（`cursor-pointer`）
- **AND** 可选：轻微 scale 放大动画（`hover:scale-105`）

### 2.2 异常路径

**SC-City-04：移动端网格适配**
- **WHEN** 首页在移动端（宽度 < 640px）查看
- **THEN** 城市网格显示为 2 列（grid-cols-2）
- **AND** 图标尺寸适当缩小，文字字号适配
- **AND** 8 个城市分 4 行显示，无溢出

---

## 3. 数据结构

### 3.1 组件树

```
CityQuickAccess (Server Component)
├── <h2> "Explore by City"
└── <div class="grid">
    └── CityIcon × 8
        ├── Icon (lucide-react)
        ├── <span> CityName
        └── <Link href="/guides/{code}">
```

### 3.2 组件 Props 定义

#### `CityQuickAccess`（主组件 · Server Component）

```typescript
interface CityQuickAccessProps {
  /**
   * 区块标题文案。
   * @default "Explore by City"
   */
  title?: string;

  /**
   * 自定义城市列表。
   * 不传时使用 MVP_CITIES 默认 8 城。
   */
  cities?: CityEntry[];
}
```

#### `CityIcon`（城市图标 · Server Component）

```typescript
interface CityIconProps {
  /** 城市数据 */
  city: CityEntry;
}

interface CityEntry {
  /** 城市编码（大写 IATA 风格，如 "BJS"） */
  code: string;
  /** 英文名（如 "Beijing"） */
  name: string;
  /** 中文名（可选，tooltip 展示） */
  nameCn?: string;
  /** lucide-react 图标组件名（如 "Landmark"） */
  icon: LucideIcon;
  /** Tailwind 背景色类（如 "bg-red-100"） */
  bgColor: string;
  /** Tailwind 图标色类（如 "text-red-600"） */
  iconColor: string;
}
```

### 3.3 静态城市常量

```typescript
// src/components/homepage/city-quick-access/city-constants.ts

import {
  Landmark,     // Beijing
  Building2,    // Shanghai
  PawPrint,     // Chengdu (熊猫)
  Castle,       // Xi'an (城墙/兵马俑)
  Utensils,     // Guangzhou (美食)
  Cpu,          // Shenzhen (科技)
  Lake,         // Hangzhou (西湖)
  Mountain,     // Guilin (山水)
} from "lucide-react";

export const MVP_CITIES: CityEntry[] = [
  { code: "BJS", name: "Beijing",    nameCn: "北京", icon: Landmark,  bgColor: "bg-red-50",    iconColor: "text-red-600" },
  { code: "SHA", name: "Shanghai",   nameCn: "上海", icon: Building2, bgColor: "bg-blue-50",   iconColor: "text-blue-600" },
  { code: "CTU", name: "Chengdu",    nameCn: "成都", icon: PawPrint,  bgColor: "bg-orange-50", iconColor: "text-orange-600" },
  { code: "SIA", name: "Xi'an",      nameCn: "西安", icon: Castle,    bgColor: "bg-amber-50",  iconColor: "text-amber-700" },
  { code: "CAN", name: "Guangzhou",  nameCn: "广州", icon: Utensils,  bgColor: "bg-green-50",  iconColor: "text-green-600" },
  { code: "SZX", name: "Shenzhen",   nameCn: "深圳", icon: Cpu,       bgColor: "bg-cyan-50",   iconColor: "text-cyan-600" },
  { code: "HGH", name: "Hangzhou",   nameCn: "杭州", icon: Lake,      bgColor: "bg-teal-50",   iconColor: "text-teal-600" },
  { code: "KWL", name: "Guilin",     nameCn: "桂林", icon: Mountain,  bgColor: "bg-emerald-50",iconColor: "text-emerald-600" },
];
```

### 3.4 文件结构

```
src/components/homepage/city-quick-access/
├── CityQuickAccess.tsx     # 主组件（Server Component）
├── CityIcon.tsx            # 城市图标子组件
├── city-constants.ts       # MVP_CITIES 静态常量
└── index.ts                # 模块导出
```

---

## 4. 验收标准（Acceptance Checklist）

### 4.1 功能验收

- [ ] **AC-01** 区块标题 "Explore by City" 可见
- [ ] **AC-02** 8 个城市图标按固定顺序排列（Beijing → Shanghai → Chengdu → Xi'an → Guangzhou → Shenzhen → Hangzhou → Guilin）
- [ ] **AC-03** 每个城市显示图标 + 英文名
- [ ] **AC-04** 点击城市图标跳转至 `/guides/<小写城市编码>`（如 `/guides/bjs`）
- [ ] **AC-05** 目标页面为 Coming Soon 占位页，返回 HTTP 200
- [ ] **AC-06** 页面加载时 Network 面板无城市相关 API 请求（纯静态）

### 4.2 交互验收

- [ ] **AC-07** 鼠标悬停城市图标时，卡片有视觉反馈（背景色加深 / 边框高亮 / scale 放大）
- [ ] **AC-08** 悬停时光标变为 pointer
- [ ] **AC-09** 城市图标支持键盘导航（Tab 可聚焦，Enter 可跳转）

### 4.3 响应式验收

- [ ] **AC-10** 移动端（< 640px）：网格 2 列（grid-cols-2），4 行
- [ ] **AC-11** 平板（640px–1024px）：网格 3 列（grid-cols-3）
- [ ] **AC-12** 桌面端（≥ 1024px）：网格 4 列（grid-cols-4），2 行

### 4.4 无障碍验收

- [ ] **AC-13** 每个城市链接有描述性 `aria-label`（如 `aria-label="Explore Beijing"`）
- [ ] **AC-14** 图标有 `aria-hidden="true"`（装饰性图标，文字已传达含义）

### 4.5 技术约束验收

- [ ] **AC-15** CityQuickAccess 为 Server Component（无 `"use client"`）
- [ ] **AC-16** 样式使用 Tailwind CSS
- [ ] **AC-17** 图标使用 `lucide-react`（非 `@ant-design/icons`）
- [ ] **AC-18** `npm run build` 通过 TypeScript strict 检查

---

## 5. 依赖关系

| 维度 | 说明 |
|------|------|
| **数据依赖** | 无（8 城为前端静态常量） |
| **组件依赖** | 被 `HomepageShell` 组装挂载 |
| **外部依赖** | `lucide-react`（8 个图标）、Tailwind CSS、Next.js `Link` |
| **前置任务** | ⚠️ Tailwind CSS + shadcn/ui 迁移（`migrate-ui-to-shadcn`，`lucide-react` 随之安装） |
| **弱依赖** | `/guides/<city>` 占位页须存在（景点攻略模块占位页） |

---

## 6. 开放问题

1. **城市编码方案**：使用 IATA 机场码（BJS/SHA）还是自定义拼音缩写（bj/sh）？影响路由 URL 形态。
2. **图标选型**：当前为每个城市手选 lucide 图标，是否需要定制 SVG 图标以更具辨识度？
3. **城市颜色**：当前为每个城市分配了浅色背景，是否需要品牌色规范统一管理？
