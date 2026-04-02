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

## 两种安装模式

knowledge-save 提供两种使用模式，可根据你的需求选择：

| 模式 | 特点 | 适用场景 |
|------|------|---------|
| **Skill 模式** | 主 agent 直接执行沉淀，能访问完整会话上下文 | 所有客户端通用，信息最完整 |
| **SubAgent 模式** | 独立轻量模型执行沉淀，不占用主 agent 上下文 | 支持 subagent 的客户端（如 OpenCode），节省上下文和成本 |

**如何选择？**

- 如果你**重视信息完整度**，或者你的客户端**不支持 subagent** → 选择 **Skill 模式**
- 如果你**重视主 agent 上下文节省**，且客户端**支持 subagent/Task 工具** → 选择 **SubAgent 模式**
- 不确定？先用 **Skill 模式**，它适用于所有客户端

> **SubAgent 模式的优势**：使用轻量模型（如 claude-sonnet）执行沉淀，不占用主 agent 上下文，同一会话内多次沉淀时可复用 subagent 的上下文。
>
> **SubAgent 模式的局限**：subagent 无法直接访问主 agent 的会话内容，依赖主 agent 传入的摘要质量。command 中已包含详细的摘要规范来最大化信息传递。

---

## 安装步骤

### Step 1: 获取文件

克隆或下载本项目：

```bash
git clone https://github.com/Zzzia/knowledge-save-skill.git
cd knowledge-save-skill
```

你需要的文件：
- `SKILL.md` — 核心 skill 定义（Skill 模式必须，SubAgent 模式可选）
- `agent/knowledge-save.md` — subagent 定义（SubAgent 模式必须）
- `command/save-skill.md` — Skill 模式的 `/save` 命令模板
- `command/save-subagent.md` — SubAgent 模式的 `/save` 命令模板

### Step 2: 替换占位符

#### 2a. 替换 Obsidian 路径

**所有模式都需要**。将 `SKILL.md` 和 `agent/knowledge-save.md` 中的 `{{VAULT_PATH}}` 替换为你的 Obsidian 目录的**绝对路径**。

例如，如果你的 Obsidian 目录在 `/home/yourname/obsidian/my-notes`：

```bash
# Linux / macOS
sed -i 's|{{VAULT_PATH}}|/home/yourname/obsidian/my-notes|g' SKILL.md agent/knowledge-save.md

# macOS（如果上面报错）
sed -i '' 's|{{VAULT_PATH}}|/home/yourname/obsidian/my-notes|g' SKILL.md agent/knowledge-save.md
```

> **注意**: 路径必须是绝对路径，不要使用 `~` 或相对路径。

#### 2b. 替换 SubAgent 模型（仅 SubAgent 模式）

如果你选择 SubAgent 模式，需要将 `agent/knowledge-save.md` 中的 `{{SUBAGENT_MODEL}}` 替换为你希望使用的模型 ID。

模型 ID 格式为 `provider/model-id`，例如：
- `anthropic/claude-sonnet-4-6` — Anthropic 直连
- `openrouter/anthropic/claude-sonnet-4-6` — 通过 OpenRouter
- `opencode/claude-sonnet-4-6` — 通过 OpenCode Zen
- 也可以使用你已配置的任意 provider 的模型

```bash
# 示例：使用 anthropic 的 claude-sonnet-4-6
sed -i 's|{{SUBAGENT_MODEL}}|anthropic/claude-sonnet-4-6|g' agent/knowledge-save.md

# macOS
sed -i '' 's|{{SUBAGENT_MODEL}}|anthropic/claude-sonnet-4-6|g' agent/knowledge-save.md
```

如果你不确定用什么模型，或者想用**客户端的默认模型**：直接删除 `agent/knowledge-save.md` frontmatter 中的 `model: {{SUBAGENT_MODEL}}` 这一行即可。

```bash
# 删除 model 行，使用默认模型
sed -i '/^model: {{SUBAGENT_MODEL}}$/d' agent/knowledge-save.md

# macOS
sed -i '' '/^model: {{SUBAGENT_MODEL}}$/d' agent/knowledge-save.md
```

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

### Step 4: 安装 obsidian-skills 依赖

