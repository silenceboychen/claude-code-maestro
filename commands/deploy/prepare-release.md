# 准备发布命令

准备并验证发布包

## 说明

请遵循以下系统化方法准备发布：**$ARGUMENTS**

1. **发布规划与验证**
   - 确定发布版本号（语义化版本控制）
   - 审查并验证发布中包含的所有功能
   - 检查所有计划的问题和功能是否完整
   - 验证发布标准和验收要求

2. **发布前检查清单**
   - 确保所有测试均已通过（单元测试、集成测试、端到端测试）
   - 验证代码覆盖率是否符合项目标准
   - 完成安全漏洞扫描
   - 执行性能测试和验证
   - 审查并批准所有待处理的拉取请求

3. **版本管理**
  ```bash
  # 检查当前版本
  git describe --tags --abbrev=0

  # 确定下一个版本（语义化版本控制）
  # MAJOR.MINOR.PATCH
  # MAJOR：重大变更
  # MINOR：新增功能（向后兼容）
  # PATCH：错误修复（向后兼容）兼容）

  # 版本更新示例
  # 1.2.3 -> 1.2.4（补丁）
  # 1.2.3 -> 1.3.0（次要版本）
  # 1.2.3 -> 2.0.0（主要版本）
  ```

4. **代码冻结和分支管理**
  ```bash
  # 从主分支创建发布分支
  git checkout main
  git pull origin main
  git checkout -b release/v1.2.3

  # 替代方案：直接使用主分支进行较小的版本发布
  # 确保发布过程中不会合并任何新功能
  ```

5. **版本号更新**
   - 更新 package.json、setup.py 或等效版本文件
   - 更新应用程序配置中的版本
   - 更新文档和 README 中的版本
   - 更新 API 版本（如适用）

   ```bash
   # Node.js 项目
   npm 版本补丁 # 或次要补丁、主要补丁

   # Python 项目
   # 更新 setup.py、__init__.py 或 pyproject.toml 中的版本

   # 手动更新版本
   sed -i 's/"version": "1.2.2"/"version": "1.2.3"/' package.json
   ```

6. **变更日志生成**
  ```markdown
  # CHANGELOG.md

  ## [1.2.3] - 2024-01-15

  ### 添加
  - 新的用户身份验证系统
  - UI 支持暗黑模式
  - API 速率限制功能

  ### 更改
  - 提升数据库查询性能
  - 更新用户界面设计
  - 增强错误处理

  ### 修复
  - 修复后台任务中的内存泄漏问题
  - 解决文件上传验证问题
  - 修复日期计算中的时区处理问题

  ### 安全
  - 使用安全补丁更新依赖项
  - 改进输入验证和过滤
  ```

7. **文档更新**
   - 使用新端点更新 API 文档
   - 修订用户文档和指南
   - 更新安装和部署说明
   - 审查并更新 README.md
   - 如有需要，更新迁移指南

8. **依赖管理**
  ```bash
  # 更新和审核依赖项
  npm audit fix
  npm update

  # Python
  pip-audit
  pip freeze > requirements.txt

  # 审查安全漏洞
  npm audit
  snyk test
  ```

9. **构建和工件生成**
  ```bash
  # 清理构建环境
  npm run clean
  rm -rf dist/ build/

  # 构建生产环境工件
  npm run build

  # 验证构建工件
  ls -la dist/

  # 测试已构建的工件
  npm run test:build
  ```

10. **测试和质量保证**
    - 运行全面的测试套件
    - 对关键功能进行手动测试
    - 执行回归测试
    - 进行用户验收测试
    - 在预发布环境中进行验证

    ```bash
    # 运行所有测试
    npm test
    npm run test:integration
    npm run test:e2e

    # 检查代码覆盖率
    npm run test:coverage

    # 性能测试
    npm run test:performance
    ```

11. **安全与合规性验证**
    - 运行安全扫描和渗透测试
    - 验证是否符合安全标准
    - 检查机密信息或凭证是否泄露
    - 验证数据保护和隐私措施

