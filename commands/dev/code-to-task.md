# 将代码分析转换为线性任务

将代码分析转换为线性任务

## 用途
此命令会扫描您的代码库，查找 TODO/FIXME 注释、技术债务标记、弃用代码以及其他应作为任务跟踪的指标。它会自动创建有序且优先级高的线性任务，以确保重要的代码改进不会被遗漏。

## 用法
```bash
# 扫描整个代码库中的 TODO 并创建任务
claude "根据代码库中的所有 TODO 注释创建任务"

# 扫描特定目录或模块
claude "在 src/api 中查找 TODO 并创建线性任务"

# 根据特定模式创建任务
claude "为所有弃用函数创建任务"

# 生成技术债务报告
claude "分析项目中的技术债务并创建改进任务"
```

## 说明

### 1. 扫描任务标记
搜索指示所需工作的常见模式：

```bash
# 查找待办事项注释
rg "TODO|FIXME|HACK|XXX|OPTIMIZE|REFACTOR" --type-add 'code:*.{js,ts,py,java,go,rb,php}' -t code

# 查找已弃用的标记
rg "@deprecated|DEPRECATED|@obsolete" -t code

# 查找临时代码
rg "TEMPORARY|TEMP|REMOVE BEFORE|DELETE ME" -t code -i

# 查找技术债务标记
rg "TECHNICAL DEBT|TECH DEBT|REFACTOR|NEEDS REFACTORING" -t code -i

# 查找安全隐患
rg "SECURITY|INSECURE|VULNERABILITY|CVE-" -t code -i

# 查找性能问题
rg "SLOW|PERFORMANCE|OPTIMIZE|BOTTLENECK" -t code -i
```

### 2. 解析注释上下文
从注释中提取有意义的信息：

```javascript
class CommentParser {
  parseComment(file, lineNumber, comment) {
    const parsed = {
      type: 'todo',
      priority: 'medium',
      title: '',
      description: '',
      author: null,
      date: null,
      tags: [],
      code_context: '',
      file_path: file,
      line_number: lineNumber
    };
    
    // Detect comment type
    if (comment.match(/FIXME/i)) {
      parsed.type = 'fixme';
      parsed.priority = 'high';
    } else if (comment.match(/HACK|XXX/i)) {
      parsed.type = 'hack';
      parsed.priority = 'high';
    } else if (comment.match(/OPTIMIZE|PERFORMANCE/i)) {
      parsed.type = 'optimization';
    } else if (comment.match(/DEPRECATED/i)) {
      parsed.type = 'deprecation';
      parsed.priority = 'high';
    } else if (comment.match(/SECURITY/i)) {
      parsed.type = 'security';
      parsed.priority = 'urgent';
    }
    
    // Extract author and date
    const authorMatch = comment.match(/@(\w+)|by (\w+)/i);
    if (authorMatch) {
      parsed.author = authorMatch[1] || authorMatch[2];
    }
    
    const dateMatch = comment.match(/(\d{4}-\d{2}-\d{2})|(\d{1,2}\/\d{1,2}\/\d{2,4})/);
    if (dateMatch) {
      parsed.date = dateMatch[0];
    }
    
    // Extract title and description
    const cleanComment = comment
      .replace(/^\/\/\s*|^\/\*\s*|\*\/\s*$|^#\s*/g, '')
      .replace(/TODO|FIXME|HACK|XXX/i, '')
      .trim();
    
    const parts = cleanComment.split(/[:\-–—]/);
    if (parts.length > 1) {
      parsed.title = parts[0].trim();
      parsed.description = parts.slice(1).join(':').trim();
    } else {
      parsed.title = cleanComment;
    }
    
    // Extract tags
    const tagMatch = comment.match(/#(\w+)/g);
    if (tagMatch) {
      parsed.tags = tagMatch.map(tag => tag.substring(1));
    }
    
    return parsed;
  }
  
  getCodeContext(file, lineNumber, contextLines = 5) {
    const lines = readFileLines(file);
    const start = Math.max(0, lineNumber - contextLines);
    const end = Math.min(lines.length, lineNumber + contextLines);
    
    return lines.slice(start, end).map((line, i) => ({
      number: start + i + 1,
      content: line,
      isTarget: start + i + 1 === lineNumber
    }));
  }
}
```

