# Knowledge Save — 安装指引

## 简介

**knowledge-save** 是一个 AI Agent Skill，用于将 AI 会话中的关键进展自动沉淀到你的 Obsidian 知识库。

它支持：
- 自动创建/更新笔记文档
- 维护主题中心页与具体笔记页的双向关联
- 更新领域索引（topic）和每日索引（daily）
- 通过 `/save` 命令在有关键进展时触发沉淀
- 首次使用时自动创建所需目录结构和 Obsidian 配置

## 前提条件

### 1. Obsidian 目录

你需要一个 Obsidian 目录（即 Obsidian 中打开的文件夹）。skill 会在首次运行时自动创建 `daily/`、`topic/`、`note/` 三个子目录，并配置 Obsidian 的 Daily Notes 插件指向 `daily/` 目录。

### 2. AI 客户端

目前已适配以下客户端（其他客户端可参考"其他客户端"部分）：

| 客户端 | Skill 目录 | Command 支持 |
|--------|-----------|-------------|
| OpenCode | `.opencode/skills/` | ✅ `.opencode/commands/` |
| Claude Code | `.claude/skills/` | ✅（作为 skill 安装） |
| Gemini CLI | `.gemini/skills/` | ✅ `.gemini/commands/` |
| Codex (OpenAI) | `.codex/skills/` | ✅ `.codex/commands/` |
| Cursor | `.cursor/rules/` | ✅（作为 rule 安装） |

---

## 安装步骤

### Step 1: 获取文件

克隆或下载本项目：

```bash
git clone https://github.com/Zzzia/knowledge-save-skill.git
cd knowledge-save-skill
```

你需要的文件：
- `SKILL.md` — 核心 skill 定义
- `command/save.md` — `/save` 命令模板

### Step 2: 替换路径占位符

**在复制文件之前**，将 `SKILL.md` 中所有的 `{{VAULT_PATH}}` 替换为你的 Obsidian 目录的**绝对路径**。

例如，如果你的 Obsidian 目录在 `/home/yourname/obsidian/my-notes`：

```bash
# Linux / macOS
sed -i 's|{{VAULT_PATH}}|/home/yourname/obsidian/my-notes|g' SKILL.md

# macOS（如果上面报错）
sed -i '' 's|{{VAULT_PATH}}|/home/yourname/obsidian/my-notes|g' SKILL.md
```

或者手动在编辑器中全局替换。

> **注意**: 路径必须是绝对路径，不要使用 `~` 或相对路径。

### Step 3: 初始化 Obsidian 目录结构

创建知识库所需的三层目录：

```bash
# 将下面的路径替换为你的实际 Obsidian 目录路径
OBSIDIAN_DIR="/home/yourname/obsidian/my-notes"

mkdir -p "$OBSIDIAN_DIR/daily"
mkdir -p "$OBSIDIAN_DIR/topic"
mkdir -p "$OBSIDIAN_DIR/note"
```

配置 Obsidian 的 Daily Notes 插件指向 `daily/` 目录：

```bash
# 如果 .obsidian 目录存在（即已用 Obsidian 打开过该目录），配置 daily-notes
if [ -d "$OBSIDIAN_DIR/.obsidian" ]; then
  echo '{"folder": "daily"}' > "$OBSIDIAN_DIR/.obsidian/daily-notes.json"
fi
```

> 如果还没用 Obsidian 打开过该目录，先用 Obsidian 打开一次，再执行上面的配置命令。

### Step 4: 按客户端安装

---

#### OpenCode

**安装 Skill（全局，所有项目可用）**:

```bash
mkdir -p ~/.config/opencode/skills/knowledge-save
cp SKILL.md ~/.config/opencode/skills/knowledge-save/SKILL.md
```

**安装 Skill（项目级，仅当前项目可用）**:

```bash
mkdir -p .opencode/skills/knowledge-save
cp SKILL.md .opencode/skills/knowledge-save/SKILL.md
```

**安装 Command**:

```bash
# 全局
mkdir -p ~/.config/opencode/commands
cp command/save.md ~/.config/opencode/commands/save.md

# 或项目级
mkdir -p .opencode/commands
cp command/save.md .opencode/commands/save.md
```

安装完成后，在 OpenCode 中输入 `/save` 即可使用。

---

#### Claude Code

Claude Code 的 skills 和 commands 已合并，统一使用 `.claude/skills/` 目录。

**安装 Skill（全局，所有项目可用）**:

```bash
mkdir -p ~/.claude/skills/knowledge-save
cp SKILL.md ~/.claude/skills/knowledge-save/SKILL.md
```

**安装 Skill（项目级，仅当前项目可用）**:

```bash
mkdir -p .claude/skills/knowledge-save
cp SKILL.md .claude/skills/knowledge-save/SKILL.md
```

**安装 Command（作为 skill 安装）**:

在 Claude Code 中，command 本质上也是 skill。安装方式：

```bash
# 全局
mkdir -p ~/.claude/skills/save
cp command/save.md ~/.claude/skills/save/SKILL.md

# 或项目级
mkdir -p .claude/skills/save
cp command/save.md .claude/skills/save/SKILL.md
```

安装完成后，在 Claude Code 中输入 `/save` 即可使用。

