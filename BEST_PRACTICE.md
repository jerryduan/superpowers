# Superpowers 主开发链路最佳实践

> 目标：把 `superpowers` 仓库里“需求到交付”的标准开发链路收束成一条稳定主线，用显式触发驱动 skill，在保证质量的前提下把返工压到最低。

## 1. 适用范围

本文以主开发链路为主体，适用于这两类主场景：

- 新增能力：新增 skill、workflow、command、跨平台接入能力
- 增量迭代：调整已有行为、约束、文档与实现的一致性

另外，本文附带覆盖 3 条常见侧链：

- 写 / 改 skill 本身
- 处理 review feedback
- 并行调度

本文**不覆盖** bug 专项排障链路；bug 场景请看 [BEST_PRACTICE_BUG.md](/Users/duanyangyang/.codex/superpowers/BEST_PRACTICE_BUG.md)。

## 2. 一条主线走到底

在 `superpowers` 里，最快的方式不是“直接开改”，而是固定走下面这条主线：

1. `using-superpowers`
2. `brainstorming`
3. `using-git-worktrees`
4. `writing-plans`
5. `subagent-driven-development`
6. `test-driven-development`
7. `requesting-code-review`
8. `verification-before-completion`
9. `finishing-a-development-branch`

本文统一采用**显式触发**，不依赖自动匹配。原则只有 4 条：

- 一个阶段只说一个目标，不把“设计、开发、评审、提 PR”混在一句话里。
- 直接点名 skill 名称，不让模型自己猜。
- 每推进一个阶段，都要说清“先不要做什么”。
- 每个阶段都要有明确产物，没有产物就不算真正进入下一步。

推荐角色分工：

- 主控代理：负责澄清需求、写 spec、写 plan、调度执行、把关验收。
- 执行代理：只负责按 plan 做实现、补测试、修 review 问题，不自行扩 scope。
- 人类：负责边界决策、关键确认、最终交付选择。

## 3. 一页看懂主流程

| 阶段 | 显式触发 | 主要输出 | 默认目录 | 是否需要人参与 |
| --- | --- | --- | --- | --- |
| 入口统一 | `先按 using-superpowers 进入标准流程` | 流程共识、阶段边界 | 无固定落盘文件 | 否 |
| 需求与方案 | `进入 brainstorming，先不要写代码` | 设计结论、设计文档 | `docs/superpowers/specs/` | 是 |
| 隔离工作区 | `开始实现前先执行 using-git-worktrees` | worktree、基线测试结果 | `.worktrees/` / `worktrees/` / `~/.config/superpowers/worktrees/` | 条件性 |
| 计划拆解 | `用 writing-plans 生成可执行计划` | implementation plan | `docs/superpowers/plans/` | 是 |
| 执行实现 | `按 subagent-driven-development + TDD 执行` | 代码、测试、变更记录 | 业务目录 + `docs/ai-changes-<requirement-name>.md` | 条件性 |
| 阶段评审 | `当前任务完成后先 requesting-code-review` | 规格评审结论、代码评审结论 | 默认不强制落盘 | 条件性 |
| 完成验证 | `先做 verification-before-completion，不要先说完成` | fresh verification 证据 | 默认不强制落盘 | 否 |
| 收尾交付 | `验证通过后按 finishing-a-development-branch 收尾` | merge / PR / 保留分支 / 丢弃方案 | Git/PR 系统 | 是 |

## 4. 固定产物清单

主链路里真正应该稳定落盘的文档，只有下面 3 类：

| 文档 | 作用 | 路径 |
| --- | --- | --- |
| 设计文档 | 固化需求、方案、边界、测试策略 | `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` |
| 实施计划 | 把设计拆成可执行小任务 | `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md` |
| AI 变更记录 | 记录跨会话改动、决策和验证摘要 | `docs/ai-changes-<requirement-name>.md` |

另外 3 类输出通常存在，但**不一定是仓库内固定文件**：

