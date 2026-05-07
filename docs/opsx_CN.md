# OPSX Workflow

> 欢迎在 [Discord](https://discord.gg/YctCnvvshC) 反馈。

## 这是什么？

OPSX 现在是 OpenSpec 的标准工作流。

这是一个用于 OpenSpec changes 的**流畅、迭代的工作流**。不再有刻板的阶段——只是你可以随时采取的行动。

## 为什么存在

遗留 OpenSpec 工作流有效，但它是**锁定的**：

- **指令硬编码** — 埋在 TypeScript 中，你不能改变
- **全有或全无** — 一个大命令创建所有内容，不能测试单个部分
- **固定结构** — 每个人相同的工作流，无法自定义
- **黑盒** — 当 AI 输出糟糕时，你不能调整 prompts

**OPSX 开放了它。** 现在任何人都可以：

1. **尝试指令** — 编辑模板，看看 AI 是否做得更好
2. **细粒度测试** — 独立验证每个 artifact 的指令
3. **自定义工作流** — 定义你自己的 artifacts 和依赖
4. **快速迭代** — 更改模板，立即测试，无需重建

```
Legacy workflow:                      OPSX:
┌────────────────────────┐           ┌────────────────────────┐
│  Hardcoded in package  │           │  schema.yaml           │◄── You edit this
│  (can't change)        │           │  templates/*.md        │◄── Or this
│        ↓               │           │        ↓               │
│  Wait for new release   │           │  Instant effect        │
│        ↓               │           │        ↓               │
│  Hope it's better      │           │  Test it yourself      │
└────────────────────────┘           └────────────────────────┘
```

**这适合所有人：**
- **团队** — 创建与你实际工作方式匹配的工作流
- **高级用户** — 调整 prompts 以获得更好的 AI 输出到你的代码库
- **OpenSpec 贡献者** — 无需发布即可尝试新方法

我们仍在学习什么最有效。OPSX 让我们一起学习。

## 用户体验

**线性工作流的问题：**
你是"在规划阶段"，然后"在实现阶段"，然后"完成"。但实际工作不是这样运作的。你实现了一些东西，意识到你的设计是错误的，需要更新 specs，继续实现。线性阶段违背了工作实际发生的方式。

**OPSX 方法：**
- **行动，而不是阶段** — 创建、实现、更新、归档——随时做任何一个
- **依赖是启用器** — 它们显示什么是可能的，而不是下一步需要什么

```
  proposal ──→ specs ──→ design ──→ tasks ──→ implement
```

## 设置

```bash
# 确保你安装了 openspec — skills 自动生成
openspec init
```

这在 `.claude/skills/`（或等效位置）中创建 AI 编程助手自动检测的 skills。

默认情况下，OpenSpec 使用 `core` 工作流 profile（`propose`、`explore`、`apply`、`sync`、`archive`）。如果你想要扩展工作流命令（`new`、`continue`、`ff`、`verify`、`bulk-archive`、`onboard`），用 `openspec config profile` 配置，然后用 `openspec update` 应用。

在设置期间，会提示你创建**项目配置**（`openspec/config.yaml`）。这是可选的但推荐。

## 项目配置

项目配置让你设置默认值并将项目特定上下文注入所有 artifacts。

### 创建配置

配置在 `openspec init` 期间创建，或手动：

```yaml
# openspec/config.yaml
schema: spec-driven

context: |
  Tech stack: TypeScript, React, Node.js
  API conventions: RESTful, JSON responses
  Testing: Vitest for unit tests, Playwright for e2e
  Style: ESLint with Prettier, strict TypeScript

rules:
  proposal:
    - Include rollback plan
    - Identify affected teams
  specs:
    - Use Given/When/Then format for scenarios
  design:
    - Include sequence diagrams for complex flows
```

### 配置字段

| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `schema` | string | 新 changes 的默认 schema（如 `spec-driven`） |
| `context` | string | 注入到所有 artifact 指令的项目上下文 |
| `rules` | object | 按 artifact ID 键控的每-artifact 规则 |

### 工作原理

**Schema 优先级**（从高到低）：
1. CLI 标志（`--schema <name>`）
2. Change 元数据（change 目录中的 `.openspec.yaml`）
3. 项目配置（`openspec/config.yaml`）
4. 默认值（`spec-driven`）

**上下文注入：**
- 上下文被预置到每个 artifact 的指令
- 包装在 `<context>...</context>` 标签中
- 帮助 AI 理解你项目的约定

**规则注入：**
- 规则仅注入匹配的 artifacts
- 包装在 `<rules>...</rules>` 标签中
- 出现在上下文之后，模板之前

### 按 Schema 的 Artifact IDs

**spec-driven**（默认）：
- `proposal` — Change proposal
- `specs` — Specifications
- `design` — Technical design
- `tasks` — Implementation tasks

### 配置验证

- `rules` 中的未知 artifact IDs 生成警告
- Schema 名称根据可用 schemas 验证
- Context 有 50KB 大小限制
- 无效 YAML 带有行号报告

### 故障排除

**"rules 中的未知 artifact ID: X"**
- 检查 artifact IDs 匹配你的 schema（见上表）
- 运行 `openspec schemas --json` 查看每个 schema 的 artifact IDs

**配置未应用：**
- 确保文件在 `openspec/config.yaml`（不是 `.yml`）
- 用验证器检查 YAML 语法
- 配置更改立即生效（无需重启）

**Context 太大：**
- Context 限制为 50KB
- 改为总结或链接到外部文档

## 命令

| 命令 | 功能 |
|---------|--------------|
| `/opsx:propose` | 一步创建 change 并生成规划 artifacts（默认快速路径） |
| `/opsx:explore` | 思考想法、调查问题、澄清需求 |
| `/opsx:new` | 开始新的 change 脚手架（扩展工作流） |
| `/opsx:continue` | 创建下一个 artifact（扩展工作流） |
| `/opsx:ff` | 快速前进规划 artifacts（扩展工作流） |
| `/opsx:apply` | 实现 tasks，根据需要更新 artifacts |
| `/opsx:verify` | 根据 artifacts 验证实现（扩展工作流） |
| `/opsx:sync` | 将 delta specs 同步到主（默认工作流，可选） |
| `/opsx:archive` | 归档完成时 |
| `/opsx:bulk-archive` | 一次归档多个已完成的 changes（扩展工作流） |
| `/opsx:onboard` | 端到端 change 的引导演练（扩展工作流） |

## 使用

### 探索一个想法
```
/opsx:explore
```
思考想法、调查问题、比较选项。无需结构——只是一个思考伙伴。当洞察清晰时，过渡到 `/opsx:propose`（默认）或 `/opsx:new`/`/opsx:ff`（扩展）。

### 开始新 change
```
/opsx:propose
```
创建 change 并生成实现前所需的规划 artifacts。

如果你已启用扩展工作流，你可以改为使用：

```text
/opsx:new        # 仅脚手架
/opsx:continue   # 一次创建一个 artifact
/opsx:ff         # 一次创建所有规划 artifacts
```

### 创建 artifacts
```
/opsx:continue
```
显示基于依赖准备好创建的内容，然后创建一个 artifact。重复使用以增量构建你的 change。

```
/opsx:ff add-dark-mode
```
一次创建所有规划 artifacts。在你清楚要构建什么时使用。

### 实现（流畅的部分）
```
/opsx:apply
```
处理 tasks，边走边勾选。如果你在处理多个 changes，你可以运行 `/opsx:apply <name>`；否则它应该从对话中推断，如果无法判断则提示你选择。

### 完成
```
/opsx:archive   # 完成后移动到归档（如果需要则提示同步 specs）
```

## 何时更新 vs 从头开始

你总是可以在实现前编辑你的 proposal 或 specs。但什么时候完善变成"这是不同的工作"？

### Proposal 捕获什么

Proposal 定义三件事：
1. **意图** — 你在解决什么问题？
2. **范围** — 什么在/不在范围内？
3. **方法** — 你将如何解决它？

问题是：哪个改变了，改变了多少？

### 更新现有 Change 当：

**相同意图，改进执行**
- 你发现了没有考虑到的 edge cases
- 方法需要调整但目标不变
- 实现揭示设计稍微不对

**范围缩小**
- 你意识到完整范围太大，想先发布 MVP
- "添加深色模式" → "添加深色模式切换（v2 中的系统偏好）"

**学习驱动的修正**
- 代码库结构不是你想象的那样
- 一个依赖不如预期工作
- "使用 CSS 变量" → "改为使用 Tailwind 的 dark: 前缀"

### 开始新 Change 当：

**意图根本改变**
- 问题本身现在不同了
- "添加深色模式" → "添加包含自定义颜色、字体、间距的综合主题系统"

**范围爆炸**
- Change 增长如此之多，它本质上是不同的工作
- 原始 proposal 更新后将无法识别
- "修复登录 bug" → "重写 auth 系统"

**原版可完成**
- 原始 change 可以标记为"完成"
- 新工作独立存在，不是完善
- 完成"添加深色模式 MVP" → 归档 → 新 change"增强深色模式"

### 启发法

```
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

| 测试 | 更新 | 新 Change |
|------|--------|------------|
| **身份** | "同样的事情，完善" | "不同的工作" |
| **范围重叠** | >50% 重叠 | <50% 重叠 |
| **完成** | 没有更改无法"完成" | 可以完成原始的，新工作独立存在 |
| **故事** | 更新链讲述连贯故事 | 补丁会比澄清更混乱 |

### 原则

> **更新保留上下文。新 change 提供清晰。**
>
> 当你的思考历史有价值时选择更新。
> 当重新开始比补丁更清晰时选择新。

把它想象成 git branches：
- 在同一功能上工作时继续提交
- 当确实是新工作时开始新分支
- 有时合并部分功能并为第二阶段重新开始

## 有什么不同？

| | 遗留（`/openspec:proposal`） | OPSX（`/opsx:*`） |
|---|---|---|
| **结构** | 一个大 proposal 文档 | 带依赖的离散 artifacts |
| **工作流** | 线性阶段：plan → implement → archive | 流畅行动——随时做 |
| **迭代** | 回去很尴尬 | 随学习更新 artifacts |
| **自定义** | 固定结构 | Schema 驱动（定义你自己的 artifacts） |

**关键洞察：** 工作不是线性的。OPSX 停止假装它是。

## 架构深度解析

本节解释 OPSX 如何在幕后工作以及它与遗留工作流的比较。
本节示例使用扩展命令集（`new`、`continue` 等）；默认 `core` 用户可以将相同流程映射到 `propose → apply → sync → archive`。

### 理念：阶段 vs 行动

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         LEGACY WORKFLOW                                      │
│                    (Phase-Locked, All-or-Nothing)                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌──────────────┐      ┌──────────────┐      ┌──────────────┐             │
│   │   PLANNING   │ ───► │ IMPLEMENTING │ ───► │   ARCHIVING  │             │
│   │    PHASE     │      │    PHASE     │      │    PHASE     │             │
│   └──────────────┘      └──────────────┘      └──────────────┘             │
│         │                     │                     │                       │
│         ▼                     ▼                     ▼                       │
│   /openspec:proposal   /openspec:apply      /openspec:archive              │
│                                                                             │
│   • Creates ALL artifacts at once                                          │
│   • Can't go back to update specs during implementation                    │
│   • Phase gates enforce linear progression                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────────────┐
│                            OPSX WORKFLOW                                     │
│                      (Fluid Actions, Iterative)                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│              ┌────────────────────────────────────────────┐                 │
│              │           ACTIONS (not phases)             │                 │
│              │                                            │                 │
│              │   new ◄──► continue ◄──► apply ◄──► archive │                 │
│              │    │          │           │           │    │                 │
│              │    └──────────┴───────────┴───────────┘    │                 │
│              │              any order                     │                 │
│              └────────────────────────────────────────────┘                 │
│                                                                             │
│   • Create artifacts one at a time OR fast-forward                         │
│   • Update specs/design/tasks during implementation                        │
│   • Dependencies enable progress, phases don't exist                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 组件架构

**遗留工作流** 在 TypeScript 中使用硬编码模板：

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      LEGACY WORKFLOW COMPONENTS                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Hardcoded Templates (TypeScript strings)                                  │
│                    │                                                        │
│                    ▼                                                        │
│   Tool-specific configurators/adapters                                      │
│                    │                                                        │
│                    ▼                                                        │
│   Generated Command Files (.claude/commands/openspec/*.md)                  │
│                                                                             │
│   • Fixed structure, no artifact awareness                                  │
│   • Change requires code modification + rebuild                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**OPSX** 使用外部 schemas 和依赖图引擎：

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         OPSX COMPONENTS                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Schema Definitions (YAML)                                                 │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  name: spec-driven                                                  │   │
│   │  artifacts:                                                         │   │
│   │    - id: proposal                                                   │   │
│   │      generates: proposal.md                                         │   │
│   │      requires: []              ◄── Dependencies                     │   │
│   │    - id: specs                                                      │   │
│   │      generates: specs/**/*.md  ◄── Glob patterns                    │   │
│   │      requires: [proposal]      ◄── Enables after proposal           │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                    │                                                        │
│                    ▼                                                        │
│   Artifact Graph Engine                                                     │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  • Topological sort (dependency ordering)                           │   │
│   │  • State detection (filesystem existence)                           │   │
│   │  • Rich instruction generation (templates + context)                │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                    │                                                        │
│                    ▼                                                        │
│   Skill Files (.claude/skills/openspec-*/SKILL.md)                          │
│                                                                             │
│   • Cross-editor compatible (Claude Code, Cursor, Windsurf)                 │
│   • Skills query CLI for structured data                                    │
│   • Fully customizable via schema files                                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 依赖图模型

Artifacts 形成有向无环图（DAG）。依赖是**启用器**，而不是门：

```
                              proposal
                             (root node)
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
                 specs                       design
              (requires:                  (requires:
               proposal)                   proposal)
                    │                           │
                    └─────────────┬─────────────┘
                                  │
                                  ▼
                               tasks
                           (requires:
                           specs, design)
                                  │
                                  ▼
                          ┌──────────────┐
                          │ APPLY PHASE  │
                          │ (requires:   │
                          │  tasks)      │
                          └──────────────┘
```

**状态转换：**

```
   BLOCKED ────────────────► READY ────────────────► DONE
      │                        │                       │
   Missing                  All deps               File exists
   dependencies             are DONE               on filesystem
```

### 信息流

**遗留工作流** — agent 接收静态指令：

```
  User: "/openspec:proposal"
           │
           ▼
  ┌─────────────────────────────────────────┐
  │  Static instructions:                   │
  │  • Create proposal.md                   │
  │  • Create tasks.md                      │
  │  • Create design.md                     │
  │  • Create specs/<capability>/spec.md    │
  │                                         │
  │  No awareness of what exists or         │
  │  dependencies between artifacts         │
  └─────────────────────────────────────────┘
           │
           ▼
  Agent creates ALL artifacts in one go
```

**OPSX** — agent 查询丰富上下文：

```
  User: "/opsx:continue"
           │
           ▼
  ┌──────────────────────────────────────────────────────────────────────────┐
  │  Step 1: Query current state                                             │
  │  ┌────────────────────────────────────────────────────────────────────┐  │
  │  │  $ openspec status --change "add-auth" --json                      │  │
  │  │                                                                    │  │
  │  │  {                                                                 │  │
  │  │    "artifacts": [                                                  │  │
  │  │      {"id": "proposal", "status": "done"},                         │  │
  │  │      {"id": "specs", "status": "ready"},      ◄── First ready      │  │
  │  │      {"id": "design", "status": "ready"},                          │  │
  │  │      {"id": "tasks", "status": "blocked", "missingDeps": ["specs"]}│  │
  │  │    ]                                                               │  │
  │  │  }                                                                 │  │
  │  └────────────────────────────────────────────────────────────────────┘  │
  │                                                                          │  │
  │  Step 2: Get rich instructions for ready artifact                        │
  │  ┌────────────────────────────────────────────────────────────────────┐  │
  │  │  $ openspec instructions specs --change "add-auth" --json          │  │
  │  │                                                                    │  │
  │  │  {                                                                 │  │
  │  │    "template": "# Specification\n\n## ADDED Requirements...",      │  │
  │  │    "dependencies": [{"id": "proposal", "path": "...", "done": true}│  │
  │  │    "unlocks": ["tasks"]                                            │  │
  │  │  }                                                                 │  │
  │  └────────────────────────────────────────────────────────────────────┘  │
  │                                                                          │  │
  │  Step 3: Read dependencies → Create ONE artifact → Show what's unlocked  │
  └──────────────────────────────────────────────────────────────────────────┘
```

### 迭代模型

**遗留工作流** — 迭代尴尬：

```
  ┌─────────┐     ┌─────────┐     ┌─────────┐
  │/proposal│ ──► │ /apply  │ ──► │/archive │
  └─────────┘     └─────────┘     └─────────┘
       │               │
       │               ├── "Wait, the design is wrong"
       │               │
       │               ├── Options:
       │               │   • Edit files manually (breaks context)
       │               │   • Abandon and start over
       │               │   • Push through and fix later
       │               │
       │               └── No official "go back" mechanism
       │
       └── Creates ALL artifacts at once
```

**OPSX** — 自然迭代：

```
  /opsx:new ───► /opsx:continue ───► /opsx:apply ───► /opsx:archive
      │                │                  │
      │                │                  ├── "The design is wrong"
      │                │                  │
      │                │                  ▼
      │                │            Just edit design.md
      │                │            and continue!
      │                │                  │
      │                │                  ▼
      │                │         /opsx:apply picks up
      │                │         where you left off
      │                │
      │                └── Creates ONE artifact, shows what's unlocked
      │
      └── Scaffolds change, waits for direction
```

### 自定义 Schemas

使用 schema 管理命令创建自定义工作流：

```bash
# 从零开始创建新 schema（交互式）
openspec schema init my-workflow

# 或 fork 现有 schema 作为起点
openspec schema fork spec-driven my-workflow

# 验证 schema 结构后再使用
openspec schema validate my-workflow

# 查看 schema 从哪里解析（用于调试）
openspec schema which my-workflow
```

Schemas 存储在 `openspec/schemas/`（项目本地，版本控制）或 `~/.local/share/openspec/schemas/`（用户全局）。

**Schema 结构：**
```
openspec/schemas/research-first/
├── schema.yaml
└── templates/
    ├── research.md
    ├── proposal.md
    └── tasks.md
```

**示例 schema.yaml：**
```yaml
name: research-first
artifacts:
  - id: research        # Added before proposal
    generates: research.md
    requires: []

  - id: proposal
    generates: proposal.md
    requires: [research]  # Now depends on research

  - id: tasks
    generates: tasks.md
    requires: [proposal]
```

**依赖图：**
```
   research ──► proposal ──► tasks
```

### 总结

| 方面 | 遗留 | OPSX |
|--------|----------|------|
| **模板** | 硬编码 TypeScript | 外部 YAML + Markdown |
| **依赖** | 无（一次全部） | 带拓扑排序的 DAG |
| **状态** | 基于阶段的心智模型 | 文件系统存在性 |
| **自定义** | 编辑源，重建 | 创建 schema.yaml |
| **迭代** | 阶段锁定 | 流畅，编辑任何东西 |
| **编辑器支持** | 工具特定的 configurator/adapters | 单一 skills 目录 |

## Schemas

Schemas 定义存在哪些 artifacts 及其依赖。当前可用：

- **spec-driven**（默认）：proposal → specs → design → tasks

```bash
# 列出可用 schemas
openspec schemas

# 查看所有 schemas 及其解析来源
openspec schema which --all

# 交互式创建新 schema
openspec schema init my-workflow

# Fork 现有 schema 用于自定义
openspec schema fork spec-driven my-workflow

# 使用前验证 schema 结构
openspec schema validate my-workflow
```

## 提示

- 在提交 change 前用 `/opsx:explore` 思考想法
- 当你知道想要什么时用 `/opsx:ff`，当探索时用 `/opsx:continue`
- 在 `/opsx:apply` 期间，如果有问题——修复 artifact，然后继续
- Tasks 通过 `tasks.md` 中的复选框跟踪进度
- 随时检查状态：`openspec status --change "name"`

## 反馈

这是粗糙的。这是故意的——我们正在学习什么有效。

发现 bug？有想法？加入我们的 [Discord](https://discord.gg/YctCnvvshC) 或在 [GitHub](https://github.com/Fission-AI/openspec/issues) 打开 issue。
