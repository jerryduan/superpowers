# Superpowers Bug 修复最佳实践

> 目标：把 `superpowers` 仓库里的 bug 修复链路收束成一条稳定主线，用显式触发驱动 skill，优先保证根因清楚、回归可复现、修复可验证，而不是追求“先改了再说”。

## 1. 适用范围

本文只覆盖 bug 修复场景，适用于这几类问题：

- 功能行为错误
- 测试失败
- 跨组件集成异常
- 构建或运行时异常
- workflow / prompt / skill 触发行为偏差

本文**不覆盖**从需求到交付的新功能主链路，也不覆盖“如何编写 skill 本身”的专项方法。

## 2. 一条 bug 主线走到底

在 `superpowers` 里，最快的 bug 修复方式不是“先试一个 patch”，而是固定走下面这条主线：

1. `using-superpowers`
2. `systematic-debugging`
3. `using-git-worktrees`
4. `test-driven-development`
5. `requesting-code-review`
6. `verification-before-completion`
7. `finishing-a-development-branch`

本文统一采用**显式触发**，不依赖自动匹配。原则只有 5 条：

- 先排查根因，再谈修复方案。
- 一个阶段只做一类事，不把“定位、修复、评审、收尾”混在一句话里。
- 直接点名 skill 名称，不让模型自己猜。
- 先说清“不要做什么”，尤其是不要直接写代码、不要一次改很多地方。
- 没有证据链、失败测试和 fresh verification，就不算修好。

推荐角色分工：

- 主控代理：负责复现、证据链、根因判断、修复边界、最终验收。
- 执行代理：只负责按已确认根因做最小修复与测试，不自行扩 scope。
- 人类：负责应急优先级、是否允许带病前进、是否接受风险、最终交付方式。

## 3. 一页看懂 bug 流程

| 阶段 | 显式触发 | 主要输出 | 默认目录 | 是否需要人参与 |
| --- | --- | --- | --- | --- |
| 入口统一 | `先按 using-superpowers 进入 bug 修复流程` | 流程共识、阶段边界 | 无固定落盘文件 | 否 |
| 根因排查 | `进入 systematic-debugging，先不要修` | 复现步骤、证据链、根因假设、最小修复方向 | 默认不强制落盘 | 是 |
| 隔离工作区 | `确认根因后再执行 using-git-worktrees` | worktree、基线测试结果 | `.worktrees/` / `worktrees/` / `~/.config/superpowers/worktrees/` | 条件性 |
| 回归测试与修复 | `按 test-driven-development 先写失败测试再修` | regression test、最小修复、相关代码改动 | 业务目录 + 测试目录 + `docs/ai-changes-<requirement-name>.md` | 条件性 |
| 阶段评审 | `修复完成后先 requesting-code-review` | 根因与修复是否匹配、代码质量结论 | 默认不强制落盘 | 条件性 |
| 完成验证 | `先做 verification-before-completion，不要先说修好` | fresh verification 证据、残余风险说明 | 默认不强制落盘 | 否 |
| 收尾交付 | `验证通过后按 finishing-a-development-branch 收尾` | merge / PR / 保留分支 / 丢弃方案 | Git/PR 系统 | 是 |

## 4. 固定产物清单

bug 修复链路里，稳定存在的产物主要有下面 4 类：

| 产物 | 作用 | 路径 |
| --- | --- | --- |
| 复现步骤 | 让问题能稳定重演，避免“修空气” | 默认在会话、issue、PR 或排查记录中 |
| 根因证据 | 证明问题断在哪一层、为什么发生 | 默认在命令输出、日志、截图、PR 评论中 |
| 回归测试 | 把 bug 变成可重复失败再可重复通过的测试 | 对应业务测试目录 |
| AI 变更记录 | 记录决策、修复范围和验证摘要 | `docs/ai-changes-<requirement-name>.md` |

默认**不强制**为 bug 修复额外创建独立 Markdown 文档；如果需要跨会话保留排查结论，建议把关键内容补进 `docs/ai-changes-<requirement-name>.md`。

## 5. 分阶段最佳实践

### 5.1 入口统一：`using-superpowers`

目标是先统一方法，再开始真正排障。

推荐显式触发：

```text
先按 using-superpowers 进入 bug 修复流程。这个问题分阶段处理，先定位根因，不要直接开始修。
```

本阶段输出：

- 明确接下来走的是一条 bug 修复链路
- 明确当前先排查，不直接修改代码

本阶段文档：

- 无固定落盘文件

人类是否必须参与：

- 不必须

完成门禁：

- AI 已明确后续按 bug 修复主线推进
- 当前会话已接受“显式触发 + 证据优先”的工作方式

### 5.2 根因排查：`systematic-debugging`

目标是先证明问题是什么、断在哪里、为什么发生，再进入修复。

推荐显式触发：

```text
进入 systematic-debugging。先不要修代码，请先给我复现步骤、证据链、根因假设和最小修复方向。
```

本阶段必须输出：

- 错误信息或异常现象
- 稳定复现步骤
- 最近变更或相关影响面
- 多层证据链
- 单一根因假设

本阶段默认产物：

