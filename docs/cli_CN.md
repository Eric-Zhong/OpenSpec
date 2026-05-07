# CLI 参考文档

OpenSpec CLI（`openspec`）提供终端命令，用于项目设置、验证、状态检查和管理。这些命令补充了 [Commands](commands.md) 中记录的 AI slash commands（如 `/opsx:propose`）。

## 概览

| 类别 | 命令 | 用途 |
|----------|----------|--------|
| **设置** | `init`, `update` | 在项目中初始化和更新 OpenSpec |
| **工作区（测试版）** | `workspace setup`, `workspace list`, `workspace ls`, `workspace link`, `workspace relink`, `workspace doctor`, `workspace open` | 设置跨链接仓库或文件夹的规划 |
| **浏览** | `list`, `view`, `show` | 浏览 changes 和 specs |
| **验证** | `validate` | 检查 changes 和 specs 的问题 |
| **生命周期** | `archive` | 完成已完成的 changes |
| **工作流** | `status`, `instructions`, `templates`, `schemas` | Artifact 驱动的工作流支持 |
| **Schemas** | `schema init`, `schema fork`, `schema validate`, `schema which` | 创建和管理自定义工作流 |
| **配置** | `config` | 查看和修改设置 |
| **工具** | `feedback`, `completion` | 反馈和 shell 集成 |

---

## 人类 vs Agent 命令

大多数 CLI 命令设计为**人类在终端使用**。某些命令也支持通过 JSON 输出用于 **agent/script 使用**。

### 仅人类命令

这些命令是交互式的，设计用于终端使用：

| 命令 | 用途 |
|---------|---------|
| `openspec init` | 初始化项目（交互式提示） |
| `openspec view` | 交互式仪表板 |
| `openspec config edit` | 在编辑器中打开配置 |
| `openspec feedback` | 通过 GitHub 提交反馈 |
| `openspec completion install` | 安装 shell 自动补全 |

### 支持 Agent 的命令

这些命令支持 `--json` 输出，用于 AI agents 和 scripts 的编程使用：

| 命令 | 人类用途 | Agent 用途 |
|---------|-----------|-----------|
| `openspec list` | 浏览 changes/specs | `--json` 获取结构化数据 |
| `openspec show <item>` | 读取内容 | `--json` 用于解析 |
| `openspec validate` | 检查问题 | `--all --json` 用于批量验证 |
| `openspec status` | 查看 artifact 进度 | `--json` 获取结构化状态 |
| `openspec instructions` | 获取下一步骤 | `--json` 获取 agent 指令 |
| `openspec templates` | 查找模板路径 | `--json` 用于路径解析 |
| `openspec schemas` | 列出可用 schemas | `--json` 用于 schema 发现 |
| `openspec workspace setup --no-interactive` | 使用明确输入创建工作区 | `--json` 获取结构化设置输出 |
| `openspec workspace list` | 浏览已知工作区 | `--json` 获取类型化工作区对象 |
| `openspec workspace link` | 链接仓库或文件夹 | `--json` 获取结构化链接输出 |
| `openspec workspace relink` | 修复链接路径 | `--json` 获取结构化链接输出 |
| `openspec workspace doctor` | 检查一个工作区 | `--json` 获取结构化状态输出 |

---

## 全局选项

这些选项适用于所有命令：

| 选项 | 描述 |
|--------|-------------|
| `--version`, `-V` | 显示版本号 |
| `--no-color` | 禁用彩色输出 |
| `--help`, `-h` | 显示命令帮助 |

---

## 设置命令

### `openspec init`

在项目中初始化 OpenSpec。创建文件夹结构并配置 AI 工具集成。

默认行为使用全局配置默认值：profile `core`，delivery `both`，workflows `propose, explore, apply, sync, archive`。

