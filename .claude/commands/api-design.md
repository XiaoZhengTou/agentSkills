---
name: api-design
description: 基于 PRD 数据模型和业务流程，设计完整的 RESTful API 接口规范，遵循 OpenAPI 3.0 标准
agent: backend
inputs:
  - .claude/project-context.json
  - .claude/handoffs/prd.md
outputs:
  - .claude/handoffs/api-spec.yaml
---

# api-design

## description
基于 PRD 数据模型和业务流程，设计完整的 RESTful API 接口规范，遵循 OpenAPI 3.0 标准。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取 PRD 中的数据模型和用户故事
3. 识别所有资源实体，定义 CRUD 端点
4. 设计请求/响应 Schema，类型风格参考 project-context 的 language（TypeScript/Python/Java/Go）
5. 根据 project-context 的 framework 决定认证方式：
   - nextjs/express/nestjs → JWT 或 NextAuth
   - fastapi/django → OAuth2/JWT
   - spring → Spring Security
6. 处理错误码和异常响应格式
7. 输出 OpenAPI 3.0 规范文档

## output
```yaml
openapi: 3.0.0
info:
  title: API 名称
  version: 1.0.0
paths:
  /resource:
    get:
      summary: 获取资源列表
      responses:
        200:
          description: 成功
          content:
            application/json:
              schema:
                type: array
components:
  schemas:
    Resource:
      type: object
      properties:
        id: { type: string }
```
