# Homepage AI Assistant Entry · 模块规格说明

> **文档版本**：v1.0
> **所属变更**：[build-homepage](../openspec/changes/build-homepage/proposal.md)
> **关联 delta spec**：[homepage-ai-assistant-entry/spec.md](../openspec/changes/build-homepage/specs/homepage-ai-assistant-entry/spec.md)
> **优先级**：P0（全局悬浮入口，挂在 Layout 层级）

---

## 0. 技术决策记录

| # | 决策项 | 结论 | 说明 |
|---|--------|------|------|
| TD1 | **挂载位置** | `(dashboard)/layout.tsx` 全局 Layout | 独立于 HomepageShell，在 dashboard 下所有页面可见，符合"随时唤起"需求 |
| TD2 | **按钮交互** | Client Component | 需处理点击事件、认证状态判断、chat 界面开关 |
| TD3 | **未登录处理** | 跳转登录页 | AI 助手为写操作（需登录），未登录用户点击跳转 `/login?redirect=<当前路径>` |
| TD4 | **本期聊天界面** | 占位跳转 | 本期不做完整聊天 UI，点击跳转 `/assistant` 占位页或显示 "Coming Soon" 提示 |

---

## 1. 模块边界

### 1.1 In Scope（包含的功能）

| # | 功能点 | 说明 |
|---|--------|------|
| 1 | **全局悬浮按钮（FAB）** | 固定在页面右下角的圆形悬浮按钮，持续可见 |
| 2 | **AI 图标 + Tooltip** | 按钮含 AI/Chat 图标，hover 时显示 "Ask AI" 提示 |
| 3 | **认证状态判断** | 点击时判断登录态：已登录 → 打开/跳转 AI 助手；未登录 → 跳转登录页 |
| 4 | **滚动持久可见** | 页面滚动时按钮始终可见（CSS `fixed` 定位） |
| 5 | **z-index 层级** | 按钮始终在最上层，不被其他内容遮挡 |

### 1.2 Out of Scope（不包含的功能）

- ❌ **AI 对话界面（Chat UI）**：本期不做聊天窗口、消息列表、输入框（仅入口按钮）
- ❌ **消息通知红点**：不展示未读消息提示
- ❌ **拖拽移动**：按钮位置固定，不可拖拽
- ❌ **自动弹出引导**：不自动弹出气泡提示用户使用 AI
- ❌ **语音输入入口**：仅按钮入口，不做语音功能
- ❌ **暗黑模式适配**：本期使用固定配色，不做主题切换（预留 CSS 变量支持）

---

## 2. 核心场景

### 2.1 正常路径

**SC-AI-01：悬浮按钮可见性**
- **WHEN** 用户浏览首页（或 dashboard 下任意页面）并在页面任意位置滚动
- **THEN** 右下角持续可见一个圆形悬浮按钮（FAB）
- **AND** 按钮显示 AI/Chat 图标（`lucide-react` 的 `Sparkles` 或 `MessageCircle`）
- **AND** 按钮 `z-index` 足够高，不被任何内容遮挡

**SC-AI-02：hover Tooltip**
- **WHEN** 用户鼠标悬停在悬浮按钮上
- **THEN** 显示 tooltip 文本 "Ask AI"
- **AND** 按钮有轻微视觉反馈（scale 放大 / 阴影增强）

**SC-AI-03：已登录用户点击**
- **WHEN** 已登录用户点击悬浮按钮
- **THEN** 浏览器跳转至 `/assistant`（AI 助手页面）
- **AND** 本期 `/assistant` 为 Coming Soon 占位页

### 2.2 异常路径

**SC-AI-04：未登录用户点击**
- **WHEN** 未登录用户（无 token cookie）点击悬浮按钮
- **THEN** 浏览器跳转至 `/login?redirect=<当前路径>`
- **AND** 登录成功后自动跳回原页面
- **AND** 不直接打开 AI 助手界面

**SC-AI-05：移动端适配**
- **WHEN** 页面在移动端查看
- **THEN** 悬浮按钮尺寸适配移动端（稍小，如 48×48px vs 桌面 56×56px）
- **AND** 按钮距右下边缘间距适配（如 `bottom-4 right-4` vs 桌面 `bottom-6 right-6`）
- **AND** 按钮不遮挡移动端关键内容

---

## 3. 数据结构

### 3.1 组件树

```
AiAssistantEntry (Client Component, 挂载于 layout.tsx)
├── <button class="fixed bottom-6 right-6 ...">
│   ├── Icon (lucide-react Sparkles)
│   └── Tooltip "Ask AI" (shadcn/ui Tooltip 或原生 title)
└── 认证逻辑:
    ├── isLoggedIn → router.push("/assistant")
    └── !isLoggedIn → router.push("/login?redirect=...")
```

### 3.2 组件 Props 定义

#### `AiAssistantEntry`（主组件 · Client Component）

```typescript
"use client";

interface AiAssistantEntryProps {
  /**
   * AI 助手目标路由。
   * @default "/assistant"
   */
  assistantRoute?: string;

  /**
   * Tooltip 文案。
   * @default "Ask AI"
   */
  tooltipText?: string;

  /**
   * 自定义按钮图标。
   * @default Sparkles (lucide-react)
   */
  icon?: LucideIcon;
}
```

**约束**：
- MUST 标记 `"use client"`（需 useRouter、路由跳转交互）
- 按钮定位 MUST 使用 `position: fixed`（Tailwind `fixed`）
- `z-index` MUST ≥ 50（确保在最上层，Tailwind `z-50`）