```
openspec init [path] [options]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `path` | 否 | 目标目录（默认：当前目录） |

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--tools <list>` | 非交互式配置 AI 工具。使用 `all`、`none` 或逗号分隔的列表 |
| `--force` | 自动清理遗留文件，不提示 |
| `--profile <profile>` | 覆盖此 init 运行的全局 profile（`core` 或 `custom`） |

`--profile custom` 使用当前在全局配置中选择的 workflows（`openspec config profile`）。

**支持的工具 ID（`--tools`）：** `amazon-q`, `antigravity`, `auggie`, `bob`, `claude`, `cline`, `codex`, `forgecode`, `codebuddy`, `continue`, `costrict`, `crush`, `cursor`, `factory`, `gemini`, `github-copilot`, `iflow`, `junie`, `kilocode`, `kimi`, `kiro`, `opencode`, `pi`, `qoder`, `lingma`, `qwen`, `roocode`, `trae`, `windsurf`

**示例：**

```bash
# 交互式初始化
openspec init

# 在特定目录中初始化
openspec init ./my-project

# 非交互式：为 Claude 和 Cursor 配置
openspec init --tools claude,cursor

# 为所有支持的工具配置
openspec init --tools all

# 覆盖此运行的 profile
openspec init --profile core

# 跳过提示并自动清理遗留文件
openspec init --force
```

**创建内容：**

```
openspec/
├── specs/              # 你的 specifications（真相来源）
├── changes/            # 提议的 changes
└── config.yaml         # 项目配置

.claude/skills/         # Claude Code skills（如果选择了 claude）
.cursor/skills/         # Cursor skills（如果选择了 cursor）
.cursor/commands/       # Cursor OPSX 命令（如果 delivery 包含 commands）
... (其他工具配置)
```

---

### `openspec update`

升级 CLI 后更新 OpenSpec 指令文件。使用当前全局 profile、选定的 workflows 和 delivery 模式重新生成 AI 工具配置文件。

```
openspec update [path] [options]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `path` | 否 | 目标目录（默认：当前目录） |

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--force` | 即使文件是最新的也强制更新 |

**示例：**

```bash
# npm 升级后更新指令文件
npm update @fission-ai/openspec
openspec update
```

---

## 工作区命令

工作区命令正在积极开发中，尚未准备就绪。请勿在此命令面上构建外部自动化、集成或长期工作流；命令行为、状态文件和 JSON 输出可能随时更改。

协调工作区是跨多个仓库或文件夹的工作的规划基地。工作区可见性不是变更承诺：链接 OpenSpec 应知道的仓库或文件夹，然后在准备好规划特定工作时创建变更。

### `openspec workspace setup`

在标准 OpenSpec 工作区位置创建工作区并链接至少一个现有仓库或文件夹。

```bash
openspec workspace setup [options]
```

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--name <name>` | 工作区名称。名称必须使用 kebab-case |
| `--link <path>` | 链接现有仓库或文件夹，并从文件夹名称推断链接名称 |
| `--link <name>=<path>` | 使用明确的链接名称链接现有仓库或文件夹 |
| `--opener <id>` | 在非交互式设置期间存储首选 opener：`codex`、`claude`、`github-copilot` 或 `editor` |
| `--no-interactive` | 禁用提示；需要 `--name` 和至少一个 `--link` |
| `--json` | 输出 JSON；需要 `--no-interactive` |

**示例：**

```bash
openspec workspace setup
openspec workspace setup --no-interactive --name platform --link /repos/api --link web=/repos/web
openspec workspace setup --no-interactive --name platform --link /repos/api --opener codex
openspec workspace setup --no-interactive --json --name checkout --link /repos/platform/apps/checkout
```

交互式设置询问首选 opener 并将其存储在机器本地工作区状态中。非交互式设置仅在提供 `--opener` 时存储首选 opener；否则 `workspace open` 在有支持的 opener 时在交互式终端中提示，或要求 scripts 传递 `--agent <tool>` 或 `--editor`。

### `openspec workspace list`

从本地注册表列出已知的 OpenSpec 工作区。

```bash
openspec workspace list [--json]
openspec workspace ls [--json]
```

列表显示每个工作区位置和链接的仓库或文件夹。报告过期的注册记录但不更改。

### `openspec workspace link`

为一个工作区记录现有仓库或文件夹。

```bash
openspec workspace link [name] <path> [options]
```

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--workspace <name>` | 从本地注册表选择已知工作区 |
| `--json` | 输出 JSON |
| `--no-interactive` | 禁用工作区选择器提示 |

