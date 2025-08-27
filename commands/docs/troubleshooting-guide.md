# 故障排除指南生成器

生成故障排除文档

## 说明

请遵循以下系统化方法创建故障排除指南：**$ARGUMENTS**

1. **系统概述和架构**
   - 记录系统架构和组件
   - 规划依赖关系和集成
   - 识别关键路径和故障点
   - 创建系统拓扑图
   - 记录数据流和通信模式

2. **常见问题识别**
   - 收集历史支持工单和问题
   - 与团队成员面谈常见问题
   - 分析错误日志和监控数据
   - 审查用户反馈和投诉
   - 识别系统故障模式

3. **故障排除框架**
   - 建立系统化诊断程序
   - 创建问题隔离方法
   - 记录升级路径和程序
   - 设置日志和监控检查点
   - 定义严重级别和响应时间

4. **诊断工具和命令**

  ```markdown
  ## 基本诊断命令

  ### 系统健康
  ```bash
  # 检查系统资源
  top # CPU 和内存使用情况
  df -h # 磁盘空间
  free -m # 内存使用情况
  netstat -tuln # 网络连接

  # 应用程序日志
  tail -f /var/log/app.log
  journalctl -u 服务名称 -f

  # 数据库连接
  mysql -u user -p -e "SELECT 1"
  psql -h host -U user -d db -c "SELECT 1"
  ```
  ```

5. **问题类别和解决方案**

  **性能问题：**
  ```markdown
  ### 响应时间慢

  **症状：**
  - API 响应时间超过 5 秒
  - 用户界面卡顿
  - 数据库超时

  **诊断步骤：**
  1. 检查系统资源（CPU、内存、磁盘）
  2. 查看应用程序日志中的错误
  3. 分析数据库查询性能
  4. 检查网络连接和延迟

  **常见原因：**
  - 数据库连接池耗尽
  - 数据库查询效率低下
  - 应用程序中的内存泄漏
  - 网络带宽限制

  **解决方案：**
  - 重启应用程序服务
  - 优化数据库查询
  - 增加连接池大小
  - 扩展基础架构资源
  ```

6. **错误代码文档**

  ```markdown
  ## 错误代码参考

  ### HTTP 状态代码
  - **500 内部服务器错误**
    - 检查应用程序日志中的堆栈跟踪
    - 验证数据库连接
    - 检查环境变量

  - **404 未找到**
    - 验证 URL 路由配置
    - 检查资源是否存在
    - 查看 API 端点文档

  - **503 服务不可用**
    - 检查服务健康状态
    - 验证负载均衡器配置
    - 检查维护模式
  ```

7. **特定环境问题**
   - 记录开发环境问题
   - 解决预发布/测试环境问题
   - 涵盖生产环境相关的故障排除
   - 包含本地开发设置问题

8. **数据库故障排除**

  ```markdown
  ### 数据库连接问题

  **症状：**
  - “连接被拒绝”错误
  - “连接过多”错误
  - 查询性能缓慢

  **诊断命令：**
  ```sql
  -- 检查活动连接数
  SHOW PROCESSLIST;

  -- 检查数据库大小
  SELECT table_schema,
  ROUND(SUM(data_length + index_length) / 1024 / 1024, 1) AS '数据库大小 (MB)'
  FROM information_schema.tables
  GROUP BY table_schema;

  -- 检查慢查询
  SHOW VARIABLES LIKE 'slow_query_log';
  ```
  ```

9. **网络和连接问题**

  ```markdown
  ### 网络故障排除

  **基本连接：**
  ```bash
  # 测试基本连接
  ping example.com
  telnet host port
  curl -v https://api.example.com/health

  # DNS 解析
  nslookup example.com
  dig example.com

  # 网络路由
  traceroute example.com
  ```

  **SSL/TLS 问题：**
  ```bash
  # 检查 SSL 证书
  openssl s_client -connect example.com:443
  curl -vI https://example.com
  ```
  ```

