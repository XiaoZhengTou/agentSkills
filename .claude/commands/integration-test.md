---
name: integration-test
description: 编写端到端集成测试，验证 API 与数据库的完整交互链路，根据 project-context 自动选择测试工具
agent: tester
inputs:
  - .claude/project-context.json
  - .claude/handoffs/api-spec.yaml
outputs:
  - .claude/handoffs/test-report.md
---

# integration-test

## description
编写端到端集成测试，验证 API 与数据库的完整交互链路。根据 project-context 自动选择测试工具和框架。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取 api-design.md 中所有端点定义
3. 根据 project-context 的 test_framework 和 framework 选择实现：
   - vitest/jest + nextjs/express → Supertest 或直接 fetch 测试
   - pytest + fastapi/django → pytest + httpx 或 Django TestClient
   - junit + spring → MockMvc 或 RestAssured
4. 搭建测试数据库环境（使用测试专用连接）
5. 为每个端点编写完整请求-响应测试
6. 测试认证流程和权限控制
7. 验证数据持久化和级联操作
8. 清理测试数据，确保测试隔离

## output
```typescript
// __tests__/integration/resources.integration.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest'
import { createServer } from '@/lib/test-server'
import { prisma } from '@/lib/prisma'

describe('Resources API Integration', () => {
  let baseUrl: string

  beforeAll(async () => {
    baseUrl = await createServer()
    await prisma.$connect()
  })

  afterAll(async () => {
    await prisma.$disconnect()
  })

  beforeEach(async () => {
    await prisma.resource.deleteMany()
  })

  it('POST /api/resources creates a resource', async () => {
    const res = await fetch(`${baseUrl}/api/resources`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: 'Test Resource', email: 'test@example.com' }),
    })
    const body = await res.json()

    expect(res.status).toBe(201)
    expect(body.data.name).toBe('Test Resource')

    const dbRecord = await prisma.resource.findUnique({ where: { id: body.data.id } })
    expect(dbRecord).not.toBeNull()
  })

  it('GET /api/resources returns created resources', async () => {
    await prisma.resource.createMany({
      data: [{ name: 'A' }, { name: 'B' }],
    })

    const res = await fetch(`${baseUrl}/api/resources`)
    const body = await res.json()

    expect(res.status).toBe(200)
    expect(body.data).toHaveLength(2)
  })
})
```