**示例：**

```bash
openspec workspace link /repos/api
openspec workspace link api-service /repos/api
openspec workspace link --workspace platform /repos/platform/apps/checkout
```

路径必须已存在。相对路径在 OpenSpec 将验证的绝对路径存储到机器本地工作区状态之前相对于命令的当前目录解析。链接路径可以是完整仓库、包、服务、应用或没有仓库本地 `openspec/` 状态的文件夹。

### `openspec workspace relink`

修复或更改现有链接的本地路径。

```bash
openspec workspace relink <name> <path> [options]
```

路径必须已存在。Relink 仅更新稳定链接名称的机器本地路径。

### `openspec workspace doctor`

检查一个工作区可以在当前机器上解析什么。

```bash
openspec workspace doctor [options]
```

Doctor 显示工作区位置、规划路径、链接的仓库或文件夹、缺失路径、存在时的仓库本地 specs 路径，以及建议的修复。它仅报告问题；不会自动修复。

需要一个工作区的命令在从工作区文件夹或子目录内运行时使用当前工作区。从其他地方运行时，传递 `--workspace <name>`，在交互式终端中选择，或依赖只有一个已知工作区时的唯一已知工作区。在 `--json` 或 `--no-interactive` 模式下，模糊选择失败并显示结构化状态错误并建议 `--workspace <name>`。

JSON 响应使用类型化对象加上 `status` 数组。主数据位于 `workspace`、`workspaces` 或 `link`；警告和错误位于 `status`。

### `openspec workspace open`

通过存储的首选 opener、一次会话 agent 覆盖或 VS Code 编辑器模式打开工作区工作集。

```bash
openspec workspace open [name] [options]
```

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--workspace <name>` | 位置工作区名称的别名 |
| `--agent <tool>` | 一次会话 agent 覆盖：`codex`、`claude` 或 `github-copilot` |
| `--editor` | 将维护的 VS Code 工作区文件作为普通编辑器工作区打开 |
| `--no-interactive` | 禁用工作区和 opener 选择器提示 |

**示例：**

```bash
openspec workspace open
openspec workspace open platform
openspec workspace open platform --agent github-copilot
openspec workspace open --agent codex
openspec workspace open --editor
```

`workspace open` 在工作区内运行时使用当前工作区，在其他地方运行时自动选择唯一已知工作区，并在多个工作区已知时要求用户选择。`--agent` 和 `--editor` 不更改存储的首选 opener。传递两个 opener 覆盖是错误；选择 `--agent <tool>` 或 `--editor`。

OpenSpec 在工作区根目录维护 `<workspace-name>.code-workspace` 用于 VS Code 编辑器和 GitHub Copilot-in-VS-Code 打开。该文件是机器本地的，默认使用特定的 `<workspace-name>.code-workspace` `.gitignore` 条目忽略，因此用户编写的 `*.code-workspace` 文件仍有资格被跟踪。

维护的 VS Code 工作区包括协调根作为 `.` 加上有效链接的仓库或文件夹作为额外根。VS Code 将这些条目显示为多根工作区。

根工作区打开支持跨链接仓库或文件夹的探索和规划。实现编辑应仅在用户明确请求和正常 OpenSpec 实现工作流后开始。

---

## 浏览命令

### `openspec list`

列出项目中的 changes 或 specs。

```
openspec list [options]
```

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--specs` | 列出 specs 而不是 changes |
| `--changes` | 列出 changes（默认） |
| `--sort <order>` | 按 `recent`（默认）或 `name` 排序 |
| `--json` | 输出为 JSON |