- 会话中的排查过程
- 命令输出、日志、截图、trace

如果是多组件问题，至少要回答：

- 问题断在哪一层
- 上一层传入了什么
- 下一层实际收到什么
- 哪一层开始偏离预期

人类必须参与的点：

- 当问题无法稳定复现
- 当基于现有证据存在 2 个以上可能根因
- 当已经尝试 2 次以上仍未验证成功
- 当问题暴露为架构性缺陷而不是单点 bug

完成门禁：

- 已形成可陈述的根因假设
- 已拿到足够证据说明“为什么不是拍脑袋猜的”
- 还没有直接开始修代码

### 5.3 隔离工作区：`using-git-worktrees`

目标是在根因基本明确后，用独立工作区做最小修复，避免把排查性试验污染当前目录。

推荐显式触发：

```text
根因已经基本确认。开始修复前先执行 using-git-worktrees；如果基线测试失败，先停下来告诉我。
```

本阶段必须输出：

- 是否已经在隔离工作区
- 若未隔离，则创建新的 worktree
- 基线依赖安装与 baseline test 结果

默认目录：

- `.worktrees/<branch-name>`
- `worktrees/<branch-name>`
- `~/.config/superpowers/worktrees/<project>/<branch-name>`

本阶段文档：

- 无固定 Markdown 文档

人类必须参与的点：

- 当尚未隔离且需要征求是否创建 worktree 时
- 当 baseline 已失败，需要决定“先修基线”还是“在已知失败背景下继续定位当前 bug”

完成门禁：

- 已确认当前实现环境隔离状态
- 已完成依赖初始化
- 已拿到 fresh baseline test 结果

### 5.4 回归测试与最小修复：`test-driven-development`

目标是先把 bug 变成失败测试，再做最小修复，而不是凭感觉改代码。

推荐显式触发：

```text
按 test-driven-development 修这个 bug。先写最小失败测试并证明它真的失败，再做最小实现，不要顺手重构其它问题。
```

本阶段必须输出：

- 最小复现测试
- 失败测试的实际输出
- 最小修复代码
- 修复后通过的测试结果
- AI 变更记录

固定文档与目录：

- 对应业务测试目录
- 对应业务代码目录
- `docs/ai-changes-<requirement-name>.md`

这里的 AI 变更记录建议至少写：

- bug 现象摘要
- 根因结论
- 修复范围
- regression test
- 已执行验证

人类必须参与的点：

- 当无法把问题稳定变成自动化测试时
- 当修复范围开始从单点扩展成大面积重构时
- 当需要在“临时防护”和“彻底修复”之间做取舍时

完成门禁：

- 失败测试已经被看见并确认是正确失败
- 修复只覆盖根因，不扩 scope
- 修复后相关测试已通过
- 变更记录已更新到 `docs/ai-changes-<requirement-name>.md`

### 5.5 阶段评审：`requesting-code-review`

目标是确认“这次改动确实修到了根因”，而不是只是把症状盖住。

推荐显式触发：

```text
修复完成后先 requesting-code-review，重点检查根因是否对上、修复是否最小、有没有引入新的风险。
```

本阶段必须输出：

- 根因与修复是否一致的评审结论
- 代码质量评审结论
- 待修项或通过结论

固定文档与目录：

- 默认无强制仓库内文件
- 如需沉淀，可把关键 review 结论摘要补进 `docs/ai-changes-<requirement-name>.md`

人类必须参与的点：

- 当 reviewer 认为这是 symptom fix 而非 root cause fix
- 当 reviewer 结论与既有业务判断冲突
- 当修复暴露出更大的架构决策问题

完成门禁：

- 重要 review 问题已关闭
- reviewer 不再认为这只是“遮住症状”
- 才允许进入最终验证

### 5.6 完成验证：`verification-before-completion`

目标是只基于 fresh evidence 说“修好了”，不能靠推测。

推荐显式触发：

```text
先不要说修好了。请执行 verification-before-completion：重新跑证明这个 bug 已修复所需的命令，再告诉我通过项和残余风险。
```

本阶段必须输出：

- 原始复现是否已消失
- regression test 是否通过
- 相关模块测试是否通过
- 是否还存在未覆盖风险

固定文档与目录：

- 默认无强制仓库内文件
- 若团队需要追溯证据，可把关键验证摘要追加到 `docs/ai-changes-<requirement-name>.md`

推荐验证维度：

- 原问题复现路径
- 新增 regression test
- 受影响模块测试
- 必要时的集成验证

人类是否必须参与：

- 不必须

完成门禁：

- 所有“bug 已修复”说法都有 fresh command output 支撑
- 已明确说明通过项、未覆盖项、残余风险

### 5.7 收尾交付：`finishing-a-development-branch`

目标是在验证通过后，用结构化选项完成交付，而不是“修完就散”。

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

## 6. 应急变体

### 6.1 线上紧急 hotfix

线上紧急场景下，可以压缩沟通，但**不能跳过 systematic-debugging、失败测试和 verification**。

推荐显式触发：

```text
这是线上紧急 hotfix。请按 systematic-debugging 快速收敛根因，给出最小修复方案；修复前仍然必须先做失败测试或最小复现，再做 verification。
```

