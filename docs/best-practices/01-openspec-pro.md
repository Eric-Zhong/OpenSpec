# OpenSpec 最佳实践

> 来源：[openspec.pro/best-practices](https://openspec.pro/best-practices/)

这些习惯将流畅的规范驱动工作流与混乱的工作流区分开来——尤其是当多人或模型同时处理同一个变更时。

---

## 1. 使用简短明确的变更名称

使用 `/opsx:new short-descriptive-slug` 创建变更，确保文件夹名称在代码审查和 `git status` 中清晰可读。

**示例：**
- ✅ 推荐：`add-oauth-refresh`
- ❌ 避免：`stuff`

---

## 2. 保持提案在两分钟内可阅读

您的提案应回答以下问题：

- **问题**：要解决什么问题？
- **解决方案**：提议的解决方案是什么？
- **范围**：包含哪些内容？不包含哪些内容？
- **风险**：潜在风险有哪些？

> 如果评审者无法快速浏览，请拆分变更或缩减范围。

---

## 3. 编写具体场景而非模糊描述

在 `specs/` 中优先使用具体场景，而非模糊的形容词。

**推荐格式：**

```text
Given <前置条件>
When <触发事件>
Then <预期结果>
```

场景将成为您的 AI 助手实现的合同。

---

## 4. 保持任务小而有序

- 大型单体任务会导致巨大的 diff
- 将工作拆分为可独立验证的编号步骤
- 当后续步骤依赖于前面的文件或迁移时，**顺序很重要**

---

## 5. 在 `/opsx:apply` 之前进行审查

快速转发（`/opsx:ff`）功能强大，将其输出视为人工设计审查。在要求模型接触生产路径之前，先调整 specs 或 tasks。

---

## 6. 在规划和编码之间重置上下文

长线程会积累噪音。规划稳定后，使用规范文件启动新会话通常会产生更清晰的实现。

> 参见 [OpenSpec + Cursor](https://openspec.pro/use-cases/cursor) 获取编辑器特定提示。

---

## 7. 完成时归档

当变更合并或放弃时使用 `/opsx:archive`，确保您的 `openspec/changes/` 目录保持为活跃工作队列，而非已完成的 graveyard。

---

## 相关资源

- [什么是规范驱动开发？](https://openspec.pro/learn/what-is-spec-driven-development)
- [工作流深度解析](https://openspec.pro/learn/workflow)
- [示例](https://openspec.pro/examples)
- [FAQ](https://openspec.pro/resources/faq)

---

> **注：** OpenSpec 是 Fission-AI 的开源项目（MIT License）。