**示例：**

```bash
# 列出所有活动 changes
openspec list

# 列出所有 specs
openspec list --specs

# 用于 scripts 的 JSON 输出
openspec list --json
```

**输出（文本）：**

```
Active changes:
  add-dark-mode     UI theme switching support
  fix-login-bug     Session timeout handling
```

---

### `openspec view`

显示用于探索 specs 和 changes 的交互式仪表板。

```
openspec view
```

打开基于终端的界面来导航项目 specifications 和 changes。

---

### `openspec show`

显示 change 或 spec 的详细信息。

```
openspec show [item-name] [options]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `item-name` | 否 | change 或 spec 的名称（如果省略则提示） |

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--type <type>` | 指定类型：`change` 或 `spec`（如果明确则自动检测） |
| `--json` | 输出为 JSON |
| `--no-interactive` | 禁用提示 |

**Change 特定选项：**

| 选项 | 描述 |
|--------|-------------|
| `--deltas-only` | 仅显示 delta specs（JSON 模式） |

**Spec 特定选项：**

| 选项 | 描述 |
|--------|-------------|
| `--requirements` | 仅显示 requirements，排除 scenarios（JSON 模式） |
| `--no-scenarios` | 排除 scenario 内容（JSON 模式） |
| `-r, --requirement <id>` | 按从 1 开始的索引显示特定 requirement（JSON 模式） |

**示例：**

```bash
# 交互式选择
openspec show

# 显示特定 change
openspec show add-dark-mode

# 显示特定 spec
openspec show auth --type spec

# 用于解析的 JSON 输出
openspec show add-dark-mode --json
```

---

## 验证命令

### `openspec validate`

验证 changes 和 specs 的结构问题。

```
openspec validate [item-name] [options]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `item-name` | 否 | 要验证的特定项目（如果省略则提示） |

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--all` | 验证所有 changes 和 specs |
| `--changes` | 验证所有 changes |
| `--specs` | 验证所有 specs |
| `--type <type>` | 当名称模糊时指定类型：`change` 或 `spec` |
| `--strict` | 启用严格验证模式 |
| `--json` | 输出为 JSON |
| `--concurrency <n>` | 最大并行验证数（默认：6，或 `OPENSPEC_CONCURRENCY` env） |
| `--no-interactive` | 禁用提示 |

**示例：**

```bash
# 交互式验证
openspec validate

# 验证特定 change
openspec validate add-dark-mode

# 验证所有 changes
openspec validate --changes

# 验证所有内容并输出 JSON（用于 CI/scripts）
openspec validate --all --json

# 严格验证并增加并行度
openspec validate --all --strict --concurrency 12
```

**输出（文本）：**

```
Validating add-dark-mode...
  ✓ proposal.md valid
  ✓ specs/ui/spec.md valid
  ⚠ design.md: missing "Technical Approach" section

1 warning found
```

**输出（JSON）：**

```json
{
  "version": "1.0.0",
  "results": {
    "changes": [
      {
        "name": "add-dark-mode",
        "valid": true,
        "warnings": ["design.md: missing 'Technical Approach' section"]
      }
    ]
  },
  "summary": {
    "total": 1,
    "valid": 1,
    "invalid": 0
  }
}
```

---

## 生命周期命令

### `openspec archive`

归档已完成的 change 并将 delta specs 合并到主 specs 中。

```
openspec archive [change-name] [options]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `change-name` | 否 | 要归档的 change（如果省略则提示） |

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `-y, --yes` | 跳过确认提示 |
| `--skip-specs` | 跳过 spec 更新（用于基础设施/工具/仅文档变更） |
| `--no-validate` | 跳过验证（需要确认） |

**示例：**

```bash
# 交互式归档
openspec archive

