# 迁移到 OPSX

本指南帮助你从遗留 OpenSpec 工作流迁移到 OPSX。迁移设计为平滑的——你现有的工作被保留，新系统提供更多灵活性。

## 有什么变化？

OPSX 用流畅的、基于行动的方法取代了旧的阶段锁定工作流。这是关键转变：

| 方面 | 遗留 | OPSX |
|--------|--------|------|
| **命令** | `/openspec:proposal`、`/openspec:apply`、`/openspec:archive` | 默认：`/opsx:propose`、`/opsx:apply`、`/opsx:sync`、`/opsx:archive`（扩展工作流命令可选） |
| **工作流** | 一次创建所有 artifacts | 增量或一次创建——你的选择 |
| **返回** | 尴尬的阶段门控 | 自然的——随时更新任何 artifact |
| **自定义** | 固定结构 | Schema 驱动，完全可 hack |
| **配置** | 带标记的 `CLAUDE.md` + `project.md` | `openspec/config.yaml` 中的干净配置 |

**理念变化：** 工作不是线性的。OPSX 停止假装它是。

---

## 开始前

### 你现有的工作安全

迁移过程设计时考虑保留：

- **`openspec/changes/` 中的活动 changes** — 完全保留。你可以用 OPSX 命令继续。
- **归档的 changes** — 原封不动。你的历史保持完整。
- **`openspec/specs/` 中的主 specs** — 原封不动。这些是你的真相来源。
- **你在 `CLAUDE.md`、`AGENTS.md` 等中写的内容** — 保留。只移除 OpenSpec 标记块；你写的所有内容保留。

### 什么被移除

只有被替换的 OpenSpec 管理的文件：

| 什么 | 为什么 |
|------|-----|
| 遗留 slash command 目录/文件 | 被新的 skills 系统替换 |
| `openspec/AGENTS.md` | 过时的 workflow 触发器 |
| `CLAUDE.md`、`AGENTS.md` 等中的 OpenSpec 标记 | 不再需要 |

**按工具的遗留命令位置**（示例——你的工具可能不同）：

- Claude Code: `.claude/commands/openspec/`
- Cursor: `.cursor/commands/openspec-*.md`
- Windsurf: `.windsurf/workflows/openspec-*.md`
- Cline: `.clinerules/workflows/openspec-*.md`
- Roo: `.roo/commands/openspec-*.md`
- GitHub Copilot: `.github/prompts/openspec-*.prompt.md`（仅 IDE 扩展；Copilot CLI 不支持）
- 等等（Augment、Continue、Amazon Q 等）

迁移检测你配置的工具并清理它们的遗留文件。

移除列表可能看起来很长，但这些都是 OpenSpec 最初创建的文件。你自己的内容永远不会被删除。

### 需要你关注的内容

一个文件需要手动迁移：

**`openspec/project.md`** — 此文件不会自动删除，因为它可能包含你写的项目上下文。你需要：

1. 审查其内容
2. 将有用的上下文移动到 `openspec/config.yaml`（见下面指导）
3. 准备好时删除文件

**为什么我们做这个改变：**

旧的 `project.md` 是被动的——agents 可能读它，可能不读，可能忘记他们读的内容。我们发现可靠性不一致。

新的 `config.yaml` 上下文**被主动注入到每个 OpenSpec 规划请求中**。这意味着当你创建 artifacts 时，你的项目约定、技术栈和规则始终存在。更高的可靠性。

**权衡：**

因为上下文被注入每个请求，你需要简洁。专注于真正重要的：
- 技术栈和关键约定
- AI 需要知道的非明显约束
- 之前经常被忽略的规则

不要担心让它完美。我们仍在学习什么最有效，我们会随着实验改进上下文注入的工作方式。

---

## 运行迁移

`openspec init` 和 `openspec update` 都检测遗留文件并引导你完成相同的清理过程。使用适合你情况的：

- 新安装默认为 profile `core`（`propose`、`explore`、`apply`、`sync`、`archive`）。
- 迁移的安装通过在需要时写入 `custom` profile 来保留你之前安装的工作流。

### 使用 `openspec init`

如果你想添加新工具或重新配置设置了哪些工具：

```bash
openspec init
```

init 命令检测遗留文件并引导你完成清理：

