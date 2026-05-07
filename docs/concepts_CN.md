# Concepts

本指南解释 OpenSpec 背后的核心概念以及它们如何组合在一起。有关实际使用，请参见 [Getting Started](getting-started.md) 和 [Workflows](workflows.md)。

## Philosophy

OpenSpec 围绕四个原则构建：

```
fluid not rigid         — no phase gates, work on what makes sense
iterative not waterfall — learn as you build, refine as you go
easy not complex        — lightweight setup, minimal ceremony
brownfield-first        — works with existing codebases, not just greenfield
```

### 为什么这些原则重要

**Fluid not rigid。** 传统的 spec 系统将你锁定在阶段中：首先规划，然后实现，然后完成。OpenSpec 更加灵活——你可以按对你的工作有意义的方式以任何顺序创建 artifacts。

**Iterative not waterfall。** 需求会变化。理解会深化。在开始时看起来好的方法在看到代码库后可能站不住脚。OpenSpec 接受这个现实。

**Easy not complex。** 一些 spec 框架需要大量设置、严格格式或重量级流程。OpenSpec 不碍事。几秒钟初始化，立即开始工作，仅在需要时自定义。

**Brownfield-first。** 大多数软件工作不是从零开始构建——而是在修改现有系统。OpenSpec 的基于 delta 的方法使得指定对现有行为的更改变得容易，而不仅仅是描述新系统。

## 大局

OpenSpec 将你的工作组织成两个主要区域：

```
┌────────────────────────────────────────────────────────────────────┐
│                        openspec/                                   │
│                                                                    │
│   ┌─────────────────────┐      ┌───────────────────────────────┐   │
│   │       specs/        │      │         changes/              │   │
│   │                     │      │                               │   │
│   │  Source of truth    │◄─────│  Proposed modifications       │   │
│   │  How your system    │ merge│  Each change = one folder     │   │
│   │  currently works    │      │  Contains artifacts + deltas  │   │
│   │                     │      │                               │   │
│   └─────────────────────┘      └───────────────────────────────┘   │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Specs** 是真相来源——它们描述你的系统当前如何运行。

**Changes** 是提议的修改——它们存在于单独的文件夹中，直到你准备好合并它们。

这种分离是关键。你可以在没有冲突的情况下并行处理多个 changes。你可以在影响主 specs 之前审查 change。当归档 change 时，它的 deltas 干净地合并到真相来源中。

## 协调工作区

工作区支持正在积极开发中，尚未准备就绪。请勿在此工作区行为上构建外部自动化、集成或长期工作流；命令、状态文件和 JSON 输出可能随时更改。

以下命令为跨链接仓库或文件夹的规划提供了第一个设置流程。

当一个仓库拥有规划、实现和归档流程时，仓库本地的 OpenSpec 项目是正确的默认值。有些工作跨越多个仓库或文件夹。对于这种情况，OpenSpec 协调工作区是持久的规划基地。

工作区心智模型是：

```text
workspace = where related cross-repo changes live
link      = a stable name for a repo or folder the workspace can plan against
change    = one feature, fix, project, or other planned piece of work
```

工作区与仓库本地项目有不同的形状：

```text
workspace-folder/
├── changes/                       # Workspace-level planning
└── .openspec-workspace/
    ├── workspace.yaml             # Shared workspace identity and link names
    └── local.yaml                 # This machine's local paths
```

仓库本地 OpenSpec 状态保持现有形状：

```text
repo-root/
└── openspec/
    ├── specs/
    └── changes/
```

这种区别很重要。工作区文件夹是跨链接仓库或文件夹进行规划的协调面。每个仓库的 `openspec/` 目录仍然是仓库拥有的 specs、仓库本地 changes 和实现规划的归宿。用户不需要在工作区文件夹内运行仓库本地的 `openspec init`。

稳定的链接名称是工作区规划如何引用仓库和文件夹。共享工作区状态保留名称如 `api`、`web` 或 `checkout`；每台机器将这些名称映射到其自己的本地路径，在 `.openspec-workspace/local.yaml` 中。

```yaml
# .openspec-workspace/workspace.yaml
version: 1
name: platform
links:
  api: {}
  web: {}
```

```yaml
# .openspec-workspace/local.yaml
version: 1
paths:
  api: /repos/api
  web: /repos/web
