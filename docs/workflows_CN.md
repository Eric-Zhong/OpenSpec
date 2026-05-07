# Workflows

本指南涵盖 OpenSpec 的常见工作流模式以及每种的使用场景。关于基本设置，请参见 [Getting Started](getting-started.md)。关于命令参考，请参见 [Commands](commands.md)。

## 理念：行动，而不是阶段

传统工作流迫使你经历阶段：规划，然后实现，然后完成。但实际工作不会整齐地放入框中。

OPSX 采用不同方法：

```text
Traditional (phase-locked):

  PLANNING ────────► IMPLEMENTING ────────► DONE
      │                    │
      │   "Can't go back"  │
      └────────────────────┘

OPSX (fluid actions):

  proposal ──► specs ──► design ──► tasks ──► implement
```

**关键原则：**

- **行动，而不是阶段** - 命令是你可以做的事情，不是你被困住的阶段
- **依赖是启用器** - 它们显示什么是可能的，而不是下一步需要什么

> **自定义：** OPSX 工作流由定义 artifact 序列的 schemas 驱动。关于创建自定义 schemas 的详细信息，请参见 [Customization](customization.md)。

## 两种模式

### 默认快速路径（`core` profile）

新安装默认为 `core`，提供：
- `/opsx:propose`
- `/opsx:explore`
- `/opsx:apply`
- `/opsx:sync`
- `/opsx:archive`

典型流程：

```text
/opsx:propose ──► /opsx:apply ──► /opsx:sync ──► /opsx:archive
```

### 扩展/完整工作流（自定义选择）

如果你想要明确的脚手架和构建命令（`/opsx:new`、`/opsx:continue`、`/opsx:ff`、`/opsx:verify`、`/opsx:bulk-archive`、`/opsx:onboard`），用以下方式启用：

```bash
openspec config profile
openspec update
```

## 工作流模式（扩展模式）

### 快速功能

当你知道要构建什么，只需要执行时：

```text
/opsx:new ──► /opsx:ff ──► /opsx:apply ──► /opsx:verify ──► /opsx:archive
```

**示例对话：**

```text
You: /opsx:new add-logout-button

AI:  Created openspec/changes/add-logout-button/
     Ready to create: proposal

You: /opsx:ff

AI:  Creating all planning artifacts...
     ✓ proposal.md
     ✓ specs/
     ✓ design.md
     ✓ tasks.md
     Ready for implementation!

You: /opsx:apply

AI:  Working through tasks...
     ✓ Added logout button to header
     ✓ Implemented session clearing
     ✓ Added confirmation dialog
     All tasks complete!

You: /opsx:archive

AI:  ✓ Merged specs
     ✓ Archived change
```

**最适合：** 小型到中型功能、bug 修复、直接的更改。

### 探索性

当需求不清晰或需要先调查时：

```text
/opsx:explore ──► /opsx:new ──► /opsx:continue ──► ... ──► /opsx:apply
```

**示例对话：**

```text
You: /opsx:explore

AI:  What would you like to explore?

You: I want to improve page load performance but I'm not sure
     where the bottlenecks are.

AI:  Let me help investigate...
     [Analyzes bundle sizes, identifies slow queries, reviews
     component render patterns]

     I found three main bottlenecks:
     1. Large unoptimized images
     2. Synchronous data fetching in ProductList
     3. Re-renders from context changes

     Which would you like to address first?

You: Let's tackle the data fetching issue.

You: /opsx:new optimize-product-list-fetching

AI:  Created change. Ready to create: proposal

You: /opsx:continue

AI:  Creating proposal based on our exploration...
     [Writes proposal incorporating investigation insights]

     Ready to create: specs, design
     Which would you like next?

You: /opsx:continue

AI:  Creating specs...
```

**最适合：** 性能优化、调试、架构决策、需求不清晰。

### 并行 Changes

同时处理多个 changes：

```text
Change A: /opsx:new ──► /opsx:ff ──► /opsx:apply (in progress)
                                         │
                                    context switch
                                         │
Change B: /opsx:new ──► /opsx:ff ──────► /opsx:apply
```

**示例对话：**

