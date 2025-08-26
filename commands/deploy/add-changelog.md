# 添加变更日志命令

生成并维护项目变更日志

## 说明

请按照以下步骤设置并维护变更日志：**$ARGUMENTS**

1. **变更日志格式（保留变更日志）**
```markdown
# 变更日志

本项目所有显著变更都将记录在此文件中。

此格式基于 [保留变更日志](https://keepachangelog.com/en/1.0.0/)，
且本项目遵循 [语义化版本控制](https://semver.org/spec/v2.0.0.html)。

## [未发布]
### 新增
- 新功能

### 变更
- 现有功能的变更

### 弃用
- 即将移除的功能

### 移除
- 已移除的功能

### 修复
- 错误修复

### 安全性
- 安全性改进
```

2. **版本条目**
```markdown
## [1.2.3] - 2025-08-26
### 添加
- 用户身份验证系统
- 暗黑模式切换
- 报告导出功能

### 修复
- 后台任务内存泄漏
- 时区处理问题
```

3. **自动化工具**
```bash
# 从 git 提交生成变更日志
npm install -D conventional-changelog-cli
npx conventional-changelog -p angular -i CHANGELOG.md -s

# 自动变更日志
npm install -D auto-changelog
npx auto-changelog
```

4. **提交约定**
```bash
# 自动生成的约定提交
feat: 添加用户身份验证
fix: 解决任务内存泄漏问题
docs: 更新 API 文档
style: 使用 Prettier 格式化代码
refactor: 重新组织用户服务
test: 添加单元授权测试
任务：更新依赖项
```

5. **与发布版本集成**
  - 每次发布前更新变更日志
  - 包含在发布说明中
  - 链接到 GitHub 发布版本
  - 始终使用标签标记版本

请务必保持条目清晰、分类，并重点关注用户可见的变更。