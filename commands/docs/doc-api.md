# API 文档生成器命令

从代码生成 API 文档

## 说明

请遵循以下系统化方法创建 API 文档：**$ARGUMENTS**

1. **代码分析与发现**
   - 扫描代码库中的 API 端点、路由和处理程序
   - 识别 REST API、GraphQL 模式和 RPC 服务
   - 规划控制器类、路由定义和中间件
   - 发现请求/响应模型和数据结构

2. **文档工具选择**
   - 根据技术栈选择合适的文档工具：
     - **OpenAPI/Swagger**：具有交互式文档的 REST API
     - **GraphQL**：GraphiQL、GraphQL Playground 或 Apollo Studio
     - **Postman**：API 集合和文档
     - **Insomnia**：API 设计和文档
     - **Redoc**：替代 OpenAPI 渲染器
     - **API Blueprint**：基于 Markdown 的 API 文档

3. **API 规范生成**

  **对于带有 OpenAPI 的 REST API：**
  ```yaml
   openapi: 3.0.0
   info:
     title: $ARGUMENTS API
     version: 1.0.0
     description: Comprehensive API for $ARGUMENTS
   servers:
     - url: https://api.example.com/v1
   paths:
     /users:
       get:
         summary: List users
         parameters:
           - name: page
             in: query
             schema:
               type: integer
         responses:
           '200':
             description: Successful response
             content:
               application/json:
                 schema:
                   type: array
                   items:
                     $ref: '#/components/schemas/User'
   components:
     schemas:
       User:
         type: object
         properties:
           id:
             type: integer
           name:
             type: string
           email:
             type: string
   ```

4. **端点文档**
   - 记录所有 HTTP 方法 (GET、POST、PUT、DELETE、PATCH)
   - 指定请求参数（路径、查询、标头、正文）
   - 定义响应模式和状态码
   - 包含错误响应和错误代码
   - 记录身份验证和授权要求

5. **请求/响应示例**
   - 为每个端点提供真实的请求示例
   - 包含格式正确的示例响应数据
   - 展示不同的响应场景（成功、错误、边缘情况）
   - 记录内容类型和编码

6. **身份验证文档**
   - 记录身份验证方法（API 密钥、JWT、OAuth）
   - 解释授权范围和权限
   - 提供身份验证示例和令牌格式
   - 记录会话管理和刷新令牌流程

7. **数据模型文档**
   - 定义所有数据模式和模型
   - 记录字段类型、约束和验证规则
   - 包含实体之间的关系
   - 提供数据结构示例

8. **错误处理文档**
   - 记录所有可能的错误响应
   - 解释错误代码及其含义
   - 提供故障排除指南
   -包含速率限制和节流信息

9. **交互式文档设置**

   **Swagger UI 集成：**
   ```html
   <!DOCTYPE html>
   <html>
   <head>
     <title>API Documentation</title>
     <link rel="stylesheet" type="text/css" href="./swagger-ui-bundle.css" />
   </head>
   <body>
     <div id="swagger-ui"></div>
     <script src="./swagger-ui-bundle.js"></script>
     <script>
       SwaggerUIBundle({
         url: './api-spec.yaml',
         dom_id: '#swagger-ui'
       });
     </script>
   </body>
   </html>
   ```

10. **代码注释和评论**
    - 为 API 处理程序添加内联文档
    - 使用特定于框架的注释工具：
      - **Java**：@ApiOperation、@ApiParam（Swagger 注释）
      - **Python**：使用 FastAPI 或 Flask-RESTX 的文档字符串
      - **Node.js**：使用 swagger-jsdoc 的 JSDoc 注释
      - **C#**：XML 文档注释

11. **自动文档生成**

  **对于 Node.js/Express：**
  ```javascript
    const swaggerJsdoc = require('swagger-jsdoc');
    const swaggerUi = require('swagger-ui-express');
    
    const options = {
      definition: {
        openapi: '3.0.0',
        info: {
          title: 'API Documentation',
          version: '1.0.0',
        },
      },
      apis: ['./routes/*.js'],
    };
    
    const specs = swaggerJsdoc(options);
    app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));
    ```

12. **测试集成**
  - 根据文档生成 API 测试集
  - 包含测试脚本和验证规则
  - 设置自动化 API 测试
  - 记录测试场景和预期结果

13. **版本管理**
  - 记录 API 版本控制策略
  - 维护多个 API 版本的文档
  - 记录弃用时间表和迁移指南
  - 跟踪版本之间的重大变更

14. **性能文档**
  - 记录速率限制和限流策略
  - 包含性能基准和 SLA
  - 记录缓存策略和标头
  - 解释分页和过滤选项

15. **SD​​K 和客户端库文档**
  - 根据 API 规范生成客户端库
  - 记录 SDK 使用方法和示例
  - 提供不同语言的快速入门指南
  - 包含集成示例和最佳实践

16. **环境特定文档**
  - 记录不同环境（开发、预发布、生产）
  - 包含特定环境的端点和配置
  - 记录部署和配置要求
  - 提供环境设置说明

17. **安全文档**
  - 记录安全最佳实践
  - 包含 CORS 和 CSP 策略
  - 记录输入验证和清理
  - 解释安全标头及其用途

18. **维护与更新**
  - 设置自动文档更新
  - 创建流程以保持文档更新
  - 定期审查和验证文档
  - 将文档审查集成到开发工作流程中

**特定框架示例：**

**FastAPI (Python)：**
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(title="My API", version="1.0.0")

class User(BaseModel):
    id: int
    name: str
    email: str

@app.get("/users/{user_id}", response_model=User)
async def get_user(user_id: int):
    """Get a user by ID."""
    return {"id": user_id, "name": "John", "email": "john@example.com"}
```

**Spring Boot (Java):**
```java
@RestController
@Api(tags = "Users")
public class UserController {
    
    @GetMapping("/users/{id}")
    @ApiOperation(value = "Get user by ID")
    public ResponseEntity<User> getUser(
        @PathVariable @ApiParam("User ID") Long id) {
        // Implementation
    }
}
```

请记住使文档与代码更改保持同步，并使内部团队和外部消费者都可以轻松访问。