# 归档特定 change
openspec archive add-dark-mode

# 无提示归档（CI/scripts）
openspec archive add-dark-mode --yes

# 归档不影响 specs 的工具变更
openspec archive update-ci-config --skip-specs
```

**执行操作：**

1. 验证 change（除非 `--no-validate`）
2. 请求确认（除非 `--yes`）
3. 将 delta specs 合并到 `openspec/specs/`
4. 将 change 文件夹移动到 `openspec/changes/archive/YYYY-MM-DD-<name>/`

---

## 工作流命令

这些命令支持 artifact 驱动的 OPSX 工作流。它们对检查进度的人类和确定下一步的 agents 都很有用。

### `openspec status`

显示 change 的 artifact 完成状态。

```
openspec status [options]
```

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--change <id>` | Change 名称（如果省略则提示） |
| `--schema <name>` | Schema 覆盖（从 change 的 config 自动检测） |
| `--json` | 输出为 JSON |

**示例：**

```bash
# 交互式状态检查
openspec status

# 特定 change 的状态
openspec status --change add-dark-mode

# 用于 agent 使用的 JSON
openspec status --change add-dark-mode --json
```

**输出（文本）：**

```
Change: add-dark-mode
Schema: spec-driven
Progress: 2/4 artifacts complete

[x] proposal
[ ] design
[x] specs
[-] tasks (blocked by: design)
```

**输出（JSON）：**

```json
{
  "changeName": "add-dark-mode",
  "schemaName": "spec-driven",
  "isComplete": false,
  "applyRequires": ["tasks"],
  "artifacts": [
    {"id": "proposal", "outputPath": "proposal.md", "status": "done"},
    {"id": "design", "outputPath": "design.md", "status": "ready"},
    {"id": "specs", "outputPath": "specs/**/*.md", "status": "done"},
    {"id": "tasks", "outputPath": "tasks.md", "status": "blocked", "missingDeps": ["design"]}
  ]
}
```

---

### `openspec instructions`

获取用于创建 artifact 或应用 tasks 的增强指令。AI agents 使用它来了解接下来要创建什么。

```
openspec instructions [artifact] [options]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `artifact` | 否 | Artifact ID：`proposal`、`specs`、`design`、`tasks` 或 `apply` |

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--change <id>` | Change 名称（在非交互模式下必填） |
| `--schema <name>` | Schema 覆盖 |
| `--json` | 输出为 JSON |

**特殊情况：** 使用 `apply` 作为 artifact 获取任务实现指令。

**示例：**

```bash
# 获取下一个 artifact 的指令
openspec instructions --change add-dark-mode

# 获取特定 artifact 指令
openspec instructions design --change add-dark-mode

# 获取 apply/实现指令
openspec instructions apply --change add-dark-mode

# 用于 agent 消费的 JSON
openspec instructions design --change add-dark-mode --json
```

**输出包括：**

- Artifact 的模板内容
- 来自 config 的项目上下文
- 来自依赖 artifacts 的内容
- 来自 config 的每-artifact 规则

---

### `openspec templates`

显示 schema 中所有 artifacts 的解析模板路径。

```
openspec templates [options]
```

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--schema <name>` | 要检查的 schema（默认：`spec-driven`） |
| `--json` | 输出为 JSON |

**示例：**

```bash
# 显示默认 schema 的模板路径
openspec templates

# 显示自定义 schema 的模板
openspec templates --schema my-workflow

# 用于编程使用的 JSON
openspec templates --json
```

**输出（文本）：**

```
Schema: spec-driven

Templates:
  proposal  → ~/.openspec/schemas/spec-driven/templates/proposal.md
  specs     → ~/.openspec/schemas/spec-driven/templates/specs.md
  design    → ~/.openspec/schemas/spec-driven/templates/design.md
  tasks     → ~/.openspec/schemas/spec-driven/templates/tasks.md