### 3. 分组和去重
智能整理发现的问题：

```javascript
class TaskGrouper {
  groupTasks(parsedComments) {
    const groups = {
      byFile: new Map(),
      byType: new Map(),
      byAuthor: new Map(),
      byModule: new Map()
    };
    
    for (const comment of parsedComments) {
      // Group by file
      if (!groups.byFile.has(comment.file_path)) {
        groups.byFile.set(comment.file_path, []);
      }
      groups.byFile.get(comment.file_path).push(comment);
      
      // Group by type
      if (!groups.byType.has(comment.type)) {
        groups.byType.set(comment.type, []);
      }
      groups.byType.get(comment.type).push(comment);
      
      // Group by module
      const module = this.extractModule(comment.file_path);
      if (!groups.byModule.has(module)) {
        groups.byModule.set(module, []);
      }
      groups.byModule.get(module).push(comment);
    }
    
    return groups;
  }
  
  mergeSimilarTasks(tasks) {
    const merged = [];
    const seen = new Set();
    
    for (const task of tasks) {
      if (seen.has(task)) continue;
      
      // Find similar tasks
      const similar = tasks.filter(t => 
        t !== task &&
        !seen.has(t) &&
        this.areSimilar(task, t)
      );
      
      if (similar.length > 0) {
        // Merge into one task
        const mergedTask = {
          ...task,
          title: this.generateMergedTitle(task, similar),
          description: this.generateMergedDescription(task, similar),
          locations: [task, ...similar].map(t => ({
            file: t.file_path,
            line: t.line_number
          }))
        };
        merged.push(mergedTask);
        seen.add(task);
        similar.forEach(t => seen.add(t));
      } else {
        merged.push(task);
        seen.add(task);
      }
    }
    
    return merged;
  }
}
```

### 4. 分析技术债务
识别代码质量问题：

```javascript
class TechnicalDebtAnalyzer {
  async analyzeFile(filePath) {
    const issues = [];
    const content = await readFile(filePath);
    const lines = content.split('\n');
    
    // Check for long functions
    const functionMatches = content.matchAll(/function\s+(\w+)|(\w+)\s*=\s*\(.*?\)\s*=>/g);
    for (const match of functionMatches) {
      const functionName = match[1] || match[2];
      const startLine = getLineNumber(content, match.index);
      const functionLength = this.getFunctionLength(lines, startLine);
      
      if (functionLength > 50) {
        issues.push({
          type: 'long_function',
          severity: functionLength > 100 ? 'high' : 'medium',
          title: `Refactor long function: ${functionName}`,
          description: `Function ${functionName} is ${functionLength} lines long. Consider breaking it into smaller functions.`,
          file_path: filePath,
          line_number: startLine
        });
      }
    }
    
    // Check for duplicate code
    const duplicates = await this.findDuplicateCode(filePath);
    for (const dup of duplicates) {
      issues.push({
        type: 'duplicate_code',
        severity: 'medium',
        title: 'Remove duplicate code',
        description: `Similar code found in ${dup.otherFile}:${dup.otherLine}`,
        file_path: filePath,
        line_number: dup.line
      });
    }
    
    // Check for complex conditionals
    const complexConditions = content.matchAll(/if\s*\([^)]{50,}\)/g);
    for (const match of complexConditions) {
      issues.push({
        type: 'complex_condition',
        severity: 'low',
        title: 'Simplify complex conditional',
        description: 'Consider extracting conditional logic into named variables or functions',
        file_path: filePath,
        line_number: getLineNumber(content, match.index)
      });
    }
    
    // Check for outdated dependencies
    if (filePath.endsWith('package.json')) {
      const outdated = await this.checkOutdatedDependencies(filePath);
      for (const dep of outdated) {
        issues.push({
          type: 'outdated_dependency',
          severity: dep.major ? 'high' : 'low',
          title: `Update ${dep.name} from ${dep.current} to ${dep.latest}`,
          description: dep.major ? 'Major version update available' : 'Minor update available',
          file_path: filePath
        });
      }
    }
    
    return issues;
  }
}
```

