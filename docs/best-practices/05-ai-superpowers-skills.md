# 为 AI 注入纪律：Superpowers 工作流规范解析

> 使用 AI 编程时最常见的痛点——需求还没聊清就直接开工、说着"修好了"却换一种方式出错——归根结底是 AI 缺乏工程师的纪律。Superpowers 通过 15 个 Skill 文件构建了一套完整的工作流规范，帮助 AI 养成可靠的开发习惯。

> **原文地址：** [給 AI 超能力？Superpowers 的設計與取捨](https://kaochenlong.com/ai-superpowers-skills) | 作者：kaochenlong | 2026-01-20

---

![Superpowers Cover](https://cdn.kaochenlong.com/rails/active_storage/representations/proxy/eyJfcmFpbHMiOnsiZGF0YSI6MjY5MywicHVyIjoiYmxvYl9pZCJ9fQ==--de8797d247661836e39d9882e687acbcabe76aee/eyJfcmFpbHMiOnsiZGF0YSI6eyJmb3JtYXQiOiJ3ZWJwIiwicmVzaXplX3RvX2ZpbGwiOlsxMjgwLDQ0OF19LCJwdXIiOiJ2YXJpYXRpb24ifX0=--ecffecc7310878204e6133d2b5591dfd779a0246/superpowers-cover.jpg)

## 为什么 AI 需要"纪律"

无论使用哪家 AI 工具，开发者都遇到过类似场景：刚描述完需求，AI 就直接开工，然后告诉你"做好了"，结果一运行根本不能用；或者说 Bug 修好了，测试后发现只是换了一种方式出错。

问题的根源不在于 AI 不够聪明，而在于 **LLM 天生缺乏纪律**：

- 不一定先想清楚再动手
- 不一定写测试
- 不一定在执行前验证

这些是工程师多年培养的习惯，AI 并不会自动具备。[Superpowers](https://github.com/obra/superpowers)（GitHub 近 3 万 Star）正是为此而生——它不是软件或库，而是一套 **给 AI 阅读的工作流规范**，类似 AI Agent 的员工手册，明确定义了"做 X 之前必须先 Y"、"完成 Z 之后如何验证"。

**核心结论：** Superpowers 用 15 个 Skill 建立完整 AI 开发工作流，规则以"必须"而非"应该"的形式强制执行，并预先封堵 AI 常找的借口。即使不直接使用，阅读后也能显著提升你指导 AI 编程的能力。

---

## 整体架构

Superpowers 由以下两部分组成：

| 组件 | 说明 |
|------|------|
| **15 个 Skill** | 每个 Skill 是一份 Markdown 文件，描述特定场景下应遵循的流程 |
| **SessionStart Hook** | 每次对话开始时自动执行，将 `using-superpowers` 入门 Skill 注入对话上下文 |

这些 Skill 串联起完整的开发工作流：需求澄清 → 设计审查 → 计划编写 → 测试驱动开发 → 代码审查 → 分支合并。

---

## 一、动手前先对话：苏格拉底式需求澄清

`brainstorming` Skill 要求在开发新功能前，通过对话式提问厘清需求。

### 操作规则

- **一次只问一个问题**，不要连环轰炸
- **优先使用选择题**，而非开放式提问
- 确认 AI 理解需求后，用 200~300 字撰写设计方案
- **逐段确认**：每写完一段，停下来问"这样对吗？"，而非一次性甩出整份设计

### 方案对比

当探索多个方案时，要求提出 2~3 种不同做法，逐一说明利弊，然后给出推荐选项及理由。

### 关键原则

> **YAGNI（You Ain't Gonna Need It）**：在设计阶段就积极砍掉不需要的功能，而不是等实现后发现做了一堆无用的东西。

> 完整文档：[brainstorming/SKILL.md](https://github.com/obra/superpowers/blob/main/skills/brainstorming/SKILL.md)

---

## 二、编写计划：假设执行者一无所知

`writing-plans` Skill 对计划的详细程度要求极高，核心假设是：**执行这份计划的人对项目毫无了解、判断力有限，且不喜欢写测试。**

### 计划编写要求

- 每个任务拆分为 **2~5 分钟可完成** 的小步骤
- 写失败测试、运行确认失败、写代码使测试通过——各自是独立步骤
- 每个步骤包含 **完整代码**，不能出现"添加适当测试"之类的模糊描述
- **文件路径、执行命令、预期输出** 全部写死

### 设计目标

任何一个全新 AI 会话或对项目陌生的工程师，仅凭这份计划就能完成全部任务，无需额外解释。

> 完整文档：[writing-plans/SKILL.md](https://github.com/obra/superpowers/blob/main/skills/writing-plans/SKILL.md)

---

## 三、测试驱动开发：先有测试，再写代码

`test-driven-development` Skill 对 TDD 的定义极为严格，甚至堵住"我遵循的是 TDD 精神"这类借口：

> **违反规则的条文，就是违反规则的精神。**

### 核心规则

**没有失败的测试，就不能写一行生产代码。**

如果先写了代码再补测试——正确做法是 **删掉代码**，从测试重新开始。不是留着参考，而是彻底删除。

### 红-绿-重构循环

```
写一个会失败的测试
    → 运行，确认失败
        → 写最小代码使其通过
            → 运行，确认通过
                → 重构
```

每个步骤必须实际执行，不能跳过。如果测试一写完就通过，说明你在测已存在的行为，这个测试没有意义。

### 常见借口与反驳

| 借口 | 反驳 |
|------|------|
| "太简单，不需要测试" | 简单代码也会出错，测试只需 30 秒 |
| "我先手动测试过" | 手动测试不系统、无记录、无法重跑 |
| "测试后写也能达到同样目标" | 测试前问的是"应该做什么"，测试后问的是"做了什么"，两者完全不同 |
| "删掉 X 小时的工作太浪费" | 沉没成本谬误，保留未经验证的代码才是真正的浪费 |

另有 `testing-anti-patterns` 文档专门列举测试常见错误，如"测试 mock 行为而非真实代码"、"在正式代码中加入仅测试使用的方法"等。

> 完整文档：[test-driven-development/SKILL.md](https://github.com/obra/superpowers/blob/main/skills/test-driven-development/SKILL.md)

---

## 四、系统性调试，杜绝盲目猜测

`systematic-debugging` Skill 将调试分为四个递进阶段，**前一阶段未完成，不可进入下一阶段。**

### 阶段一：根因调查

- 逐字阅读错误信息
- 稳定复现问题，无法复现则不猜测
- 用 `git diff` 检查最近变更
- 在多组件系统中，在边界处加日志定位数据异常点

### 阶段二：模式分析

- 找出正常运作的相似代码，对比差异
- 若实现某种 Pattern，读完参考资料再动手

### 阶段三：假设与验证

- 提出具体假设并记录
- 做最小修改来验证（一次只改变一个变量）
- 假设被推翻则提出新假设，不要在错误方向上继续堆代码

### 阶段四：修复

- 先写复现问题的测试，再修复，确认测试通过
- **修复失败 3 次以上？** 停下来质疑架构本身是否存在根本问题

### 配套技术文档

- **defense-in-depth**：修复 Bug 后，在每一层（API 边界、业务逻辑、环境层面）添加验证，使 Bug 结构上不可能再发生
- **condition-based-waiting**：不要猜等多久，等到条件真正成立

```javascript
// 不推荐：猜测等待时间
await new Promise((r) => setTimeout(r, 50))
const result = getResult()
expect(result).toBeDefined()

// 推荐：等待条件成立
await waitFor(() => getResult() !== undefined)
const result = getResult()
expect(result).toBeDefined()
```

- **root-cause-tracing**：从错误点逆向追溯至问题源头

### 常见借口与反驳

| 借口 | 反驳 |
|------|------|
| "问题简单，不需要流程" | 简单问题也有根因，流程对简单 Bug 同样高效 |
| "紧急，没时间走流程" | 系统性调试比盲目试错更快 |
| "我看到问题了，让我修" | 看到症状 ≠ 理解根因 |
| "再试一次（已连续失败 2 次以上）" | 3+ 次失败 = 架构问题，应质疑设计而非继续修 |

> 完整文档：[systematic-debugging/SKILL.md](https://github.com/obra/superpowers/blob/main/skills/systematic-debugging/SKILL.md)

---

## 五、验证优先：说完成之前先跑一遍

`verification-before-completion` Skill 针对 AI 的常见坏习惯——未验证就宣布完成。

> **宣称完成但未验证，是不诚实而非高效。**

### 规则

在说"完成了""修好了""测试通过了"之前，必须：

1. 运行对应验证命令
2. 读取完整输出
3. 确认结果与宣称一致

不能用"应该可以""我有信心""看起来对"代替证据。

### 验证对照表

| 宣称 | 对应验证 |
|------|----------|
| 测试通过 | 测试命令输出显示 0 个失败 |
| Build 成功 | 退出码为 0 |
| Bug 修好了 | 重现原始症状，确认不再出现 |
| Subagent 完成任务 | 检查 `git diff` 确认有实际变更 |

### 预警信号

出现以下情况时，应立即停下来验证：

- 使用"应该""大概""似乎"等模糊词
- 说"完成"之前感到满意或兴奋
- 想跳过验证

> 完整文档：[verification-before-completion/SKILL.md](https://github.com/obra/superpowers/blob/main/skills/verification-before-completion/SKILL.md)

---

## 六、两阶段审查：先看规格，再看品质

`subagent-driven-development` Skill 定义了每任务完成后的两道审查流程，**顺序不可调换**。

### 第一道：规格符合性审查

- 启动一个 Subagent，检查代码是否完全符合规格
- **多做** 和 **少做** 都是问题
- 关注点："这是我们要的东西吗？"

### 第二道：代码品质审查

- 仅在规格审查通过后执行
- 关注点：测试覆盖率、代码结构、可维护性

### 为什么要分开

"代码写得好但不是我们要的"是常见问题。先确认方向正确，再打磨品质，避免在错误的代码上投入大量精力。

### 常见陷阱

- 跳过任一审查
- 在规格符合性上接受"差不多就行"
- 让实现者自我审查代替真正的审查

> 完整文档：[subagent-driven-development/SKILL.md](https://github.com/obra/superpowers/blob/main/skills/subagent-driven-development/SKILL.md)

---

## 七、正确应对代码审查意见

`receiving-code-review` Skill 规范了收到审查意见后的应对方式。

### 禁止的行为

不使用表演性赞同时，如：

- "你说得对！"
- "好建议！"
- "让我现在实现"（验证前）

这些话语提供情绪价值，但不含技术信息。更糟的是，审查者可能错了，你却因不想显得不配合而照做。

### 正确流程

```
理解 → 验证 → 评估 → 回应
```

1. **理解**：用自己的话复述要求，确认未误解
2. **验证**：检查建议在当前代码库中是否正确，是否影响现有功能
3. **评估**：判断建议是否合理，还是审查者缺乏上下文
4. **回应**：给出技术回应或直接开始修改

### 审查者说错了怎么办？

用技术理由推回，而非默默接受。若不便直接拒绝，可使用暗号：

> *Strange things are afoot at the Circle K*

这句来自 1989 年电影《Bill & Ted's Excellent Adventure》，作为约定暗号，提醒人类开发者"我有话想说但不好直接说"。

如果推回后发现自己是错的，简洁回应即可："你是对的，我查了 X 确认是 Y，现在来修。"

> 完整文档：[receiving-code-review/SKILL.md](https://github.com/obra/superpowers/blob/main/skills/receiving-code-review/SKILL.md)

---

## 八、编写 Skill 本身也是 TDD

`writing-skills` Skill 将 TDD 理念延伸到文档编写：**写 Skill = 面向文档的 TDD。**

### 编写流程

```
设计压力测试场景
    → 启动无 Skill 的 Subagent，观察它如何违反规则（Red）
        → 编写 Skill，封堵发现的问题
            → 启动有 Skill 的 Subagent，验证它遵守规则（Green）
                → 发现新绕过方式？补充封堵措施，重新测试
```

### 关键洞察

- 不要只写"请遵守 TDD"，而要 **预判 AI 的每一个借口** 并逐一反驳
- Skill 描述字段应写"何时使用"，而非"做什么"——否则 AI 可能只看描述就开始做，跳过正文
  - 例如描述写"每个任务间做 code review"，AI 可能只做一次 review，即使正文要求两次

### TDD 与 Skill 编写对照

| TDD 概念 | Skill 编写对应 |
|----------|----------------|
| 测试用例 | 带 Subagent 的压力场景 |
| 生产代码 | Skill 文档 (SKILL.md) |
| 测试失败 (Red) | 无 Skill 时 Agent 违反规则 |
| 测试通过 (Green) | 有 Skill 时 Agent 遵守规则 |
| 重构 | 封堵漏洞，同时保持合规 |

> 完整文档：[writing-skills/SKILL.md](https://github.com/obra/superpowers/blob/main/skills/writing-skills/SKILL.md)

---

## 完整工作流程

将上述 Skill 串联后，典型的开发流程如下：

```
brainstorming
  → 需求澄清，AI 通过提问逐步明确需求
      → 提出多个方案，用户选择后 AI 撰写设计文档
          → using-git-worktrees
              → 在新 worktree 中建立隔离工作环境
                  → writing-plans
                      → 将设计拆分为细粒度任务，每个含完整指令和代码
                          → subagent-driven-development
                              → 每个任务由独立 Subagent 执行
                              → 完成后由另两个 Subagent 分别做规格审查和品质审查
                              → 执行过程中 TDD Skill 确保先写测试
                                  → finishing-a-development-branch
                                      → 运行完整测试
                                      → 与用户确认：合并 / 开 PR / 保留 / 丢弃
```

---

## 与 OpenSpec 搭配使用

Superpowers 的局限在于所有流程存在于对话中，Session 结束后难以追溯。对于长期维护的项目，建议搭配 [OpenSpec](https://github.com/Fission-AI/OpenSpec) 留存规格记录。

### 推荐搭配流程

```
brainstorming（复杂任务时才需要，简单需求可跳过）
  → openspec proposal（正式化规格，生成 tasks.md）
      → writing-plans（将 tasks.md 展开为细粒度步骤，非必需）
          → openspec apply（开始实现）
              → openspec archive（归档）
```

OpenSpec 的 `tasks.md` 为高层任务清单，简单任务可直接执行；复杂任务再用 `writing-plans` 展开。

![OpenSpec GUI - Spectra](https://cdn.kaochenlong.com/rails/active_storage/representations/proxy/eyJfcmFpbHMiOnsiZGF0YSI6MjY5NSwicHVyIjoiYmxvYl9pZCJ9fQ==--a9320bb931565b3ee0929c15aded6057b16796aa/eyJfcmFpbHMiOnsiZGF0YSI6eyJmb3JtYXQiOiJ3ZWJwIiwicmVzaXplX3RvX2xpbWl0IjpbMTI4MCwxMDI0XSwicXVhbGl0eSI6ODV9LCJwdXIiOiJ2YXJpYXRpb24ifX0=--0b69e9fe8af003acacbcd26b0aac87dbbf711297/ai-superpowers-skills-1.webp)

> 作者正在开发 OpenSpec 的 GUI 工具 Spectra，用于浏览、修改 Proposal 和 Spec Delta，以及管理归档。

---

## 设计取舍分析

### 四大设计亮点

| 设计决策 | 说明 |
|----------|------|
| **强制而非建议** | 使用"必须"而非"应该"，堵住"我遵循的是精神"这类借口 |
| **理性化预防表** | 每个重要 Skill 列举常见借口并逐一反驳，堵漏洞而非讲道理 |
| **步骤级验证** | 每个步骤完成后必须确认，大幅增加步骤数，但大幅减少"以为好了其实没好" |
| **原子化任务** | 每个任务独立执行，不依赖前置上下文或执行者判断，可安全委派给新 Subagent |

---

## 限制与问题

| 问题 | 说明 |
|------|------|
| **依赖 AI 自律** | 纯文档系统，无技术手段强制执行；AI 常以"我知道意思"为由跳过 Skill |
| **Token 成本** | 每个任务开 3 个 Subagent（1 实现 + 2 审查），50 个任务的 Token 消耗可观 |
| **计划膨胀** | 大型功能的计划文件可达数百步骤，中途修改困难 |
| **串行执行** | 即使使用 Subagent，任务仍逐个执行，非真正并行 |
| **环境假设** | 假设 Git 已配置、测试套件已就绪、build/test 命令已设定，非标准项目需额外调整 |
| **YAGNI 悖论** | 系统强调避免过度设计，但本身缺乏强制机制来确保这一点——教你避免过度设计的系统，本身可能就是过度设计 |

---

## 适用场景

### 推荐使用

- 需要高质量代码、愿意投入时间做审查和验证
- 长期维护的项目，需要建立稳定的 AI 协作纪律
- 团队刚开始使用 AI 编程，缺乏成熟的协作流程

### 不推荐使用

- 快速原型验证，只想尽快出个 Demo
- 一次性脚本或简单修改
- 团队已有成熟的开发流程
- Bug 修复、错别字修改、配置文件调整等琐事

---

## 总结

Superpowers 的核心理念：**AI 需要纪律，纪律需要明确的规则，而非模糊的建议。**

它用激进的方式强制规则执行，不给 AI 任何理性化的空间。每个 Skill 都假设 AI 会尝试绕过，预先封堵各种借口。但这种方法的有效性最终仍取决于 AI 是否真的遵守——目前尚无技术手段保证这一点，需要提示工程、反复测试和人工监督三者结合。

即使不直接使用这套 Skill，通读后也能显著提升你对"如何指导 AI 编程"的理解。尤其是那些**理性化预防表格**，记录了 AI 在实际开发中常找的借口，这些实战经验比任何理论都更有价值。
