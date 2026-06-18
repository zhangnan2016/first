# Superpowers — 技能方法论

Superpowers 是一套为 AI 编码智能体设计的 **完整软件开发方法论**，由一组可组合的技能（skills）和初始指令构成，确保智能体在正确的时刻使用正确的技能。

> 来源理念：[obra/superpowers](https://github.com/obra/superpowers)，本项目已适配 Qoder 平台。

## 核心理念

- **测试驱动开发（TDD）** — 永远先写测试
- **系统化优于即兴** — 流程优于猜测
- **降低复杂度** — 简洁是首要目标
- **证据优于声明** — 验证后再宣布成功

## 技能优先级

当多个技能可能适用时，按此顺序使用：

1. **流程技能优先（Process skills）** — brainstorming、systematic-debugging：决定"如何"着手
2. **实现技能其次（Implementation skills）** — TDD、plans：指导执行

```
"让我们构建 X"  → 先 brainstorming，再实现技能
"修复这个 bug"  → 先 systematic-debugging，再领域技能
```

## 技能列表

| 技能 | 类型 | 作用 |
|------|------|------|
| [using-superpowers](skills/using-superpowers/SKILL.md) | Meta | 技能系统入口与触发规则 |
| [brainstorming](skills/brainstorming/SKILL.md) | Process | 写代码前先头脑风暴、提炼设计 |
| [systematic-debugging](skills/systematic-debugging/SKILL.md) | Process | 4 阶段根因定位 |
| [writing-plans](skills/writing-plans/SKILL.md) | Process | 设计拆解为可执行任务清单 |
| [executing-plans](skills/executing-plans/SKILL.md) | Process | 按计划分批执行 + 检查点 |
| [test-driven-development](skills/test-driven-development/SKILL.md) | Implementation | RED-GREEN-REFACTOR 循环 |
| [verification-before-completion](skills/verification-before-completion/SKILL.md) | Implementation | 完成前验证 |
| [requesting-code-review](skills/requesting-code-review/SKILL.md) | Implementation | 代码审查请求 |

## 技能类型

- **刚性技能（Rigid）**：TDD、debugging — 严格遵循，不要弱化纪律
- **弹性技能（Flexible）**：patterns — 根据上下文调整原则

技能本身会告诉你它属于哪一类。

## 基本工作流

1. **brainstorming** — 写代码前激活，通过提问提炼想法，探索替代方案，分段展示设计供验证
2. **writing-plans** — 设计批准后激活，拆分为小任务（每个 2-5 分钟），包含精确文件路径和验证步骤
3. **executing-plans** — 带计划激活，分批执行并设置人工检查点
4. **test-driven-development** — 实现期间激活，强制 RED-GREEN-REFACTOR
5. **systematic-debugging** — 遇到 bug 时激活，4 阶段根因流程
6. **requesting-code-review** — 任务间激活，对照计划审查
7. **verification-before-completion** — 完成时激活，确保真正修复

**智能体在任何任务前都会检查相关技能。** 这是强制工作流，而非建议。

## 与 OpenSpec 的协同

Superpowers 解决"怎么做"，OpenSpec 解决"做什么"。协同方式见项目根 [AGENTS.md](../AGENTS.md)：

```
brainstorming → /opsx:propose（写规范）→ writing-plans → /opsx:apply（实现）→ TDD/调试 → /opsx:archive
```
