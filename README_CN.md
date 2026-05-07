<p align="center">
  <a href="https://github.com/Fission-AI/OpenSpec">
    <picture>
      <source srcset="assets/openspec_bg.png">
      <img src="assets/openspec_bg.png" alt="OpenSpec logo">
    </picture>
  </a>
</p>

<p align="center">
  <a href="https://github.com/Fission-AI/OpenSpec/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/Fission-AI/OpenSpec/actions/workflows/ci.yml/badge.svg" /></a>
  <a href="https://www.npmjs.com/package/@fission-ai/openspec"><img alt="npm version" src="https://img.shields.io/npm/v/@fission-ai/openspec?style=flat-square" /></a>
  <a href="./LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square" /></a>
  <a href="https://discord.gg/YctCnvvshC"><img alt="Discord" src="https://discord.gg/YctCnvvshC?style=flat-square&logo=discord&logoColor=white&label=Discord&suffix=%20online" /></a>
</p>

<details>
<summary><strong>最受喜爱的 spec 框架。</strong></summary>

[![Stars](https://img.shields.io/github/stars/Fission-AI/OpenSpec?style=flat-square&label=Stars)](https://github.com/Fission-AI/OpenSpec/stargazers)
[![Downloads](https://img.shields.io/npm/dm/@fission-ai/openspec?style=flat-square&label=Downloads/mo)](https://www.npmjs.com/package/@fission-ai/openspec)
[![Contributors](https://img.shields.io/github/contributors/Fission-AI/OpenSpec?style=flat-square&label=Contributors)](https://github.com/Fission-AI/OpenSpec/graphs/contributors)

</details>
<p></p>
我们的理念：

```text
→ 流畅而非刻板
→ 迭代而非瀑布式
→ 简单而非复杂
→ 为 brownfield 而生，不只是 greenfield
→ 可从个人项目扩展到企业级应用
```

> [!TIP]
> **全新工作流现已推出！** 我们已经使用新的 artifact 引导工作流重建了 OpenSpec。
>
> 运行 `/opsx:propose "你的想法"` 开始使用。 → [了解更多](docs/opsx.md)

<p align="center">
  关注 <a href="https://x.com/0xTab">@0xTab on X</a> 获取更新 · 加入 <a href="https://discord.gg/YctCnvvshC">OpenSpec Discord</a> 获取帮助和讨论问题。
</p>

<!-- TODO: Add GIF demo of /opsx:propose → /opsx:archive workflow -->

## 实际演示

```text
You: /opsx:propose add-dark-mode
AI:  Created openspec/changes/add-dark-mode/
     ✓ proposal.md — 我们为什么要做这件事，正在改变什么
     ✓ specs/       — 需求和场景
     ✓ design.md    — 技术方案
     ✓ tasks.md     — 实现清单
     Ready for implementation!

You: /opsx:apply
AI:  Implementing tasks...
     ✓ 1.1 Add theme context provider
     ✓ 1.2 Create toggle component
     ✓ 2.1 Add CSS variables
     ✓ 2.2 Wire up localStorage
     All tasks complete!

You: /opsx:archive
AI:  Archived to openspec/changes/archive/2025-01-23-add-dark-mode/
     Specs updated. Ready for the next feature.
```

<details>
<summary><strong>OpenSpec Dashboard</strong></summary>

<p align="center">
  <img src="assets/openspec_dashboard.png" alt="OpenSpec dashboard preview" width="90%">
</p>

</details>

## 快速开始

**需要 Node.js 20.19.0 或更高版本。**

全局安装 OpenSpec：

```bash
npm install -g @fission-ai/openspec@latest
```

然后进入你的项目目录并初始化：

```bash
cd your-project
openspec init
```

现在告诉你的 AI：`/opsx:propose <你想构建的内容>`

如果你想要扩展工作流（`/opsx:new`, `/opsx:continue`, `/opsx:ff`, `/opsx:verify`, `/opsx:bulk-archive`, `/opsx:onboard`），使用 `openspec config profile` 选择并通过 `openspec update` 应用。

> [!NOTE]
> 不确定你的工具是否支持？ [查看完整列表](docs/supported-tools.md) – 我们支持 25+ 种工具，并且还在增加。
>
> 同时支持 pnpm、yarn、bun 和 nix。[查看安装选项](docs/installation.md)。

## 文档

→ **[Getting Started](docs/getting-started.md)**: 入门第一步<br>
→ **[Workflows](docs/workflows.md)**: 组合和模式<br>
→ **[Commands](docs/commands.md)**: slash commands 和 skills<br>
→ **[CLI](docs/cli.md)**: 终端命令参考<br>
→ **[Supported Tools](docs/supported-tools.md)**: 工具集成和安装路径<br>
→ **[Concepts](docs/concepts.md)**: 核心概念<br>
→ **[Multi-Language](docs/multi-language.md)**: 多语言支持<br>
→ **[Customization](docs/customization.md)**: 定制化配置

## 社区 schemas

通过独立仓库分发的第三方 schema 包 — 这些提供了整合 OpenSpec 与其他工具的个性化工作流，类似于 [github/spec-kit's community extension catalog](https://github.com/github/spec-kit/tree/main/extensions) 处理工具集成的方式。

→ 在自定义文档中 **[浏览目录](docs/customization.md#community-schemas)**。

## 为什么选择 OpenSpec？

AI 编程助手功能强大，但当需求只存在于聊天记录中时，结果往往不可预测。OpenSpec 添加了一个轻量级的 spec 层，让你在编写代码之前先就构建内容达成一致。

- **先达成共识再构建** — 人类和 AI 在代码编写之前通过 specs 对齐
- **保持有序** — 每个变更都有独立的文件夹，包含 proposal、specs、design 和 tasks
- **流畅工作** — 随时更新任何 artifact，没有刻板的阶段门控
- **使用你的工具** — 通过 slash commands 支持 20+ 种 AI 助手

### 我们的对比

**vs. [Spec Kit](https://github.com/github/spec-kit)** (GitHub) — 全面但重量级。严格的阶段门控、大量 Markdown、Python 配置。OpenSpec 更轻量，让你可以自由迭代。

**vs. [Kiro](https://kiro.dev)** (AWS) — 功能强大但被锁定在他们的 IDE 中，且仅限于 Claude 模型。OpenSpec 使用你已有的工具。

**vs. 什么都没有** — 没有 specs 的 AI 编程意味着模糊的提示和不可预测的结果。OpenSpec 带来可预测性而无需繁琐流程。

## 更新 OpenSpec

**升级包**

```bash
npm install -g @fission-ai/openspec@latest
```

**刷新 agent 指令**

在每个项目中运行此命令来重新生成 AI 指导并确保最新的 slash commands 可用：

```bash
openspec update
```

## 使用注意事项

**模型选择**：OpenSpec 最适合高推理能力的模型。我们推荐 Opus 4.5 和 GPT 5.2 用于规划和实现。

**Context 卫生**：OpenSpec 受益于干净的 context 窗口。在开始实现之前清除你的 context，并在整个会话中保持良好的 context 卫生。

## 贡献

**小型修复** — Bug 修复、拼写纠正和小型改进可以直接提交 PR。

**大型变更** — 对于新功能、重大重构或架构变更，请先提交 OpenSpec 变更提案，以便我们在实施之前对齐意图和目标。

编写提案时，请牢记 OpenSpec 的理念：我们服务于使用不同编程 agent、模型和用例的各种用户。变更应该对每个人都适用。

**欢迎 AI 生成的代码** — 只要经过测试和验证即可。包含 AI 生成代码的 PR 应提及所使用的编程 agent 和模型（例如："Generated with Claude Code using claude-opus-4-5-20251101"）。

### 开发

- 安装依赖：`pnpm install`
- 构建：`pnpm run build`
- 测试：`pnpm test`
- 本地开发 CLI：`pnpm run dev` 或 `pnpm run dev:cli`
- Conventional commits（一行格式）：`type(scope): subject`

## 其他

<details>
<summary><strong>遥测数据</strong></summary>

OpenSpec 收集匿名使用统计。

我们只收集命令名称和版本以了解使用模式。不收集参数、路径、内容或个人身份信息。在 CI 中自动禁用。

**选择退出：** `export OPENSPEC_TELEMETRY=0` 或 `export DO_NOT_TRACK=1`

</details>

<details>
<summary><strong>维护者和顾问</strong></summary>

请参阅 [MAINTAINERS.md](MAINTAINERS.md) 了解帮助指导项目的核心维护者和顾问列表。

</details>



## 许可证

MIT