- review 结论：默认在会话、评论或 PR 中，不强制要求单独建文件
- verification 证据：默认来自刚执行过的命令输出，不要求额外造文档
- 交付结果：可能是本地 merge，也可能是 PR，本质上是 Git/平台状态，不一定对应仓库内 Markdown

## 5. 分阶段最佳实践

### 5.1 入口统一：`using-superpowers`

目标是先统一方法，再开始真正工作。

推荐显式触发：

```text
先按 using-superpowers 进入标准流程。这个任务分阶段推进，不要直接开始实现。
```

本阶段输出：

- 明确接下来要走的是一条主开发链路
- 明确当前先做设计，不直接写代码

本阶段文档：

- 无固定落盘文件

人类是否必须参与：

- 不必须

完成门禁：

- AI 已明确后续按阶段推进
- 当前会话已接受“显式触发 + 阶段门禁”的工作方式

### 5.2 需求澄清与方案设计：`brainstorming`

目标是把“想法”收束成“已确认设计”，并且在动手前锁定边界。

推荐显式触发：

```text
进入 brainstorming。先不要写代码，请基于当前 superpowers 仓库做需求澄清、影响面分析、方案比较和推荐方案。
```

本阶段必须输出：

- 需求边界
- 影响范围
- 2 到 3 个方案及推荐方案
- 设计文档

固定文档与目录：

- `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`

设计文档至少应覆盖：

- 本次要解决的问题
- 改动范围与不改范围
- 涉及目录与下游影响
- 错误处理与兼容性策略
- 测试策略

人类必须参与的点：

- 逐步确认设计方向
- 审核已落盘 spec
- 明确是否需要调整边界或缩 scope

完成门禁：

- 设计方案已被人确认
- spec 已写入 `docs/superpowers/specs/`
- user 已确认可以从 spec 进入 plan

### 5.3 隔离工作区：`using-git-worktrees`

目标是在不污染当前工作区的前提下开始实现。

推荐显式触发：

```text
开始实现前先执行 using-git-worktrees，优先使用隔离 worktree；如果基线测试失败，先停下来告诉我。
```

本阶段必须输出：

- 是否已经在隔离工作区
- 若未隔离，则创建新的 worktree
- 基线依赖安装与基线测试结果

默认目录：

- `.worktrees/<branch-name>`
- `worktrees/<branch-name>`
- `~/.config/superpowers/worktrees/<project>/<branch-name>`

本阶段文档：

- 无固定 Markdown 文档

人类必须参与的点：

- 当尚未隔离且需要征求是否创建 worktree 时
- 当基线测试失败，需要决定“先修基线”还是“带着已知失败继续”

完成门禁：

- 已确认当前实现环境隔离状态
- 已完成依赖初始化
- 已拿到 fresh baseline test 结果

### 5.4 计划拆解：`writing-plans`

目标是把已确认设计拆成可执行、可验证、可交给代理的任务。

推荐显式触发：

```text
设计已确认。请使用 writing-plans 生成 implementation plan，不要直接写实现；任务粒度控制在 2 到 5 分钟，并写清文件、测试和验证命令。
```

本阶段必须输出：

- plan 文档
- 文件修改映射
- 逐任务步骤
- 每一步的测试与验证命令

固定文档与目录：

- `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`

计划文档至少应覆盖：

- 目标与架构摘要
- 每个任务涉及哪些文件
- 每个任务如何先写失败测试
- 每个任务完成后如何验证

人类必须参与的点：

- 确认 plan 粒度是否合适
- 在 `subagent-driven-development` 与 `executing-plans` 之间做选择

完成门禁：

- plan 已落盘到 `docs/superpowers/plans/`
- spec 中每个关键要求都能在 plan 里找到对应任务
- 已决定采用哪种执行模式

### 5.5 实施执行：`subagent-driven-development` + `test-driven-development`

目标是按 plan 做小步交付，每步都走 TDD。

推荐显式触发：

```text
按 subagent-driven-development 执行这个 plan。所有任务必须遵守 test-driven-development，先写失败测试，再做最小实现。
```

本阶段必须输出：

- 代码改动
- 回归测试/单元测试
- 小步提交
- AI 变更记录

