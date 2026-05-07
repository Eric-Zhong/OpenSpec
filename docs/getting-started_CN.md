# Getting Started

本指南解释安装和初始化 OpenSpec 后 OpenSpec 如何工作。关于安装说明，请参见 [main README](../README.md#quick-start)。

## 工作原理

OpenSpec 帮助你和你的 AI 编程助手在编写任何代码之前就构建内容达成一致。

**默认快速路径（core profile）：**

```text
/opsx:propose ──► /opsx:apply ──► /opsx:sync ──► /opsx:archive
```

**扩展路径（自定义工作流选择）：**

```text
/opsx:new ──► /opsx:ff or /opsx:continue ──► /opsx:apply ──► /opsx:verify ──► /opsx:archive
```

默认全局 profile 是 `core`，包括 `propose`、`explore`、`apply`、`sync` 和 `archive`。你可以用 `openspec config profile` 启用扩展工作流命令，然后用 `openspec update`。

## OpenSpec 创建的内容

运行 `openspec init` 后，你的项目有这个结构：

```
openspec/
├── specs/              # 真相来源（你的系统行为）
│   └── <domain>/
│       └── spec.md
├── changes/            # 提议的更新（每个 change 一个文件夹）
│   └── <change-name>/
│       ├── proposal.md
│       ├── design.md
│       ├── tasks.md
│       └── specs/      # Delta specs（正在改变什么）
│           └── <domain>/
│               └── spec.md
└── config.yaml         # 项目配置（可选）
```

**两个关键目录：**

- **`specs/`** - 真相来源。这些 specs 描述你的系统当前如何运行。按领域组织（例如 `specs/auth/`、`specs/payments/`）。

- **`changes/`** - 提议的修改。每个 change 有自己的文件夹，包含所有相关 artifacts。当 change 完成时，其 specs 合并到主 `specs/` 目录。

## 理解 Artifacts

每个 change 文件夹包含指导工作的 artifacts：

| Artifact | 用途 |
|----------|---------|
| `proposal.md` | "为什么"和"是什么" - 捕获意图、范围和方法 |
| `specs/` | Delta specs 显示 ADDED/MODIFIED/REMOVED requirements |
| `design.md` | "如何" - 技术方法和架构决策 |
| `tasks.md` | 带复选框的实现清单 |

**Artifacts 相互构建：**

```
proposal ──► specs ──► design ──► tasks ──► implement
   ▲           ▲          ▲                    │
   └───────────┴──────────┴────────────────────┘
            update as you learn
```

你总是可以在实现过程中学到更多时回去完善早期的 artifacts。

## Delta Specs 如何工作

Delta specs 是 OpenSpec 的关键概念。它们显示相对于你当前 specs 的变化。

### 格式

Delta specs 使用部分来指示变更类型：

```markdown
# Delta for Auth

## ADDED Requirements

### Requirement: Two-Factor Authentication
The system MUST require a second factor during login.

#### Scenario: OTP required
- GIVEN a user with 2FA enabled
- WHEN the user submits valid credentials
- THEN an OTP challenge is presented

## MODIFIED Requirements

### Requirement: Session Timeout
The system SHALL expire sessions after 30 minutes of inactivity.
(Previously: 60 minutes)

#### Scenario: Idle timeout
- GIVEN an authenticated session
- WHEN 30 minutes pass without activity
- THEN the session is invalidated

## REMOVED Requirements

### Requirement: Remember Me
(Deprecated in favor of 2FA)
```

### 归档时发生什么

归档 change 时：

1. **ADDED** requirements 被追加到主 spec
2. **MODIFIED** requirements 替换现有版本
3. **REMOVED** requirements 从主 spec 中删除

change 文件夹移动到 `openspec/changes/archive/` 以供审计历史。

## 示例：你的第一个 Change

让我们走过为应用程序添加深色模式。

### 1. 开始 Change（默认）

```text
You: /opsx:propose add-dark-mode

AI:  Created openspec/changes/add-dark-mode/
     ✓ proposal.md — why we're doing this, what's changing
     ✓ specs/       — requirements and scenarios
     ✓ design.md    — technical approach
     ✓ tasks.md     — implementation checklist
     Ready for implementation!
```

如果你已启用扩展工作流 profile，你也可以分两步做：`/opsx:new` 然后 `/opsx:ff`（或 `/opsx:continue` 增量）。

### 2. 创建了什么

**proposal.md** - 捕获意图：

```markdown
# Proposal: Add Dark Mode

## Intent
Users have requested a dark mode option to reduce eye strain
during nighttime usage.

## Scope
- Add theme toggle in settings
- Support system preference detection
- Persist preference in localStorage

## Approach
Use CSS custom properties for theming with a React context
for state management.
```

**specs/ui/spec.md** - 显示新 requirements 的 delta：

```markdown
# Delta for UI

## ADDED Requirements

### Requirement: Theme Selection
The system SHALL allow users to choose between light and dark themes.

#### Scenario: Manual toggle
- GIVEN a user on any page
- WHEN the user clicks the theme toggle
- THEN the theme switches immediately
- AND the preference persists across sessions

#### Scenario: System preference
- GIVEN a user with no saved preference
- WHEN the application loads
- THEN the system's preferred color scheme is used
```

**tasks.md** - 实现清单：

```markdown
# Tasks

## 1. Theme Infrastructure
- [ ] 1.1 Create ThemeContext with light/dark state
- [ ] 1.2 Add CSS custom properties for colors
- [ ] 1.3 Implement localStorage persistence

## 2. UI Components
- [ ] 2.1 Create ThemeToggle component
- [ ] 2.2 Add toggle to settings page
- [ ] 2.3 Update Header to include quick toggle

## 3. Styling
- [ ] 3.1 Define dark theme color palette
- [ ] 3.2 Update components to use CSS variables
```

### 3. 实现

```
You: /opsx:apply

AI:  Working through tasks...
     ✓ 1.1 Created ThemeContext with light/dark state
     ✓ 1.2 Added CSS custom properties to globals.css
     ✓ 1.3 Implemented localStorage persistence
     ✓ 2.1 Created ThemeToggle component
     ...
     All tasks complete!
```

在实现过程中，如果你发现 design 需要调整，只需更新 artifact 并继续。

### 4. 归档

```
You: /opsx:archive

AI:  Archiving add-dark-mode...
     ✓ Merged specs into openspec/specs/ui/spec.md
     ✓ Moved to openspec/changes/archive/2025-01-24-add-dark-mode/
     Done! Ready for the next feature.
```

你的 delta specs 现在是主 specs 的一部分，记录你的系统如何运行。

## 验证和审查

使用 CLI 检查你的 changes：

```bash
# 列出活动 changes
openspec list

# 查看 change 详情
openspec show add-dark-mode

# 验证 spec 格式
openspec validate add-dark-mode

# 交互式仪表板
openspec view
```

## 下一步

- [Workflows](workflows.md) - 常见模式和每种命令的使用场景
- [Commands](commands.md) - 所有 slash commands 的完整参考
- [Concepts](concepts.md) - 更深入理解 specs、changes 和 schemas
- [Customization](customization.md) - 让 OpenSpec 按你的方式工作