### 5. 创建线性任务
将发现转化为可执行的任务：

```javascript
async function createLinearTasks(groupedTasks, options = {}) {
  const created = [];
  const skipped = [];
  
  // Check for existing tasks to avoid duplicates
  const existingTasks = await linear.searchTasks('TODO OR FIXME');
  const existingTitles = new Set(existingTasks.map(t => t.title));
  
  // Create parent task for large groups
  if (options.createEpic && groupedTasks.length > 10) {
    const epic = await linear.createTask({
      title: `Technical Debt: ${options.module || 'Codebase'} Cleanup`,
      description: `Parent task for ${groupedTasks.length} code improvements`,
      priority: 2,
      labels: ['technical-debt', 'code-quality']
    });
    options.parentId = epic.id;
  }
  
  for (const task of groupedTasks) {
    // Skip if similar task exists
    if (existingTitles.has(task.title)) {
      skipped.push({ task, reason: 'duplicate' });
      continue;
    }
    
    // Build task description
    const description = buildTaskDescription(task);
    
    // Map priority
    const priorityMap = {
      urgent: 1,
      high: 2,
      medium: 3,
      low: 4
    };
    
    try {
      const linearTask = await linear.createTask({
        title: task.title,
        description,
        priority: priorityMap[task.priority] || 3,
        labels: getLabelsForTask(task),
        parentId: options.parentId,
        estimate: estimateTaskSize(task)
      });
      
      created.push({
        linear: linearTask,
        source: task
      });
      
      // Add code link as comment
      await linear.createComment({
        issueId: linearTask.id,
        body: `📍 Code location: \`${task.file_path}:${task.line_number}\``
      });
      
    } catch (error) {
      skipped.push({ task, reason: error.message });
    }
  }
  
  return { created, skipped };
}

function buildTaskDescription(task) {
  let description = task.description || '';
  
  // Add code context
  if (task.code_context) {
    description += '\n\n### Code Context\n```\n';
    task.code_context.forEach(line => {
      const prefix = line.isTarget ? '>>> ' : '    ';
      description += `${prefix}${line.number}: ${line.content}\n`;
    });
    description += '```\n';
  }
  
  // Add metadata
  description += '\n\n### Details\n';
  description += `- **Type**: ${task.type}\n`;
  description += `- **File**: \`${task.file_path}\`\n`;
  description += `- **Line**: ${task.line_number}\n`;
  
  if (task.author) {
    description += `- **Author**: @${task.author}\n`;
  }
  if (task.date) {
    description += `- **Date**: ${task.date}\n`;
  }
  if (task.tags.length > 0) {
    description += `- **Tags**: ${task.tags.join(', ')}\n`;
  }
  
  // Add suggestions
  if (task.type === 'deprecated') {
    description += '\n### Suggested Actions\n';
    description += '1. Identify all usages of this deprecated code\n';
    description += '2. Update to use the recommended alternative\n';
    description += '3. Add deprecation warnings if not present\n';
    description += '4. Schedule for removal in next major version\n';
  }
  
  return description;
}
```

### 6. 生成总结报告
创建调查结果概述：

```javascript
function generateReport(scanResults, createdTasks) {
  const report = {
    summary: {
      totalFound: scanResults.length,
      tasksCreated: createdTasks.created.length,
      tasksSkipped: createdTasks.skipped.length,
      byType: {},
      byPriority: {},
      byFile: {}
    },
    details: [],
    recommendations: []
  };
  
  // Analyze distribution
  for (const result of scanResults) {
    report.summary.byType[result.type] = (report.summary.byType[result.type] || 0) + 1;
    report.summary.byPriority[result.priority] = (report.summary.byPriority[result.priority] || 0) + 1;
  }
  
  // Generate recommendations
  if (report.summary.byType.security > 0) {
    report.recommendations.push({
      priority: 'urgent',
      action: 'Address security-related TODOs immediately',
      tasks: scanResults.filter(r => r.type === 'security').length
    });
  }
  
  if (report.summary.byType.deprecated > 5) {
    report.recommendations.push({
      priority: 'high',
      action: 'Create deprecation removal sprint',
      tasks: report.summary.byType.deprecated
    });
  }
  
  return report;
}
```

### 7. 错误处理

```javascript
// Handle access errors
try {
  await scanDirectory(path);
} catch (error) {
  if (error.code === 'EACCES') {
    console.warn(`Skipping ${path} - permission denied`);
  }
}