```

OpenSpec 创建的工作区默认从可移植协作状态中排除 `.openspec-workspace/local.yaml`。`.openspec-workspace/workspace.yaml` 保持可移植，因为它存储工作区名称和稳定链接名称，而不是一个用户的绝对 checkout 路径。

链接路径可以是完整仓库、大型 monorepo 内的文件夹或其他现有文件夹。它们在参与工作区规划之前不需要仓库本地的 `openspec/` 状态。后续实现、verify 或归档工作流可能需要更多仓库准备，但规划可见性从链接开始。

```text
multi-repo:
  api      -> /repos/api
  web      -> /repos/web

large monorepo:
  billing  -> /repos/platform/services/billing
  checkout -> /repos/platform/apps/checkout
```

托管工作区位于标准 OpenSpec 数据目录下：

```text
getGlobalDataDir()/workspaces
```

这意味着当设置 `XDG_DATA_HOME` 时为 `$XDG_DATA_HOME/openspec/workspaces`，Unix 风格回退为 `~/.local/share/openspec/workspaces`，以及原生 Windows 回退为 `%LOCALAPPDATA%\openspec\workspaces`。原生 Windows shell、PowerShell 和 WSL2 各自为运行 OpenSpec 的运行时保留路径字符串。此基础不会在 `D:\repo`、`/mnt/d/repo` 和 UNC WSL 路径之间转换。

OpenSpec 还在以下位置保留机器本地注册表：

```text
getGlobalDataDir()/workspaces/registry.yaml
```

注册表将工作区名称映射到工作区位置，以便稍后全局命令可以从任何地方列出或选择已知工作区。它只是一个索引。每个工作区文件夹对其自己的 `.openspec-workspace/workspace.yaml` 和 `.openspec-workspace/local.yaml` 保持权威，因此过期的注册记录可以报告和修复，而无需重新定义工作区本身。

工作区可见性不是变更承诺。在 OpenSpec 应该知道哪些仓库或文件夹相关时设置工作区；在准备好规划功能、修复、项目或其他工作时创建变更。

有用的命令：

```bash
# Guided setup
openspec workspace setup

# Automation-friendly setup
openspec workspace setup --no-interactive --name platform --link /repos/api --link web=/repos/web
openspec workspace setup --no-interactive --name platform --link /repos/api --opener codex

# See known workspaces from the local registry
openspec workspace list
openspec workspace ls

# Add or repair links for the selected workspace
openspec workspace link /repos/api
openspec workspace link api-service /repos/api
openspec workspace relink api-service /new/path/to/api

# Check what this machine can resolve
openspec workspace doctor
openspec workspace doctor --workspace platform

# Open the linked working set
openspec workspace open
openspec workspace open platform --agent github-copilot
openspec workspace open --editor
```

`workspace setup` 始终在标准工作区位置创建工作区，将其记录在本地注册表中，显示工作区位置，并需要至少一个链接的仓库或文件夹。交互式设置询问首选 opener。非交互式设置仅在提供 `--opener codex`、`--opener claude`、`--opener github-copilot` 或 `--opener editor` 时存储一个。

OpenSpec 还维护根工作区打开文件：`AGENTS.md` 中 OpenSpec 管理的指导块、用于 VS Code 和 GitHub Copilot-in-VS-Code 打开的机器本地 `<workspace-name>.code-workspace` 文件，以及该维护的 `.code-workspace` 文件的特定忽略条目。用户编写的 `*.code-workspace` 文件仍然可跟踪，因为忽略规则仅针对维护的文件。

维护的 VS Code 工作区包括协调根作为 `.` 加上有效链接的仓库或文件夹作为额外根。VS Code 将这些条目显示为多根工作区。

`workspace open` 打开链接的工作集，使用存储的首选 opener，除非为该会话传递了 `--agent <tool>` 或 `--editor`。传递两个 opener 覆盖是错误。选择 `--agent <tool>` 或 `--editor`。

根工作区打开使链接的仓库和文件夹可见以进行探索和规划；实现仅在用户明确请求实现工作后开始。

`workspace link` 和 `workspace relink` 仅记录现有文件夹；它们不创建、复制、移动、初始化或编辑链接的仓库或文件夹。成功链接或重新链接后，OpenSpec 刷新管理的指导、VS Code 工作区文件和忽略规则。

需要一个工作区的工作区命令可以从任何地方通过 `--workspace <name>` 运行。如果你在工作区文件夹或子目录内运行它们，OpenSpec 使用该当前工作区。如果有多个已知工作区可用且你没有传递 `--workspace <name>`，人类命令显示选择器；`--json` 和 `--no-interactive` 失败并显示结构化状态错误而不是提示。

直接工作区命令支持用于 scripts 的 JSON 输出。JSON 响应将主数据保留在 `workspace`、`workspaces` 或 `link` 对象中，并在 `status` 数组中报告警告或错误。健康对象使用 `status: []`。

## Specs

Specs 使用结构化 requirements 和 scenarios 描述你的系统行为。

### 结构

```
openspec/specs/
├── auth/
│   └── spec.md           # Authentication behavior
├── payments/
│   └── spec.md           # Payment processing
├── notifications/
│   └── spec.md           # Notification system
└── ui/
    └── spec.md           # UI behavior and themes