建议采用的压缩链路：

1. `using-superpowers`
2. `systematic-debugging`
3. `using-git-worktrees`
4. `test-driven-development`
5. `verification-before-completion`
6. `finishing-a-development-branch`

hotfix 场景允许压缩的地方：

- 可减少过程性汇报
- review 可聚焦关键风险而非全面优化
- 可优先验证最关键的用户路径

hotfix 场景**不能**压缩的地方：

- 不能跳过根因判断
- 不能直接改线上症状代码试运气
- 不能省掉失败测试或最小复现
- 不能没有 fresh verification 就宣称恢复

必须由人参与的点：

- 确认这真的是 hotfix 而非普通缺陷
- 接受“先止血、后补完”的范围边界
- 决定是否允许只做最小覆盖验证
- 决定最终发布或合并方式

hotfix 完成门禁：

- 已明确止血目标
- 已明确哪些验证做了、哪些因为时效没有做
- 已明确是否需要后续补充回归、补评审或补长期修复

### 6.2 基线已坏时的应急修复

如果在开始修 bug 前，baseline 就已经失败，最佳实践不是假装全绿，而是先把“旧问题”和“本次问题”切开。

推荐显式触发：

```text
当前 baseline 已坏。请先按 systematic-debugging 区分既有失败和本次 bug，给出可隔离的最小复现和修复边界；没有边界前不要直接修。
```

本变体必须额外输出：

- baseline 失败清单
- 与本次 bug 直接相关的失败项
- 与本次 bug 无关但已存在的失败项
- 是否需要先修基线的建议

推荐决策顺序：

1. 先识别 baseline 中哪些失败与当前 bug 无关
2. 为当前 bug 找到独立复现方式
3. 在独立失败测试上完成修复
4. 最终验证时明确“当前 bug 已修复”与“仓库 baseline 仍有旧失败”这两个事实

必须由人参与的点：

- 决定是否先修 baseline
- 决定是否接受“当前 bug 修好，但仓库仍非全绿”
- 决定是否拆成两个问题分别处理

完成门禁：

- 当前 bug 的独立复现已被修复
- baseline 历史失败没有被错误描述成“本次修复引入”
- 最终汇报中已清楚区分“本次修复结果”和“既有系统状态”

## 7. 必须有人参与的节点

bug 修复链路里，下面 6 个节点不应该完全交给 AI 自行决定：

1. 是否进入应急模式
   普通 bug、线上 hotfix、基线已坏，这三种场景的人类优先级判断不同。

2. 根因升级
   当 2 次以上尝试仍无法验证，或开始暴露架构性问题时，必须由人判断是否改方向。

3. 自动化测试可行性判断
   当 bug 难以自动化复现时，需要由人判断是否接受最小手工复现脚本或观测性补强。

4. 修复范围取舍
   是做最小止血，还是顺带做彻底治理，必须由人决定。

5. baseline 异常处置
   仓库本身已坏时，必须由人决定先修基线还是继续隔离修当前 bug。

6. 最终交付选择
   merge、PR、保留分支、丢弃工作，必须由人决定。

## 8. 最容易踩坑的反模式

- 还没做 `systematic-debugging` 就开始改代码
- 只凭报错文本猜问题，不做复现和证据链
- 一次改很多地方，看哪个能碰巧好
- 没看到失败测试就直接补实现
- 把“症状消失”当成“根因已修”
- baseline 已坏却仍声称“全部通过”
- hotfix 场景下用“先救火”为理由跳过验证
- 没有 fresh verification 就说“修好了”

## 9. 推荐提示词模板

### 9.1 普通 bug 修复

```text
先按 using-superpowers 进入 bug 修复流程，然后进入 systematic-debugging。先不要改代码，请先给我复现步骤、证据链、根因假设和最小修复方向。
```

### 9.2 根因确认后进入修复

```text
根因已经基本确认。开始修复前先执行 using-git-worktrees。然后按 test-driven-development 先写失败测试，再做最小修复，不要顺手扩 scope。
```

### 9.3 修复后进入评审

```text
修复完成后先 requesting-code-review，重点检查这是不是 root cause fix，而不是 symptom fix；有重要问题先修，不要直接收尾。
```

### 9.4 进入最终验证

```text
先不要说修好了。请执行 verification-before-completion，重新跑能证明这个 bug 已修复的命令，再告诉我通过项、未覆盖项和残余风险。
```

### 9.5 线上紧急 hotfix

```text
这是线上紧急 hotfix。请按 systematic-debugging 快速收敛根因，给出最小修复方案；修复前仍然必须先做失败测试或最小复现，再做 verification。
```

### 9.6 基线已坏时的修复

```text
当前 baseline 已坏。请先区分既有失败和本次 bug，给出可隔离的最小复现与修复边界；没有边界前不要直接修。
```

## 10. 一句话总结

在 `superpowers` 仓库里，bug 修复的最佳实践不是“先 patch 再解释”，而是：

**显式触发 skill + 根因先行 + 失败测试先行 + 最小修复 + 评审确认不是 symptom fix + fresh verification + 人类把关应急与交付决策。**
