# Customization

OpenSpec 提供三个级别的自定义：

| 级别 | 功能 | 适合 |
|-------|--------------|----------|
| **Project Config** | 设置默认值，注入上下文/规则 | 大多数团队 |
| **Custom Schemas** | 定义你自己的工作流 artifacts | 有独特流程的团队 |
| **Global Overrides** | 跨所有项目共享 schemas | 高级用户 |

---

## 项目配置

`openspec/config.yaml` 文件是为团队自定义 OpenSpec 的最简单方式。它允许你：

- **设置默认 schema** - 跳过每个命令的 `--schema`
- **注入项目上下文** - AI 看到你的技术栈、约定等
- **添加每-artifact 规则** - 针对特定 artifacts 的自定义规则

### 快速设置

```bash
openspec init
```

这将引导你交互式创建配置。或者手动创建一个：

```yaml
# openspec/config.yaml
schema: spec-driven

context: |
  Tech stack: TypeScript, React, Node.js, PostgreSQL
  API style: RESTful, documented in docs/api.md
  Testing: Jest + React Testing Library
  We value backwards compatibility for all public APIs

rules:
  proposal:
    - Include rollback plan
    - Identify affected teams
  specs:
    - Use Given/When/Then format
    - Reference existing patterns before inventing new ones
```

### 工作原理

**默认 schema：**

```bash
# 没有 config
openspec new change my-feature --schema spec-driven

# 有 config - schema 是自动的
openspec new change my-feature
```

**上下文和规则注入：**

生成任何 artifact 时，你的上下文和规则被注入 AI prompt：

```xml
<context>
Tech stack: TypeScript, React, Node.js, PostgreSQL
...
</context>

<rules>
- Include rollback plan
- Identify affected teams
:</rules>

<template>
[Schema's built-in template]
</template>
```

- **Context** 出现在所有 artifacts 中
- **Rules** 仅出现在匹配的 artifacts 中

### Schema 解析顺序

当 OpenSpec 需要 schema 时，按此顺序检查：

1. CLI 标志：`--schema <name>`
2. Change 元数据（`.openspec.yaml` 在 change 文件夹中）
3. 项目配置（`openspec/config.yaml`）
4. 默认值（`spec-driven`）

---

## 自定义 Schemas

当项目配置不够时，创建具有完全自定义工作流你自己的 schema。自定义 schemas 住在项目的 `openspec/schemas/` 目录中，与你的代码一起版本控制。

```text
your-project/
├── openspec/
│   ├── config.yaml        # Project config
│   ├── schemas/           # Custom schemas live here
│   │   └── my-workflow/
│   │       ├── schema.yaml
│   │       └── templates/
│   └── changes/           # Your changes
└── src/
```

### Fork 现有 Schema

自定义最快的方式是 fork 内置 schema：

```bash
openspec schema fork spec-driven my-workflow
```

这将整个 `spec-driven` schema 复制到 `openspec/schemas/my-workflow/`，你可以自由编辑。

**你获得的内容：**

```text
openspec/schemas/my-workflow/
├── schema.yaml           # Workflow definition
└── templates/
    ├── proposal.md       # Template for proposal artifact
    ├── spec.md           # Template for specs
    ├── design.md         # Template for design
    └── tasks.md          # Template for tasks
```

现在编辑 `schema.yaml` 更改工作流，或编辑模板以更改 AI 生成的内容。

### 从零创建 Schema

对于完全新鲜的工作流：

```bash
# 交互式
openspec schema init research-first

# 非交互式
openspec schema init rapid \
  --description "Rapid iteration workflow" \
  --artifacts "proposal,tasks" \
  --default
```

### Schema 结构

Schema 定义工作流中的 artifacts 及其相互依赖：

```yaml
# openspec/schemas/my-workflow/schema.yaml
name: my-workflow
version: 1
description: My team's custom workflow

artifacts:
  - id: proposal
    generates: proposal.md
    description: Initial proposal document
    template: proposal.md
    instruction: |
      Create a proposal that explains WHY this change is needed.
      Focus on the problem, not the solution.
    requires: []

  - id: design
    generates: design.md
    description: Technical design
    template: design.md
    instruction: |
      Create a design document explaining HOW to implement.
    requires:
      - proposal    # Can't create design until proposal exists

  - id: tasks
    generates: tasks.md
    description: Implementation checklist
    template: tasks.md
    requires:
      - design

apply:
  requires: [tasks]
  tracks: tasks.md
```

**关键字段：**

