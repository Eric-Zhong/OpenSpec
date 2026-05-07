# Commands

这是 OpenSpec slash commands 的参考文档。这些命令在你的 AI 编程助手的聊天界面中调用（例如 Claude Code、Cursor、Windsurf）。

有关工作流模式和每种命令的使用场景，请参见 [Workflows](workflows.md)。有关 CLI 命令，请参见 [CLI](cli.md)。

## 快速参考

### 默认快速路径（`core` profile）

| 命令 | 用途 |
|---------|---------|
| `/opsx:propose` | 一步创建 change 并生成规划 artifacts |
| `/opsx:explore` | 在提交 change 之前思考想法、调查问题、澄清需求 |
| `/opsx:apply` | 实现来自 change 的 tasks |
| `/opsx:sync` | 将 delta specs 合并到主 specs |
| `/opsx:archive` | 归档已完成的 change |

### 扩展工作流命令（自定义工作流选择）

| 命令 | 用途 |
|---------|---------|
| `/opsx:new` | 开始新的 change 脚手架 |
| `/opsx:continue` | 基于依赖创建下一个 artifact |
| `/opsx:ff` | 快速前进：一次创建所有规划 artifacts |
| `/opsx:verify` | 验证实现与 artifacts 匹配 |
| `/opsx:bulk-archive` | 一次归档多个 changes |
| `/opsx:onboard` | 完整工作流的引导教程 |

默认全局 profile 是 `core`。要启用扩展工作流命令，运行 `openspec config profile`，选择 workflows，然后在你的项目中运行 `openspec update`。

---

## 命令参考

### `/opsx:propose`

一步创建新 change 并生成规划 artifacts。这是 `core` profile 中的默认开始命令。

**语法：**
```text
/opsx:propose [change-name-or-description]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `change-name-or-description` | 否 | Kebab-case 名称或纯文本 change 描述 |

**执行操作：**
- 创建 `openspec/changes/<change-name>/`
- 生成实现前所需的 artifacts（对于 `spec-driven`：proposal、specs、design、tasks）
- 在 change 准备好 `/opsx:apply` 时停止

**示例：**
```text
You: /opsx:propose add-dark-mode

AI:  Created openspec/changes/add-dark-mode/
     ✓ proposal.md
     ✓ specs/ui/spec.md
     ✓ design.md
     ✓ tasks.md
     Ready for implementation. Run /opsx:apply.
```

**提示：**
- 使用这个获得最快的端到端路径
- 如果你想要逐步控制 artifacts，启用扩展工作流并使用 `/opsx:new` + `/opsx:continue`

---

### `/opsx:explore`

在提交 change 之前思考想法、调查问题、澄清需求。

**语法：**
```
/opsx:explore [topic]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `topic` | 否 | 你想探索或调查的内容 |

**执行操作：**
- 打开探索性对话，无需结构
- 调查代码库以回答问题
- 比较选项和方法
- 创建可视化图表以澄清思路
- 当洞察清晰时可以过渡到 `/opsx:propose`（默认）或 `/opsx:new`（扩展工作流）

**示例：**
```text
You: /opsx:explore

AI:  What would you like to explore?

You: How should we handle authentication for the mobile app?

AI:  Let me investigate your current auth setup...
     [Analyzes codebase]

     I see you're using session-based auth for the web app.
     For mobile, we have three main options:

     1. JWT tokens (stateless, works offline)
     2. OAuth2 (if you need third-party login)
     3. Extend current sessions (simpler, but requires connectivity)

     Your API already has CORS configured. Which direction interests you?

You: Let's go with JWT. Can we start a change for that?

AI:  Ready when you are. Run /opsx:propose add-jwt-auth to begin.
```

**提示：**
- 当需求不清晰或需要调查时使用
- 探索期间不创建 artifacts
- 适合在决定前比较多种方法
- 可以读取文件和搜索代码库

---

### `/opsx:new`

开始新的 change 脚手架。创建 change 文件夹并等待你用 `/opsx:continue` 或 `/opsx:ff` 生成 artifacts。