---

#### Gemini CLI

Gemini CLI 使用 `GEMINI.md` 和 `.gemini/` 目录。

**安装 Skill（全局）**:

```bash
mkdir -p ~/.gemini/skills/knowledge-save
cp SKILL.md ~/.gemini/skills/knowledge-save/SKILL.md
```

**安装 Skill（项目级）**:

```bash
mkdir -p .gemini/skills/knowledge-save
cp SKILL.md .gemini/skills/knowledge-save/SKILL.md
```

**安装 Command**:

```bash
# 全局
mkdir -p ~/.gemini/commands
cp command/save.md ~/.gemini/commands/save.md

# 或项目级
mkdir -p .gemini/commands
cp command/save.md .gemini/commands/save.md
```

安装完成后，在 Gemini CLI 中输入 `/save` 即可使用。

---

#### Codex (OpenAI)

Codex 使用 `.codex/` 目录。

**安装 Skill（全局）**:

```bash
mkdir -p ~/.codex/skills/knowledge-save
cp SKILL.md ~/.codex/skills/knowledge-save/SKILL.md
```

**安装 Skill（项目级）**:

```bash
mkdir -p .codex/skills/knowledge-save
cp SKILL.md .codex/skills/knowledge-save/SKILL.md
```

**安装 Command**:

```bash
# 全局
mkdir -p ~/.codex/commands
cp command/save.md ~/.codex/commands/save.md

# 或项目级
mkdir -p .codex/commands
cp command/save.md .codex/commands/save.md
```

安装完成后，在 Codex 中输入 `/save` 即可使用。

---

#### Cursor

Cursor 使用 `.cursor/rules/` 目录。

**安装 Skill（项目级）**:

```bash
mkdir -p .cursor/rules
cp SKILL.md .cursor/rules/knowledge-save.md
```

**安装 Command**:

```bash
cp command/save.md .cursor/rules/save.md
```

> Cursor 的规则文件在打开对应项目时自动加载。

---

#### 其他客户端

对于以上未列出的 AI 客户端，可以将 `SKILL.md` 的内容添加到客户端的 **System Prompt** 或 **Custom Instructions** 中。具体方式取决于客户端支持的配置机制。

---

## 使用方式

### 基本用法

在会话有关键进展时，输入：

```
/save
```

AI 会自动回顾当前会话内容，将关键知识沉淀到你的 Obsidian 知识库。

### 带参数用法

你可以指定要沉淀的方向：

```
/save TEE参数修改的最新发现
```

```
/save 今天讨论的知识库架构方案
```

AI 会以你提供的参数为指导方向，有针对性地沉淀内容。

### 预期行为

执行 `/save` 后，AI 会：

1. 获取当前真实时间（通过系统命令）
2. 分析会话中的关键知识点
3. 扫描知识库现有结构，确定归属（如有目录缺失会自动创建）
4. 告知你它的归属判断（你可以纠正）
5. 创建或更新相关文档
6. 输出操作摘要

### 示例输出

```
📋 知识沉淀完成

已创建：
- note/Android-TEE环境/TEE证书认证.md（新建具体笔记页）

已更新：
- note/Android-TEE环境/Android-TEE环境.md（追加子文档、更新结论）
- daily/2026-03-16.md（追加今日记录和新增笔记）

新增关联：
- [[note/Android-TEE环境/TEE证书认证]] → [[note/Android-TEE环境/Android-TEE环境]] → [[topic/Android]]
```

---

## 自定义

### 修改模板结构

如果你的知识库使用不同的组织方式，可以修改 `SKILL.md` 中的模板部分来适配：

- **修改区块名称**: 编辑 `## 三、文件模板` 中的区块标题
- **添加/删除区块**: 在模板中增减 `##` 区块
- **修改 frontmatter 字段**: 编辑 `3.4 具体笔记页` 中的 frontmatter 模板
- **调整约束规则**: 编辑 `## 五、约束规则汇总` 中的各项规则

### 调整 tags 数量

默认限制为 2~4 个。如需修改，编辑 SKILL.md 中 `五、约束规则汇总 > 标签（tags）` 部分。

### 修改 wikilink 格式

默认使用带路径前缀的格式（如 `[[note/主题名/笔记名]]`）。如果你的 Obsidian 配置了不同的链接格式，编辑 SKILL.md 中 `Wikilink 格式` 部分。

---

## 常见问题

### Q: AI 没有按照模板格式写文件

确认 SKILL.md 已正确安装到对应客户端的 skill 目录，并且 `{{VAULT_PATH}}` 已替换为实际路径。

### Q: AI 创建了太多子文档

这说明 AI 没有很好地执行"优先更新已有子文档"的规则。你可以：
1. 在 `/save` 时明确指定更新哪个文件
2. 在 SKILL.md 的约束规则中加强描述

### Q: 新建了不需要的 topic

正常情况下 AI 应该在新建 topic 前询问你。如果没有，检查 SKILL.md 的步骤 4 是否完整。

### Q: 路径替换后文件中还有 `{{VAULT_PATH}}`

使用全局搜索确认所有占位符都已替换：

```bash
grep -n '{{VAULT_PATH}}' SKILL.md
```

如果有输出，说明还有未替换的占位符。