```

---

### `openspec schemas`

列出具有描述和 artifact 流程的可用工作流 schemas。

```
openspec schemas [options]
```

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--json` | 输出为 JSON |

**示例：**

```bash
openspec schemas
```

**输出：**

```
Available schemas:

  spec-driven (package)
    The default spec-driven development workflow
    Flow: proposal → specs → design → tasks

  my-custom (project)
    Custom workflow for this project
    Flow: research → proposal → tasks
```

---

## Schema 命令

用于创建和管理自定义工作流 schemas 的命令。

### `openspec schema init`

创建新的项目本地 schema。

```
openspec schema init <name> [options]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `name` | 是 | Schema 名称（kebab-case） |

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--description <text>` | Schema 描述 |
| `--artifacts <list>` | 逗号分隔的 artifact IDs（默认：`proposal,specs,design,tasks`） |
| `--default` | 设置为项目默认 schema |
| `--no-default` | 不提示设置为默认 |
| `--force` | 覆盖现有 schema |
| `--json` | 输出为 JSON |

**示例：**

```bash
# 交互式 schema 创建
openspec schema init research-first

# 具有特定 artifacts 的非交互式创建
openspec schema init rapid \
  --description "Rapid iteration workflow" \
  --artifacts "proposal,tasks" \
  --default
```

**创建内容：**

```
openspec/schemas/<name>/
├── schema.yaml           # Schema 定义
└── templates/
    ├── proposal.md       # 每个 artifact 的模板
    ├── specs.md
    ├── design.md
    └── tasks.md
```

---

### `openspec schema fork`

复制现有 schema 到项目进行自定义。

```
openspec schema fork <source> [name] [options]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `source` | 是 | 要复制的 Schema |
| `name` | 否 | 新 schema 名称（默认：`<source>-custom`） |

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--force` | 覆盖现有目标 |
| `--json` | 输出为 JSON |

**示例：**

```bash
# Fork 内置的 spec-driven schema
openspec schema fork spec-driven my-workflow
```

---

### `openspec schema validate`

验证 schema 的结构和模板。

```
openspec schema validate [name] [options]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `name` | 否 | 要验证的 schema（如果省略则验证所有） |

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--verbose` | 显示详细验证步骤 |
| `--json` | 输出为 JSON |

**示例：**

```bash
# 验证特定 schema
openspec schema validate my-workflow

# 验证所有 schemas
openspec schema validate
```

---

### `openspec schema which`

显示 schema 从哪里解析（用于调试优先级）。

```
openspec schema which [name] [options]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `name` | 否 | Schema 名称 |

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--all` | 列出所有 schemas 及其来源 |
| `--json` | 输出为 JSON |

**示例：**

```bash
# 检查 schema 来自哪里
openspec schema which spec-driven
```

**输出：**

```
spec-driven resolves from: package
  Source: /usr/local/lib/node_modules/@fission-ai/openspec/schemas/spec-driven
```

**Schema 优先级：**

1. 项目：`openspec/schemas/<name>/`
2. 用户：`~/.local/share/openspec/schemas/<name>/`
3. 包：内置 schemas

---

## 配置命令

### `openspec config`

查看和修改全局 OpenSpec 配置。

```
openspec config <subcommand> [options]
```

**子命令：**

| 子命令 | 描述 |
|------------|-------------|
| `path` | 显示配置文件位置 |
| `list` | 显示所有当前设置 |
| `get <key>` | 获取特定值 |
| `set <key> <value>` | 设置值 |
| `unset <key>` | 删除键 |
| `reset` | 重置为默认值 |
| `edit` | 在 `$EDITOR` 中打开 |
| `profile [preset]` | 交互式或通过预设配置 workflow profile |

**示例：**

