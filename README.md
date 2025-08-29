<div align="center">
  
### **Claude Code 完整用户指南**

> 本指南涵盖了截至2025年8月的**所有可发现的Claude Code命令**，  
> 包括许多在基础教程中不广为人知或未记录的功能。

**这是最完整的Claude Code命令参考手册。**

---
</div>

# 目录

1. [快速开始](#快速开始)
2. [安装与设置](#安装与设置)
3. [命令介绍](#命令介绍)
4. [子代理（Sub Agents）](#子代理)
5. [MCP集成](#mcp集成)
6. [安全性与权限](#安全性与权限)
7. [思考关键词](#思考关键词)
8. [故障排除](#故障排除)
9. [高级功能](#高级功能)
10. [最佳实践](#最佳实践)

---

## 快速开始

```bash
# 快速安装

## 方法1 – NPM (全局) ⭐️ 官方推荐
npm install -g @anthropic-ai/claude-code
# 需要macOS / Linux / WSL上的Node 18+ | 如果只输入"claude"不工作，尝试"npx claude"

## 方法2 MacOS
brew install node
npm install -g @anthropic-ai/claude-code
# claude bot出现问题？运行 export PATH="$PATH:$(npm bin -g)"

## 方法3 – Arch Linux AUR
yay -S claude-code        # 或 paru -S claude-code
# 与npm版本保持同步

## 方法4 – Docker (容器化)
mkdir -p ~/.docker/cli-plugins
curl -SL https://github.com/docker/buildx/releases/download/v0.11.2/buildx-v0.11.2.linux-amd64 -o ~/.docker/cli-plugins/docker-buildx
chmod +x ~/.docker/cli-plugins/docker-buildx
curl -O https://raw.githubusercontent.com/RchGrav/claudebox/main/claudebox
chmod +x claudebox
# 当你无法触及主机系统时很有用

## 方法5 – Windows via WSL (Anthropic推荐路径)
# 1) 启用WSL 2并安装Ubuntu
# 2) 在Ubuntu内：
sudo apt update && sudo apt install -y nodejs npm
npm install -g @anthropic-ai/claude-code
# 分步指南

# 你也可以在vscode或cursor中轻松打开你的整个项目：
cursor .
# 或
code .
# cd到你想要工作的目录并运行上述命令！
# 记得安装claude code扩展

## 检查claude是否已安装
which claude
claude --version


# 交互模式
claude                      # 启动交互式REPL
claude "your question"      # 带初始提示启动

# 单次模式  
claude -p "analyze this"    # 快速查询并退出
cat file | claude -p "fix"  # 处理管道内容

# 管理
claude config              # 配置设置
claude update              # 更新到最新版本
claude mcp                 # 设置MCP服务器
claude /agents             # 配置/设置用于不同任务的子代理
```

---

## 安装与设置

**系统要求：**
- 操作系统：macOS 10.15+、Ubuntu 20.04+/Debian 10+，或通过WSL的Windows
- 硬件：最少4GB内存，推荐8GB+
- 软件：
  - Node.js 18+
  - git 2.23+ (可选)
  - 用于PR工作流的GitHub或GitLab CLI (可选)
- API调用需要互联网连接

**支持的平台：**
- macOS (Intel/Apple Silicon)
- Linux (Ubuntu, Debian, CentOS)
- Windows 10/11 (**推荐WSL**，因为目前不支持常规Windows终端)

### 安装方法

#### 方法1：NPM安装 

如果您已安装 Node.js 18 或更新版本：

```bash
# 全局安装
npm install -g @anthropic-ai/claude-code
```

#### 方法2：原生安装（推荐，npm安装有时无法自动升级）

```bash
# macOS、Linux、WSL：
curl -fsSL claude.ai/install.sh | bash

#Windows PowerShell：
irm https://claude.ai/install.ps1 | iex
```

### 初始设置

#### 1. API密钥配置 
```bash
# 必需：从 https://console.anthropic.com 获取你的API密钥
export ANTHROPIC_API_KEY="sk-your-key-here"

# 永久设置（选择你的shell）
# Bash
echo 'export ANTHROPIC_API_KEY="sk-your-key-here"' >> ~/.bashrc
source ~/.bashrc

# Zsh
echo 'export ANTHROPIC_API_KEY="sk-your-key-here"' >> ~/.zshrc
source ~/.zshrc

# Fish
echo 'set -gx ANTHROPIC_API_KEY "sk-your-key-here"' >> ~/.config/fish/config.fish
```

#### 2. 基础配置
```bash
# 交互式设置
claude config

# 设置基本默认值 
claude config set -g verbose true
claude config set -g outputFormat text

# 测试安装
claude "Hello, Claude!"
claude /doctor
claude update
```

### 健康检查与测试
```bash
claude /doctor
# 预期输出可能包括：
# ✅ API Key: Valid
# ✅ Network: Connected  
# ✅ Model Access: Available

# 基本功能测试 
claude "Explain what you can do"

# 打印模式测试 
claude -p "What is 2+2?"

# 工具权限测试
claude "Create a file called test.txt with 'Hello World'"
# 应该提示Edit权限

# 会话连续性测试  
claude -c  # 应该从上一个会话继续
```

---

## 命令介绍

### 内置斜杠命令（交互式）

| 斜杠命令             | 用途                     |
| -------------------- | ------------------------ |
| `/help`              | 列出斜杠命令             |
| `/add-dir`           | 添加更多工作目录         |
| `/agents`            | 列出/创建/编辑子代理     |
| `/bashes`            | 列出并管理后台 bash shell  |
| `/bug`               | 向Anthropic报告错误      |
| `/clear`             | 清除聊天历史             |
| `/compact`           | 压缩对话                 |
| `/config`            | 配置菜单                 |
| `/context`           | 将当前上下文使用情况可视化为彩色网格  |
| `/cost`              | Token使用情况            |
| `/doctor`            | 健康检查                 |
| `/exit`              | 退出REPL                 |
| `/export`            | 将当前对话导出到文件或剪贴板     |
| `/hooks`             | 管理工具事件的钩子配置     |
| `/ide`               | 管理 IDE 集成并显示状态     |
| `/init`              | 生成CLAUDE.md            |
| `/install-github-app`| 为存储库设置 Claude GitHub Actions   |
| `/login` / `/logout` | 认证切换                 |
| `/mcp`               | 管理MCP服务器            |
| `/memory`            | 编辑记忆                 |
| `/migrate-installer` | 从全局 npm 安装迁移到本地安装    |
| `/model`             | 更改模型                 |
| `/output-style`      | 直接设置输出样式或从选择菜单中设置   |
| `/output-style:new`  | 创建自定义输出样式         |
| `/permissions`       | 工具权限                 |
| `/pr-comments`       | 查看PR评论               |
| `/release-notes`     | 查看发行说明             |
| `/resume`            | 恢复对话             |
| `/review`            | 请求代码审查             |
| `/security-review`   | 完成对当前分支上待处理更改的安全审查   |
| `/status`            | 系统/账户状态            |
| `/statusline`        | 创建自定义状态栏            |
| `/terminal-setup`    | 安装Shift+Enter绑定      |
| `/todo`              | 列出当前待办事项      |
| `/upgrade`           | 升级升级到 Max 可获得更高的速率限制和更多 Opus      |
| `/vim`               | 切换vim模式              |

### 核心命令

| 命令                                | 描述                   | 示例                                   |
| ---------------------------------- | ---------------------- | ------------------------------------- |
| `claude`                           | 启动交互式REPL         | `claude`                              |
| `claude "<prompt>"`                | 带初始提示的REPL       | `claude "help debug"`                 |
| `claude -p "<prompt>"`             | 单次打印模式           | `claude -p "explain fn"`              |
| `cat file \| claude -p "<prompt>"` | 通过管道传递STDIN给Claude | `cat logs.txt \| claude -p "explain"` |
| `claude update`                    | 自更新到最新版本       | `claude update`                       |
| `claude mcp`                       | 启动MCP向导            | `claude mcp`                          |

### 会话命令

| 命令                    | 描述                   | 示例                         |
| ----------------------- | ---------------------- | --------------------------- |
| `claude -c` / `--continue` | 继续上次会话           | `claude -c`                 |
| `claude -c -p "<prompt>"` | 继续 + 新提示（打印）    | `claude -c -p "check types"` |
| `claude -r <id>`        | 通过会话ID恢复         | `claude -r abc123`          |
| `claude --resume <id>`  | 长格式恢复             | `claude --resume abc123`    |
| `claude --resume <name>` | 通过保存的名称恢复     | `claude --resume sprint-review` |


### 配置命令

| 命令                              | 描述        | 示例                          |
| --------------------------------- | ----------- | ----------------------------- |
| `claude config`                   | 交互式向导  | `claude config`               |
| `claude config list`              | 列出所有键  | `claude config list`          |
| `claude config get <key>`         | 获取值      | `claude config get theme`     |
| `claude config set <key> <val>`   | 设置值      | `claude config set theme dark` |
| `claude config add <key> <vals…>` | 追加到数组  | `claude config add env DEV=1` |
| `claude config remove <key> <vals…>` | 移除项目 | `claude config remove env DEV=1` |

### 参数定义

#### 基础

| 标志         | 简写 | 描述          | 示例               |
| ------------ | ---- | ------------- | ------------------ |
| `--print`    | `-p` | 非交互模式    | `claude -p "help"` |
| `--continue` | `-c` | 恢复上次会话  | `claude --continue` |
| `--resume`   | `-r` | 通过ID恢复    | `claude --resume abc` |
| `--help`     | `-h` | 显示帮助      | `claude --help`    |
| `--version`  | `-v` | 显示版本      | `claude --version` |

#### 输出与流程（配合-p使用）

| 标志                     | 选项                       | 示例                              |
| ------------------------ | -------------------------- | --------------------------------- |
| `--output-format`        | `text`, `json`, `stream-json` | `--output-format json`         |
| `--input-format`         | `text`, `stream-json`      | `--input-format stream-json`      |
| `--max-turns`            | 整数                       | `--max-turns 3`                   |
| `--system-prompt`        | 字符串                     | `--system-prompt "You are strict"` |
| `--append-system-prompt` | 字符串                     | `--append-system-prompt "Add tests"` |

#### 安全与权限

| 标志                              | 描述                    | 示例                                      |
| --------------------------------- | ----------------------- | ----------------------------------------- |
| `--allowedTools <list>`           | 工具白名单              | `--allowedTools "Read,View"`              |
| `--disallowedTools <list>`        | 工具黑名单              | `--disallowedTools "Bash"`                |
| `--permission-mode <mode>`        | 以权限模式启动          | `--permission-mode plan`                  |
| `--permission-prompt-tool <tool>` | 用于权限检查的MCP工具   | `--permission-prompt-tool mcp__auth__prompt` |
| `--dangerously-skip-permissions`  | 跳过所有提示（⚠️）      | `--dangerously-skip-permissions`          |

#### 额外

| 标志                  | 用途               | 示例                     |
| --------------------- | ------------------ | ------------------------ |
| `--add-dir <paths>`   | 额外工作目录       | `--add-dir ../lib ../src` |
| `--model <name>`      | 选择模型           | `--model claude-opus-4-20250514` |
| `--verbose`           | 详细日志           | `--verbose`              |
| `--mcp-config <file>` | 加载MCP服务器      | `--mcp-config servers.json` |

---

## Hooks钩子

Hooks（钩子）是Claude Code中的强大功能，允许你在特定工具调用事件发生时执行自定义shell命令。

### 配置文件位置说明

| 配置文件路径 | 作用范围 | 描述 | 版本控制 |
|-------------|----------|------|----------|
| `~/.claude/settings.json` | 用户级全局 | 影响所有Claude Code会话的全局设置 | 不提交 |
| `.claude/settings.json` | 项目级 | 仅影响当前项目，团队共享配置 | 可提交 |
| `.claude/settings.local.json` | 本地项目级 | 个人化项目配置，覆盖项目设置 | 不提交 |

### Hook事件说明

| 事件类型 | 触发时机 | 用途 | 示例场景 |
|----------|----------|------|----------|
| `PreToolUse` | 工具执行前 | 预处理、权限检查、阻止工具使用 | 执行前备份、权限验证 |
| `PostToolUse` | 工具成功执行后 | 后续处理、验证、记录结果 | 自动提交代码、运行测试 |
| `Notification` | Claude发送通知时 | 处理权限请求、空闲提醒 | 自动授权、提醒通知 |
| `UserPromptSubmit` | 用户提交提示时 | 添加上下文、验证输入、阻止特定提示 | 添加项目信息、输入过滤 |
| `Stop` | Claude响应完成时 | 主要响应完成处理 | 保存响应日志 |
| `SubagentStop` | 子代理响应完成时 | 子代理任务完成处理 | 子任务结果统计 |
| `SessionEnd` | 会话结束时 | 清理、记录会话统计 | 资源清理、会话统计 |
| `PreCompact` | 执行压缩操作前 | 压缩前准备工作 | 备份对话历史 |
| `SessionStart` | 会话开始时 | 初始化、加载开发上下文 | 环境准备、上下文加载 |

### 匹配器说明

匹配器用于确定hook何时触发，支持以下模式：

| 匹配器类型 | 语法 | 示例 | 说明 |
|-----------|------|------|------|
| 精确匹配 | `"ToolName"` | `"Write"` | 仅匹配指定工具 |
| 正则表达式 | `"Tool1\|Tool2"` | `"Edit\|Write"` | 匹配多个工具 |
| 通配符 | `"*"` | `"*"` | 匹配所有工具 |

**PreToolUse/PostToolUse支持的工具匹配器：**
- `Task` - 子代理任务
- `Bash` - Shell命令执行  
- `Glob`、`Grep` - 搜索操作
- `Read`、`Write`、`Edit` - 文件操作
- `WebFetch` - 网络获取

**PreCompact匹配器：**
- `manual` - 通过/compact命令手动触发
- `auto` - 由于上下文窗口限制自动触发

**SessionStart匹配器：**
- `startup` - 新会话启动
- `resume` - 恢复现有会话
- `clear` - 清除后重新开始

**SessionEnd触发原因：**
- `clear` - 用户清除会话
- `logout` - 用户登出
- `prompt_input_exit` - 提示输入退出
- `other` - 其他原因

### 配置管理

```bash
# 打开hooks配置界面
/hooks

# 查看当前hook配置  
claude config get hooks

# 清除特定hook
claude config remove hooks.preToolUse
```

---

## 子代理

> agents/目录下提供了多种不同类别的Sub Agents的最佳实践，可直接点击[**子代理最佳实践**](agents/README.md)查看。

子代理（Sub Agents）是专用助手，拥有**自己的提示、工具和隔离的上下文窗口**。将其视为你为每个代码库**组合**的"专家混合体"。

**何时使用它们**
- 你需要高信号响应（计划、审查、差异）而不会偏离主题。
- 你希望版本控制的提示和工具策略与代码库一起。
- 你在PR驱动的团队中工作，希望按角色进行范围化编辑。

**为你的项目设计规则**
- 为每个代理定义**一个明确的职责**。
- 保持该角色所需的**最少**工具集。
- 对于分析/审查任务，偏向**只读**代理。
- 尽可能少地给予代理编辑权限。

### 如何配置代理

保持代理**在项目中**，以便它们与仓库一起版本化，并通过PR演进。

```bash
# 打开代理面板，根据指示进行配置
/agents
```

### 创建你的核心代理
- **planner**（只读）：将功能/问题转化为小的、可测试的任务；输出任务列表或plan.md。
- **codegen**（具有编辑能力）：实现任务；限制在`src/` + `tests/`。
- **tester**（只读或仅补丁）：编写*一个*失败的测试或最小重现。
- **reviewer**（只读）：留下结构化审查评论；从不编辑。
- **docs**（具有编辑能力）：仅更新`README.md`/`docs/`。

> **策略提示：** 对于具有编辑能力的代理，偏向*补丁输出*，以便更改通过你的正常Git工作流程落地。


### 示例提示
保持提示简短、可测试且特定于仓库。将它们放入`agents/`目录：

<img width="700" height="217" alt="image" src="https://github.com/user-attachments/assets/b4f92591-ff5c-4775-aec2-051f145b9616" />

*说明：**test-coverage-analyzer**代理的示例提示。*

**tester.prompt.md（示例）**
```
角色：为我描述的特定场景编写一个单一的、专注的失败测试。
范围：仅在tests/下创建/修改测试。不要更改src/。
输出：简要理由 + 统一差异或补丁。
如果场景不清楚，问一个明确的澄清问题。
```

### 预期输出
你的测试代理应该产生一个小的差异或补丁加上简短的理由：

<img width="700" height="273" alt="image" src="https://github.com/user-attachments/assets/839151ce-02c9-4283-a53b-9dd105802ada" />

*说明：来自**test-coverage-analyzer**代理的示例响应。*

**验收清单**
- [ ] 输出是单个变更集。
- [ ] 只触及允许路径中的文件。
- [ ] 理由解释意图和边缘情况。
- [ ] 如果被阻止，代理询问了一个明确的问题。

### 为什么子代理很重要

**操作优势**
- **更少的上下文切换：** 你保持一种思维模式；代理做其余的事情。
- **更清晰的PR：** 狭窄的提示 + 有限的工具 → 更小的、可审查的差异。
- **更少的回归：** 测试/审查代理在合并前捕获差距。
- **可重复性：** 提示 + 策略存在于仓库中并与分支一起传递。

**安全与治理**
- 按路径限制写入权限（例如，`src/`、`tests/`、`docs/`）。
- 对高风险区域优先进行只读分析。
- 记录/提交助手输出作为补丁以供审计。

### 思维模式转变

**应该做的**
- 将代理视为有工作描述的队友。
- 从只读开始；*最后*授予写访问权限。
- 将提示保存在版本控制中并通过PR迭代。

**不应该做的**
- 要求一个代理在单次操作中计划、编码和测试。
- 授予一揽子写入权限。
- 要求进行单次测试时，接受多文件差异。

---

## MCP集成

### 理解MCP（模型上下文协议）

**什么是MCP？**

MCP通过连接外部服务、数据库、API和工具来扩展Claude的能力。

**MCP架构：**
```
Claude Code ←→ MCP协议 ←→ MCP服务器 ←→ 外部服务
```

### MCP设置与配置

#### 基本MCP命令
```bash
claude mcp                    # 交互式MCP配置
claude mcp list              # 列出已配置的服务器            
claude mcp add <name> <cmd>  # 添加新服务器
claude mcp remove <name>     # 移除服务器
```

#### MCP配置文件 

用户/全局范围：

**位置**：`~/.claude.json` 

项目范围：

项目范围的服务器存储在项目根目录的`.mcp.json`文件中

```json
{
  "mcpServers": {
    "git": {
      "command": "git-mcp-server",
      "args": [],
      "env": {}
    },
    "postgres": {
      "command": "postgres-mcp-server", 
      "args": ["--host", "localhost", "--port", "5432"],
      "env": {
        "POSTGRES_USER": "developer",
        "POSTGRES_PASSWORD": "dev_password",
        "POSTGRES_DB": "myapp_development"
      }
    }
  }
}
```

### MCP服务器

**注意**：下面的确切包名和安装命令可能不准确。请查阅官方MCP文档获取当前的服务器包。

#### 开发工具
```bash
# npm install -g git-mcp-server         

# claude mcp add git "git-mcp-server"
# claude mcp add github "github-mcp-server --token $GITHUB_TOKEN"
```

#### 数据库集成
```bash
npm install -g postgres-mcp-server               
npm install -g mysql-mcp-server                  
npm install -g sqlite-mcp-server               

# 设置示例可能如下所示：
# export POSTGRES_URL="postgresql://user:password@localhost:5432/mydb"
# claude mcp add postgres "postgres-mcp-server --url $POSTGRES_URL"
```

### MCP工具权限

```bash
# 允许特定的MCP工具 
claude --allowedTools "mcp__git__commit,mcp__git__push"

# 允许来自特定服务器的所有工具
claude --allowedTools "mcp__postgres__*"

# 与内置工具组合
claude --allowedTools "Edit,View,mcp__git__*"
```

---


## 安全性与权限

### 权限系统

**工作原理**：
- Claude在使用工具前请求权限
- 权限在每个会话中被记住
- 危险操作需要确认

#### 权限级别
| 级别 | 描述 | 风险 | 使用场景 |
|-------|-------------|------|----------|
| **交互式** | 每个操作都提示 | **低** | 开发工作 |
| **允许列表** | 仅预批准的工具 | **中等** | 自动化脚本 |
| **危险** | 跳过所有权限 | **严重** | 仅容器 |

#### 工具权限模式 
```bash
# 允许特定工具，可以选择自己的，也可以像这样使用：Bash(*)
claude --allowedTools "Edit,View"

# 允许工具类别  
claude --allowedTools "Edit,View,Bash"

# 限定范围的权限（仅Git操作）
claude --allowedTools "Bash(git:*)"

# 多个范围
claude --allowedTools "Bash(git:*),Bash(npm:*)"
```

### 危险模式（严重安全功能）

```bash
# 危险 - 可能导致数据丢失
claude --dangerously-skip-permissions

# 仅在隔离环境中使用：
# ✅ 安全：隔离的Docker容器  
# ❌ 绝不：生产系统、共享机器、有重要数据的系统
```

### 安全最佳实践

#### 1. 从严格开始
```bash
# 好：特定权限
claude --allowedTools "Edit,View,Bash(git:status)"

# 坏：广泛权限  
claude --allowedTools "Bash"
```

#### 2. 保护敏感数据
```bash
# 好：环境变量
export DATABASE_URL="postgresql://user:pass@host/db"

# 坏：在命令中硬编码凭证
# claude "connect to postgresql://user:password123@host/db"
```

#### 3. 定期安全审计
```bash
# 检查当前权限
claude config get allowedTools
claude config get disallowedTools

# 审查配置
claude config list
```

---

## 思考关键词

Claude Code支持**扩展思考** — 为更难的问题提供额外的预回答规划时间。你可以使用特定的触发词来推动逐渐增加的"思考预算"。

### 关键词

按预算增加的顺序排列：`think` < `think hard` < `think harder` < `ultrathink`。

> **仅**使用上述四个关键词。其他短语（例如，"think more"、"megathink"等）**未记录**，不应依赖。

### 扩展思考的作用

在Claude开始产生最终答案之前，它花费更多时间：
- 规划解决方案，
- 分解步骤，
- 考虑替代方案和权衡，
- 检查约束和边缘情况。

### 如何使用

你可以在提示中的任何地方放置关键词（不区分大小写）。如果出现多个，假设**最强的一个获胜**。

```bash
# 小幅提升
claude -p "Think. Outline a plan to refactor the auth module."

# 中等提升
claude -p "Think harder. Draft a migration plan for moving from REST to gRPC."

# 最大规划预算
claude -p "Ultrathink. Propose a step‑by‑step strategy to fix flaky payments tests and add guardrails."
```

### 注意事项和注意点

- 这是**Claude Code (CLI)行为**，不是公共API参数；命名或效果可能随时间演变。
- 更高的预算通常增加**延迟**和**token使用**。偏向能完成工作的最小关键词。
- 保持提示简洁。关键词要求Claude规划；你的提示应该仍然提供目标、约束和成功标准。

---

## 故障排除

### 诊断命令

```bash
# 基本健康检查
claude --version             
claude --help                   
claude config list                 
claude /doctor                  
```

### 常见问题与解决方案

#### 1. 认证问题
```bash
# 检查API密钥
echo $ANTHROPIC_API_KEY

# 测试连接
claude -p "test" --verbose

# 重置认证 
```

#### 2. 安装问题  
```bash
# 重新安装
npm uninstall -g @anthropic-ai/claude-code     
npm install -g @anthropic-ai/claude-code      

# 检查Node.js版本
node --version  # 应该是18+
```

#### 3. 权限问题
```bash
# 检查当前权限
claude config get allowedTools

# 重置权限
claude config set allowedTools "[]"
claude config set allowedTools '["Edit", "View"]'
```

#### 4. MCP问题
```bash
# 调试MCP 
claude --mcp-debug
claude mcp list  
```

### 调试模式
```bash
# 启用详细日志
claude --verbose

# 检查日志（验证日志位置）
```

---

## 高级功能

### 上下文管理

#### CLAUDE.md - 项目记忆
**用途**：存储Claude记住的持久项目信息

**位置**：项目根目录

**创建方式**
- 手动创建CLAUDE.md文件
- 通过斜杠命令``/init``创建（推荐）

```markdown
# 项目：我的应用程序

## 概述
这是一个使用PostgreSQL数据库的React/Node.js应用程序。

## 架构
- 前端：React 18 with TypeScript
- 后端：Node.js with Express  
- 数据库：PostgreSQL 14

## 当前目标
- [ ] 实现认证
- [ ] 添加API文档
- [ ] 设置CI/CD管道

## 开发指南
- 对所有新代码使用TypeScript
- 遵循ESLint配置
- 为新功能编写测试
```

#### 记忆命令 
```bash
claude /memory           # 编辑项目记忆
claude /memory view      # 查看当前记忆
```

### 高级思考
```bash
# 这些"思考"短语可能有效：
claude "think hard about the security implications of this code"
claude "analyze this thoroughly and provide detailed recommendations"
```

### 多目录工作空间
```bash
# 添加多个目录
claude --add-dir ../frontend ../backend ../shared

# 项目级分析
claude "analyze the entire application architecture"
```

---

## 最佳实践

### 有效提示
```bash
# 好：具体且详细
claude "Review UserAuth.js for security vulnerabilities, focusing on JWT handling"

# 坏：模糊  
claude "check my code"
```

### 安全最佳实践
1. **从最小权限开始**：`claude --allowedTools "View"`
2. **使用环境变量**：`export API_KEY="secret"`
3. **定期审计**：`claude config get allowedTools`
4. **避免危险模式**：仅在容器中使用`--dangerously-skip-permissions`

### 性能技巧
1. **使用适当的输出格式**：自动化使用`--output-format json`
2. **在提示中要具体**：更好的结果，更快的执行
3. **定期清理**：删除旧会话和缓存

### 监控与警报

**1. 健康检查自动化**
```bash
# 定期健康检查
*/15 * * * * /usr/local/bin/claude /doctor > /dev/null || echo "Claude health check failed" | mail -s "Alert" admin@company.com
```

**2. 日志分析**
```bash
# 每日日志分析
0 6 * * * tail -1000 /var/log/app.log | claude -p "analyze for issues" --output-format json > /tmp/daily-analysis.json
```

### 协作最佳实践

#### 团队工作流程

**1. 共享配置模板**
```bash
# 创建团队模板
mkdir -p ~/.claude/templates/
cat > ~/.claude/templates/team-frontend.json << EOF
{
  "allowedTools": ["Edit", "View", "Bash(npm:*)", "mcp__git__*"],
  "model": "claude-sonnet-4",
  "systemPrompt": "You are working on our React frontend. Follow our coding standards and use TypeScript."
}
EOF

# 使用模板
claude config import ~/.claude/templates/team-frontend.json
```

**2. 文档自动化**
```bash
# 自动化文档更新
claude "update README.md with recent changes to the API endpoints"
claude "generate TypeScript definitions from the new database schema"
```

**3. 代码审查标准**
```bash
# 标准化审查流程
claude --allowedTools "View,mcp__git__*" \
"使用我们团队的标准审查 PR #123：
- 安全最佳实践
- 性能考量
- 代码风格合规
- 测试覆盖率充分性"
```

#### 知识共享

**1. 创建项目 Runbook**
```bash
# 生成 Runbook
claude "为此应用程序创建部署 Runbook，包含所有步骤和故障排除"
claude "记录新开发人员的入职流程"
```

**2. 架构文档**
```bash
# 维护架构文档
claude "更新架构文档以反映最新的微服务变更"
claude "为新的身份验证流程创建序列图"
```

### 常见陷阱避免

#### 安全陷阱

**❌ 不要做：**
- 在生产系统上使用`--dangerously-skip-permissions`
- 在命令或配置中硬编码密码
- 授予过于宽泛的权限
- 不必要地以提升的权限运行

**✅ 应该做：**
- 对密码使用环境变量
- 从最小权限开始
- 定期安全审计
- 隔离敏感操作

#### 性能陷阱

**❌ 不要做：**
- 不必要地加载整个大型代码库
- 对简单任务使用最大思考预算
- 运行多个并发Claude实例
- 忽略内存和缓存清理

**✅ 应该做：**
- 使用`--add-dir`集中上下文
- 匹配思考预算与任务复杂性
- 监控资源使用
- 定期清理

#### 工作流程陷阱

**❌ 不要做：**
- 跳过项目上下文设置 (CLAUDE.md)
- 使用模糊、含糊的提示
- 忽略错误消息和日志
- 未先测试就进行自动化

**✅ 应做的：**
- 维护全面的项目上下文
- 请求内容具体详细
- 监控和分析日志
- 在安全的环境中测试自动化
---