// Handle Linear API limits
const rateLimiter = {
  tasksCreated: 0,
  resetTime: Date.now() + 3600000,
  
  async createTask(taskData) {
    if (this.tasksCreated >= 50) {
      console.log('Rate limit approaching, batching remaining tasks...');
      // Create single task with list of TODOs
      return this.createBatchTask(remainingTasks);
    }
    this.tasksCreated++;
    return linear.createTask(taskData);
  }
};

// Handle malformed comments
const safeParser = {
  parse(comment) {
    try {
      return this.parseComment(comment);
    } catch (error) {
      return {
        type: 'todo',
        title: comment.substring(0, 50) + '...',
        priority: 'low',
        parseError: true
      };
    }
  }
};
```

## 示例输出

```
扫描代码库中的 TODO 和技术债务……

📊 扫描结果：
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

在 23 个文件中发现 47 项：
• 24 个 TODO
• 8 个 FIXME
• 5 个弃用函数
• 3 个安全隐患
• 7 个性能优化

🔍 细分按优先级排序：
🔴 紧急：3（安全相关）
🟠 高：13（修复 + 弃用）
🟡 中：24（标准待办事项）
🟢 低：7（优化）

📁 热点文件：
1. src/api/auth.js - 8 项
2. src/utils/validation.js - 6 项
3. src/models/User.js - 5 项

🚨 关键发现：

1. 安全：硬编码 API 密钥
文件：src/config/api.js:45
待办事项：移除硬编码密钥并使用环境变量
→ 创建“紧急”优先级的任务

2. 弃用：旧式身份验证方法
文件：src/api/auth.js:120
在 4 个文件中发现多种用法
→ 创建迁移任务

3. 修复：并发更新中的竞争条件
文件：src/services/sync.js:78
作者：@alice (2024-01-03)
→ 创建高优先级错误任务

📝 任务创建摘要：
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ 创建了 32 个线性任务：
- 史诗级：“第一季度技术债务”清理 (LIN-456)
- 3 项紧急安全任务
- 10 项高优先级修复
- 19 项中优先级改进

⏭️ 跳过 15 项：
- 8 项重复（任务已存在）
- 4 条低价值评论（例如“TODO：考虑一下”）
- 3 个外部依赖项（正在等待上游）

📊 预估：
- 总故事点数：89
- 预计工作量：2-3 个冲刺
- 建议团队规模：2-3 名开发人员

🎯 建议操作：
1. 立即安排安全冲刺（3 项紧急任务）
2. 将弃用移除任务分配到下一个冲刺（5 项）
3. 创建编码标准以减少未来 TODO
4. 设置预提交钩子以限制新的 TODO

查看所有已创建任务任务：
https://linear.app/yourteam/project/q1-technical-debt-cleanup
```

## 高级功能

### 自定义模式
定义项目特定模式：
```bash
# 添加自定义标记进行扫描
claude "扫描 REVIEW、QUESTION 和 ASSUMPTION 注释"
```

### 与 CI/CD 集成
```bash
# 如果发现关键 TODO，则构建失败
claude "检查 SECURITY 或 FIXME 注释，如果发现则出错退出"
```

### 计划扫描
```bash
# 每周技术债务报告
claude "生成每周技术债务报告并为新项目创建任务"
```

## 提示
- 定期运行以防止 TODO 累积
- 在整个团队中使用一致的注释格式
- 在 TODO 中包含作者和日期
- 尽可能将 TODO 链接到现有的线性问题
- 设置 IDE 代码片段以正确格式化 TODO
- 审查并关闭已完成的 TODO 任务
- 在 PR 审查中使用 TODO 注释作为质量门限