```text
You: I'm working on add-dark-mode but got pulled into a bug fix

You: /opsx:new fix-login-redirect

AI:  Created openspec/changes/fix-login-redirect/

You: /opsx:ff

AI:  ✓ Created planning artifacts

You: /opsx:apply

AI:  Working on fix-login-redirect...
     [Implements the fix]
     All tasks complete!

You: /opsx:archive

AI:  ✓ Archived fix-login-redirect

You: Let's get back to dark mode

You: /opsx:apply add-dark-mode

AI:  Resuming add-dark-mode...
     Picking up at task 2.3: Update Header...
```

**最适合：** 并行工作流、紧急中断、团队协作。

当你有多个已完成的 changes 时，使用 `/opsx:bulk-archive`：

```text
You: /opsx:bulk-archive

AI:  Found 3 completed changes:
     - add-dark-mode (all tasks done)
     - fix-login-redirect (all tasks done)
     - update-footer (all tasks done)

     Checking for spec conflicts...
     ⚠ add-dark-mode and update-footer both touch specs/ui/

     Inspecting codebase to resolve...
     Both changes are implemented. Will apply in chronological order.

     Archive all 3 changes?

You: Yes

AI:  ✓ Archived add-dark-mode
     ✓ Archived fix-login-redirect
     ✓ Archived update-footer
     Specs merged in order: add-dark-mode → update-footer
```

批量归档检测多个 changes 触及同一个 specs 并通过检查实际实现的内容解决冲突。

### 完成 Change

推荐的完成流程：

```text
/opsx:apply ──► /opsx:verify ──► /opsx:archive
                    │                 │
              validates          prompts to sync
              implementation     if needed
```

#### Verify：检查你的工作

`/opsx:verify` 从三个维度验证实现与你的 artifacts：

```text
You: /opsx:verify

AI:  Verifying add-auth...

     COMPLETENESS
     ✓ All 12 tasks in tasks.md are checked
     ✓ All requirements in specs have corresponding code
     ⚠ Scenario "Session timeout after inactivity" not tested

     CORRECTNESS
     ✓ Implementation matches spec intent
     ✓ Edge cases from scenarios are handled
     ✓ Error states match spec definitions

     COHERENCE
     ✓ Design decisions reflected in code structure
     ✓ Naming conventions consistent with design.md
     ⚠ Design mentions "event-driven" but implementation uses polling

     SUMMARY
     ─────────────────────────────
     Critical issues: 0
     Warnings: 2
     Ready to archive: Yes (with warnings)

     Recommendations:
     1. Add test for session timeout scenario
     2. Consider refactoring to event-driven as designed, or update design.md
```

**verify 检查什么：**

| 维度 | 验证内容 |
|-----------|------------------|
| Completeness | 所有 tasks 完成，所有 requirements 实现，scenarios 覆盖 |
| Correctness | 实现匹配 spec 意图，edge cases 处理 |
| Coherence | 设计决策反映在代码中，模式一致 |

Verify 不会阻止归档，但它揭示你可能想先解决的问题。

#### Archive：完成 Change

`/opsx:archive` 完成 change 并将其移动到归档：

```text
You: /opsx:archive

AI:  Archiving add-auth...

     Artifact status:
     ✓ proposal.md exists
     ✓ specs/ exists
     ✓ design.md exists
     ✓ tasks.md exists (12/12 tasks complete)

     Delta specs: Not yet synced to main
     → Would you like to sync now?

You: Yes

AI:  ✓ Synced specs to openspec/specs/auth/spec.md
     ✓ Moved to openspec/changes/archive/2025-01-24-add-auth/

     Change archived successfully.
```

归档如果 specs 未同步会提示。它不会因未完成的 tasks 阻止，但会警告你。

## 何时使用什么

### `/opsx:ff` vs `/opsx:continue`

| 情况 | 使用 |
|-----------|-----|
| 需求清晰，准备构建 | `/opsx:ff` |
| 探索，想审查每一步 | `/opsx:continue` |
| 想在 specs 前迭代 proposal | `/opsx:continue` |
| 时间压力大，需要快速 | `/opsx:ff` |
| 复杂 change，想要控制 | `/opsx:continue` |

**经验法则：** 如果你能提前描述完整范围，使用 `/opsx:ff`。如果你边做边搞清楚，使用 `/opsx:continue`。