此命令是扩展工作流集的一部分（不包含在默认 `core` profile 中）。

**语法：**
```
/opsx:new [change-name] [--schema <schema-name>]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `change-name` | 否 | change 文件夹的名称（如果未提供则提示） |
| `--schema` | 否 | 要使用的工作流 schema（默认：从 config 或 `spec-driven`） |

**执行操作：**
- 创建 `openspec/changes/<change-name>/` 目录
- 在 change 文件夹中创建 `.openspec.yaml` 元数据文件
- 显示准备好创建的第一个 artifact 模板
- 如果未提供则提示 change 名称和 schema

**创建内容：**
```
openspec/changes/<change-name>/
└── .openspec.yaml    # Change 元数据（schema、创建日期）
```

**示例：**
```
You: /opsx:new add-dark-mode

AI:  Created openspec/changes/add-dark-mode/
     Schema: spec-driven

     Ready to create: proposal
     Use /opsx:continue to create it, or /opsx:ff to create all artifacts.
```

**提示：**
- 使用描述性名称：`add-feature`、`fix-bug`、`refactor-module`
- 避免通用名称如 `update`、`changes`、`wip`
- Schema 也可以在项目配置（`openspec/config.yaml`）中设置

---

### `/opsx:continue`

在依赖链中创建下一个 artifact。一次创建一个 artifact 以实现增量进度。

**语法：**
```
/opsx:continue [change-name]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `change-name` | 否 | 要继续的 change（如果未提供则从上下文推断） |

**执行操作：**
- 查询 artifact 依赖图
- 显示哪些 artifacts 已就绪 vs 被阻止
- 创建第一个就绪的 artifact
- 读取依赖文件以获取上下文
- 显示创建后可用的内容

**示例：**
```
You: /opsx:continue

AI:  Change: add-dark-mode

     Artifact status:
     ✓ proposal    (done)
     ◆ specs       (ready)
     ◆ design      (ready)
     ○ tasks       (blocked - needs: specs)

     Creating specs...

     [Reads proposal.md, creates specs based on requirements]

     ✓ Created openspec/changes/add-dark-mode/specs/ui/spec.md

     Now available: tasks
     Run /opsx:continue to create the next artifact.
```

**提示：**
- 当你想在继续之前审查每个 artifact 时使用
- 适合你想要的控制的复杂 changes
- 多个 artifacts 可能同时就绪
- 你可以在继续之前编辑已创建的 artifacts

---

### `/opsx:ff`

快速前进 artifact 创建。一次创建所有规划 artifacts。

**语法：**
```
/opsx:ff [change-name]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `change-name` | 否 | 要快速前进的 change（如果未提供则从上下文推断） |

**执行操作：**
- 按依赖顺序创建所有 artifacts
- 通过 todo 列表跟踪进度
- 当所有 `apply-required` artifacts 完成时停止
- 在创建下一个 artifact 之前读取每个依赖

**示例：**
```
You: /opsx:ff add-dark-mode

AI:  Fast-forwarding add-dark-mode...

     ✓ Creating proposal.md
     ✓ Creating specs/ui/spec.md
     ✓ Creating design.md
     ✓ Creating tasks.md

     All planning artifacts complete!
     Ready for implementation. Run /opsx:apply to begin.
```

**提示：**
- 当你清楚自己要构建什么时使用
- 比 `/opsx:continue` 对于简单的 changes 更快
- 你之后仍然可以编辑 artifacts
- 适合小型到中型功能

---

### `/opsx:apply`

实现来自 change 的 tasks。处理任务列表，编写代码并勾选项目。

**语法：**
```
/opsx:apply [change-name]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `change-name` | 否 | 要实现的 change（如果未提供则从上下文推断） |

**执行操作：**
- 读取 `tasks.md` 并识别未完成的 tasks
- 逐一处理 tasks
- 根据需要编写代码、创建文件、运行测试
- 用复选框 `[x]` 标记 tasks 完成