10. **特定应用程序故障排除**

  **内存问题：**
  ```markdown
  ### 内存不足错误

  **Java 应用程序：**
  ```bash
  # 检查堆使用情况
  jstat -gc [PID]
  jmap -dump:format=b,file=heapdump.hprof [PID]

  # 分析堆转储
  jhat heapdump.hprof
  ```

  **Node.js 应用程序：**
  ```bash
  # 监控内存使用情况
  node --inspect app.js
  # 使用 Chrome DevTools 进行内存分析
  ```
  ```

11. **安全和身份验证问题**

  ```markdown
  ### 身份验证失败

  **症状：**
  - 401 未授权响应
  - 令牌验证错误
  - 会话超时问题

  **诊断步骤：**
  1. 验证凭据和令牌
  2. 检查令牌是否过期
  3. 验证身份验证服务
  4. 检查 CORS 配置

  **常见解决方案：**
  - 刷新身份验证令牌
  - 清除浏览器 Cookie/缓存
  - 验证 CORS 标头
  - 检查 API 密钥权限
  ```

12. **部署和配置问题**

  ```markdown
  ### 部署失败

  **容器问题：**
  ```bash
  # 检查容器状态
  docker ps -a
  docker logs container-name

  # 检查资源限制
  docker stats

  # 调试容器
  docker exec -it container-name /bin/bash
  ```

  **Kubernetes 问题：**
  ```bash
  # 检查 Pod 状态
  kubectl get pods
  kubectl describe pod pod-name
  kubectl logs pod-name

  # 检查服务连接性
  kubectl get svc
  kubectl port-forward pod-name 8080:8080
  ```
  ```

13. **监控和警报设置**
  - 配置健康检查和监控
  - 设置日志聚合和分析
  - 实施关键问题警报
  - 创建系统指标仪表板
  - 记录监控阈值

14. **升级程序**

  ```markdown
  ## 升级矩阵

  ### 严重程度

  **严重 (P1)：**系统崩溃，数据丢失
  - 需要立即响应
  - 升级至值班工程师
  - 30 分钟内通知管理层

  **高 (P2)：**主要功能受损
  - 2 小时内响应
  - 升级至高级工程师
  - 每小时提供更新

  **中 (P3)：**轻微功能问题
  - 8 小时内响应
  - 分配给合适的团队成员
  - 提供每日更新
  ```

15. **恢复程序**
    - 记录系统恢复步骤
    - 创建数据备份和恢复程序
    - 建立部署回滚程序
    - 记录灾难恢复流程
    - 定期测试恢复程序

16. **预防措施**
    - 实施监控和警报
    - 设置自动健康检查
    - 创建部署验证程序
    - 建立代码审查流程
    - 记录维护程序

17. **知识库集成**
    - 链接至相关文档
    - 参考 API 文档
    - 包含监控仪表板链接
    - 连接团队沟通渠道
    - 与工单系统集成

18. **团队沟通**

  ```markdown
  ## 沟通渠道

  ### 立即响应
  - Slack：#incidents Channels
  - 电话：轮班值班
  - 电子邮件：alerts@company.com

  ### 状态更新
  - 状态页面：status.company.com
  - Twitter：@company_status
  - 内部 Wiki：故障排除部分
  ```

19. **文档维护**
    - 定期审查和更新
    - 故障排除指南的版本控制
    - 收集用户反馈
    - 与事件事后分析集成
    - 持续改进流程

20. **自助服务工具**
    - 创建诊断脚本和工具
    - 构建自动恢复程序
    - 实施自我修复系统
    - 提供用户友好的诊断界面
    - 为常见问题创建聊天机器人集成

**高级故障排除技术：**

**日志分析：**
```bash
# 搜索特定错误
grep -i "error" /var/log/app.log | tail -50

# 分析日志模式
awk '{print $1}' access.log | sort | uniq -c | sort -nr

# 实时监控日志
tail -f /var/log/app.log | grep -i "exception"
```

**性能分析：**
```bash
# 系统性能
iostat -x 1
sar -u 1 10
vmstat 1 10

# 应用程序分析
strace -p [PID]
perf record -p [PID]
```

请记住：
- 保持故障排除指南的更新
- 定期测试所有记录的流程
- 收集用户反馈并改进指南
- 在有用的位置添加屏幕截图和视觉辅助工具
- 使指南易于搜索且条理清晰