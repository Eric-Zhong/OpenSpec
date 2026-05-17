# OpenSpec Issues 汇总（第 1 页，共 25 个 Issue）

> 数据截止时间：2026-05-17 | 排序：按 Issue 编号从高到低

---

## 有效 Issue 清单

### #1091 — [Feature Request] 为 CLI feedback 命令添加 'feedback' label

- **状态：** Open
- **作者：** @OmniaZ1
- **日期：** 2026-05-14
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1091
- **问题：** `openspec feedback` 命令因仓库缺少 `feedback` label 而报 404 错误
- **建议方案：** 创建 `feedback` label（颜色 `#FBA4D4`），或 CLI 回退到 `question`/`enhancement` label

---

### #1087 — Protect OpenSpec from AI slop PRs

- **状态：** Open
- **作者：** @kimjune01
- **日期：** 2026-05-13
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1087
- **问题：** 过去两周内 38 个 PR 中有 8 个应在审查前被关闭
- **建议方案：** 安装 GitHub Action 过滤低质量 PR：
  - [PR Quality Gate](https://github.com/kimjune01/sweep/blob/master/action.yml) — 判断 PR 质量而非作者身份
  - [anti-slop](https://github.com/peakoss/anti-slop) — 34 项可配置检查
  - [agentscan-action](https://github.com/MatteoGabriele/agentscan-action) — 深度分析作者账户行为

---

### #1084 — Incomplete artifacts marked as "done" after workflow interruption

- **状态：** Open（有 PR #1098 关联）
- **作者：** @jiehu03
- **日期：** 2026-05-12
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1084
- **问题：** 工作流中断后，部分写入的 artifact 文件残留磁盘上；重新执行时系统检测到这些文件，将其标记为 `status: "done"` 并跳过，导致不一致状态
- **根因：** `detectCompleted()` 仅检查文件是否存在，不验证内容
- **PR #1098 修复方案：** 新增 `artifactOutputContentValid` 和 `artifactOutputComplete`，要求非空白、非纯注释内容

---

### #1081 — [Feature Request] 扩展包机制：自定义 schema、技能、命令、钩子和配置文件

- **状态：** Open
- **作者：** @harikrishnan83（Contributor）
- **日期：** 2026-05-11
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1081
- **问题：** OpenSpec 支持自定义 schema 和 profile，但复杂工作流需要更多资产（技能、命令、生命周期钩子、默认配置），目前只能手动组装
- **建议方案：** 提供轻量级的扩展包机制，让社区工作流可分享、安装、检查和更新
- **附带仓库：**
  - https://github.com/intent-driven-dev/openspec-schemas
  - https://github.com/intent-driven-dev/intent-driven-template

---

### #1080 — Support latest TRAE slash command

- **状态：** Open
- **作者：** @Joy-Zhang
- **日期：** 2026-05-11
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1080
- **问题：** TRAE 已支持斜杠命令，OpenSpec 的 opsx 命令需要更新以替换现有技能

---

### #1077 — For complex requirements, how can specs be generated and maintained?

- **状态：** Open
- **作者：** @beston123
- **日期：** 2026-05-10
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1077
- **问题：** 复杂需求下，如何生成和维护 spec？能否支持 Gherkin 的 Scenario Outline 和 Background
- **评论：** @marcindulak 指出与 #508 讨论相关

---

### #1076 — /opsx:* commands not visible in Claude Code 2.1.119 slash menu

- **状态：** Open
- **作者：** @aygzs123
- **日期：** 2026-05-10
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1076
- **环境：** OpenSpec CLI 1.3.1，Claude Code 2.1.119，Windows 11
- **问题：** 子目录命令与 Claude Code 新的 commands→skills 合并不兼容，`/opsx:` 菜单为空
- **评论：** @xiaods 建议升级到 CC 2.1.129

---

### #1075 — openspec init 选择 codex 后，在 codex cli 中没有 openspec 相关的 command

- **状态：** Open
- **作者：** @peakxy
- **日期：** 2026-05-09
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1075
- **问题：** `openspec init` 选择 codex 后，codex CLI 中没有 openspec 相关命令，只有 skill
- **评论：**
  - @great123456-1：使用 `$openxxxx` 就有，codex 确实有点抽象
  - @MingYuan998：同样问题，openspec 1.3.1 + codex-cli 0.130.0，`openspec init` 后没有生成 `.codex` 文件夹

---

### #1074 — Support schema extension or partial overrides to avoid full shadowing of built-in schemas

- **状态：** Open
- **作者：** @javigomez
- **日期：** 2026-05-09
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1074
- **问题：** 当前自定义 schema 需要完整复制内置 schema，上游改进不会自动接收
- **三种候选方案：**
  1. Schema 继承（`extends` 字段 + 按 ID 合并 artifact + 增量指令钩子）
  2. 项目配置层增加 prompt 扩展钩子
  3. 保留完整 shadow，增加上游追踪工具（`openspec schema diff/update/rebase`）
- **评论：** @harikrishnan83 表示自定义 schema 也有重复问题，希望支持 schema 元素定义与 schema 定义的分离

---

### #1073 — Add a pre-submit semantic cleanup checkpoint before verify/archive

- **状态：** Open
- **作者：** @streaker303
- **日期：** 2026-05-09
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1073
- **问题：** AI 辅助实现常遗留早期规划阶段的无用代码（未使用的 API 函数、类型字段、占位逻辑等），这些代码可通过 lint 和测试
- **建议方案：** 引入可选的「提交前语义清理检查点」，在 verify/archive 之前审计实际实现与 spec 的差异

---

### #1072 — opsx-sync command is missing on antigravity

- **状态：** Closed（已完成）
- **作者：** @asarkar1990
- **日期：** 2026-05-08
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1072
- **问题：** 无描述
- **解决：** @xiaods 建议使用 `openspec config profile`

---

### #1071 — Inquiry: Would a Chinese README be welcome?

- **状态：** Open
- **作者：** @HowardYan888（Contributor）
- **日期：** 2026-05-08
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1071
- **建议：** 创建 `README.zh-CN.md`，翻译核心内容（项目介绍、快速开始、为什么选择 OpenSpec 等）
- **待确认：** 多语言 README 是否已有计划、翻译质量要求

---

### #1067 — Restart your IDE for slash commands to take effect (CLI 工具不该显示此提示)

- **状态：** Open（有 PR #1097 关联）
- **作者：** @raul-junc-tgs
- **日期：** 2026-05-07
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1067
- **问题：** 无论选择哪个 agent，安装后都显示「Restart your IDE for slash commands to take effect」，重启电脑也没用
- **PR #1097 修复：** 为每个工具配置添加 `requiresIdeRestart` 标志，CLI 工具不再显示此消息

---

### #1065 — Add i18n / multilingual support for the website

- **状态：** Open
- **作者：** @nektobit
- **日期：** 2026-05-07
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1065
- **问题：** 网站仅支持英语，对非英语开发者形成障碍
- **建议方案：** 使用标准 i18n 方案，优先支持中文、西班牙语、俄语、葡萄牙语、日语
- **附注：** 作者愿意贡献实现

---

### #1064 — opsx:explore in codex does not actively raise decision prompts

- **状态：** Open
- **作者：** @narapeka
- **日期：** 2026-05-07
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1064
- **问题：** Codex 中的 explore 模式更像自由对话，agent 没有主动提出分步决策引导用户完成构思
- **评论：** 多位用户表示有同感，但 draft proposal 输出仍清晰可用

---

### #1049 — multiple language support

- **状态：** Open
- **作者：** @YanwuZeng
- **日期：** 2026-05-05
- **链接：** https://github.com/Fission-AI/OpenSpec/issues/1049
- **问题：** 当前通过修改 config.yaml 支持多语言，建议增加 `openspec init --language language_name` 或初始化步骤中选择语言
- **评论：** HowardYan888 和 nektobit 均表示支持

---

## 明显无意义的 Issue（已跳过）

| Issue 编号 | 标题 | 原因 |
|-----------|------|------|
| #1094 | test | 明显是测试 Issue |
| #1093 | test 123111 | 明显是测试 Issue |
| #1092 | test | 明显是测试 Issue |

---

## 统计摘要

| 分类 | 数量 |
|------|------|
| 有效 Issue | 16 |
| 测试/垃圾 Issue | 3 |
| 已有 PR 修复 | 2 (#1084→#1098, #1067→#1097) |
| 已关闭 | 1 (#1072) |
| 功能请求 | 7 |
| Bug 报告 | 5 |
| 讨论/咨询 | 4 |
