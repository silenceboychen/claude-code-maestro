# 回滚部署命令

将部署回滚到之前的版本

## 说明

请遵循以下系统性回滚程序：**$ARGUMENTS**

1. **事件评估与决策**
   - 评估当前部署问题的严重性和影响
   - 确定是否需要回滚或向前修复是否更好
   - 识别受影响的系统、用户和业务功能
   - 考虑数据完整性和一致性的影响
   - 记录决策依据和时间表

2. **紧急响应设置**
  ```bash
  # 启动事件响应团队
  # 建立沟通渠道
  # 立即通知利益相关者

  # 紧急通知示例
  echo "🚨 已启动回滚
  问题：v1.3.0 部署后性能严重下降
  操作：回滚到 v1.2.9
  预计完成时间：15 分钟
  影响：可能出现服务暂时中断
  状态通道：#incident-rollback-202401"
  ```

3. **回滚前安全检查**
  ```bash
  # 验证当前生产版本
  curl -s https://api.example.com/version
  kubectl get deploys -o wide

  # 检查系统状态
  curl -s https://api.example.com/health | jq .

  # 确定目标回滚版本
  git tag --sort=-version:refname | head -5

  # 验证回滚目标是否存在且可部署
  git show v1.2.9 --stat
  ```

4. **数据库注意事项**
  ```bash
  # 检查自上一版本以来的数据库迁移
  ./check-migrations.sh v1.2.9 v1.3.0

  # 如果存在迁移，请计划数据库回滚
  # 警告：数据库回滚可能导致数据丢失
  # 如果存在迁移，请考虑进行前向修复

  # 回滚前创建数据库备份
  ./backup-database.sh "pre-rollback-$(date +%Y%m%d-%H%M%S)"
  ```

5. **流量管理准备**
  ```bash
  # 准备重定向流量
  # 选项 1：维护页面
  ./enable-maintenance-mode.sh

  # 选项 2：负载均衡器管理
  ./drain-traffic.sh --gradual

  # 选项 3：熔断器激活
  ./activate-circuit-breaker.sh
  ```

6. **容器/Kubernetes回滚**
  ```bash
  # Kubernetes 回滚
  kubectl rollout history deployment/app-deployment
  kubectl rollout undo deployment/app-deployment

  # 或回滚到特定版本
  kubectl rollout undo deployment/app-deployment --to-revision=3

  # 监控回滚进度
  kubectl rollout status deployment/app-deployment --timeout=300s

  # 验证 pod 是否正在运行
  kubectl get pods -l app=your-app
  ```

7. **Docker Swarm 回滚**
```bash
# 列出服务历史记录
docker service ps app-service --no-trunc

# 回滚到之前的版本
docker service update --rollback app-service

# 或更新到特定镜像
docker service update --image app:v1.2.9 app-service

# 监控回滚
docker service ps app-service
```

8. **传统部署回滚**
```bash
# 蓝绿部署回滚
./switch-to-blue.sh # 或 green，取决于当前版本

# 滚动部署回滚
./deploy-version.sh v1.2.9 --rolling

# 基于符号链接的回滚
ln -sfn /releases/v1.2.9 /current
sudo systemctl restart app-service
```

9. **负载均衡器和 CDN 更新**
```bash
# 更新负载均衡器指向旧版本
aws elbv2 modify-target-group --target-group-arn $TG_ARN --targets Id=old-instance

# 如有需要，清除 CDN 缓存
aws cloudfront create-invalidation --distribution-id $DIST_ID --paths \"/*\"

# 如有需要，更新 DNS（最后手段，有传播延迟）
# aws route53 change-resource-record-sets ...
```

10. **配置回滚**