固定文档与目录：

- 业务代码与测试目录：按任务涉及文件落盘
- `docs/ai-changes-<requirement-name>.md`

这里的 AI 变更记录建议至少写：

- 本轮改动摘要
- 涉及文件/模块
- 关键决策
- 已执行验证

人类必须参与的点：

- 只有在出现 blocker、计划错误、架构冲突、命名/调用链需要改向时才介入
- 常规机械执行不应每步都打断人

完成门禁：

- 每个任务都经历 Red -> Green -> Refactor
- 任务内测试已通过
- 变更记录已更新到 `docs/ai-changes-<requirement-name>.md`

### 5.6 阶段评审：`requesting-code-review`

目标是在问题还便宜时把它们拦下来，而不是等到最后一起返工。

推荐显式触发：

```text
当前任务完成后先 requesting-code-review，先检查规格符合性，再检查代码质量；有重要问题就先修，不要继续下一个任务。
```

本阶段必须输出：

- 规格符合性评审结论
- 代码质量评审结论
- 待修项或通过结论

固定文档与目录：

- 主链路默认无强制仓库内文件
- 如需沉淀，可把关键 review 结论摘要补进 `docs/ai-changes-<requirement-name>.md`

人类必须参与的点：

- 当 review 暴露出需求理解偏差、架构争议、边界重议时
- 当 AI 对 reviewer 结论需要升级判断时

完成门禁：

- 规格问题已关闭
- 重要代码质量问题已关闭
- 才允许进入下一任务或最终验证

### 5.7 完成验证：`verification-before-completion`

目标是只基于 fresh evidence 说话，不基于感觉收尾。

推荐显式触发：

```text
先不要说完成。请执行 verification-before-completion：基于这次改动列出验证清单，跑完 fresh verification 后再报告结果。
```

本阶段必须输出：

- 本次 claim 对应的验证命令
- 最新命令结果
- 是否还有未验证风险

固定文档与目录：

- 默认无强制仓库内文件
- 若团队需要可追溯证据，可把关键验证摘要追加到 `docs/ai-changes-<requirement-name>.md`

推荐按改动面选择验证命令，例如：

- `cd tests/brainstorm-server && npm test`
- `./tests/skill-triggering/run-all.sh`
- `./tests/opencode/run-tests.sh`
- `./tests/claude-code/run-skill-tests.sh`
- `./tests/claude-code/run-skill-tests.sh --integration`

人类是否必须参与：

- 不必须

完成门禁：

- 所有“已完成”说法都有 fresh command output 支撑
- 已明确说明通过项、未覆盖项、残余风险

### 5.8 收尾交付：`finishing-a-development-branch`

目标是在验证通过后，用结构化选项完成交付，而不是含糊收尾。

推荐显式触发：

```text
验证通过后按 finishing-a-development-branch 收尾。先给我标准选项，再按我的选择执行，不要自己替我决定 merge、PR 还是丢弃。
```

本阶段必须输出：

- 交付选项
- 执行结果
- 分支 / worktree / PR 状态

可能的交付结果：

- 本地 merge 回基线分支
- push 并创建 PR
- 保留分支稍后处理
- 丢弃此次工作

固定文档与目录：

- 无固定 Markdown 文档
- 若创建 PR，产物主要在 Git 平台

人类必须参与的点：

- 必须由人选择最终交付方式
- 若选择丢弃，必须由人明确确认

完成门禁：

- 已重新验证测试
- 已确认分支去向
- worktree 是否清理已明确

## 6. 必须有人参与的节点

主链路里，下面 5 个节点不应该完全交给 AI 自行决定：

1. 设计确认
   design section、推荐方案、scope 边界必须由人拍板。

2. 书面 spec 审核
   `docs/superpowers/specs/...` 写完后，必须由人确认再进入 plan。

3. 执行模式选择
   plan 写完后，至少要确认是 `subagent-driven-development` 还是 `executing-plans`。

4. 异常升级
   基线失败、架构冲突、命名争议、调用链变更，都需要人判断是否改变方向。

