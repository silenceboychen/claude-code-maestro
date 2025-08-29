# 修补程序部署命令

快速部署关键修补程序

## 说明

请遵循以下紧急修补程序部署流程：**$ARGUMENTS**

1. **紧急情况评估与分类**
   - 评估问题的严重性和影响
   - 确定修补程序是否有必要或是否可以等待
   - 识别受影响的系统和用户影响
   - 评估时间敏感性和业务影响
   - 记录事件和决策依据

2. **事件响应设置**
   - 在事件管理系统中创建事件跟踪
   - 建立作战室或沟通渠道
   - 通知利益相关者和值班团队成员
   - 建立清晰的沟通协议
   - 记录初始事件详细信息和时间表

3. **分支和环境设置**
  ```bash
  # 从生产标签创建修补程序分支
  git fetch --tags
  git checkout tags/v1.2.3 # 最新生产版本
  git checkout -b hotfix/critical-auth-fix

  # 替代方案：如果使用，则从主分支创建分支基于主干的开发
  git checkout main
  git pull origin main
  git checkout -b hotfix/critical-auth-fix
  ```

4. **快速开发流程**
   - 尽量减少变更，并专注于关键问题
   - 避免重构、优化或无关的改进
   - 使用经过充分测试的模式和成熟的方法
   - 添加最少的日志记录以进行故障排除
   - 遵循现有的代码约定和模式

5. **加速测试**
  ```bash
  # 运行与修复相关的重点测试
  npm test -- --testPathPattern=auth
  npm run test:security

  # 手动测试清单
  # [ ] 核心功能正常运行
  # [ ] 热修复解决了关键问题
  # [ ] 没有引入新问题
  # [ ] 关键用户流程保持正常运行
  ```

6. **快速代码审查**
   - 获得高级团队成员的快速审查
   - 重点审查安全性和正确性
   - 如果条件允许且时间允许，请使用结对编程
   - 快速记录审查决策和理由
   - 即使在时间压力下也要确保审批流程的合理性

7. **版本和标记**
  ```bash
  # 更新修补程序版本
  # 1.2.3 -> 1.2.4（修补程序版本）
  # 或 1.2.3 -> 1.2.3-hotfix.1（修补程序标识符）

  # 提交并显示详细信息
  git add .
  git commit -m "hotfix: 修复关键身份验证漏洞

  - 修复密码验证逻辑
  - 解决允许绕过的安全问题
  - 进行最小更改以降低部署风险

  修复：#1234"

  # 标记修补程序版本
  git tag -a v1.2.4 -m "Hotfix v1.2.4: 关键身份验证安全修复"
  git push origin hotfix/critical-auth-fix
  git push origin v1.2.4
  ```

8. **暂存部署和验证**
  ```bash
  # 部署到暂存环境进行最终验证
  ./deploy-staging.sh v1.2.4

  # 关键路径测试
  curl -X POST staging.example.com/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"testpass"}'

  # 运行冒烟测试
  npm run test:smoke:staging
  ```

9. **生产部署策略**

  **蓝绿部署：**
  ```bash
  # 部署到蓝色环境
  ./deploy-blue.sh v1.2.4

  # 验证蓝色环境健康状况
  ./health-check-blue.sh

  # 将流量切换到蓝色环境
  ./switch-to-blue.sh

  # 监控部署指标
  ./monitor-deployment.sh
  ```

  **滚动部署：**
  ```bash
  # 首先部署到部分服务器
  ./deploy-rolling.sh v1.2.4 --batch-size 1

  # 监控每个批次部署
  ./monitor-batch.sh

  # 如果健康，继续下一个批次
  ./deploy-next-batch.sh
  ```

10. **部署前检查清单**
  ```bash
  # 验证所有先决条件已满足
  #[] 数据库备份已成功完成
  #[] 回滚计划已记录并准备就绪
  #[] 监控警报已配置并处于活动状态
  #[] 团队成员随时待命提供支持
  #[] 沟通渠道已建立

  # 执行生产部署
  ./deploy-production.sh v1.2.4

  # 立即运行部署后验证
  ./validate-hotfix.sh
  ```