```
Upgrading to the new OpenSpec:

OpenSpec now uses agent skills, the emerging standard across coding
agents. This simplifies your setup while keeping everything working
as before.

Files to remove
No user content to preserve:
  • .claude/commands/openspec/
  • openspec/AGENTS.md

Files to update
OpenSpec markers will be removed, your content preserved:
  • CLAUDE.md
  • AGENTS.md

Needs your attention
  • openspec/project.md
    We won't delete this file. It may contain useful project context.

    The new openspec/config.yaml has a "context:" section for planning
    context. This is included in every OpenSpec request and works more
    reliably than the old project.md approach.

    Review project.md, move any useful content to config.yaml's context
    section, then delete the file when ready.

? Upgrade and clean up legacy files? (Y/n)
```

**当你说是时发生什么：**

1. 遗留 slash command 目录被移除
2. OpenSpec 标记从 `CLAUDE.md`、`AGENTS.md` 等中剥离（你的内容保留）
3. `openspec/AGENTS.md` 被删除
4. 新 skills 安装在 `.claude/skills/`
5. 使用默认 schema 创建 `openspec/config.yaml`

### 使用 `openspec update`

如果你只想迁移并将现有工具刷新到最新版本：

```bash
openspec update
```

update 命令也检测并清理遗留 artifacts，然后刷新生成的 skills/commands 以匹配你当前的 profile 和 delivery 设置。

### 非交互式 / CI 环境

用于脚本化迁移：

```bash
openspec init --force --tools claude
```

`--force` 标志跳过提示并自动接受清理。

---

## 将 project.md 迁移到 config.yaml

旧的 `openspec/project.md` 是用于项目上下文的自由格式 markdown 文件。新的 `openspec/config.yaml` 是结构化的，关键的是——**注入到每个规划请求**，所以你的约定在 AI 工作时始终存在。

### 之前（project.md）

```markdown
# Project Context

This is a TypeScript monorepo using React and Node.js.
We use Jest for testing and follow strict ESLint rules.
Our API is RESTful and documented in docs/api.md.

## Conventions:

- All public APIs must maintain backwards compatibility
- New features should include tests
- Use Given/When/Then format for specifications
```

### 之后（config.yaml）

```yaml
schema: spec-driven

context: |
  Tech stack: TypeScript, React, Node.js
  Testing: Jest with React Testing Library
  API: RESTful, documented in docs/api.md
  We maintain backwards compatibility for all public APIs

rules:
  proposal:
    - Include rollback plan for risky changes
  specs:
    - Use Given/When/Then format for scenarios
    - Reference existing patterns before inventing new ones
  design:
    - Include sequence diagrams for complex flows
```

### 关键差异

| project.md | config.yaml |
|------------|-------------|
| 自由格式 markdown | 结构化 YAML |
| 一段文本 | 分开的 context 和每-artifact 规则 |
| 不清楚何时使用 | Context 出现在所有 artifacts；规则仅出现在匹配的 artifacts |
| 没有 schema 选择 | 明确的 `schema:` 字段设置默认工作流 |

### 保留什么、放弃什么

迁移时要有选择性。问自己："AI 需要这个用于*每个*规划请求吗？"

**适合 `context:` 的好候选**
- 技术栈（语言、框架、数据库）
- 关键架构模式（monorepo、微服务等）
- 非明显约束（"我们不能使用库 X 因为..."）
- 经常被忽略的关键约定

**改为移动到 `rules:`**
- Artifact 特定格式（"在 specs 中使用 Given/When/Then"）
- 审查标准（"proposals 必须包括回滚计划"）
- 这些仅出现在匹配的 artifacts，保持其他请求更轻

**完全省略**
- AI 已经知道的通用最佳实践
- 可以总结的冗长解释
- 不影响当前工作的历史上下文

### 迁移步骤

1. **创建 config.yaml**（如果 init 尚未创建）：
   ```yaml
   schema: spec-driven
   ```

2. **添加你的上下文**（要简洁——这会进入每个请求）：
   ```yaml
   context: |
     Your project background goes here.
     Focus on what the AI genuinely needs to know.
   ```

3. **添加每-artifact 规则**（可选）：
   ```yaml
   rules:
     proposal:
       - Your proposal-specific guidance
     specs:
       - Your spec-writing rules
   ```

4. **一旦你移动了所有有用的内容就删除 project.md。**