```bash
# 显示配置文件路径
openspec config path

# 列出所有设置
openspec config list

# 获取特定值
openspec config get telemetry.enabled

# 设置值
openspec config set telemetry.enabled false

# 显式设置字符串值
openspec config set user.name "My Name" --string

# 删除自定义设置
openspec config unset user.name

# 重置所有配置
openspec config reset --all --yes

# 在编辑器中编辑配置
openspec config edit

# 使用操作向导配置 profile
openspec config profile

# 快速预设：切换 workflows 到 core（保持 delivery 模式）
openspec config profile core
```

`openspec config profile` 从当前状态摘要开始，然后让你选择：
- 更改 delivery + workflows
- 仅更改 delivery
- 仅更改 workflows
- 保持当前设置（退出）

如果你保持当前设置，则不写入更改，也不显示更新提示。
如果配置没有更改，但当前项目文件与全局 profile/delivery 不同步，OpenSpec 将显示警告并建议运行 `openspec update`。
按 `Ctrl+C` 也会干净地取消流程（无堆栈跟踪）并以代码 `130` 退出。
在工作流清单中，`[x]` 表示 workflows 在全局配置中被选中。要将这些选择应用到项目文件，运行 `openspec update`（或在项目内提示时选择 `Apply changes to this project now?`）。

**交互式示例：**

```bash
# 仅 delivery 更新
openspec config profile
# 选择：仅更改 delivery
# 选择 delivery：仅 Skills

# 仅 workflows 更新
openspec config profile
# 选择：仅更改 workflows
# 在清单中切换 workflows，然后确认
```

---

## 工具命令

### `openspec feedback`

提交关于 OpenSpec 的反馈。创建 GitHub issue。

```
openspec feedback <message> [options]
```

**参数：**

| 参数 | 必填 | 描述 |
|----------|----------|-------------|
| `message` | 是 | 反馈消息 |

**选项：**

| 选项 | 描述 |
|--------|-------------|
| `--body <text>` | 详细描述 |

**要求：** 必须安装并认证 GitHub CLI（`gh`）。

**示例：**

```bash
openspec feedback "Add support for custom artifact types" \
  --body "I'd like to define my own artifact types beyond the built-in ones."
```

---

### `openspec completion`

管理 OpenSpec CLI 的 shell 自动补全。

```
openspec completion <subcommand> [shell]
```

**子命令：**

| 子命令 | 描述 |
|------------|-------------|
| `generate [shell]` | 输出补全脚本到 stdout |
| `install [shell]` | 为你的 shell 安装补全 |
| `uninstall [shell]` | 移除已安装的补全 |

**支持的 shell：** `bash`, `zsh`, `fish`, `powershell`

**示例：**

```bash
# 安装补全（自动检测 shell）
openspec completion install

# 为特定 shell 安装
openspec completion install zsh

# 生成脚本用于手动安装
openspec completion generate bash > ~/.bash_completion.d/openspec

# 卸载
openspec completion uninstall
```

---

## 退出代码

| 代码 | 含义 |
|------|---------|
| `0` | 成功 |
| `1` | 错误（验证失败、文件缺失等） |

---

## 环境变量

| 变量 | 描述 |
|----------|-------------|
| `OPENSPEC_TELEMETRY` | 设置为 `0` 禁用遥测 |
| `DO_NOT_TRACK` | 设置为 `1` 禁用遥测（标准 DNT 信号） |
| `OPENSPEC_CONCURRENCY` | 批量验证的默认并发数（默认：6） |
| `EDITOR` 或 `VISUAL` | 用于 `openspec config edit` 的编辑器 |
| `NO_COLOR` | 设置时禁用彩色输出 |

---

## 相关文档

- [Commands](commands.md) - AI slash commands（`/opsx:propose`、`/opsx:apply` 等）
- [Workflows](workflows.md) - 常见模式和每种命令的使用场景
- [Customization](customization.md) - 创建自定义 schemas 和模板
- [Getting Started](getting-started.md) - 首次设置指南
