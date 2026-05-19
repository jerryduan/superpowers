# Superpowers AI Coding 最佳实践

> 目标：在当前 `superpowers` 项目中，建立一套“先保证质量，再追求速度”的 AI Coding 标准打法，把返工成本降到最低，把有效开发速度提到最高。

## 1. 先统一一个原则

在这个项目里，**最快的开发方式不是“立刻写代码”**，而是：

1. 先把需求和边界说清楚
2. 再把实现拆成极小任务
3. 每个任务都走 TDD
4. 每个任务都做 review 和 fresh verification
5. 最后再做联调和收口

Superpowers 已经把这套流程拆成了现成 skills。最佳实践不是绕过这些 skills，而是把它们串成一个稳定流水线。

推荐把 AI 分成两种角色：

- **主控代理**：负责澄清需求、沉淀 spec、写 plan、调度子代理、汇总结论、把关验收。
- **执行代理**：只负责按任务实现、补测试、修 review 问题，不自己扩 scope。

这样做的核心收益是：

- 速度快：减少“写了又推翻”的返工
- 质量高：每一步都有测试、评审、验证兜底
- 可追踪：spec、plan、review、测试命令都有落盘证据

## 2. 结合当前仓库的项目特征

这个仓库本身就是一个由 skills 驱动的开发系统，主要改动面集中在这些目录：

- `skills/`：各类流程技能本体
- `commands/`：对外暴露的命令入口
- `agents/`：评审/执行等代理提示模板
- `hooks/`：会话和自动化钩子
- `.codex/`、`.opencode/`、`.claude-plugin/`、`.cursor-plugin/`：多平台接入层
- `tests/`：技能触发、平台接入、brainstorm server 等测试
- `docs/superpowers/specs/`、`docs/superpowers/plans/`：需求设计与实现计划沉淀位置

因此，这个项目的 AI Coding 最佳实践必须满足两个约束：

- 既要保证某个 skill 或插件改动本身正确
- 也要保证它不会破坏其他平台或其他工作流

## 3. 最推荐的标准流水线

对于绝大多数开发任务，建议固定使用下面这条流水线：

1. `using-superpowers`
2. `brainstorming`
3. `using-git-worktrees`
4. `writing-plans`
5. `subagent-driven-development`
6. `test-driven-development`
7. `requesting-code-review`
8. `verification-before-completion`
9. `finishing-a-development-branch`

如果是 bug 修复，则把第 2 步替换成：

1. `using-superpowers`
2. `systematic-debugging`
3. `test-driven-development`
4. `requesting-code-review`
5. `verification-before-completion`

这条顺序不要轻易改，因为它对应的是“先理解，再实现；先证明问题，再证明修复；先局部闭环，再整体交付”。

### 3.1 Skills 的触发总规则

在 Superpowers 里，skill 触发有两种方式：

1. **自动匹配触发**
   AI 根据用户任务和 skill 的 `description` 自动判断是否适用。
2. **显式点名触发**
   用户在对话里直接说出 skill 名称，或者明确要求按某条流程执行。

要想触发稳定，建议优先遵守下面 4 条：

- **把任务说成动作，不要只说目标。**
  例如“帮我设计并拆计划”比“做一个新功能”更容易稳定触发 `brainstorming` 和 `writing-plans`。
- **把阶段说清楚。**
  例如“先不要写代码”“先定位根因”“开始执行计划”“准备收尾”这类阶段信号，对 skill 触发非常关键。
- **尽量点名 skill 或流程名。**
  比如直接说“按 `systematic-debugging` 流程排查”“进入 `subagent-driven-development` 执行”。
- **一个提示词只强调一个主阶段。**
  如果一句话里同时要求“设计、开发、评审、提 PR”，自动触发容易混乱；更好的做法是按阶段推进。

### 3.2 当前项目最关键 skills 的触发方式

下面这张表适合直接当“提示词速查表”来用。