| 字段 | 用途 |
|-------|---------|
| `id` | 唯一标识符，用于命令和规则 |
| `generates` | 输出文件名（支持 glob 如 `specs/**/*.md`） |
| `template` | `templates/` 目录中的模板文件 |
| `instruction` | 创建此 artifact 的 AI 指令 |
| `requires` | 依赖 - 哪些 artifacts 必须先存在 |

### 模板

模板是指导 AI 的 markdown 文件。在创建该 artifact 时注入到 prompt 中。

```markdown
<!-- templates/proposal.md -->
## Why:

<!-- Explain the motivation for this change. What problem does this solve? -->

## What Changes:

<!-- Describe what will change. Be specific about new capabilities or modifications. -->

## Impact:

<!-- Affected code, APIs, dependencies, systems -->
```

模板可以包括：
- AI 应该填写的章节标题
- 为 AI 提供指导的 HTML 注释
- 显示预期结构的示例格式

### 验证你的 Schema

使用自定义 schema 前，验证它：

```bash
openspec schema validate my-workflow
```

这检查：
- `schema.yaml` 语法正确
- 所有引用的模板存在
- 没有循环依赖
- Artifact IDs 有效

### 使用你的自定义 Schema

创建后，使用你的 schema：

```bash
# 在命令上指定
openspec new change feature --schema my-workflow

# 或在 config.yaml 中设置为默认值
schema: my-workflow
```

### 调试 Schema 解析

不确定使用哪个 schema？检查：

```bash
# 查看特定 schema 从哪里解析
openspec schema which my-workflow

# 列出所有可用 schemas
openspec schema which --all
```

输出显示它是来自你的项目、用户目录还是包：

```text
Schema: my-workflow
Source: project
Path: /path/to/project/openspec/schemas/my-workflow
```

---

> **注意：** OpenSpec 还支持用户级 schemas 在 `~/.local/share/openspec/schemas/` 用于跨项目共享，但建议使用 `openspec/schemas/` 中的项目级 schemas，因为它们与你的代码一起版本控制。

---

## 示例

### 快速迭代工作流

用于快速迭代的最小工作流：

```yaml
# openspec/schemas/rapid/schema.yaml
name: rapid
version: 1
description: Fast iteration with minimal overhead

artifacts:
  - id: proposal
    generates: proposal.md
    description: Quick proposal
    template: proposal.md
    instruction: |
      Create a brief proposal for this change.
      Focus on what and why, skip detailed specs.
    requires: []

  - id: tasks
    generates: tasks.md
    description: Implementation checklist
    template: tasks.md
    requires: [proposal]

apply:
  requires: [tasks]
  tracks: tasks.md
```

### 添加 Review Artifact

Fork 默认并添加审查步骤：

```bash
openspec schema fork spec-driven with-review
```

然后编辑 `schema.yaml` 添加：

```yaml
  - id: review
    generates: review.md
    description: Pre-implementation review checklist
    template: review.md
    instruction: |
      Create a review checklist based on the design.
      Include security, performance, and testing considerations.
    requires:
      - design

  - id: tasks
    # ... existing tasks config ...
    requires:
      - specs
      - design
      - review    # Now tasks require review too
```

---

## 社区 Schemas

OpenSpec 还支持通过独立仓库分发的社区维护 schemas。这些提供了将 OpenSpec 与其他工具或系统集成的个性化工作流，类似于 [github/spec-kit's community extension catalog](https://github.com/github/spec-kit/tree/main/extensions) 为 spec-kit 处理工具集成的方式。

社区 schemas 不是 vendored 到 OpenSpec core——它们住在自己的仓库中，有自己的发布节奏。要使用一个，将 schema bundle 复制到你的项目的 `openspec/schemas/<schema-name>/` 目录中（每个仓库的 README 有安装说明）。

| Schema | 维护者 | 仓库 | 描述 |
|--------|-----------|-----------|-------------|
| `superpowers-bridge` | @JiangWay | [JiangWay/openspec-schemas](https://github.com/JiangWay/openspec-schemas/tree/main/superpowers-bridge) | 将 OpenSpec 的 artifact 治理与 [obra/superpowers](https://github.com/obra/superpowers) 执行技能（头脑风暴、写作计划、通过 subagents 进行 TDD、代码审查、完成）集成。添加了 evidence-first `retrospective` artifact 来填补 Superpowers 原生不覆盖的空白。 |

> 想贡献社区 schema？打开一个 issue 并链接到你的仓库，或提交 PR 在此表中添加一行。

---

## 另见

- [CLI Reference: Schema Commands](cli.md#schema-commands) - 完整命令文档
