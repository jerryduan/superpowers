# Superpowers

Superpowers 是一套面向你的编码代理的完整软件开发工作流，构建在一组可组合的“skills”之上，并配有一套初始指令，用来确保你的代理会使用这些 skills。

## 工作原理

它从你启动编码代理的那一刻开始工作。只要代理发现你在构建某个东西，它*不会*立刻跳进去写代码。相反，它会先退一步，问清楚你真正想完成的是什么。

一旦它从对话里梳理出一份规格说明，就会把内容拆成足够短的片段展示给你，方便你真正读完并理解。

当你确认设计后，代理会整理出一份实现计划。这份计划会清晰到足以让一个热情很高、品味欠佳、缺少判断力、不了解项目上下文、而且不爱测试的初级工程师也能照着执行。它强调真正的红/绿 TDD、YAGNI（You Aren't Gonna Need It，你不会需要它）以及 DRY。

接下来，只要你说一句“开始”，它就会启动一个 *subagent-driven-development* 流程，让多个代理分别处理每项工程任务、检查和审查彼此的工作，并持续推进。Claude 通常可以连续数小时自主工作，同时依然严格遵循你们一起制定的计划，这种情况并不少见。

系统里还有很多其他内容，但以上就是核心机制。并且因为这些 skills 会自动触发，你不需要做任何额外操作。你的编码代理会直接获得 Superpowers。

## 赞助

如果 Superpowers 帮你完成了能带来收入的工作，而你也愿意支持，我会非常感谢你考虑[赞助我的开源工作](https://github.com/sponsors/obra)。

谢谢！

- Jesse

## 安装

**注意：** 不同平台的安装方式不同。Claude Code 和 Cursor 有内置插件市场；Codex 和 OpenCode 需要手动配置。

### Claude Code 官方市场

你可以通过[官方 Claude 插件市场](https://claude.com/plugins/superpowers)获取 Superpowers。

在 Claude 市场中安装插件：

```bash
/plugin install superpowers@claude-plugins-official
```

### Claude Code（通过插件市场）

在 Claude Code 中，先注册这个市场：

```bash
/plugin marketplace add obra/superpowers-marketplace
```

然后从这个市场安装插件：

```bash
/plugin install superpowers@superpowers-marketplace
```

### Cursor（通过插件市场）

在 Cursor Agent 聊天中，从市场安装：

```text
/add-plugin superpowers
```

或者直接在插件市场里搜索 “superpowers”。

### Codex

对 Codex 这样说：

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md
```

**详细文档：** [docs/README.codex.md](docs/README.codex.md)

### OpenCode

对 OpenCode 这样说：

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

**详细文档：** [docs/README.opencode.md](docs/README.opencode.md)

### Gemini CLI

```bash
gemini extensions install https://github.com/obra/superpowers
```

更新命令：

```bash
gemini extensions update superpowers
```

### 验证安装

在你选择的平台里开启一个新会话，然后提出一个本应触发某项 skill 的请求（例如“帮我规划这个功能”或“我们来排查这个问题”）。代理应该会自动调用对应的 superpowers skill。

## 基础工作流

1. **brainstorming** - 在写代码之前触发。通过提问打磨粗略想法、探索替代方案，并将设计分段展示给你确认。最后保存设计文档。

2. **using-git-worktrees** - 在设计获批后触发。会在新分支上创建隔离工作区，运行项目初始化，并验证测试基线是干净的。

3. **writing-plans** - 在设计获批后触发。把工作拆成一个个小任务（每个 2 到 5 分钟）。每个任务都包含精确文件路径、完整代码和验证步骤。

4. **subagent-driven-development** 或 **executing-plans** - 在有计划后触发。为每个任务分派全新的子代理并执行两阶段审查（先看是否符合规格，再看代码质量），或者按批次执行并设置人工检查点。

5. **test-driven-development** - 在实现阶段触发。强制执行 RED-GREEN-REFACTOR：先写失败测试，看它失败；再写最少代码，看它通过；然后提交。任何先于测试写出的代码都会被删除。

6. **requesting-code-review** - 在任务之间触发。对照计划进行审查，并按严重程度报告问题。关键问题会阻止继续推进。

7. **finishing-a-development-branch** - 在任务完成时触发。验证测试、展示可选项（合并 / PR / 保留 / 丢弃），并清理 worktree。

**代理会在执行任何任务前先检查是否有相关 skill。** 这些是强制工作流，不是建议而已。

## 包含内容

### Skills 库

**测试**
- **test-driven-development** - RED-GREEN-REFACTOR 循环（包含测试反模式参考）

**调试**
- **systematic-debugging** - 四阶段根因定位流程（包含 root-cause-tracing、defense-in-depth、condition-based-waiting 等技术）
- **verification-before-completion** - 确保问题真的已经修复

**协作**
- **brainstorming** - 苏格拉底式设计澄清
- **writing-plans** - 详细实现计划
- **executing-plans** - 带检查点的批量执行
- **dispatching-parallel-agents** - 并行子代理工作流
- **requesting-code-review** - 预审查清单
- **receiving-code-review** - 如何响应反馈
- **using-git-worktrees** - 并行开发分支
- **finishing-a-development-branch** - 合并 / PR 决策流程
- **subagent-driven-development** - 带两阶段审查的快速迭代（先看是否符合规格，再看代码质量）

**元能力**
- **writing-skills** - 按最佳实践创建新 skill（包含测试方法）
- **using-superpowers** - skills 系统介绍

## 理念

- **测试驱动开发** - 永远先写测试
- **系统化优先于临时应对** - 用流程取代猜测
- **降低复杂度** - 以简洁为第一目标
- **证据优先于声称** - 在宣布成功前先验证

了解更多：[Superpowers for Claude Code](https://blog.fsck.com/2025/10/09/superpowers/)

## 贡献

Skills 直接存放在这个仓库里。若要贡献：

1. Fork 这个仓库
2. 为你的 skill 创建一个分支
3. 按照 `writing-skills` skill 的方式创建并测试新的 skill
4. 提交 PR

完整指南请见 `skills/writing-skills/SKILL.md`。

## 更新

当你更新插件时，skills 会自动更新：

```bash
/plugin update superpowers
```

## 许可证

MIT License，详情见 LICENSE 文件

## 支持

- **Issues**: https://github.com/obra/superpowers/issues
- **Marketplace**: https://github.com/obra/superpowers-marketplace