| Skill | 何时触发 | 最容易触发的表达 | 不建议的模糊说法 |
| --- | --- | --- | --- |
| `using-superpowers` | 新会话开始、任何任务开始前 | `先按 superpowers 流程来处理这个任务` | `帮我看看` |
| `brainstorming` | 新功能、行为变更、需求设计、创意方案讨论 | `先不要写代码，先做需求澄清和方案设计` | `把这个功能做了` |
| `using-git-worktrees` | 进入正式实现前，需要隔离工作区时 | `开始实现前先创建隔离 worktree` | `新建个分支吧` |
| `writing-plans` | 已有 spec / 需求，准备进入实现前 | `基于确认后的方案，拆成可执行计划` | `你自己安排着做` |
| `subagent-driven-development` | 已有实现计划，任务相对独立，准备在当前会话执行 | `按 subagent-driven-development 执行这个 plan` | `开始做吧` |
| `executing-plans` | 已有实现计划，但不适合用子代理串行执行时 | `按 executing-plans 分批执行并在检查点停下` | `按计划推进` |
| `test-driven-development` | 新功能、bug 修复、重构、行为变更 | `先写失败测试，再做最小实现` | `顺手补点测试` |
| `requesting-code-review` | 单个任务完成后、主要功能完成后、合并前 | `这一步做完后先发起 code review，再继续` | `帮我检查一下` |
| `systematic-debugging` | 任何 bug、测试失败、异常行为、集成问题 | `先按 systematic-debugging 排查根因，不要直接修` | `这个报错修一下` |
| `verification-before-completion` | 准备说“完成了”、准备提交、准备提 PR 前 | `不要直接说完成，先跑验证命令给我结果` | `应该没问题了吧` |
| `finishing-a-development-branch` | 实现完成且验证通过后，需要合并 / PR / 清理时 | `测试通过后，按 finishing-a-development-branch 收尾` | `收个尾` |

### 3.3 面向当前仓库的推荐触发句式

这个仓库最常见的是 skill、workflow、跨平台兼容、测试体系这几类改动，所以建议直接使用下面这种句式。

#### 需求设计阶段

目标是稳定触发 `brainstorming`。

推荐说法：

```text
我要改 superpowers 里的 <skill / workflow / plugin 行为>。先不要写代码，请基于当前仓库做需求澄清、影响面分析和方案比较。
```

```text
这是一个行为变更，不是简单改文案。请先进入 brainstorming，给我 2 到 3 个方案和推荐方案。
```

#### 计划拆解阶段

目标是稳定触发 `writing-plans`。

推荐说法：

```text
设计我确认了。请基于当前 spec 生成一个可执行 implementation plan，任务粒度控制在 2 到 5 分钟，并写清文件、测试和验证命令。
```

```text
请进入 writing-plans，把这个需求拆成能交给子代理执行的计划，不要直接写实现。
```

#### 正式开发阶段

目标是稳定触发 `using-git-worktrees`、`subagent-driven-development`、`test-driven-development`。

推荐说法：

```text
开始实现前，先创建隔离 worktree。然后按 subagent-driven-development 执行计划，所有实现必须走 TDD。
```

```text
请在当前 superpowers 仓库里按 worktree + plan + TDD 的方式实现，不要跳过 failing test。
```

#### Bug 排查阶段

目标是稳定触发 `systematic-debugging` 和 `test-driven-development`。

推荐说法：

```text
这个问题先不要直接修。请按 systematic-debugging 排查根因，给出复现、证据链和最小修复方案；修复时必须先写失败测试。
```

```text
这是一个跨组件问题。请先定位断在哪一层，再把 bug 变成 regression test，最后做最小修复。
```

#### Review 阶段

目标是稳定触发 `requesting-code-review`。

推荐说法：

```text
当前这一步完成后先不要继续下一个任务，请先发起 code review，分开检查规格符合性和代码质量。
```

```text
这个功能完成后，先 review 再继续；如果有重要问题，先修掉。
```

#### 完成与收尾阶段

目标是稳定触发 `verification-before-completion` 和 `finishing-a-development-branch`。

推荐说法：

```text
先不要说完成。请先跑 fresh verification，确认测试和关键命令都通过，再按 finishing-a-development-branch 给出合并或 PR 建议。
```

```text
当你准备收尾时，先验证，再给我 merge / PR / 保留分支 / 清理 worktree 这几个选项。
```

### 3.4 哪些提示词最容易导致触发失败

下面这些说法常常会让 skill 触发变得不稳定，或者直接把 AI 引向“先写代码”的路径：

- `帮我搞一下这个`
- `做了它`
- `这个报错修一下`
- `你看着办`
- `顺手把测试也补了`
- `全部做完再告诉我`

更好的改写方式是：

- 把**阶段**讲清楚：先设计 / 先排查 / 开始执行 / 开始收尾
- 把**约束**讲清楚：不要直接写代码 / 必须先写失败测试 / 必须先 review
- 把**输出物**讲清楚：spec、plan、diff、验证命令、PR 建议

