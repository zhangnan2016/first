---
trigger: always_on
---
# 规范驱动开发规则

始终生效，适用于所有智能会话和行间会话请求。

## 核心原则

**规范是 source of truth（唯一真相源），代码是规范的派生产物。**

在编写任何功能代码之前，必须先确认变更规范已就绪。

## 变更工作流

所有功能开发遵循 OpenSpec 三步工作流：

```
/opsx:propose → /opsx:apply → /opsx:archive
```

### 1. 提案阶段（/opsx:propose）

- 明确 **意图**（为什么做）、**范围**（做什么/不做什么）、**方法**（技术方案）
- 编写 delta spec 描述规范变更（ADDED / MODIFIED / REMOVED）
- 确认 tasks.md 已生成且任务清单清晰

### 2. 实现阶段（/opsx:apply）

- 按 tasks.md 逐项实现，完成一项勾选一项 `[x]`
- 实现期间遵循 TDD（RED-GREEN-REFACTOR）
- 可以随时更新任何工件（proposal/design/tasks），规范是流动的不是僵化的

### 3. 归档阶段（/opsx:archive）

- 确认所有任务完成且验证通过
- 归档后 delta spec 自动合并到 openspec/specs/（真相源）
- 变更移至 changes/archive/ 保留审计记录

## 规范编写要求

- spec 描述 **行为**（what），不描述 **实现**（how）
- 使用 RFC 2119 关键词：SHALL/MUST（必须）、SHOULD（应当）、MAY（可以）
- 每个需求附带 **可测试的场景**（Given/When/Then）
- 内部类名、函数名、库选择不属于 spec，属于 design.md