```

按领域组织 specs——对你的系统有意义的逻辑分组。常见模式：

- **按功能区域**：`auth/`、`payments/`、`search/`
- **按组件**：`api/`、`frontend/`、`workers/`
- **按有界上下文**：`ordering/`、`fulfillment/`、`inventory/`

### Spec 格式

Spec 包含 requirements，每个 requirement 有 scenarios：

```markdown
# Auth Specification

## Purpose
Authentication and session management for the application.

## Requirements

### Requirement: User Authentication
The system SHALL issue a JWT token upon successful login.

#### Scenario: Valid credentials
- GIVEN a user with valid credentials
- WHEN the user submits login form
- THEN a JWT token is returned
- AND the user is redirected to dashboard

#### Scenario: Invalid credentials
- GIVEN invalid credentials
- WHEN the user submits login form
- THEN an error message is displayed
- AND no token is issued

### Requirement: Session Expiration
The system MUST expire sessions after 30 minutes of inactivity.

#### Scenario: Idle timeout
- GIVEN an authenticated session
- WHEN 30 minutes pass without activity
- THEN the session is invalidated
- AND the user must re-authenticate
```

**关键元素：**

| 元素 | 用途 |
|---------|---------|
| `## Purpose` | 此 spec 领域的高级描述 |
| `### Requirement:` | 系统必须具有的特定行为 |
| `#### Scenario:` | 需求在行动中的具体示例 |
| SHALL/MUST/SHOULD | 表示需求强度的 RFC 2119 关键字 |

### 为什么这样构建 Specs

**Requirements 是"什么"** — 它们陈述系统应该做什么，而不指定实现。

**Scenarios 是"何时"** — 它们提供可以验证的具体示例。好的 scenarios：
- 可测试（你可以为它们编写自动化测试）
- 覆盖 happy path 和 edge cases
- 使用 Given/When/Then 或类似的结构化格式

**RFC 2119 关键字**（SHALL、MUST、SHOULD、MAY）传达意图：
- **MUST/SHALL** — 绝对需求
- **SHOULD** — 推荐，但存在例外
- **MAY** — 可选

### Spec 是什么（不是什么）

Spec 是**行为契约**，而不是实现计划。

好的 spec 内容：
- 用户或下游系统依赖的可观察行为
- 输入、输出和错误条件
- 外部约束（安全、隐私、可靠性、兼容性）
- 可以测试或明确验证的 scenarios

避免在 specs 中：
- 内部类/函数名
- 库或框架选择
- 逐步实现细节
- 详细执行计划（这些属于 `design.md` 或 `tasks.md`）

快速测试：
- 如果实现可以在不改变外部可见行为的情况下改变，它可能不属于 spec。

### 保持轻量：渐进式严格

OpenSpec 旨在避免官僚主义。使用仍然使变更可验证的最轻级别。

**精简 spec（默认）：**
- 简短的以行为优先的需求
- 清晰的范围和非目标
- 一些具体的验收检查

**完整 spec（用于更高风险）：**
- 跨团队或跨仓库变更
- API/契约变更、迁移、安全/隐私问题
- 模糊可能导致昂贵返工的变更

大多数 changes 应该保持在精简模式。

### 人类 + Agent 协作

在许多团队中，人类探索和 agents 起草 artifacts。预期的循环是：

1. 人类提供意图、上下文和约束。
2. Agent 将其转换为以行为优先的需求和 scenarios。
3. Agent 将实现细节保留在 `design.md` 和 `tasks.md` 中，而不是 `spec.md`。
4. 验证在实现前确认结构和清晰度。

这保持 specs 对人类可读，对 agents 一致。

## Changes