#### 认证状态检查

```typescript
// 认证状态通过 httpOnly cookie 判断（客户端无法直接读取）
// 使用客户端间接判断：检查 Zustand store 中的 userId 是否存在

// 方案：从 useUserStore 读取 userId
import { useUserStore } from "@/stores/user";

function handleClick() {
  const userId = useUserStore.getState().userId;
  if (userId) {
    router.push(assistantRoute);
  } else {
    const currentPath = window.location.pathname;
    router.push(`/login?redirect=${encodeURIComponent(currentPath)}`);
  }
}
```

### 3.3 样式常量

```typescript
// src/components/homepage/ai-assistant-entry/ai-entry-constants.ts

export const AI_ENTRY_DEFAULTS = {
  assistantRoute: "/assistant",
  tooltipText: "Ask AI",
  comingSoonMessage: "AI Assistant is coming soon!",
} as const;

/** 悬浮按钮 Tailwind 类名（桌面端） */
export const FAB_CLASS_DESKTOP =
  "fixed bottom-6 right-6 z-50 h-14 w-14 rounded-full " +
  "bg-primary text-primary-foreground shadow-lg " +
  "flex items-center justify-center " +
  "transition-all hover:scale-110 hover:shadow-xl " +
  "focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring";

/** 悬浮按钮 Tailwind 类名（移动端覆盖） */
export const FAB_CLASS_MOBILE = "sm:bottom-6 sm:right-6 bottom-4 right-4 sm:h-14 sm:w-14 h-12 w-12";
```

### 3.4 文件结构

```
src/components/homepage/ai-assistant-entry/
├── AiAssistantEntry.tsx       # 主组件（Client Component）
├── ai-entry-constants.ts      # 默认值与样式常量
└── index.ts                   # 模块导出

# 挂载位置（非本组件目录，在 layout 中引入）
src/app/(dashboard)/layout.tsx # 在 AppLayout 内挂载 <AiAssistantEntry />
```

---

## 4. 验收标准（Acceptance Checklist）

### 4.1 功能验收

- [ ] **AC-01** 首页及 dashboard 下任意页面，右下角可见圆形悬浮按钮
- [ ] **AC-02** 按钮显示 AI 图标（`Sparkles` 或等效 lucide 图标）
- [ ] **AC-03** 鼠标悬停时显示 tooltip "Ask AI"
- [ ] **AC-04** 按钮有 hover 视觉反馈（scale 放大 / 阴影增强）

### 4.2 滚动持久性验收

- [ ] **AC-05** 页面向下滚动时，按钮始终保持在视口右下角（fixed 定位）
- [ ] **AC-06** 按钮不被任何页面内容遮挡（z-index ≥ 50）

### 4.3 认证状态验收

- [ ] **AC-07** 已登录用户（Zustand store 有 userId）点击按钮，跳转至 `/assistant`
- [ ] **AC-08** 未登录用户点击按钮，跳转至 `/login?redirect=<当前路径>`
- [ ] **AC-09** 登录成功后自动跳回原页面

### 4.4 响应式验收

- [ ] **AC-10** 桌面端按钮尺寸 56×56px（h-14 w-14），距边缘 24px（bottom-6 right-6）
- [ ] **AC-11** 移动端按钮尺寸 48×48px（h-12 w-12），距边缘 16px（bottom-4 right-4）
- [ ] **AC-12** 移动端按钮不遮挡底部导航或其他关键 UI

### 4.5 无障碍验收

- [ ] **AC-13** 按钮有 `aria-label="Ask AI"`（屏幕阅读器可识别）
- [ ] **AC-14** 按钮支持键盘聚焦（Tab 可到达），聚焦时有 `focus-visible:ring` 样式
- [ ] **AC-15** 按钮 Enter 键可触发点击

### 4.6 技术约束验收

- [ ] **AC-16** AiAssistantEntry 为 Client Component（有 `"use client"`）
- [ ] **AC-17** 组件挂载在 `(dashboard)/layout.tsx`（非 HomepageShell 内）
- [ ] **AC-18** 样式使用 Tailwind CSS（无 inline style）
- [ ] **AC-19** `npm run build` 通过 TypeScript strict 检查

---

## 5. 依赖关系

| 维度 | 说明 |
|------|------|
| **数据依赖** | 无（按钮 UI 为静态；认证状态从 Zustand store 读取，不发起 API 请求） |
| **组件依赖** | 独立于 HomepageShell，挂在 Layout 层级 |
| **外部依赖** | `lucide-react`（Sparkles 图标）、Tailwind CSS、Next.js `useRouter`、Zustand `useUserStore` |
| **前置任务** | ⚠️ Tailwind CSS + shadcn/ui 迁移；`/assistant` 占位页须存在 |

---

## 6. 开放问题

1. **Tooltip 实现方式**：使用 shadcn/ui `Tooltip` 组件还是原生 `title` 属性？shadcn Tooltip 体验更好但需 Provider 包裹。
2. **认证判断方式**：当前通过 Zustand store 的 `userId` 判断登录态。如果用户刷新页面后 store 被重置但 cookie 仍在，是否需要额外的 `/auth/me` 检查？（建议本期信任 store，刷新后 layout 的 `getUserId` 会重新设置）
3. **动画效果**：是否需要按钮入场动画（如页面加载后从右下角滑入）？