**不要过度思考。** 从要点开始迭代。如果你注意到 AI 遗漏了重要的东西，添加它。如果上下文感觉臃肿，削减它。这是一份活的文档。

### 需要帮助？使用此 Prompt

如果你不确定如何提炼你的 project.md，问你的 AI 助手：

```
I'm migrating from OpenSpec's old project.md to the new config.yaml format.

Here's my current project.md:
[paste your project.md content]

Please help me create a config.yaml with:
1. A concise `context:` section (this gets injected into every planning request, so keep it tight—focus on tech stack, key constraints, and conventions that often get ignored)
2. `rules:` for specific artifacts if any content is artifact-specific (e.g., "use Given/When/Then" belongs in specs rules, not global context)

Leave out anything generic that AI models already know. Be ruthless about brevity.
```

AI 会帮助你识别什么是必要的 vs 什么可以削减。

---

## 新命令

命令可用性依赖于 profile：

**默认（`core` profile）：**

| 命令 | 用途 |
|---------|---------|
| `/opsx:propose` | 一步创建 change 并生成规划 artifacts |
| `/opsx:explore` | 无结构地思考想法 |
| `/opsx:apply` | 实现来自 tasks.md 的 tasks |
| `/opsx:archive` | 完成并归档 change |

**扩展工作流（自定义选择）：**

| 命令 | 用途 |
|---------|---------|
| `/opsx:new` | 开始新的 change 脚手架 |
| `/opsx:continue` | 一次创建一个 artifact |
| `/opsx:ff` | 快速前进——一次创建规划 artifacts |
| `/opsx:verify` | 验证实现与 specs 匹配 |
| `/opsx:sync` | 将 delta specs 合并到主 specs |
| `/opsx:bulk-archive` | 一次归档多个 changes |
| `/opsx:onboard` | 引导端到端入职工作流 |

用 `openspec config profile` 启用扩展命令，然后运行 `openspec update`。

### 命令映射从遗留

| 遗留 | OPSX 等价 |
|--------|-----------------|
| `/openspec:proposal` | `/opsx:propose`（默认）或 `/opsx:new` 然后 `/opsx:ff`（扩展） |
| `/openspec:apply` | `/opsx:apply` |
| `/openspec:archive` | `/opsx:archive` |

### 新能力

这些能力是扩展工作流命令集的一部分。

**细粒度 artifact 创建：**
```
/opsx:continue
```
基于依赖一次创建一个 artifact。在你想审查每一步时使用。

**探索模式：**
```
/opsx:explore
```
在提交 change 前与伙伴一起思考想法。

---

## 理解新架构

### 从阶段锁定到流畅

遗留工作流强制线性进展：

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   PLANNING   │ ───► │ IMPLEMENTING │ ───► │   ARCHIVING  │
│    PHASE     │      │    PHASE     │      │    PHASE     │
└──────────────┘      └──────────────┘      └──────────────┘

If you're in implementation and realize the design is wrong?
Too bad. Phase gates don't let you go back easily.
```

OPSX 使用行动，而不是阶段：

```
         ┌───────────────────────────────────────────────┐
         │           ACTIONS (not phases)                │
         │                                               │
         │     new ◄──► continue ◄──► apply ◄──► archive │
         │      │          │           │             │   │
         │      └──────────┴───────────┴─────────────┘   │
         │                    any order                  │
         └───────────────────────────────────────────────┘
```

### 依赖图

Artifacts 形成有向图。依赖是**启用器**，而不是门：

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
```

当你运行 `/opsx:continue` 时，它检查什么就绪并提供下一个 artifact。你也可以按任何顺序创建多个就绪的 artifacts。

### Skills vs Commands

遗留系统使用工具特定的命令文件：

```
.claude/commands/openspec/
├── proposal.md
├── apply.md
└── archive.md
```

OPSX 使用新兴的 **skills** 标准：

```
.claude/skills/
├── openspec-explore/SKILL.md
├── openspec-new-change/SKILL.md
├── openspec-continue-change/SKILL.md
├── openspec-apply-change/SKILL.md
└── ...
```

Skills 在多个 AI 编程工具中被认可，并提供更丰富的元数据。

---

## 继续现有 Changes

你的进行中的 changes 与 OPSX 命令无缝协作。

**有遗留工作流的进行中 change？**

```
/opsx:apply add-my-feature
```

