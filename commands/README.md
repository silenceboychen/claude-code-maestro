# Claude 斜杠命令 - 命名空间组织

欢迎使用 Claude 斜杠命令！这套命令被组织成逻辑命名空间，帮助您快速找到适合您任务的工具。

## 命令命名空间

### 💻 `/dev:*` - 开发工具
必备的开发实用程序，包括代码审查、调试、重构以及专用的 AI 模式（prime、sentient、ultra-think），以提供增强的辅助功能。

- [`/dev:all-tools`](dev/all-tools.md) - 显示所有可用的开发工具
- [`/dev:architecture-scenario-explorer`](dev/architecture-scenario-explorer.md) - 通过场景分析探索架构决策
- [`/dev:clean-branches`](dev/clean-branches.md) - 清理已合并和过时的 git 分支
- [`/dev:code-permutation-tester`](dev/code-permutation-tester.md) - 通过模拟测试多种代码变体
- [`/dev:code-review`](dev/code-review.md) - 全面的代码质量审查
- [`/dev:code-to-task`](dev/code-to-task.md) - 将代码分析转换为线性任务
- [`/dev:debug-error`](dev/debug-error.md) - 系统地调试和修复错误
- [`/dev:dir​​ectory-deep-dive`](dev/directory-deep-dive.md) - 分析目录结构和用途
- [`/dev:explain-code`](dev/explain-code.md) - 分析和解释代码功能
- [`/dev:fix-issue`](dev/fix-issue.md) - 识别并解决代码问题
- [`/dev:generate-linear-worklog`](dev/generate-linear-worklog.md) - 根据最近的 git 提交，为线性问题生成技术工作日志注释。
- [`/dev:git-status`](dev/git-status.md) - 显示详细的 git 仓库状态
- [`/dev:prime`](dev/prime.md) - 增强 AI 模式，适用于复杂任务
- [`/dev:refactor-code`](dev/refactor-code.md) - 智能重构，提升代码质量
- [`/dev:rule2hook`](dev/rule2hook.md) - 根据规则自动生成hook
- [`/dev:ultra-think`](dev/ultra-think.md) - 深度分析和解决问题模式

### 🧪 `/test:*` - 测试套件
全面的测试工具，涵盖单元测试、集成测试、端到端测试、覆盖率分析、变异测试和可视化回归测试。

- [`/test:generate-test-cases`](test/generate-test-cases.md) - 自动生成全面的测试用例
- [`/test:write-tests`](test/write-tests.md) - 编写单元测试和集成测试
- [`/test:test-coverage`](test/test-coverage.md) - 分析并报告测试覆盖率
- [`/test:setup-comprehensive-testing`](test/setup-comprehensive-testing.md) - 设置完整的测试基础架构
- [`/test:e2e-setup`](test/e2e-setup.md) - 配置端到端测试套件
- [`/test:setup-visual-testing`](test/setup-visual-testing.md) - 设置可视化回归测试
- [`/test:setup-load-testing`](test/setup-load-testing.md) - 配置负载和性能测试
- [`/test:add-mutation-testing`](test/add-mutation-testing.md) - 设置变异测试以提高代码质量
- [`/test:add-property-based-testing`](test/add-property-based-testing.md) - 实现基于属性的测试框架

### 🔒 `/security:*` - 安全与合规
安全审计、依赖项扫描、身份验证实施和安全强化工具，确保您的代码库安全无虞。

- [`/security:security-audit`](security/security-audit.md) - 执行全面的安全评估
- [`/security:dependency-audit`](security/dependency-audit.md) - 审计依赖项以查找安全漏洞
- [`/security:security-hardening`](security/security-hardening.md) - 强化应用程序安全配置
- [`/security:add-authentication-system`](security/add-authentication-system.md) - 实施安全的用户身份验证系统

### 📦 `/deploy:*` - 部署与发布
发布准备、自动化部署、回滚功能、容器化和 Kubernetes 部署管理。

- [`/deploy:prepare-release`](deploy/prepare-release.md) - 准备并验证发布包
- [`/deploy:hotfix-deploy`](deploy/hotfix-deploy.md) - 快速部署关键修补程序
- [`/deploy:rollback-deploy`](deploy/rollback-deploy.md) - 将部署回滚到之前的版本
- [`/deploy:setup-automated-releases`](deploy/setup-automated-releases.md) - 设置自动发布工作流程
- [`/deploy:containerize-application`](deploy/containerize-application.md) - 将应用程序容器化以进行部署
- [`/deploy:setup-kubernetes-deployment`](deploy/setup-kubernetes-deployment.md) - 配置 Kubernetes 部署清单
- [`/deploy:ci-setup`](deploy/ci-setup.md) - 设置持续集成流水线
- [`/deploy:add-changelog`](deploy/add-changelog.md) - 生成并维护项目变更日志

### 📚 `/docs:*` - 文档生成
用于 API、架构图、入门指南和故障排除文档的自动化文档工具。

- [`/docs:generate-api-documentation`](docs/generate-api-documentation.md) - 自动生成 API 参考文档
- [`/docs:doc-api`](docs/doc-api.md) - 从代码生成 API 文档
- [`/docs:create-architecture-documentation`](docs/create-architecture-documentation.md) - 生成全面的架构文档
- [`/docs:create-onboarding-guide`](docs/create-onboarding-guide.md) - 创建开发者入门指南
- [`/docs:migration-guide`](docs/migration-guide.md) - 创建更新迁移指南
- [`/docs:troubleshooting-guide`](docs/troubleshooting-guide.md) - 生成故障排除文档

## 说明

概述了 Claude Command Suite 命名空间的组织结构。要使用任何特定命令，请导航到相应的命名空间目录并选择所需的命令。每个命令文件都包含详细的使用说明。

## 用法

命令遵循命名空间模式：`/<namespace>:<command>`

例如：
- `/dev:code-review` - 执行代码审查
- `/test:generate-test-cases` - 生成测试用例
- `/security:security-audit` - 运行安全审计

## 快速入门

1. **需要调试？** 尝试 `/dev:debug-error`
2. **编写测试？** 使用 `/test:generate-test-cases`
3. **存在安全问题？** 运行 `/security:security-audit`


## 查找命令

每个命名空间目录都包含一个 README.md 文件，其中包含可用命令的详细描述。导航到任何命名空间文件夹即可查看其特定命令及其用途。

## 命令可用性

命令加载自：
- **项目级**：`.claude/commands/`（此目录）
- **用户级**：`~/.claude/commands/`（您的个人命令）

当存在命名冲突时，项目级命令优先。