knowledge-save 依赖 [obsidian-skills](https://github.com/kepano/obsidian-skills)（Obsidian 官方 Agent Skills）来编写更高质量的 Obsidian 文档。安装 knowledge-save 之前，请先确保该依赖已安装到当前客户端的 skills 目录中。

**检测与安装步骤**：

1. 检查当前客户端的 skills 目录下是否已存在 `obsidian-skills`（包含 `obsidian-markdown`、`obsidian-bases`、`json-canvas` 等子技能）
2. 如果不存在，将 obsidian-skills 仓库克隆到对应的 skills 目录：
   ```bash
   git clone https://github.com/kepano/obsidian-skills.git <客户端skills目录>/obsidian-skills
   ```
3. 确认 `<客户端skills目录>/obsidian-skills/skills/` 下包含 `obsidian-markdown/SKILL.md` 等文件即为安装成功

> 各客户端的 skills 目录位置不同（如 OpenCode 为 `~/.config/opencode/skills/`，Claude Code 为 `~/.claude/skills/`），请根据实际客户端自行适配路径。obsidian-skills 仓库 README 中也有各客户端的安装说明可供参考。

### Step 5: 按客户端安装

根据你选择的模式和使用的客户端，按以下说明安装。

---

#### OpenCode

**Skill 模式**:

```bash
# 安装 Skill（全局）
mkdir -p ~/.config/opencode/skills/knowledge-save
cp SKILL.md ~/.config/opencode/skills/knowledge-save/SKILL.md

# 安装 Command
mkdir -p ~/.config/opencode/commands
cp command/save-skill.md ~/.config/opencode/commands/save.md
```

**SubAgent 模式**:

```bash
# 安装 SubAgent 定义
mkdir -p ~/.config/opencode/agents
cp agent/knowledge-save.md ~/.config/opencode/agents/knowledge-save.md

# 安装 Command（SubAgent 版本）
mkdir -p ~/.config/opencode/commands
cp command/save-subagent.md ~/.config/opencode/commands/save.md

# （可选）同时安装 Skill，以便在不使用 /save 时手动加载
mkdir -p ~/.config/opencode/skills/knowledge-save
cp SKILL.md ~/.config/opencode/skills/knowledge-save/SKILL.md
```

**SubAgent 模式还需要配置 AGENTS.md**（全局 `~/.config/opencode/AGENTS.md` 或项目级 `.opencode/AGENTS.md`），添加以下规则：

```markdown
## 沉淀知识库

在有突破性进展后、或用户通过 `/save` 命令要求时，需要将经验沉淀到个人知识库中。

**沉淀机制**：通过 Task 工具调用 `knowledge-save` subagent（轻量模型），不要自己执行沉淀操作，也不要加载 knowledge-save skill。

**调用规范**：
- 使用 Task 工具，`subagent_type` 设为 `knowledge-save`
- 在 `prompt` 中提供：要沉淀的**内容摘要**（关键进展、结论、方法论的精炼总结）+ 可选的**方向指示**（如果用户指定了的话）
- **上下文复用**：同一会话中首次调用后，Task 工具会返回 `task_id`。后续调用时**必须传入这个 task_id**，让 subagent 在之前的上下文上继续工作，避免重复扫描知识库
- 将 subagent 返回的操作摘要展示给用户

**关于 topic 新建确认**：如果 subagent 返回信息中提到"需要新建 topic"，你需要询问用户确认，然后将确认结果传给 subagent（通过 task_id 复用继续对话）。

**内容摘要质量要求**（subagent 只能看到你传入的摘要，无法访问原始会话，因此摘要质量直接决定沉淀质量）：
- **记录过程而非仅结论**：包括尝试过的方案、失败的原因、最终选择的理由
- **保留关键细节**：涉及的文件路径、函数名、配置项、命令、错误信息等具体信息
- **包含决策上下文**：为什么选 A 不选 B，各方案的权衡和取舍
- **保留技术知识点**：发现的 API 用法、框架机制、系统行为等可复用的技术细节
- **结构化组织**：用清晰的标题分段（背景、过程、结论、关键知识点），方便 subagent 理解和归类
```

安装完成后，在 OpenCode 中输入 `/save` 即可使用。

项目级安装方式（将上面的全局路径替换为项目路径）：

| 文件 | 全局路径 | 项目级路径 |
|------|---------|-----------|
| Skill | `~/.config/opencode/skills/knowledge-save/SKILL.md` | `.opencode/skills/knowledge-save/SKILL.md` |
| Agent | `~/.config/opencode/agents/knowledge-save.md` | `.opencode/agents/knowledge-save.md` |
| Command | `~/.config/opencode/commands/save.md` | `.opencode/commands/save.md` |

---

#### Claude Code

Claude Code 的 skills 和 commands 已合并，统一使用 `.claude/skills/` 目录。

**Skill 模式（推荐）**:

```bash
# 安装 Skill（全局）
mkdir -p ~/.claude/skills/knowledge-save
cp SKILL.md ~/.claude/skills/knowledge-save/SKILL.md

# 安装 Command（作为 skill 安装）
mkdir -p ~/.claude/skills/save
cp command/save-skill.md ~/.claude/skills/save/SKILL.md
```

**Skill 模式（项目级）**:

```bash
mkdir -p .claude/skills/knowledge-save
cp SKILL.md .claude/skills/knowledge-save/SKILL.md

mkdir -p .claude/skills/save
cp command/save-skill.md .claude/skills/save/SKILL.md
```

安装完成后，在 Claude Code 中输入 `/save` 即可使用。

---

#### Gemini CLI

Gemini CLI 使用 `GEMINI.md` 和 `.gemini/` 目录。

**Skill 模式**:

```bash
# 安装 Skill（全局）
mkdir -p ~/.gemini/skills/knowledge-save
cp SKILL.md ~/.gemini/skills/knowledge-save/SKILL.md

# 安装 Command
mkdir -p ~/.gemini/commands
cp command/save-skill.md ~/.gemini/commands/save.md
```

**Skill 模式（项目级）**:

```bash
mkdir -p .gemini/skills/knowledge-save
cp SKILL.md .gemini/skills/knowledge-save/SKILL.md

mkdir -p .gemini/commands
cp command/save-skill.md .gemini/commands/save.md
```

安装完成后，在 Gemini CLI 中输入 `/save` 即可使用。

---

#### Codex (OpenAI)

Codex 使用 `.codex/` 目录。

**Skill 模式**:

```bash
# 安装 Skill（全局）
mkdir -p ~/.codex/skills/knowledge-save
cp SKILL.md ~/.codex/skills/knowledge-save/SKILL.md

# 安装 Command
mkdir -p ~/.codex/commands
cp command/save-skill.md ~/.codex/commands/save.md
```

**Skill 模式（项目级）**:

```bash
mkdir -p .codex/skills/knowledge-save
cp SKILL.md .codex/skills/knowledge-save/SKILL.md

mkdir -p .codex/commands
cp command/save-skill.md .codex/commands/save.md
```

安装完成后，在 Codex 中输入 `/save` 即可使用。

---

#### Cursor

Cursor 使用 `.cursor/rules/` 目录。

**Skill 模式（项目级）**:

```bash
mkdir -p .cursor/rules
cp SKILL.md .cursor/rules/knowledge-save.md
cp command/save-skill.md .cursor/rules/save.md
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

如果你的知识库使用不同的组织方式，可以修改 `SKILL.md`（或 SubAgent 模式下的 `agent/knowledge-save.md`）中的模板部分来适配：

- **修改区块名称**: 编辑文件模板中的区块标题
- **添加/删除区块**: 在模板中增减 `##` 区块
- **修改 frontmatter 字段**: 编辑具体笔记页的 frontmatter 模板
- **调整约束规则**: 编辑约束规则汇总中的各项规则

> **注意**: 如果同时使用了 Skill 和 SubAgent 模式，需要保持两个文件中模板的一致性。

### 调整 tags 数量

默认限制为 2~4 个。如需修改，编辑对应文件中标签相关的规则。

### 修改 wikilink 格式

默认使用带路径前缀的格式（如 `[[note/主题名/笔记名]]`）。如果你的 Obsidian 配置了不同的链接格式，编辑对应文件中 Wikilink 格式相关的规则。

### SubAgent 模式：切换模型

修改 `agent/knowledge-save.md` frontmatter 中的 `model` 字段：

```yaml
---
model: anthropic/claude-sonnet-4-6  # 修改为你想用的模型
---
```

删除 `model` 行则使用客户端的默认模型。

---

## 常见问题

### Q: AI 没有按照模板格式写文件

确认 SKILL.md（或 agent/knowledge-save.md）已正确安装到对应客户端的目录，并且 `{{VAULT_PATH}}` 已替换为实际路径。

### Q: AI 创建了太多子文档

这说明 AI 没有很好地执行"优先更新已有子文档"的规则。你可以：
1. 在 `/save` 时明确指定更新哪个文件
2. 在规范文件的约束规则中加强描述

### Q: 新建了不需要的 topic

正常情况下 AI 应该在新建 topic 前询问你。如果没有，检查规范文件的 topic 管理规则是否完整。

### Q: 路径替换后文件中还有 `{{VAULT_PATH}}`

使用全局搜索确认所有占位符都已替换：

```bash
grep -rn '{{VAULT_PATH}}' SKILL.md agent/knowledge-save.md
```

如果有输出，说明还有未替换的占位符。

### Q: SubAgent 模式下信息不完整

SubAgent 无法直接访问主 agent 的会话上下文，只能看到主 agent 传入的摘要。command 中已包含详细的摘要规范。如果仍觉得不够完整，可以：
1. 在 `/save` 时附加更多上下文，如 `/save 重点记录 TEE 签名验证的调试过程和最终解决方案`
2. 如果信息完整度是首要需求，切换回 Skill 模式

### Q: Skill 模式和 SubAgent 模式可以同时安装吗？

可以。两者的文件互不冲突（Skill 在 `skills/` 目录，SubAgent 在 `agents/` 目录）。`/save` command 只需安装一个版本，决定了默认使用哪种模式。