### 何时更新 vs 从头开始

一个常见问题：什么时候更新现有 change 是可以的，什么时候应该开始新的？

**更新现有 change 当：**

- 相同意图，改进执行
- 范围缩小（MVP 先，剩余以后）
- 学习驱动的修正（代码库不是你预期的）
- 基于实现发现的设计调整

**开始新 change 当：**

- 意图根本改变
- 范围爆炸到完全不同的工作
- 原始 change 可以独立标记为"完成"
- 补丁会比澄清更混乱

```text
                     ┌─────────────────────────────────────┐
                     │     Is this the same work?          │
                     └──────────────┬──────────────────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼
          Same intent?      >50% overlap?      Can original
          Same problem?     Same scope?        be "done" without
                 │                  │          these changes?
                 │                  │                  │
       ┌────────┴────────┐  ┌──────┴──────┐   ┌───────┴───────┐
       │                 │  │             │   │               │
      YES               NO YES           NO  NO              YES
       │                 │  │             │   │               │
       ▼                 ▼  ▼             ▼   ▼               ▼
    UPDATE            NEW  UPDATE       NEW  UPDATE          NEW
```

**示例："添加深色模式"**

- "还需要支持自定义主题" → 新 change（范围爆炸）
- "系统偏好检测比预期难" → 更新（相同意图）
- "先发布切换，稍后添加偏好" → 更新然后归档，然后新 change

## 最佳实践

### 保持 Changes 专注

每个 change 一个逻辑工作单元。如果你要做"添加功能 X 同时重构 Y"，考虑两个单独的 changes。

**为什么重要：**
- 更易于审查和理解
- 更干净的历史记录
- 可以独立发布
- 需要时回滚更简单

### 对不清晰需求使用 `/opsx:explore`

在提交 change 前，探索问题空间：

```text
You: /opsx:explore

AI:  What would you like to explore?

You: How should we handle rate limiting for our API?

AI:  Let me investigate your current setup and options...
     [Researches codebase, considers patterns]

     I see three approaches we could take:
     1. Middleware-based (simple, per-endpoint)
     2. Redis-backed (scalable, shared state)
     3. Token bucket per user (fair, complex)

     Your current stack suggests #1 or #2. What's your scale?
```

探索在创建 artifacts 前澄清思路。

### 归档前验证

使用 `/opsx:verify` 检查实现与 artifacts 匹配：

```text
You: /opsx:verify

AI:  Verifying add-dark-mode...

     ✓ All tasks in tasks.md are checked
     ✓ Requirements in specs have corresponding code
     ✓ Design decisions are reflected in implementation

     Ready to archive!
```

在关闭 change 前捕获不匹配。

### 清晰命名 Changes

好名称使 `openspec list` 有用：

```text
Good:                          Avoid:
add-dark-mode                  feature-1
fix-login-redirect             update
optimize-product-query         changes
implement-2fa                  wip
```

## 命令快速参考

关于完整命令详情和选项，请参见 [Commands](commands.md)。

| 命令 | 用途 | 何时使用 |
|---------|---------|-------------|
| `/opsx:propose` | 创建 change + 规划 artifacts | 快速默认路径（`core` profile） |
| `/opsx:explore` | 思考想法 | 需求不清晰，调查 |
| `/opsx:new` | 开始 change 脚手架 | 扩展模式，明确 artifact 控制 |
| `/opsx:continue` | 创建下一个 artifact | 扩展模式，逐步 artifact 创建 |
| `/opsx:ff` | 创建所有规划 artifacts | 扩展模式，清晰范围 |
| `/opsx:apply` | 实现 tasks | 准备好编写代码 |
| `/opsx:verify` | 验证实现 | 扩展模式，归档前 |
| `/opsx:sync` | 合并 delta specs | 扩展模式，可选 |
| `/opsx:archive` | 完成 change | 所有工作完成 |
| `/opsx:bulk-archive` | 归档多个 changes | 扩展模式，并行工作 |

## 下一步

- [Commands](commands.md) - 带有选项的完整命令参考
- [Concepts](concepts.md) - 深入了解 specs、artifacts 和 schemas
- [Customization](customization.md) - 创建自定义工作流