11. **实时监控**
  ```bash
  # 监控关键应用指标
  watch -n 10 'curl -s https://api.example.com/health | jq .'

  # 监控错误率和日志
  tail -f /var/log/app/error.log | grep -i "auth"

  # 跟踪关键指标：
  # - 响应时间和延迟
  # - 错误率和异常计数
  # - 用户身份验证成功率
  # - 系统资源使用情况（CPU、内存）
  ```

12. **部署后验证**
  ```bash
  # 运行全面的验证测试
  ./test-critical-paths.sh

  # 测试用户身份验证功能
  curl -X POST https://api.example.com/auth/login \
      -H "Content-Type: application/json" \
      -d '{"email":"test@example.com","password":"testpass"}'

  # 验证安全修复的有效性
  ./security-validation.sh

  # 检查整体系统性能
  ./performance-check.sh
  ```

13. **沟通和状态更新**
    - 定期向利益相关者提供状态更新
    - 使用一致的沟通渠道
    - 记录部署进度和结果
    - 更新事件跟踪系统
    - 通知部署完成相关团队

14. **回滚程序**
  ```bash
  # 自动回滚脚本
  #!/bin/bash
  PREVIOUS_VERSION="v1.2.3"

  if [ "$1" = "rollback" ]; then
    echo "回滚到 $PREVIOUS_VERSION"
    ./deploy-production.sh $PREVIOUS_VERSION
    ./validate-rollback.sh
    echo "回滚已成功完成"
  fi

  # 如果自动化操作失败，请执行手动回滚步骤：
  # 1. 将负载均衡器切换回上一版本
  # 2. 验证上一版本的运行状况和功能
  # 3. 回滚后监控系统稳定性
  # 4. 将回滚状态告知团队
  ```

15. **部署后监控期**
    - 部署后监控系统 2-4 小时
    - 密切关注错误率和性能指标
    - 检查用户反馈和支持工单数量
    - 验证修补程序是否解决了原始问题
    - 记录任何问题或意外行为

16. **文档和事件报告**
    - 记录完整的修补程序流程和时间表
    - 记录经验教训和流程改进
    - 更新事件管理系统并记录解决方案
    - 创建事件后审查材料
    - 与团队共享知识以供将来参考

17. **合并回主分支**
  ```bash
  # 成功部署和验证修补程序后
  git checkout main
  git pull origin main
  git merge hotfix/critical-auth-fix
  git push origin main

  # 清理修补程序分支
  git branch -d hotfix/critical-auth-fix
  git push origin --delete hotfix/critical-auth-fix
  ```

18. **事件后活动**
    - 安排并进行事件后审查会议
    - 更新运行手册和应急程序
    - 识别并实施流程改进
    - 更新监控和警报配置
    - 规划预防措施以避免类似问题发生

**热修复最佳实践：**

- **保持简单：** 仅针对关键问题进行少量更改
- **全面测试：** 即使在时间压力下也要保持测试标准
- **清晰沟通：** 在整个过程中让所有利益相关者了解情况
- **密切监控：** 在生产环境中仔细观察修复过程
- **记录所有内容：** 记录所有决策和行动以供事件后审查
- **回滚计划：** 始终拥有经过测试的快速恢复更改的方法
- **学习和改进：** 利用每次事件来强化流程和程序

**紧急情况升级指南：**

```bash
# 紧急联系人信息
ON_CALL_ENGINEER="+1-555-0123"
SENIOR_ENGINEER="+1-555-0124"
ENGINEERING_MANAGER="+1-555-0125"
INCIDENT_COMMANDER="+1-555-0126"

# 升级时间阈值：
# 15 分钟：升级至高级工程师
# 30 分钟：升级至工程经理
# 60 分钟：升级至事件指挥官
```

**重要提醒：**

- 修补程序应仅用于真正的生产紧急情况
- 如对严重性存疑，请遵循正常的发布流程
- 始终优先考虑系统稳定性而非部署速度
- 为所有紧急变更保留清晰的审计线索
- 定期演练有助于确保团队做好应对真正紧急情况的准备