```bash
# Rollback configuration files\n    git checkout v1.2.9 -- config/\n    \n    # Restart services with old configuration\n    sudo systemctl restart nginx\n    sudo systemctl restart app-service\n    \n    # Rollback environment variables\n    ./restore-env-vars.sh v1.2.9\n    \n    # Update feature flags\n    ./update-feature-flags.sh --disable-new-features\n    ```\n\n11. **Database Rollback (if necessary)**\n    ```sql\n    -- EXTREME CAUTION: Can cause data loss\n    \n    -- Check migration status\n    SELECT * FROM schema_migrations ORDER BY version DESC LIMIT 5;\n    \n    -- Rollback specific migrations (framework dependent)\n    -- Rails: rake db:migrate:down VERSION=20240115120000\n    -- Django: python manage.py migrate app_name 0001\n    -- Node.js: npm run migrate:down\n    \n    -- Verify database state\n    SHOW TABLES;\n    DESCRIBE critical_table;\n    ```\n\n12. **Service Health Validation**\n    ```bash\n    # Health check script\n    #!/bin/bash\n    \n    echo \"Validating rollback...\"\n    \n    # Check application health\n    if curl -f -s https://api.example.com/health > /dev/null; then\n        echo \"✅ Health check passed\"\n    else\n        echo \"❌ Health check failed\"\n        exit 1\n    fi\n    \n    # Check critical endpoints\n    endpoints=(\n        \"/api/users/me\"\n        \"/api/auth/status\"\n        \"/api/data/latest\"\n    )\n    \n    for endpoint in \"${endpoints[@]}\"; do\n        if curl -f -s \"https://api.example.com$endpoint\" > /dev/null; then\n            echo \"✅ $endpoint working\"\n        else\n            echo \"❌ $endpoint failed\"\n        fi\n    done\n    ```\n\n13. **Performance and Metrics Validation**\n    ```bash\n    # Check response times\n    curl -w \"Response time: %{time_total}s\\n\" -s -o /dev/null https://api.example.com/\n    \n    # Monitor error rates\n    tail -f /var/log/app/error.log | head -20\n    \n    # Check system resources\n    top -bn1 | head -10\n    free -h\n    df -h\n    \n    # Validate database connectivity\n    mysql -u app -p -e \"SELECT 1;\"\n    ```\n\n14. **Traffic Restoration**\n    ```bash\n    # Gradually restore traffic\n    ./restore-traffic.sh --gradual\n    \n    # Disable maintenance mode\n    ./disable-maintenance-mode.sh\n    \n    # Re-enable circuit breakers\n    ./deactivate-circuit-breaker.sh\n    \n    # Monitor traffic patterns\n    ./monitor-traffic.sh --duration 300\n    ```\n\n15. **Monitoring and Alerting**\n    ```bash\n    # Enable enhanced monitoring during rollback\n    ./enable-enhanced-monitoring.sh\n    \n    # Watch key metrics\n    watch -n 10 'curl -s https://api.example.com/metrics | jq .'\n    \n    # Monitor logs in real-time\n    tail -f /var/log/app/*.log | grep -E \"ERROR|WARN|EXCEPTION\"\n    \n    # Check application metrics\n    # - Response times\n    # - Error rates\n    # - User sessions\n    # - Database performance\n    ```\n\n16. **User Communication**\n    ```markdown\n    ## Service Update - Rollback Completed\n    \n    **Status:** ✅ Service Restored\n    **Time:** 2024-01-15 15:45 UTC\n    **Duration:** 12 minutes of degraded performance\n    \n    **What Happened:**\n    We identified performance issues with our latest release and \n    performed a rollback to ensure optimal service quality.\n    \n    **Current Status:**\n    - All services operating normally\n    - Performance metrics back to baseline\n    - No data loss occurred\n    \n    **Next Steps:**\n    We're investigating the root cause and will provide updates \n    on our status page.\n    ```\n\n17. **Post-Rollback Validation**\n    ```bash\n    # Extended monitoring period\n    ./monitor-extended.sh --duration 3600  # 1 hour\n    \n    # Run integration tests\n    npm run test:integration:production\n    \n    # Check user-reported issues\n    ./check-support-tickets.sh --since \"1 hour ago\"\n    \n    # Validate business metrics\n    ./check-business-metrics.sh\n    ```\n\n18. **Documentation and Reporting**\n    ```markdown\n    # Rollback Incident Report\n    \n    **Incident ID:** INC-2024-0115-001\n    **Rollback Version:** v1.2.9 (from v1.3.0)\n    **Start Time:** 2024-01-15 15:30 UTC\n    **End Time:** 2024-01-15 15:42 UTC\n    **Total Duration:** 12 minutes\n    \n    **Timeline:**\n    - 15:25 - Performance degradation detected\n    - 15:30 - Rollback decision made\n    - 15:32 - Traffic drained\n    - 15:35 - Rollback initiated\n    - 15:38 - Rollback completed\n    - 15:42 - Traffic fully restored\n    \n    **Impact:**\n    - 12 minutes of degraded performance\n    - ~5% of users experienced slow responses\n    - No data loss or corruption\n    - No security implications\n    \n    **Root Cause:**\n    Memory leak in new feature causing performance degradation\n    \n    **Lessons Learned:**\n    - Need better performance testing in staging\n    - Improve monitoring for memory usage\n    - Consider canary deployments for major releases\n    ```\n\n19. **Cleanup and Follow-up**\n    ```bash\n    # Clean up failed deployment artifacts\n    docker image rm app:v1.3.0\n    \n    # Update deployment status\n    ./update-deployment-status.sh \"rollback-completed\"\n    \n    # Reset feature flags if needed\n    ./reset-feature-flags.sh\n    \n    # Schedule post-incident review\n    ./schedule-postmortem.sh --date \"2024-01-16 10:00\"\n    ```\n\n20. **Prevention and Improvement**\n    - Analyze what went wrong with the deployment\n    - Improve testing and validation procedures\n    - Enhance monitoring and alerting\n    - Update rollback procedures based on learnings\n    - Consider implementing canary deployments\n\n**Rollback Decision Matrix:**\n\n| Issue Severity | Data Impact | Time to Fix | Decision |\n|---------------|-------------|-------------|----------|\n| Critical | None | > 30 min | Rollback |\n| High | Minor | > 60 min | Rollback |\n| Medium | None | > 2 hours | Consider rollback |\n| Low | None | Any | Forward fix |\n\n**Emergency Rollback Script Template:**\n```bash\n#!/bin/bash\nset -e\n\n# Emergency rollback script\nPREVIOUS_VERSION=\"${1:-v1.2.9}\"\nCURRENT_VERSION=$(curl -s https://api.example.com/version)\n\necho \"🚨 EMERGENCY ROLLBACK\"\necho \"From: $CURRENT_VERSION\"\necho \"To: $PREVIOUS_VERSION\"\necho \"\"\n\n# Confirm rollback\nread -p \"Proceed with rollback? (yes/no): \" confirm\nif [ \"$confirm\" != \"yes\" ]; then\n    echo \"Rollback cancelled\"\n    exit 1\nfi\n\n# Execute rollback\necho \"Starting rollback...\"\nkubectl set image deployment/app-deployment app=app:$PREVIOUS_VERSION\nkubectl rollout status deployment/app-deployment --timeout=300s\n\n# Validate\necho \"Validating rollback...\"\nsleep 30\ncurl -f https://api.example.com/health\n\necho \"✅ Rollback completed successfully\"
```

记住：回滚应该是最后的手段。始终优先考虑向前修复，尤其是在涉及数据库迁移时。