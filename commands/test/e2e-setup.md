# 端到端测试设置

配置端到端测试套件

## 说明

请遵循以下系统化方法实施端到端测试：**$ARGUMENTS**

1. **技术栈评估**
   - 确定应用程序类型（Web 应用、移动应用、API 服务）
   - 审查现有测试基础架构
   - 确定目标浏览器和设备
   - 评估当前的部署和预发布环境

2. **端到端框架选择**
   - 根据技术栈选择合适的端到端测试框架：
     - **Playwright**：现代、快速，支持多种浏览器
     - **Cypress**：开发者友好，优秀的调试工具
     - **Selenium WebDriver**：跨浏览器，成熟的生态系统
     - **Puppeteer**：以 Chrome 为核心，适合性能测试
     - **TestCafe**：无需 WebDriver，设置简便
   - 考虑团队专业知识和项目需求

3. **测试环境设置**
   - 设置专用测试环境（预发布、质量保证 (QA)
   - 使用示例数据配置测试数据库
   - 设置环境变量和配置
   - 确保环境隔离性和可重复性

4. **框架安装和配置**

  **对于 Playwright：**
  ```bash
  npm install -D @playwright/test
  npx playwright install
  npx playwright codegen # 录制测试
  ```

  **对于 Cypress：**
  ```bash
  npm install -D cypress
  npx cypress open
  ```

  **对于 Selenium：**
  ```bash
  npm install -D selenium-webdriver
  # 安装浏览器驱动程序
  ```

5. **测试结构组织**
   - 创建逻辑测试文件夹结构：
    ```
     e2e/
     ├── tests/
     │   ├── auth/
     │   ├── user-flows/
     │   └── api/
     ├── fixtures/
     ├── support/
     │   ├── commands/
     │   └── page-objects/
     └── config/
     ```
   - 按功能或用户旅程组织测试
   - 将 API 测试与 UI 测试分开

6. **页面对象模型实现**
   - 创建页面对象类以提高可维护性
   - 封装元素选择器和交互
   - 为常用操作实现可复用方法
   - 遵循页面对象的单一职责原则

   **页面对象示例：**
   ```javascript
   class LoginPage {
     constructor(page) {
       this.page = page;
       this.emailInput = page.locator('#email');
       this.passwordInput = page.locator('#password');
       this.loginButton = page.locator('#login-btn');
     }

     async login(email, password) {
       await this.emailInput.fill(email);
       await this.passwordInput.fill(password);
       await this.loginButton.click();
     }
   }
   ```

7. **测试数据管理**
   - 创建测试夹具和样本数据
   - 实现动态测试数据的数据工厂
   - 设置数据库种子以确保一致的测试状态
   - 使用特定于环境的测试数据
   - 实施测试数据清理策略

7. **测试数据管理**
   - 创建测试夹具和示例数据
   - 实现动态测试数据的数据工厂
   - 设置数据库种子以确保一致的测试状态
   - 使用特定于环境的测试数据
   - 实现测试数据清理策略

8. **核心用户旅程测试**
   - 实现关键用户流程：
   - 用户注册和身份验证
   - 主要应用程序工作流
   - 支付和交易流程
   - 搜索和过滤功能
   - 表单提交和验证

9. **跨浏览器测试设置**
   - 配置跨多个浏览器的测试
   - 设置特定于浏览器的配置
   - 实现响应式设计测试
   - 在不同视口尺寸下进行测试

   **Playwright 浏览器配置：**
   ```javascript
   module.exports = {
     projects: [
       { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
       { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
       { name: 'webkit', use: { ...devices['Desktop Safari'] } },
       { name: 'mobile', use: { ...devices['iPhone 12'] } },
     ],
   };
   ```

10. **API 测试集成**
    - 与 UI 测试同时测试 API 端点
    - 实现 API 请求/响应验证
    - 测试身份验证和授权
    - 验证 API 和 UI 之间的数据一致性

11. **可视化测试设置**
    - 实现屏幕截图对比测试
    - 设置可视化回归测试
    - 配置可视化变化的容忍度
    - 整理可视化基线和更新

12. **测试实用程序和助手**
    - 创建自定义命令和实用程序
    - 实现常用断言助手
    - 设置身份验证助手
    - 创建数据库和状态管理实用程序

13. **错误处理和调试**
    - 配置合适的错误报告和屏幕截图
    - 设置失败测试的视频录制
    - 实现不稳定测试的重试机制
    - 创建调试工具和助手

14. **CI/CD 集成**
    - 在 CI/CD 流水线中配置端到端测试
    - 设置并行测试执行
    - 实现合适的测试报告
    - 配置测试环境配置

    **GitHub Actions 示例：**
    ```yaml
    - name: Run Playwright tests
      run: npx playwright test
    - uses: actions/upload-artifact@v3
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
    ```

15. **性能测试集成**
    - 为端到端测试添加性能断言
    - 监控页面加载时间和指标
    - 在不同网络条件下进行测试
    - 实现灯塔审核集成

16. **无障碍测试**
    - 集成无障碍测试工具 (axe-core)
    - 测试键盘导航流程
    - 验证屏幕阅读器兼容性
    - 检查颜色对比度和 WCAG 合规性

17. **移动测试设置**
    - 配置移动设备模拟
    - 测试响应式设计断点
    - 实现触摸手势测试
    - 测试特定于移动设备的功能

18. **报告和监控**
    - 设置全面的测试报告
    - 配置测试结果通知
    - 实现测试指标和分析
    - 创建用于测试健康监控的仪表盘

19. **测试维护策略**
    - 实现测试稳定性监控
    - 设置 UI 更改的自动测试更新
    - 创建测试审查和更新流程
    - 记录测试维护程序

20. **安全测试集成**
    - 测试身份验证和授权流程
    - 实现安全标头验证
    - 测试输入过滤和 XSS 预防
    - 验证 HTTPS 和安全 Cookie 处理

**E2E 测试示例：**
```javascript
test('用户可以完成购买流程', async ({ page }) => {
  // 导航并登录
  await page.goto('/login');
  await page.fill('#email', 'test@example.com');
  await page.fill('#password', 'password');
  await page.click('#login-btn');

  // 将商品添加到购物车
  await page.goto('/products');
  await page.click('[data-testid="product-1"]');
  await page.click('#add-to-cart');

  // 完成结账
  await page.goto('/checkout');
  await page.fill('#card-number', '41111111111111111');
  await page.click('#place-order');

  // 验证是否成功
  await expect(page.locator('#order-confirmation')).toBeVisible();
});
```

记住要从关键的用户旅程开始，并逐步扩大覆盖范围。专注于提供真正价值的稳定、可维护的测试。