Change 是提议的对系统的修改，打包为包含理解和实现它所需一切的文件夹。

### Change 结构

```
openspec/changes/add-dark-mode/
├── proposal.md           # Why and what
├── design.md             # How (technical approach)
├── tasks.md              # Implementation checklist
├── .openspec.yaml        # Change metadata (optional)
└── specs/                # Delta specs
    └── ui/
        └── spec.md       # What's changing in ui/spec.md
```

每个 change 都是自包含的。它有：
- **Artifacts** — 捕获意图、设计和 tasks 的文档
- **Delta specs** — 关于正在添加、修改或删除什么的规范
- **元数据** — 此特定 change 的可选配置

### 为什么 Changes 是文件夹

将 change 打包为文件夹有几个好处：

1. **一切在一起。** Proposal、design、tasks 和 specs 在一个地方。无需在不同位置搜索。

2. **并行工作。** 多个 changes 可以同时存在而不会冲突。在 `add-dark-mode` 工作的同时 `fix-auth-bug` 也可以进行。

3. **干净的历史。** 归档时，changes 移动到 `changes/archive/`，并保留其完整上下文。你可以回顾并理解不仅是改变了什么，还有为什么。

4. **适合审查。** change 文件夹易于审查——打开它，阅读 proposal，检查 design，查看 spec deltas。

## Artifacts

Artifacts 是 change 中指导工作的文档。

### Artifact 流程

```
proposal ──────► specs ──────► design ──────► tasks ──────► implement
    │               │             │              │
   why            what           how          steps
 + scope        changes       approach      to take
```

Artifacts 相互构建。每个 artifact 为下一个提供上下文。

### Artifact 类型

#### Proposal（`proposal.md`）

Proposal 捕获高级别的**意图**、**范围**和**方法**。

```markdown
# Proposal: Add Dark Mode

## Intent
Users have requested a dark mode option to reduce eye strain
during nighttime usage and match system preferences.

## Scope
In scope:
- Theme toggle in settings
- System preference detection
- Persist preference in localStorage

Out of scope:
- Custom color themes (future work)
- Per-page theme overrides

## Approach
Use CSS custom properties for theming with a React context
for state management. Detect system preference on first load,
allow manual override.
```

**何时更新 proposal：**
- 范围变化（缩小或扩大）
- 意图澄清（更好地理解问题）
- 方法根本性转变

#### Specs（`specs/` 中的 delta specs）