OPSX 读取现有 artifacts 并从你离开的地方继续。

**想向现有 change 添加更多 artifacts？**

```
/opsx:continue add-my-feature
```

显示基于已存在内容的准备好创建的内容。

**需要查看状态？**

```bash
openspec status --change add-my-feature
```

---

## 新配置系统

### config.yaml 结构

```yaml
# 必填：新 changes 的默认 schema
schema: spec-driven

# 可选：项目上下文（最大 50KB）
# 注入到所有 artifact 指令
context: |
  Your project background, tech stack,
  conventions, and constraints.

# 可选：每-artifact 规则
# 仅注入到匹配的 artifacts
rules:
  proposal:
    - Include rollback plan
  specs:
    - Use Given/When/Then format
  design:
    - Document fallback strategies
  tasks:
    - Break into 2-hour maximum chunks
```

### Schema 解析

确定使用哪个 schema 时，OPSX 按顺序检查：

1. **CLI 标志**：`--schema <name>`（最高优先级）
2. **Change 元数据**：change 目录中的 `.openspec.yaml`
3. **项目配置**：`openspec/config.yaml`
4. **默认值**：`spec-driven`

### 可用 Schemas

| Schema | Artifacts | 适合 |
|--------|-----------|----------|
| `spec-driven` | proposal → specs → design → tasks | 大多数项目 |

列出所有可用 schemas：

```bash
openspec schemas
```

### 自定义 Schemas

创建你自己的工作流：

```bash
openspec schema init my-workflow
```

或 fork 一个现有的：

```bash
openspec schema fork spec-driven my-workflow
```

详情请参见 [Customization](customization.md)。

---

## 故障排除

### "Legacy files detected in non-interactive mode"

你在 CI 或非交互环境中运行。使用：

```bash
openspec init --force
```

### 迁移后命令不出现

重启你的 IDE。Skills 在启动时检测。

### "Unknown artifact ID in rules"

检查你的 `rules:` 键是否匹配 schema 的 artifact IDs：

- **spec-driven**：`proposal`、`specs`、`design`、`tasks`

运行此命令查看有效的 artifact IDs：

```bash
openspec schemas --json
```

### 配置未应用

1. 确保文件在 `openspec/config.yaml`（不是 `.yml`）
2. 验证 YAML 语法
3. 配置更改立即生效——无需重启

### project.md 未迁移

系统故意保留 `project.md` 因为它可能包含你的自定义内容。手动审查，将有用的部分移动到 `config.yaml`，然后删除它。

### 想看看会清理什么？

运行 init 并拒绝清理提示——你会看到完整检测摘要，而不进行任何更改。

---

## 快速参考

### 迁移后的文件

```
project/
├── openspec/
│   ├── specs/                    # 不变
│   ├── changes/                  # 不变
│   │   └── archive/              # 不变
│   └── config.yaml               # 新：项目配置
├── .claude/
│   └── skills/                   # 新：OPSX skills
│       ├── openspec-propose/     # default core profile
│       ├── openspec-explore/
│       ├── openspec-apply-change/
│       ├── openspec-sync-specs/
│       └── ...                   # expanded profile adds new/continue/ff/etc.
├── CLAUDE.md                     # OpenSpec 标记移除，你的内容保留
└── AGENTS.md                     # OpenSpec 标记移除，你的内容保留
```

### 什么消失了

- `.claude/commands/openspec/` — 被 `.claude/skills/` 替换
- `openspec/AGENTS.md` — 过时
- `openspec/project.md` — 迁移到 `config.yaml`，然后删除
- `CLAUDE.md`、`AGENTS.md` 等中的 OpenSpec 标记块

### 命令速查

```text
/opsx:propose      快速开始（默认 core profile）
/opsx:apply        实现 tasks
/opsx:archive      完成并归档

# 扩展工作流（如果启用）：
/opsx:new          脚手架一个 change
/opsx:continue     创建下一个 artifact
/opsx:ff           创建规划 artifacts
```

---

## 获取帮助

- **Discord**: [discord.gg/YctCnvvshC](https://discord.gg/YctCnvvshC)
- **GitHub Issues**: [github.com/Fission-AI/OpenSpec/issues](https://github.com/Fission-AI/OpenSpec/issues)
- **文档**: 完整 OPSX 参考请参见 [docs/opsx.md](opsx.md)
