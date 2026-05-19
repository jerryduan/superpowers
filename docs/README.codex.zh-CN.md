# 适用于 Codex 的 Superpowers

这是一份通过原生 skill 发现机制在 OpenAI Codex 中使用 Superpowers 的指南。

## 快速安装

对 Codex 这样说：

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md
```

## 手动安装

### 前置条件

- OpenAI Codex CLI
- Git

### 步骤

1. 克隆仓库：
   ```bash
   git clone https://github.com/obra/superpowers.git ~/.codex/superpowers
   ```

2. 创建 skills 的符号链接：
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/superpowers/skills ~/.agents/skills/superpowers
   ```

3. 重启 Codex。

4. **针对子代理类 skill**（可选）：像 `dispatching-parallel-agents` 和 `subagent-driven-development` 这样的 skill 需要 Codex 的 collab 功能。请将以下配置加入你的 Codex 配置中：
   ```toml
   [features]
   collab = true
   ```

### Windows

请使用 junction 代替符号链接（无需开启 Developer Mode 也能工作）：

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
cmd /c mklink /J "$env:USERPROFILE\.agents\skills\superpowers" "$env:USERPROFILE\.codex\superpowers\skills"
```

## 工作原理

Codex 原生支持 skill 发现机制，它会在启动时扫描 `~/.agents/skills/`，解析 `SKILL.md` 的 frontmatter，并按需加载对应的 skill。Superpowers 的 skill 会通过一个符号链接对 Codex 可见：

```
~/.agents/skills/superpowers/ → ~/.codex/superpowers/skills/
```

`using-superpowers` 这个 skill 会被自动发现，并负责强制执行 skill 使用纪律，不需要额外配置。

## 使用方式

Skills 会被自动发现。Codex 会在以下场景激活它们：
- 你按名称提到某个 skill（例如 “use brainstorming”）
- 当前任务符合某个 skill 的描述
- `using-superpowers` skill 指示 Codex 去使用它

### 个人 Skills

你可以在 `~/.agents/skills/` 中创建自己的 skill：

```bash
mkdir -p ~/.agents/skills/my-skill
```

创建 `~/.agents/skills/my-skill/SKILL.md`：

```markdown
---
name: my-skill
description: Use when [condition] - [what it does]
---

# My Skill

[Your skill content here]
```

`description` 字段决定了 Codex 何时会自动激活这个 skill，所以请把它写成清晰的触发条件说明。

## 更新

```bash
cd ~/.codex/superpowers && git pull
```

由于使用了符号链接，skills 会立刻更新生效。

## 卸载

```bash
rm ~/.agents/skills/superpowers
```

**Windows（PowerShell）：**
```powershell
Remove-Item "$env:USERPROFILE\.agents\skills\superpowers"
```

如有需要，也可以删除克隆目录：`rm -rf ~/.codex/superpowers`（Windows：`Remove-Item -Recurse -Force "$env:USERPROFILE\.codex\superpowers"`）。

## 故障排查

### Skills 没有显示

1. 验证符号链接：`ls -la ~/.agents/skills/superpowers`
2. 检查 skills 是否存在：`ls ~/.codex/superpowers/skills`
3. 重启 Codex，因为 skills 会在启动时被发现

### Windows junction 问题

Junction 通常不需要特殊权限即可工作。如果创建失败，尝试以管理员身份运行 PowerShell。

## 获取帮助

- 提交问题：https://github.com/obra/superpowers/issues
- 主文档：https://github.com/obra/superpowers