5. 最终交付选择
   merge、PR、保留分支、丢弃工作，必须由人决定。

## 7. 最容易踩坑的反模式

- 还没做 `brainstorming` 就开始改代码
- spec 没落盘，只靠对话记忆推进
- plan 只有目标，没有文件、步骤、测试命令
- 让执行代理自己猜需求和边界
- 没看见失败测试就直接写实现
- review 留到最后一次才做
- 没有 fresh verification 就说“已经好了”
- 收尾时 AI 自己替人决定 merge 或删分支

## 8. 主链之外的 3 条常见侧链

这 3 条内容不是用来替代主链，而是在特定时机**插入主链**。

| 侧链 | 何时插入 | 对应 skill | 典型位置 |
| --- | --- | --- | --- |
| 写 / 改 skill 本身 | 当交付对象不是普通代码，而是 `skills/<name>/SKILL.md` | `writing-skills` | 设计 / plan 之后，实施执行之前 |
| 处理 review feedback | 当 `requesting-code-review` 或外部 reviewer 已经给出反馈 | `receiving-code-review` | 阶段评审之后，继续实现之前 |
| 并行调度 | 当存在 2 个以上彼此独立的任务或问题域 | `dispatching-parallel-agents` | 实施执行阶段内部 |

### 8.1 写 / 改 skill 本身：`writing-skills`

目标是当交付物本身是一个 skill 时，仍然沿用主链，但把“实现”理解成“用 TDD 的方式写流程文档”。

推荐显式触发：

```text
这次交付物是 skill 本身。请在主流程里切换到 writing-skills，按 skill 的 TDD 方式完成新增或修改，不要把它当普通文档直接改。
```

它插入主链的位置：

- `brainstorming` 之后仍然需要确认设计边界
- `writing-plans` 之后，如果真正开始落 skill 内容，就应切到 `writing-skills`
- 它本质上替代的是“普通代码实现”的方法论，不替代前后的 review 和 verification

本侧链必须输出：

- skill 目录结构
- `SKILL.md`
- 必要的 supporting file
- baseline pressure scenario
- skill 生效后的验证结果

固定文档与目录：

- `skills/<skill-name>/SKILL.md`
- `skills/<skill-name>/supporting-file.*` 或同目录 supporting assets
- `docs/ai-changes-<requirement-name>.md`

写 / 改 skill 时特别要补的内容：

- `name` 与 `description` 是否满足发现性要求
- `description` 只描述何时使用，不总结流程
- 是否先验证“没有 skill 时会失败”，再验证“有 skill 后会遵守”
- 是否把大段重参考资料拆成 supporting file

人类必须参与的点：

- skill 的适用边界与命名
- `description` 的触发条件是否准确
- 这个能力是否真的值得沉淀成 skill，而不是项目局部规则

完成门禁：

- skill 目录已落盘
- 已完成至少一轮“无 skill 会失败 / 有 skill 会通过”的思路验证
- 相关变更已记录到 `docs/ai-changes-<requirement-name>.md`

### 8.2 处理 review feedback：`receiving-code-review`

目标是收到 review 后先核实、再响应、再逐项实现，而不是表演式认同或盲改。

推荐显式触发：

```text
现在进入 receiving-code-review。先逐项核实 feedback 的技术正确性和适用范围；不要先表态，更不要在没确认前直接改。
```

它插入主链的位置：

- 紧跟在 `requesting-code-review` 之后
- 外部 reviewer、PR 评论、人类同伴 review 意见到达后立即生效

本侧链必须输出：

- 已理解的 feedback 清单
- 不清楚项的澄清问题
- 采纳项、驳回项、待确认项
- 按优先级排序后的修复动作

固定文档与目录：

- 默认无强制仓库内文件
- 如需沉淀，可把关键 review 结论和处理结果补进 `docs/ai-changes-<requirement-name>.md`

处理 review feedback 时必须遵守：

- 不做 performative agreement
- 不清楚的项先问清楚，再动手
- 外部 reviewer 的意见要先核实是否适用于当前代码库
- valid feedback 要一项一测，不批量盲改

