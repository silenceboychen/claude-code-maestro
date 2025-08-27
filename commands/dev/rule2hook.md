# 任务：将项目规则转换为 Claude Code Hook

您是将自然语言项目规则转换为 Claude Code 钩子配置的专家。您的任务是分析给定的规则，并根据 Claude Code 钩子官方规范生成合适的钩子配置。

## 说明

1. 如果规则以参数形式提供，则分析这些规则
2. 如果未提供参数，则从以下位置读取并分析 CLAUDE.md 文件：
   - `./CLAUDE.md`（项目内存）
   - `./CLAUDE.local.md`（本地项目内存）
   - `~/.claude/CLAUDE.md`（用户内存）

3. 对于每条规则，确定：
   - 相应的钩子事件（PreToolUse、PostToolUse、Stop、Notification）
   - 工具匹配器模式（确切的工具名称或正则表达式）
   - 要执行的命令

4. 按照精确的 JSON 结构生成完整的钩子配置
5. 将其保存到 `~/.claude/hooks.json`（如果存在，则与现有钩子合并）
6. 提供配置内容的摘要

## 钩子事件

### PreToolUse
- **何时**：在工具执行之前运行
- **常用关键字**："before"、"check"、"validate"、"prevent"、"scan"、"verify"
- **可用的工具匹配器**：
  - `Task` - 启动代理任务之前
  - `Bash` - 运行 shell 命令之前
  - `Glob` - 文件模式匹配之前
  - `Grep` - 内容搜索之前
  - `Read` - 读取文件之前
  - `Edit` - 编辑单个文件之前
  - `MultiEdit` - 批量编辑文件之前
  - `Write` - 写入/创建文件之前
  - `WebFetch` - 获取网页内容之前
  - `WebSearch` - 网页搜索之前
  - `TodoRead` - 读取待办事项列表之前
  - `TodoWrite` - 更新待办事项列表之前
- **特殊功能**：如果命令返回非零退出代码，则可以阻止工具执行

### PostToolUse
- **何时**：工具成功完成后运行
- **常用关键字**："after"、"following"、"once done"、"when finish"
- **可用工具匹配器**：与 PreToolUse 相同
- **常用用途**：文件更改后格式化、linting、构建和测试

### Stop
- **何时**：Claude Code 完成响应后运行
- **常用关键字**："finish"、"complete"、"end task"、"done"、"wrap up"
- **无需匹配器**：适用于所有完成操作
- **常用用途**：最终状态检查、摘要、清理

### Notification
- **何时**：Claude Code 发送通知时运行
- **常用关键字**："notify"、"alert"、"inform"、"message"
- **特殊用途**：很少用于规则转换

## Hook配置结构

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolName|AnotherTool|Pattern.*",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here"
          }
        ]
      }
    ]
  }
}
```

## 匹配器模式

- **精确匹配**：“Edit” - 仅匹配“编辑”工具
- **多工具**：“Edit|MultiEdit|Write” - 匹配以下任意一项
- **正则表达式**：“.*Edit” - 匹配“编辑”和“多编辑”
- **所有工具**：完全省略匹配器字段

## 分析示例

### 示例 1：Python 格式化
**规则**：“编辑之后使用black对Python 文件格式化”
**分析**：
- 关键字“之后” → PostToolUse
- “editing” → Edit|MultiEdit|Write 工具
- “Python 文件” → 命令应以 .py 文件为目标

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|MultiEdit|Write",
      "hooks": [{
        "type": "command",
        "command": "black . --quiet 2>/dev/null || true"
      }]
    }]
  }
}
```

### 示例 2：Git 状态检查
**规则**：“完成任务时运行 git status”
**分析**：
- “完成” → 停止事件
- 未提及特定工具 → 无需匹配器

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "git status"
      }]
    }]
  }
}
```

### 示例 3：安全扫描
**规则**：“保存任何文件之前检查硬编码密钥”
**分析**：
- “之前” → PreToolUse
- “保存任何文件” → 写入|编辑|多重编辑

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Write|Edit|MultiEdit",
      "hooks": [{
        "type": "command",
        "command": "git secrets --scan 2>/dev/null || echo 'No secrets found'"
      }]
    }]
  }
}
```

### 示例 4：测试运行器
**规则**：“修改 tests/ 目录中的文件后运行 npm test”
**分析**：
- “修改后” → PostToolUse
- “文件” → Edit|MultiEdit|Write
- 注意：路径过滤发生在命令中，而不是匹配器中

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|MultiEdit|Write",
      "hooks": [{
        "type": "command",
        "command": "npm test 2>/dev/null || echo 'Tests need attention'"
      }]
    }]
  }
}
```

### 示例 5：命令日志记录
**规则**：“在执行前记录所有 bash 命令”
**分析**：
- “执行前” → PreToolUse
- “bash 命令” → 具体为 Bash 工具

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "echo \"[$(date)] Executing bash command\" >> ~/.claude/command.log"
      }]
    }]
  }
}
```

## 命令生成的最佳实践

1. **错误处理**：添加 `|| true` 或 `2>/dev/null` 以防止钩子失败阻塞 Claude
2. **安静模式**：尽可能使用安静标志 (--quiet、-q)
3. **路径安全**：使用相对路径或检查路径是否存在
4. **性能**：保持命令快速运行，避免降低 Claude 的速度
5. **日志记录**：重定向详细输出，避免 Claude 界面混乱

## 常用规则模式

- “编辑后格式化 [language] 文件” → PostToolUse + Edit|MultiEdit|Write
- “提交前运行 [command]” → PreToolUse + Bash（检测到 git commit 时）
- “保存前检查 [pattern]” → PreToolUse + Write|Edit|MultiEdit
- “完成后执行 [command]” → Stop 事件
- “运行命令前验证 [something]” → PreToolUse + Bash
- “修改配置后清除缓存” → PostToolUse + Edit|MultiEdit|Write
- “在 [condition] 时通知” → 通常 PostToolUse 会使用特定的匹配器

## 重要提示

1. 始终与现有钩子合并 - 不要覆盖
2. 在添加钩子之前测试命令是否有效
3. 考虑钩子的性能影响
4. 尽可能使用特定的匹配器以避免不必要的执行
5. 命令以完全用户权限运行 - 谨慎处理破坏性操作

## 用户输入
$ARGUMENTS