### 3.5 一个实用原则：想要稳定，就显式点名

如果你对自动匹配没有把握，最佳实践不是“多解释一点”，而是**直接点名 skill**。

比如：

- 想先设计，就直接说 `进入 brainstorming`
- 想先排障，就直接说 `按 systematic-debugging 排查`
- 想先拆计划，就直接说 `使用 writing-plans`
- 想按严格开发流水线推进，就直接说 `按 using-git-worktrees + writing-plans + subagent-driven-development + TDD 来做`

对这个项目而言，**显式点名 skill 名称 + 说明当前阶段**，是最稳定、最快速的触发方式。

## 4. 场景一：项目 0 启动开发

这里的“项目 0”包括两类场景：

- 从 0 到 1 新增一项 skill / command / plugin 能力
- 现有能力很大，需要当成一个独立子项目重新启动

### 推荐流程

1. 用 `brainstorming` 先收敛需求，不允许直接开写。
2. 输出 2 到 3 个技术方案，明确推荐方案和放弃方案。
3. 将确认后的设计写入 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`。
4. 使用 `using-git-worktrees` 在独立 worktree 和新分支上执行，避免污染主工作区。
5. 用 `writing-plans` 生成可执行计划，任务粒度控制在 2 到 5 分钟。
6. 进入 `subagent-driven-development`，每个任务一个新子代理。
7. 每个任务完成后先做规格符合性 review，再做代码质量 review。
8. 所有任务完成后做一次总 review，再进入分支收尾。

### 最佳实践要点

- spec 不要写成“大而全愿景文档”，只写当前 iteration 真正要做的最小闭环。
- plan 里必须写清楚修改文件、测试文件、命令和预期结果，不能只写“补充实现”这种抽象描述。
- 新能力优先做成可验证的小能力，再在下一轮迭代扩展；不要第一轮就做成“大一统框架”。
- 对这个仓库来说，新增 skill 时要同时考虑文档、触发规则、测试入口和多平台可见性。

### 推荐提示词

```text
我想在 superpowers 项目里新增一个 <能力名>。请先基于当前仓库结构做需求澄清和方案比较，不要直接写代码。输出分阶段设计，待我确认后再进入 plan。
```

## 5. 场景二：需求迭代

这是本项目最常见的场景，比如：

- 调整某个 skill 的触发条件
- 增加某个 workflow 的约束
- 为 Codex / OpenCode / Claude 增加新的兼容行为
- 补文档、补命令、补测试，但不想推翻原有设计

### 推荐流程

1. 把这次需求当成“增量设计”，而不是直接 patch。
2. 先让 AI 识别受影响的目录、文件、测试范围和下游平台。
3. 用 `brainstorming` 产出 delta design，只描述“本次要改什么、为什么改、不改什么”。
4. 用 `writing-plans` 生成一个小计划，不需要把整个系统重写一遍。
5. 使用 `subagent-driven-development` 或 `executing-plans` 分批执行。
6. 每一批结束都做 review 和 verification，再进入下一批。
7. 结束前做一次集成联调，确认没有破坏其他工作流。

### 最佳实践要点

- 每次迭代只解决一个真实问题，不顺手打包多个 unrelated 优化。
- 明确“兼容旧行为”还是“有意改变行为”，并把行为变化写进 spec 和 plan。
- 如果改动的是 skill 说明文字，也要考虑测试断言是否依赖这些文字。
- 如果改动的是提示词或 workflow，优先补“行为测试”而不是只补静态文本测试。

### 推荐提示词

```text
这是一次需求迭代，不是重做。请先基于当前 superpowers 仓库识别本次需求影响面，给出最小改动方案、回归测试范围和联调清单；我确认后再进入实现。
```

## 6. 场景三：Bug 修复

对于 bug 修复，最佳实践不是“尽快改”，而是“尽快找到根因并形成回归保护”。

### 推荐流程

1. 先触发 `systematic-debugging`，禁止拍脑袋修。
2. 明确复现步骤、错误信息、影响范围、最近变更。
3. 如果链路跨多个组件，先加诊断信息，确认断点在哪一层。
4. 在根因明确后，先写失败的回归测试。
5. 用 `test-driven-development` 做最小修复。
6. 修复后跑 bug 对应测试、相关模块测试、必要的集成测试。
7. 做 code review，确认不是 symptom fix。
8. 用 `verification-before-completion` 重新执行验证命令，确认输出。

### 最佳实践要点

- 任何“先修一下再说”的冲动都要压住，尤其是 workflow、prompt、跨平台兼容 bug。
- 对于 skill 相关 bug，往往真正的问题不在文本本身，而在触发条件、优先级、上下文传递或平台差异。
- bug 修复完成后，一定要留下 regression test；否则只是“这次碰巧修好了”。

### 推荐提示词

```text
请按 superpowers 的 systematic-debugging 流程处理这个问题：先定位根因，给出证据链和最小复现，再设计回归测试与最小修复方案，不要直接改代码。
```

## 7. 按流程阶段的最佳实践

下面这部分是“无论什么场景都适用”的阶段性建议。

### 7.1 需求设计

目标是把“用户想法”变成“可实现、可验证、可拒绝 scope creep 的设计输入”。

推荐做法：

- 先读当前仓库上下文，再开始问问题。
- 一次只问一个关键问题，优先问目标、边界、兼容性、验收标准。
- 至少给出 2 到 3 个方案，并明确推荐方案。
- 设计必须覆盖：架构边界、改动范围、异常处理、测试策略、回滚策略。
- 设计确认后必须落盘，避免后续对话漂移。

对本仓库尤其重要的是：

- 修改 `skills/` 时，要明确是改“行为规则”还是改“说明表达”。
- 修改多平台目录时，要明确哪些平台必须保持一致，哪些可以差异化。

### 7.2 技术方案设计

目标是把需求设计进一步压缩成“一个工程师可以稳定执行”的结构化方案。

推荐做法：

- 先做文件结构映射，列出新增文件、修改文件和测试文件。
- 把每个文件的职责说清楚，避免一个文件承担过多角色。
- 把高风险点提前写出来，例如跨平台兼容、prompt 行为漂移、测试脆弱性。
- 主动写清楚“不做什么”，用来防止需求膨胀。

### 7.3 代码开发

目标是把大任务拆成低风险、可回退、可 review 的连续小步。

推荐做法：

- 严格按 `writing-plans` 的小任务粒度拆解，每一步 2 到 5 分钟。
- 优先让子代理执行机械性任务，主控代理只做调度和验收。
- 每个任务都带完整上下文，不让子代理自己猜。
- 每个任务结束最好有一次小提交，保持 diff 可读。

对本仓库的建议：

- 改 `skills/*` 时不要同时大改 `agents/*` 和 `commands/*`，除非 spec 明确要求。
- 改测试时，优先先让测试表达预期行为，再让实现追上来。

### 7.4 代码单元测试

这里必须坚持 TDD，因为本项目的很多改动都属于“行为规则变更”，最怕“看起来合理、实际上没被验证”。

推荐做法：

- 先写失败测试，再写实现。
- 测试名称要表达业务行为，而不是表达内部实现。
- 尽量测真实行为，少测 mock 交互。
- 修 bug 时必须先把 bug 变成可重复失败的测试。
- 每次 green 之后再考虑重构，不要把重构、修复、增强揉在一起。

### 7.5 代码评审

推荐固定执行两层 review：

1. **规格符合性 review**
   检查是否严格满足 spec / plan，是否漏需求、超 scope、改坏边界。

2. **代码质量 review**
   检查可读性、可维护性、测试质量、命名、常量抽取、错误处理等。

最佳实践：

- 不要把 review 留到最后一次做；每个任务或每一小批任务后都做。
- 规格问题必须在继续前修掉，避免后面叠加返工。
- review 的输入应是精确的任务上下文和 diff，而不是整个会话历史。

### 7.6 联调

联调的目标不是“再看一眼”，而是验证这次改动在真实集成边界上成立。

这个仓库的联调建议按受影响面选择：

- 改 `skills/`：至少验证 skill 触发和核心行为
- 改 `.opencode/`：跑 OpenCode 插件加载和相关测试
- 改 `tests/brainstorm-server/`：跑 Node 测试
- 改跨平台共享逻辑：做一次跨目录回归检查

联调时推荐回答三个问题：

1. 本次改动最直接影响哪条工作流？
2. 哪些相邻工作流最容易被误伤？
3. 什么命令能提供 fresh evidence 证明“它真的可用”？

## 8. 当前仓库的测试与验证映射

为了兼顾质量和效率，测试不要无脑全量跑，而要按改动面分层执行。

### 第一层：最小验证

- 文档改动：检查链接、结构、diff 和格式即可
- 单个 skill 文案改动：先跑对应 skill 相关测试
- 单个脚本或 server 改动：先跑对应目录下的单元测试

### 第二层：模块级验证

- `tests/brainstorm-server/`
  运行：
  ```bash
  cd tests/brainstorm-server && npm test
  ```

- `tests/skill-triggering/`
  运行：
  ```bash
  ./tests/skill-triggering/run-all.sh
  ```

- `tests/opencode/`
  运行：
  ```bash
  ./tests/opencode/run-tests.sh
  ```

- `tests/claude-code/`
  快速验证：
  ```bash
  ./tests/claude-code/run-skill-tests.sh
  ```
  集成验证：
  ```bash
  ./tests/claude-code/run-skill-tests.sh --integration
  ```

### 第三层：发布前验证

在以下情况建议做更完整的回归：

- 改动多个 skills 的协作顺序
- 改动 review / plan / debugging 这类核心流程
- 改动多平台接入目录
- 改动会影响已有测试基线的共享逻辑

发布前至少要确认：

- 目标行为通过
- 相关回归测试通过
- 没有引入明显的跨平台破坏
- 文档与实现一致

## 9. 如何做到“质量不降的最快速度”

关键不是让 AI 写得更快，而是让 AI **少走弯路**。

推荐遵守下面 6 条：

1. **先设计再实现**
   没有 design 和 plan，就不要让 AI 直接开始改代码。

2. **一个会话只做一件事**
   一个主目标一个 worktree，一个 spec 一个 plan，避免上下文污染。

3. **小步快跑，不做大跃进**
   每个任务 2 到 5 分钟，问题更早暴露，回退更容易。

4. **让便宜代理干活，让强代理把关**
   机械实现用快模型，设计和 review 用强模型，兼顾成本与质量。

5. **验证只看 fresh evidence**
   没有刚跑过的命令输出，就不要说“已经好了”。

6. **把返工前置到最便宜的阶段**
   在 design 阶段推翻方案，成本远低于在联调阶段推翻实现。

## 10. 推荐的团队使用方式

如果团队要长期使用 Superpowers，建议统一成下面这种协作模式：

- 产品 / 需求方负责确认目标、边界、验收标准
- 主控 AI 负责设计、计划、调度、验收
- 执行 AI 负责按任务实现和补测试
- 人类工程师重点审核设计质量、边界判断和最终交付物

最好的分工不是“人写代码，AI 打下手”，也不是“AI 全包，人不管”，而是：

- 人类决定方向、约束、取舍
- AI 负责拆解、执行、验证、暴露问题

## 11. 可直接复用的提示词模板

### 项目 0 启动

```text
请基于当前 superpowers 仓库，按 brainstorming → writing-plans → subagent-driven-development 的方式推进这个新能力。先做需求设计和技术方案比较，不要直接写代码。目标是在保证质量的前提下实现最小可用闭环。
```

### 需求迭代

```text
这是一次增量迭代。请先识别本次需求影响的 skills、commands、agents、tests 和平台目录，输出最小改动方案、验证策略和联调范围，待我确认后再进入实现。
```

### Bug 修复

```text
请按 systematic-debugging + TDD 的流程处理这个 bug：先给出复现、证据链和根因分析，再设计失败测试和最小修复，最后给出需要跑的回归与联调命令。
```

### 合并前验收

```text
请不要直接说完成。先基于这次 diff 生成验证清单，逐项执行 verification-before-completion 所需命令，再告诉我哪些已确认、哪些仍有风险。
```

## 12. 明确禁止的反模式

- 没有 spec 就直接开始实现
- plan 过于粗糙，只写目标不写步骤
- 不写失败测试，直接补实现
- 修 bug 只改症状，不找根因
- 一个任务里混入需求变更、重构、修复三件事
- 把 review 放到最后一次才做
- 没有 fresh verification 就宣称“完成”
- 为了省几分钟跳过联调，最后花几小时返工

## 13. 一句话总结

在 `superpowers` 项目里，**最高效的 AI Coding 不是让 AI 更激进，而是让流程更严格**：

**设计前置 + 小步计划 + TDD + 持续 review + fresh verification + 按影响面联调**，这就是“保证代码质量基础上的最快开发效率”。