人类必须参与的点：

- review 意见互相冲突
- reviewer 建议与既有架构决策冲突
- 需要决定是继续做，还是按 YAGNI 直接砍掉某项需求

完成门禁：

- 所有不清楚项都已澄清或升级
- 已采纳项已逐项验证
- 若有 pushback，已经给出技术理由而非情绪化回应

### 8.3 并行调度：`dispatching-parallel-agents`

目标是在多个任务彼此独立时，把串行等待变成并行推进，但不制造冲突。

推荐显式触发：

```text
当前存在多个独立任务。请进入 dispatching-parallel-agents，先拆分独立问题域，再决定哪些可以并行，不要把相关问题硬拆开。
```

它插入主链的位置：

- 属于 `subagent-driven-development` 或调试执行阶段内的加速器
- 不单独替代 `writing-plans`、`requesting-code-review` 或 `verification-before-completion`

本侧链必须输出：

- 独立问题域划分
- 每个 agent 的 scope、约束和预期输出
- 并行执行后的整合结论
- 冲突检查与整体验证结果

固定文档与目录：

- 默认无强制仓库内文件
- 如需沉淀，可把并行拆分策略和集成摘要补进 `docs/ai-changes-<requirement-name>.md`

适合并行的情况：

- 2 个以上 task 彼此独立
- 3 个以上失败分布在不同文件或子系统
- 不同 agent 不会改同一批文件

不适合并行的情况：

- 问题共享同一根因
- 需要先理解全局系统状态
- 不同 agent 会争抢同一写入范围

人类必须参与的点：

- 无法判断任务是否真的独立
- 需要在“更快并行”与“更稳串行”之间做取舍
- 并行结果出现冲突，需要决定优先级

完成门禁：

- 每个 agent 的 scope 已明确且彼此不重叠
- 返回结果已做冲突检查
- 最终集成结果已重新验证，而不是只信任 agent 自报完成

## 9. 推荐提示词模板

### 9.1 需求进入设计

```text
先按 using-superpowers 进入标准流程，然后进入 brainstorming。先不要写代码，请基于当前 superpowers 仓库做需求澄清、影响面分析、方案比较和推荐方案。
```

### 9.2 设计进入计划

```text
设计我确认了。请用 writing-plans 生成 implementation plan，任务粒度控制在 2 到 5 分钟，写清文件、测试和验证命令，不要直接写实现。
```

### 9.3 计划进入开发

```text
开始实现前先执行 using-git-worktrees。然后按 subagent-driven-development 执行 plan，整个过程必须遵守 test-driven-development，先写失败测试，再做最小实现。
```

### 9.4 开发进入评审

```text
当前任务完成后先 requesting-code-review，先看规格符合性，再看代码质量；如果有重要问题，先修掉，不要继续下一个任务。
```

### 9.5 评审进入收尾

```text
先不要说完成。请执行 verification-before-completion，给我 fresh verification 结果；验证通过后，再按 finishing-a-development-branch 给出标准收尾选项。
```

### 9.6 写 / 改 skill 本身

```text
这次交付物是 skill 本身。请在主流程里切换到 writing-skills，按 skill 的 TDD 方式完成新增或修改，不要把它当普通文档直接改。
```

### 9.7 处理 review feedback

```text
现在进入 receiving-code-review。先逐项核实 feedback 的技术正确性和适用范围；不清楚的先问清，再逐项修改并验证。
```

### 9.8 并行调度

```text
当前存在多个独立任务。请进入 dispatching-parallel-agents，先拆分独立问题域，再决定哪些可以并行；并行完成后统一做冲突检查和整体验证。
```

## 10. 一句话总结

在 `superpowers` 仓库里，主开发链路的最佳实践不是“让 AI 一把梭”，而是：

**显式触发 skill + spec 落盘 + plan 落盘 + TDD 小步执行 + 持续 review + fresh verification + 人类把关关键决策；当对象变成 skill、本轮进入 review 反馈处理、或任务可安全并行时，再插入对应侧链。**