12. **发布说明准备**
  ```markdown
  # 发布说明 v1.2.3

  ## 🎉 新功能
  - **暗黑模式**：用户现在可以在设置中切换到暗黑模式
  - **增强安全性**：通过双重身份验证 (2FA) 支持改进身份验证
  - **性能**：页面加载时间提升 40%

  ## 🔧 改进
  - 更清晰的表单验证错误信息
  - 提升移动端响应速度
  - 增强辅助功能

  ## 🐛 错误修复
  - 修复 Safari 浏览器中的文件下载问题
  - 解决后台任务中的内存泄漏问题
  - 修复时区显示问题

  ## 📚 文档
  - 更新 API 文档
  - 新用户入门指南
  - 增强故障排除部分

  ## 🔄 迁移指南
  - 此版本无重大变更
  - 包含自动数据库迁移
  - 详情请参阅[迁移指南](链接)
  ```

13. **发布标记和版本控制**
  ```bash
  # 创建带注释的标签
  git add .
  git commit -m "chore: prepare release v1.2.3"
  git tag -a v1.2.3 -m "Release version 1.2.3

  功能：
  - 支持暗黑模式
  - 增强身份验证

  错误修复：
  - 修复文件上传问题
  - 解决内存泄漏问题

  # 将标签推送到远程
  git push origin v1.2.3
  git push origin release/v1.2.3
  ```

14. **部署准备**
    - 准备部署脚本和配置
    - 更新环境变量和密钥
    - 规划部署策略（蓝绿部署、滚动部署、金丝雀部署）
    - 设置发布监控和告警
    - 准备回滚流程

15. **暂存环境验证**
  ```bash
  # 部署到暂存环境
  ./deploy-staging.sh v1.2.3

  # 运行冒烟测试
  npm run test:smoke:staging

  # 手动验证清单
  # [ ] 用户登录/注销
  # [ ] 核心功能
  # [ ] 新功能
  # [ ] 性能指标
  # [ ] 安全检查
  ```

16. **生产部署规划**
    - 安排部署窗口
    - 通知利益相关者和用户
    - 如有需要，准备维护模式
    - 设置部署监控
    - 规划沟通策略

17. **发布自动化设置**
    ```yaml
    # GitHub Actions Release Workflow
    name: Release
    
    on:
      push:
        tags:
          - 'v*'
    
    jobs:
      release:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v3
          - name: Setup Node.js
            uses: actions/setup-node@v3
            with:
              node-version: '18'
          
          - name: Install dependencies
            run: npm ci
          
          - name: Run tests
            run: npm test
          
          - name: Build
            run: npm run build
          
          - name: Create Release
            uses: actions/create-release@v1
            env:
              GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
            with:
              tag_name: ${{ github.ref }}
              release_name: Release ${{ github.ref }}
              draft: false
              prerelease: false
    ```

18. **沟通与公告**
    - 准备发布公告
    - 更新状态页面和文档
    - 通知客户和用户
    - 在相关沟通渠道上分享
    - 更新社交媒体和营销材料

19. **发布后监控**
    - 监控应用程序性能和错误
    - 跟踪用户对新功能的采用情况
    - 监控系统指标和警报
    - 收集用户反馈和问题
    - 如有需要，准备修补程序

20. **发布回顾**
    - 记录经验教训
    - 审查发布流程的有效性
    - 识别改进机会
    - 更新发布程序
    - 规划下一个发布周期

**版本类型和注​​意事项：**

**补丁版本 (1.2.3 → 1.2.4)：**
- 仅修复错误
- 无新功能
- 只需进行少量测试
- 快速部署

**次要版本 (1.2.3 → 1.3.0)：**
- 新增功能（向后兼容）
- 功能增强
- 全面测试
- 需要用户沟通

**主要版本 (1.2.3 → 2.0.0)：**
- 重大变更
- 重要新功能
- 需要迁移指南
- 延长测试期
- 用户培训和支持

**修补程序版本：**
```bash
# 紧急修补程序流程
git checkout main
git pull origin main
git checkout -b hotfix/critical-bug-fix

# 进行少量修复
git add .
git commit -m "hotfix: 修复关键安全漏洞"

# 快速测试和部署
npm test
git tag -a v1.2.4-hotfix.1 -m "关键安全问题热修复"
git push origin hotfix/critical-bug-fix
git push origin v1.2.4-hotfix.1
```

请记住：
- 发布前彻底测试所有内容
- 与所有利益相关者清晰沟通
- 准备好回滚流程
- 部署后密切监控发布情况
- 记录所有内容，以备将来发布