---
name: unit-test
description: 为后端路由和前端组件编写单元测试，确保核心逻辑的正确性，根据 project-context 自动选择测试框架
agent: tester
inputs:
  - .claude/project-context.json
outputs:
  - .claude/handoffs/test-report.md
---

# unit-test

## description
为后端路由和前端组件编写单元测试，确保核心逻辑的正确性。根据 project-context 自动选择测试框架。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取待测试的源码文件
3. 根据 project-context 的 test_framework 选择实现：
   - vitest → TypeScript/JS 项目，Mock 用 vi.fn()
   - jest → TypeScript/JS 项目，Mock 用 jest.fn()
   - pytest → Python 项目，Mock 用 pytest-mock
   - junit → Java 项目，Mock 用 Mockito
4. 识别需要测试的函数和组件
5. 为每个单元编写正向和边界测试用例
6. Mock 外部依赖（数据库、fetch、环境变量）
7. 验证输入输出和副作用，确保覆盖率 ≥ 80%

## output
```typescript
// __tests__/api/resources.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { GET, POST } from '@/app/api/resources/route'
import { prisma } from '@/lib/prisma'

vi.mock('@/lib/prisma', () => ({
  prisma: {
    resource: {
      findMany: vi.fn(),
      create: vi.fn(),
    },
  },
}))

describe('GET /api/resources', () => {
  beforeEach(() => vi.clearAllMocks())

  it('returns list of resources', async () => {
    const mockData = [{ id: '1', name: 'Test' }]
    vi.mocked(prisma.resource.findMany).mockResolvedValue(mockData)

    const req = new Request('http://localhost/api/resources')
    const res = await GET(req)
    const body = await res.json()

    expect(res.status).toBe(200)
    expect(body.data).toEqual(mockData)
  })

  it('returns 500 on database error', async () => {
    vi.mocked(prisma.resource.findMany).mockRejectedValue(new Error('DB error'))

    const req = new Request('http://localhost/api/resources')
    const res = await GET(req)

    expect(res.status).toBe(500)
  })
})
```