**示例：**
```
You: /opsx:apply

AI:  Implementing add-dark-mode...

     Reading tasks.md:
     - [ ] 1.1 Create ThemeContext
     - [ ] 1.2 Add CSS custom properties
     - [ ] 1.3 Implement localStorage persistence

     Working on 1.1: Create ThemeContext...
     [Creates src/context/ThemeContext.tsx]

     ✓ 1.1 Complete

     Working on 1.2: Add CSS custom properties...
     [Updates src/styles/globals.css]

     ✓ 1.2 Complete
     ...
```

**提示：**
- 如果中断，可以从中断处继续
- 通过指定 change 名称用于并行 changes
- 完成状态通过 `tasks.md` 复选框跟踪

---

### `/opsx:verify`

验证实现与你的 change artifacts 匹配。检查完整性、正确性和连贯性。

**语法：**
```
/opsx:verify [change-name]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `change-name` | 否 | 要验证的 change（如果未提供则从上下文推断） |

**执行操作：**
- 从三个维度检查实现质量
- 搜索代码库中的实现证据
- 报告分类为 CRITICAL、WARNING 或 SUGGESTION 的问题
- 不阻止归档，但揭示问题

**验证维度：**

| 维度 | 验证内容 |
|-----------|-------------------|
| **Completeness** | 所有 tasks 完成，所有 requirements 实现，scenarios 覆盖 |
| **Correctness** | 实现匹配 spec 意图，edge cases 处理 |
| **Coherence** | 设计决策反映在代码中，模式一致 |

**示例：**
```
You: /opsx:verify

AI:  Verifying add-dark-mode...

     COMPLETENESS
     ✓ All 8 tasks in tasks.md are checked
     ✓ All requirements in specs have corresponding code
     ⚠ Scenario "System preference detection" has no test coverage

     CORRECTNESS
     ✓ Implementation matches spec intent
     ✓ Edge cases from scenarios are handled
     ✓ Error states match spec definitions

     COHERENCE
     ✓ Design decisions reflected in code structure
     ✓ Naming conventions consistent with design.md
     ⚠ Design mentions "CSS variables" but implementation uses Tailwind classes

     SUMMARY
     ─────────────────────────────
     Critical issues: 0
     Warnings: 2
     Ready to archive: Yes (with warnings)

     Recommendations:
     1. Add test for system preference detection
     2. Update design.md to reflect Tailwind usage, or refactor to use CSS variables
```

**提示：**
- 在归档前运行以尽早发现不匹配
- 警告不会阻止归档，但表示潜在问题
- 适合在提交前审查 AI 的工作
- 可以揭示 artifacts 和实现之间的漂移

---

### `/opsx:sync`

**可选命令。** 将 change 的 delta specs 合并到主 specs。归档会在需要时提示同步，所以通常你不需要手动运行此命令。

**语法：**
```
/opsx:sync [change-name]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `change-name` | 否 | 要同步的 change（如果未提供则从上下文推断） |

**执行操作：**
- 从 change 文件夹读取 delta specs
- 解析 ADDED/MODIFIED/REMOVED/RENAMED 部分
- 将更改合并到主 `openspec/specs/` 目录
- 保留未在 delta 中提及的现有内容
- 不归档 change（保持活动）

**示例：**
```text
You: /opsx:sync

AI:  Syncing add-dark-mode delta specs...

     Reading: openspec/changes/add-dark-mode/specs/ui/spec.md
     Target:  openspec/specs/ui/spec.md

     Changes to apply:
     ✓ ADDED: Theme Selection requirement (2 scenarios)
     ✓ ADDED: Accessibility requirement (1 scenario)

     Merging...
     ✓ openspec/specs/ui/spec.md updated

     Change remains active. Run /opsx:archive when ready to complete.
```

**何时手动使用：**

| 场景 | 使用 sync？ |
|----------|--------|
| 长期运行的 change，希望在归档前 specs 进入主版本 | 是 |
| 多个并行 changes 需要更新的基础 specs | 是 |
| 想单独预览/审查合并 | 是 |
| 快速 change，直接归档 | 否（archive 处理） |

