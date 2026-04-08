# Knowledge Save — 安装指引

## 简介

`knowledge-save` 是一个将 AI 会话关键进展沉淀到 Obsidian 知识库的 skill。

当前推荐安装方式是：
- 安装 `knowledge-save`
- 安装 `obsidian-skills` 依赖
- 配置 Obsidian Vault 路径

本项目的可安装运行时目录是 `runtime/knowledge-save/`。

## 前提条件

### 1. Obsidian Vault

你需要一个 Obsidian Vault 的绝对路径。

知识库默认会使用三层目录：

- `daily/`
- `topic/`
- `note/`

### 2. 安装 obsidian-skills 依赖

`knowledge-save` 依赖 [obsidian-skills](https://github.com/kepano/obsidian-skills) 中的 `obsidian-markdown`。

`knowledge-save` 会使用 `obsidian-markdown` 规范来组织 Markdown 内容。安装完成后，请确认以下 4 个文件存在：

- `obsidian-markdown/SKILL.md`
- `obsidian-markdown/references/PROPERTIES.md`
- `obsidian-markdown/references/CALLOUTS.md`
- `obsidian-markdown/references/EMBEDS.md`

因此，`obsidian-skills` 需要安装在与 `knowledge-save` 相同的 skills 根目录下。

## Step 1: 获取项目文件

```bash
git clone https://github.com/Zzzia/knowledge-save-skill.git
cd knowledge-save-skill
```

## Step 2: 替换占位符

### 2a. Vault 路径

把 `runtime/knowledge-save/SKILL.md` 中的 `{{VAULT_PATH}}` 替换为你的 Obsidian Vault 绝对路径。

示例：

```bash
# Linux
sed -i 's|{{VAULT_PATH}}|/home/yourname/obsidian/my-notes|g' runtime/knowledge-save/SKILL.md

# macOS
sed -i '' 's|{{VAULT_PATH}}|/home/yourname/obsidian/my-notes|g' runtime/knowledge-save/SKILL.md
```

## Step 3: 初始化 Obsidian 目录结构

```bash
OBSIDIAN_DIR="/home/yourname/obsidian/my-notes"

mkdir -p "$OBSIDIAN_DIR/daily"
mkdir -p "$OBSIDIAN_DIR/topic"
mkdir -p "$OBSIDIAN_DIR/note"
```

如果 Vault 已经被 Obsidian 打开过，可以顺手配置 Daily Notes：

```bash
if [ -d "$OBSIDIAN_DIR/.obsidian" ]; then
  echo '{"folder": "daily"}' > "$OBSIDIAN_DIR/.obsidian/daily-notes.json"
fi
```

## Step 4: 安装 obsidian-skills

把 `obsidian-skills` 安装到你的客户端 skills 根目录中。

示例：

```bash
# Codex CLI
mkdir -p ~/.codex/skills
git clone https://github.com/kepano/obsidian-skills.git ~/.codex/skills/obsidian-skills

# OpenCode
mkdir -p ~/.config/opencode/skills
git clone https://github.com/kepano/obsidian-skills.git ~/.config/opencode/skills/obsidian-skills
```

安装完成后，确认下面 4 个文件都存在：

- `<skills-root>/obsidian-skills/skills/obsidian-markdown/SKILL.md`
- `<skills-root>/obsidian-skills/skills/obsidian-markdown/references/PROPERTIES.md`
- `<skills-root>/obsidian-skills/skills/obsidian-markdown/references/CALLOUTS.md`
- `<skills-root>/obsidian-skills/skills/obsidian-markdown/references/EMBEDS.md`

## Step 5: 安装 knowledge-save

把 `runtime/knowledge-save/` 安装到与你的 `obsidian-skills` 相同的 skills 根目录中，目录名使用 `knowledge-save`。

如果你使用的是支持“从 GitHub 仓库某个路径安装 skill”的通用安装器，请选择：

```text
runtime/knowledge-save
```

手动安装示例：

```bash
# Codex CLI
mkdir -p ~/.codex/skills
cp -R runtime/knowledge-save ~/.codex/skills/

# OpenCode
mkdir -p ~/.config/opencode/skills
cp -R runtime/knowledge-save ~/.config/opencode/skills/
```

其他支持 skills 的客户端也遵循同样的原则：把 `runtime/knowledge-save/` 放到该客户端的 skills 根目录下，并确保它与 `obsidian-skills` 位于同一个 skills 根目录中。

## Step 6: 重启客户端

安装完后，重启客户端，让新 skill 被识别。

## 使用方式

安装完成并重启客户端后，你可以通过两种方式使用它。

### 方式一：显式调用 `/knowledge-save`

```text
/knowledge-save
```

也可以带方向指定：

```text
/knowledge-save 重点记录这次关于缓存击穿排查的过程和最终方案
```

### 方式二：在全局 `AGENTS.md` 中配置自动记录

推荐把规则写进全局 `AGENTS.md`，让 AI 在出现关键进展时自动调用 `knowledge-save` 进行沉淀。

## 常见问题

### Q: 安装时应该使用仓库里的哪个目录？

使用 `runtime/knowledge-save/` 作为安装源目录。

### Q: 为什么需要安装 `obsidian-skills`？

因为 `knowledge-save` 依赖其中的 `obsidian-markdown` 来组织 Obsidian Markdown 内容。

### Q: 如何确认依赖安装正确？

确认以下两件事：

1. `knowledge-save` 和 `obsidian-skills` 安装在同一个 skills 根目录下
2. `obsidian-markdown` 的 4 个文件都存在，且路径可读

### Q: 安装完成后，客户端里的 `knowledge-save` 目录通常包含什么？

通常只需要一个 `SKILL.md`。
