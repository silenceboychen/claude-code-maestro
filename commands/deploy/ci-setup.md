# CI/CD 设置命令

设置持续集成流水线

## 说明

请遵循以下系统化方法实施 CI/CD：**$ARGUMENTS**

1. **项目分析**
  - 确定技术栈和部署需求
  - 审查现有的构建和测试流程
  - 了解部署环境（开发、预发布、生产）
  - 评估当前的版本控制和分支策略

2. **CI/CD 平台选择**
  - 根据需求选择合适的 CI/CD 平台：
    - **GitHub Actions**：原生 GitHub 集成，广泛的市场
    - **GitLab CI**：内置 GitLab，全面的 DevOps 平台
    - **Jenkins**：自托管、高度可定制、丰富的插件
    - **CircleCI**：基于云，速度优化
    - **Azure DevOps**：Microsoft 生态系统集成
    - **AWS CodePipeline**：AWS 原生解决方案

3. **代码库设置**
  - 确保 `.gitignore` 配置正确
  - 设置分支保护规则
  - 配置合并要求和审核
  - 建立语义版本控制策略

4. **构建Pipeline配置**
   
  **GitHub Actions Example:**
  ```yaml
  name: CI/CD Pipeline
  
  on:
    push:
      branches: [ main, develop ]
    pull_request:
      branches: [ main ]
  
  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - name: Setup Node.js
          uses: actions/setup-node@v3
          with:
            node-version: '18'
            cache: 'npm'
        - run: npm ci
        - run: npm run test
        - run: npm run build
  ```

  **GitLab CI Example:**
  ```yaml
  stages:
    - test
    - build
    - deploy
  
  test:
    stage: test
    script:
      - npm ci
      - npm run test
    cache:
      paths:
        - node_modules/
  ```

5. **环境配置**
  - 设置环境变量和密钥
  - 配置不同的环境（开发、预发布、生产）
  - 实现特定环境的配置
  - 设置安全的密钥管理

6. **自动化测试集成**
  - 配置单元测试执行
  - 设置集成测试运行
  - 实现端到端测试执行
  - 配置测试报告和覆盖率

    **Multi-stage Testing:**
    ```yaml
    test:
      strategy:
        matrix:
          node-version: [16, 18, 20]
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - uses: actions/setup-node@v3
          with:
            node-version: ${{ matrix.node-version }}
        - run: npm ci
        - run: npm test
    ```

7. **代码质量门控**
  - 集成 linting 和格式检查
  - 设置静态代码分析（SonarQube、CodeClimate）
  - 配置安全漏洞扫描
  - 实现代码覆盖率阈值

8. **构建优化**
  - 配置构建缓存策略
  - 实现并行作业执行
  - 优化 Docker 镜像构建
  - 设置工件管理

    **Caching Example:**
    ```yaml
    - name: Cache node modules
      uses: actions/cache@v3
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
        ${{ runner.os }}-node-
    ```

9. **Docker 集成**
  - 创建优化的 Dockerfile
  - 设置多阶段构建
  - 配置容器镜像仓库集成
  - 实现镜像安全扫描

    **多阶段 Dockerfile：**
    ```dockerfile
    FROM node:18-alpine AS builder
    WORKDIR /app
    COPY package*.json ./
    RUN npm ci --only=production
    
    FROM node:18-alpine AS runtime
    WORKDIR /app
    COPY --from=builder /app/node_modules ./node_modules
    COPY . .
    EXPOSE 3000
    CMD ["npm", "start"]
    ```

10.  **部署策略**
  - 实现蓝绿部署
  - 设置金丝雀发布
  - 配置滚动更新
  - 实现功能开关集成

11. **基础设施即代码**
- 使用 Terraform、CloudFormation 或类似工具
- 版本控制基础设施定义
- 实施基础设施测试
- 设置自动化基础设施配置

12.  **监控和可观察性**
- 设置应用程序性能监控
- 配置日志聚合和分析
- 实施健康检查和警报
- 设置部署通知

13.  **安全集成**
- 实施依赖项漏洞扫描
- 设置容器安全扫描
- 配置 SAST（静态应用程序安全测试）
- 实施机密扫描

  **Security Scanning Example:**
   ```yaml
   security:
     runs-on: ubuntu-latest
     steps:
       - uses: actions/checkout@v3
       - name: Run Snyk to check for vulnerabilities
         uses: snyk/actions/node@master
         env:
           SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
   ```

14. **数据库迁移处理**
- 自动执行数据库模式迁移
- 实施回滚策略
- 设置测试数据库种子
- 配置备份和恢复程序

15. **性能测试集成**
- 设置流水线中的负载测试
- 配置性能基准测试
- 实施性能回归检测
- 设置性能监控

16. **多环境部署**
- 配置预发布环境部署
- 设置需要审批的生产部署
- 实施环境升级工作流
- 配置特定于环境的配置

  **Environment Deployment:**
   ```yaml
   deploy-staging:
     needs: test
     if: github.ref == 'refs/heads/develop'
     runs-on: ubuntu-latest
     steps:
       - name: Deploy to staging
         run: |
           # Deploy to staging environment
   
   deploy-production:
     needs: test
     if: github.ref == 'refs/heads/main'
     runs-on: ubuntu-latest
     environment: production
     steps:
       - name: Deploy to production
         run: |
           # Deploy to production environment
   ```

17. **回滚与恢复**
- 实施自动回滚程序
- 设置部署验证测试
- 配置故障检测和警报
- 记录手动恢复程序

18. **通知与报告**
- 设置 Slack/Teams 集成以接收通知
- 配置故障电子邮件警报
- 实施部署状态报告
- 设置指标仪表板

19. **合规与审计**
- 实施部署审计跟踪
- 设置合规性检查（SOC 2、HIPAA 等）
- 为敏感部署配置审批工作流
- 记录变更管理流程

20. **管道优化**
- 监控管道性能和成本
- 实施管道并行化
- 优化资源分配
- 设置管道分析和报告

**最佳实践：**

1. **快速失败**：实现早期故障检测
2. **并行执行**：并行运行独立作业
3. **缓存**：缓存依赖项和构建工件
4. **安全**：切勿在日志中泄露机密信息
5. **文档**：记录流水线流程和规程
6. **监控**：监控流水线的健康状况和性能
7. **测试**：测试功能分支中的流水线变更
8. **回滚**：始终制定回滚策略

**示例完整Pipeline:**
```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run test:coverage
      - run: npm run build

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Security scan
        run: npm audit --audit-level=high

  deploy-staging:
    needs: [lint-and-test, security-scan]
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to staging
        run: echo "Deploying to staging"

  deploy-production:
    needs: [lint-and-test, security-scan]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to production
        run: echo "Deploying to production"
```

从基本的 CI 开始，随着团队和项目的成熟逐渐添加更复杂的功能。