**提示：**
- Sync 是智能的，不是复制粘贴
- 可以在不重复的情况下向现有 requirements 添加 scenarios
- Sync 后 change 保持活动（不归档）
- 大多数用户永远不需要直接调用此命令——archive 在需要时提示

---

### `/opsx:archive`

归档已完成的 change。完成 change 并将其移动到归档文件夹。

**语法：**
```
/opsx:archive [change-name]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `change-name` | 否 | 要归档的 change（如果未提供则从上下文推断） |

**执行操作：**
- 检查 artifact 完成状态
- 检查 task 完成状态（如果未完成则警告）
- 如果尚未同步则提供同步 delta specs
- 将 change 文件夹移动到 `openspec/changes/archive/YYYY-MM-DD-<name>/`
- 保留所有 artifacts 以供审计追踪

**示例：**
```text
You: /opsx:archive

AI:  Archiving add-dark-mode...

     Artifact status:
     ✓ proposal.md exists
     ✓ specs/ exists
     ✓ design.md exists
     ✓ tasks.md exists (8/8 tasks complete)

     Delta specs: Not yet synced
     → Sync now? (recommended)

You: Yes

AI:  ✓ Synced specs to openspec/specs/ui/spec.md
     ✓ Moved to openspec/changes/archive/2025-01-24-add-dark-mode/

     Change archived successfully.
```

**提示：**
- Archive 不会因未完成的 tasks 阻止，但会警告
- Delta specs 可以在归档期间或之前同步
- 归档的 changes 保留以供历史查看
- 先使用 `/opsx:verify` 发现问题

---

### `/opsx:bulk-archive`

一次归档多个已完成的 changes。处理 changes 之间的 spec 冲突。

**语法：**
```
/opsx:bulk-archive [change-names...]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `change-names` | 否 | 要归档的特定 changes（如果未提供则提示选择） |

**执行操作：**
- 列出所有已完成的 changes
- 归档前验证每个 change
- 检测跨 changes 的 spec 冲突
- 通过检查实际实现的内容解决冲突
- 按时间顺序归档

**示例：**
```
You: /opsx:bulk-archive

AI:  Found 3 completed changes:
     - add-dark-mode (8/8 tasks complete)
     - fix-login-bug (3/3 tasks complete)
     - update-footer (2/2 tasks complete)

     Checking for spec conflicts...
     ⚠ add-dark-mode and update-footer both touch specs/ui/

     Inspecting codebase to resolve...
     Both changes are implemented. Will merge in chronological order:
     1. add-dark-mode (created Jan 20)
     2. update-footer (created Jan 22)

     Archive all 3 changes?

You: Yes

AI:  ✓ Archived add-dark-mode
     ✓ Archived fix-login-bug
     ✓ Archived update-footer
     Specs merged: add-dark-mode → update-footer
```

**提示：**
- 适合并行工作流
- 冲突解决是 agent 性的（检查代码库）
- Changes 按创建顺序归档
- 在覆盖 spec 内容前提示

---

### `/opsx:onboard`

完整 OpenSpec 工作流的引导入职。通过你的实际代码库的交互式教程。

**语法：**
```
/opsx:onboard
```

**执行操作：**
- 通过叙述引导完成整个工作流周期
- 扫描你的代码库以寻找真实的改进机会
- 创建具有真实 artifacts 的实际 change
- 实现实际工作（小、安全的更改）
- 归档已完成的 change
- 在发生时解释每个步骤

**阶段：**
1. 欢迎和代码库分析
2. 寻找改进机会
3. 创建 change（`/opsx:new`）
4. 编写 proposal
5. 创建 specs
6. 编写 design
7. 创建 tasks
8. 实现 tasks（`/opsx:apply`）
9. 验证实现
10. 归档 change
11. 总结和后续步骤