Delta specs 描述相对于当前 specs 的**变化**。见下面的 [Delta Specs](#delta-specs)。

#### Design（`design.md`）

Design 捕获**技术方法**和**架构决策**。

````markdown
# Design: Add Dark Mode

## Technical Approach
Theme state managed via React Context to avoid prop drilling.
CSS custom properties enable runtime switching without class toggling.

## Architecture Decisions:

### Decision: Context over Redux
Using React Context for theme state because:
- Simple binary state (light/dark)
- No complex state transitions
- Avoids adding Redux dependency

### Decision: CSS Custom Properties
Using CSS variables instead of CSS-in-JS because:
- Works with existing stylesheet
- No runtime overhead
- Browser-native solution

## Data Flow
```
ThemeProvider (context)
       │
       ▼
ThemeToggle ◄──► localStorage
       │
       ▼
CSS Variables (applied to :root)
```

## File Changes
- `src/contexts/ThemeContext.tsx` (new)
- `src/components/ThemeToggle.tsx` (new)
- `src/styles/globals.css` (modified)
````

**何时更新 design：**
- 实现揭示方法不起作用
- 发现更好的解决方案
- 依赖或约束改变

#### Tasks（`tasks.md`）

Tasks 是**实现清单** — 带有复选框的具体步骤。

```markdown
# Tasks

## 1. Theme Infrastructure
- [ ] 1.1 Create ThemeContext with light/dark state
- [ ] 1.2 Add CSS custom properties for colors
- [ ] 1.3 Implement localStorage persistence
- [ ] 1.4 Add system preference detection

## 2. UI Components
- [ ] 2.1 Create ThemeToggle component
- [ ] 2.2 Add toggle to settings page
- [ ] 2.3 Update Header to include quick toggle

## 3. Styling
- [ ] 3.1 Define dark theme color palette
- [ ] 3.2 Update components to use CSS variables
- [ ] 3.3 Test contrast ratios for accessibility
```

**Task 最佳实践：**
- 将相关 tasks 分组在标题下
- 使用层次编号（1.1、1.2 等）
- 保持 tasks 小到可以在一次会话中完成
- 完成后勾选 tasks

## Delta Specs

Delta specs 是使 OpenSpec 适用于 brownfield 开发的关键概念。它们描述**正在改变什么**，而不是重述整个 spec。

### 格式

```markdown
# Delta for Auth

## ADDED Requirements

### Requirement: Two-Factor Authentication
The system MUST support TOTP-based two-factor authentication.

#### Scenario: 2FA enrollment
- GIVEN a user without 2FA enabled
- WHEN the user enables 2FA in settings
- THEN a QR code is displayed for authenticator app setup
- AND the user must verify with a code before activation

#### Scenario: 2FA login
- GIVEN a user with 2FA enabled
- WHEN the user submits valid credentials
- THEN an OTP challenge is presented
- AND login completes only after valid OTP

## MODIFIED Requirements

### Requirement: Session Expiration
The system MUST expire sessions after 15 minutes of inactivity.
(Previously: 30 minutes)

#### Scenario: Idle timeout
- GIVEN an authenticated session
- WHEN 15 minutes pass without activity
- THEN the session is invalidated

## REMOVED Requirements

### Requirement: Remember Me
(Deprecated in favor of 2FA. Users should re-authenticate each session.)
```

### Delta 部分

| 部分 | 含义 | 归档时发生什么 |
|---------|---------|------------------------|
| `## ADDED Requirements` | 新行为 | 追加到主 spec |
| `## MODIFIED Requirements` | 更改的行为 | 替换现有 requirement |
| `## REMOVED Requirements` | 弃用的行为 | 从主 spec 中删除 |

### 为什么是 Deltas 而不是完整 Specs

**清晰。** Delta 显示确切的变化。阅读完整 spec，你需要在大脑中 diff 它与当前版本。

**避免冲突。** 两个 changes 可以触及同一个 spec 文件，只要它们修改不同的 requirements，就不会冲突。

**审查效率。** 审查者看到变化，而不是不变的上下文。专注于重要的事情。

**Brownfield 适合。** 大多数工作修改现有行为。Deltas 使修改成为一等公民，而不是事后考虑。

## Schemas

Schemas 定义工作流的 artifact 类型及其依赖关系。

### Schemas 如何工作

```yaml
# openspec/schemas/spec-driven/schema.yaml
name: spec-driven
artifacts:
  - id: proposal
    generates: proposal.md
    requires: []              # No dependencies, can create first

  - id: specs
    generates: specs/**/*.md
    requires: [proposal]      # Needs proposal before creating

  - id: design
    generates: design.md
    requires: [proposal]      # Can create in parallel with specs

  - id: tasks
    generates: tasks.md
    requires: [specs, design] # Needs both specs and design first
```

**Artifacts 形成依赖图：**

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

**依赖是启用器，不是门。** 它们显示可以创建什么，而不是你下一步必须创建什么。如果不需要 design，你可以跳过它。你可以在 design 之前或之后创建 specs——两者都只依赖于 proposal。

### 内置 Schemas

**spec-driven**（默认）

spec 驱动开发的标准化工作流：

```
proposal → specs → design → tasks → implement
```

最适合：大多数你想在实现前同意 specs 的功能工作。

### 自定义 Schemas

为团队工作流创建自定义 schemas：

```bash
# 从零开始创建
openspec schema init research-first

# 或 fork 一个现有的
openspec schema fork spec-driven research-first
```

**自定义 schema 示例：**

```yaml
# openspec/schemas/research-first/schema.yaml
name: research-first
artifacts:
  - id: research
    generates: research.md
    requires: []           # Do research first

  - id: proposal
    generates: proposal.md
    requires: [research]   # Proposal informed by research

  - id: tasks
    generates: tasks.md
    requires: [proposal]   # Skip specs/design, go straight to tasks
```

有关创建和使用自定义 schemas 的详细信息，请参见 [Customization](customization.md)。

## Archive

归档通过将其 delta specs 合并到主 specs 并保留 change 以供历史来完成 change。

### 归档时会发生什么

```
Before archive:

openspec/
├── specs/
│   └── auth/
│       └── spec.md ◄────────────────┐
└── changes/                         │
    └── add-2fa/                     │
        ├── proposal.md              │ merge
        ├── design.md                │
        ├── tasks.md                 │
        └── specs/                   │
            └── auth/                │
                └── spec.md ─────────┘


After archive:

openspec/
├── specs/
│   └── auth/
│       └── spec.md        # Now includes 2FA requirements
└── changes/
    └── archive/
        └── 2025-01-24-add-2fa/    # Preserved for history
            ├── proposal.md
            ├── design.md
            ├── tasks.md
            └── specs/
                └── auth/
                    └── spec.md
```

### 归档过程

1. **合并 deltas。** 每个 delta spec 部分（ADDED/MODIFIED/REMOVED）被应用到相应的主 spec。

2. **移动到归档。** change 文件夹移动到 `changes/archive/`，并带有日期前缀以进行时间排序。

3. **保留上下文。** 所有 artifacts 保持完整在归档中。你可以随时回顾以理解为什么做了变更。

### 为什么归档重要

**干净的状态。** 活动 changes（`changes/`）仅显示进行中的工作。已完成的工作移开。

**审计追踪。** 归档保留每个变更的完整上下文——不仅是改变了什么，还有解释为什么的 proposal、解释如何的设计，以及显示完成工作的 tasks。

**Spec 演进。** Specs 随着 changes 归档而有机增长。每个归档合并其 deltas，随着时间构建全面的规范。

## 它如何组合在一起

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                              OPENSPEC FLOW                                   │
│                                                                              │
│   ┌────────────────┐                                                         │
│   │  1. START      │  /opsx:propose (core) or /opsx:new (expanded)           │
│   │     CHANGE     │                                                         │
│   └───────┬────────┘                                                         │
│           │                                                                  │
│           ▼                                                                  │
│   ┌────────────────┐                                                         │
│   │  2. CREATE     │  /opsx:ff or /opsx:continue (expanded workflow)         │
│   │     ARTIFACTS  │  Creates proposal → specs → design → tasks              │
│   │                │  (based on schema dependencies)                         │
│   └───────┬────────┘                                                         │
│           │                                                                  │
│           ▼                                                                  │
│   ┌────────────────┐                                                         │
│   │  3. IMPLEMENT  │  /opsx:apply                                            │
│   │     TASKS      │  Work through tasks, checking them off                  │
│   │                │◄──── Update artifacts as you learn                      │
│   └───────┬────────┘                                                         │
│           │                                                                  │
│           ▼                                                                  │
│   ┌────────────────┐                                                         │
│   │  4. VERIFY     │  /opsx:verify (optional)                                │
│   │     WORK       │  Check implementation matches specs                     │
│   └───────┬────────┘                                                         │
│           │                                                                  │
│           ▼                                                                  │
│   ┌────────────────┐     ┌──────────────────────────────────────────────┐    │
│   │  5. ARCHIVE    │────►│  Delta specs merge into main specs           │    │
│   │     CHANGE     │     │  Change folder moves to archive/             │    │
│   └────────────────┘     │  Specs are now the updated source of truth   │    │
│                          └──────────────────────────────────────────────┘    │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

**良性循环：**

1. Specs 描述当前行为
2. Changes 提议修改（作为 deltas）
3. 实现使变更成为现实
4. 归档将 deltas 合并到 specs
5. Specs 现在描述新行为
6. 下一个 change 在更新的 specs 基础上构建

## 词汇表

| 术语 | 定义 |
|------|------------|
| **Artifact** | Change 中的文档（proposal、design、tasks 或 delta specs） |
| **Archive** | 完成 change 并将其 deltas 合并到主 specs 的过程 |
| **Change** | 提议的对系统的修改，打包为包含 artifacts 的文件夹 |
| **Delta spec** | 相对于当前 specs 描述更改（ADDED/MODIFIED/REMOVED）的 spec |
| **Domain** | Specs 的逻辑分组（例如 `auth/`、`payments/`） |
| **Requirement** | 系统必须具有的特定行为 |
| **Scenario** | 需求的具体示例，通常采用 Given/When/Then 格式 |
| **Schema** | Artifact 类型及其依赖关系的定义 |
| **Spec** | 描述系统行为、包含 requirements 和 scenarios 的规范 |
| **Source of truth** | `openspec/specs/` 目录，包含当前商定的行为 |

## 下一步

- [Getting Started](getting-started.md) - 实际第一步
- [Workflows](workflows.md) - 常见模式和每种命令的使用场景
- [Commands](commands.md) - 完整命令参考
- [Customization](customization.md) - 创建自定义 schemas 和配置项目