**示例：**
```
You: /opsx:onboard

AI:  Welcome to OpenSpec!

     I'll walk you through the complete workflow using your actual codebase.
     We'll find something small to improve, create a proper change for it,
     implement it, and archive it.

     Let me scan your codebase for opportunities...

     [Analyzes codebase]

     I found a few things we could work on:
     1. Add input validation to the contact form
     2. Improve error messages in the auth flow
     3. Add loading states to async buttons

     Which interests you? (or suggest something else)
```

**提示：**
- 最适合学习工作流的新用户
- 使用真实代码，不是玩具示例
- 创建你可以保留或丢弃的实际 change
- 需要 15-30 分钟完成

---

## AI 工具的命令语法

不同的 AI 工具使用略微不同的命令语法。使用与你的工具匹配的格式：

| 工具 | 语法示例 |
|------|----------------|
| Claude Code | `/opsx:propose`, `/opsx:apply` |
| Cursor | `/opsx-propose`, `/opsx-apply` |
| Windsurf | `/opsx-propose`, `/opsx-apply` |
| Copilot (IDE) | `/opsx-propose`, `/opsx-apply` |
| Kimi CLI | 基于 skill 的调用，如 `/skill:openspec-propose`、`/skill:openspec-apply-change`（不生成 `opsx-*` 命令文件） |
| Trae | 基于 skill 的调用，如 `/openspec-propose`、`/openspec-apply-change`（不生成 `opsx-*` 命令文件） |

所有工具的意图相同，但命令如何呈现可能因集成方式而异。

> **注意：** GitHub Copilot 命令（`.github/prompts/*.prompt.md`）仅在 IDE 扩展中可用（VS Code、JetBrains、Visual Studio）。GitHub Copilot CLI 目前不支持自定义 prompt 文件 — 有关详细信息和解决方法，请参见 [Supported Tools](supported-tools.md)。

---

## 遗留命令

这些命令使用较旧的"一次性"工作流。它们仍然有效，但推荐使用 OPSX 命令。

| 命令 | 执行操作 |
|---------|--------------|
| `/openspec:proposal` | 一次创建所有 artifacts（proposal、specs、design、tasks） |
| `/openspec:apply` | 实现 change |
| `/openspec:archive` | 归档 change |

**何时使用遗留命令：**
- 使用旧工作流的现有项目
- 简单的 changes，你不需要增量 artifact 创建
- 偏好全有或全无方法

**迁移到 OPSX：**
遗留 changes 可以用 OPSX 命令继续。Artifact 结构兼容。

---

## 故障排除

### "Change not found"

命令无法识别要处理的 change。

**解决方案：**
- 显式指定 change 名称：`/opsx:apply add-dark-mode`
- 检查 change 文件夹是否存在：`openspec list`
- 验证你在正确的项目目录中

### "No artifacts ready"

所有 artifacts 要么完成，要么被缺失的依赖阻止。

**解决方案：**
- 运行 `openspec status --change <name>` 查看是什么在阻止
- 检查所需的 artifacts 是否存在
- 先创建缺失的依赖 artifacts

### "Schema not found"

指定的 schema 不存在。

**解决方案：**
- 列出可用 schemas：`openspec schemas`
- 检查 schema 名称的拼写
- 如果是自定义的则创建 schema：`openspec schema init <name>`

### Commands not recognized

AI 工具不识别 OpenSpec 命令。

**解决方案：**
- 确保 OpenSpec 已初始化：`openspec init`
- 重新生成 skills：`openspec update`
- 检查 `.claude/skills/` 目录是否存在（对于 Claude Code）
- 重启你的 AI 工具以获取新的 skills

### Artifacts not generating properly

AI 创建的 artifacts 不完整或不正确。

**解决方案：**
- 在 `openspec/config.yaml` 中添加项目上下文
- 添加特定于 artifacts 的规则以获取具体指导
- 在 change 描述中提供更多细节
- 使用 `/opsx:continue` 而不是 `/opsx:ff` 以获得更多控制

---

## 下一步

- [Workflows](workflows.md) - 常见模式和每种命令的使用场景
- [CLI](cli.md) - 用于管理和验证的终端命令
- [Customization](customization.md) - 创建自定义 schemas 和工作流
