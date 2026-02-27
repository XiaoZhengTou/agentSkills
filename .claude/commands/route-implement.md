---
name: route-implement
description: 基于 API 设计规范和数据库 Schema，实现后端路由/控制器，包含业务逻辑、数据验证和错误处理
agent: backend
inputs:
  - .claude/project-context.json
  - .claude/handoffs/api-spec.yaml
outputs:
  - .claude/handoffs/route-implement.md
---

# route-implement

## description
基于 API 设计规范和数据库 Schema，实现后端路由/控制器，包含业务逻辑、数据验证和错误处理。根据 project-context 自动适配框架。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取 api-design.md 中的端点规范
3. 根据 project-context 的 framework 选择实现方式：
   - nextjs → app/api/[resource]/route.ts，使用 Zod 验证
   - express/fastify → src/routes/resource.ts，使用 Zod 或 Joi
   - nestjs → src/resource/resource.controller.ts + service.ts
   - fastapi → routers/resource.py，使用 Pydantic
   - django → views.py + serializers.py（DRF）
   - spring → ResourceController.java + Service.java
   - gin → handlers/resource.go
4. 根据 project-context 的 orm 调用对应数据库客户端
5. 实现统一错误处理和响应格式
6. 添加认证中间件（与 api-design 保持一致）
7. 路由实现完成后，自动执行 `/unit-test`，传入本次产出的路由文件列表，验证核心逻辑

## output
```typescript
// app/api/[resource]/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { prisma } from '@/lib/prisma'
import { z } from 'zod'

const CreateSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
})

export async function GET(req: NextRequest) {
  try {
    const items = await prisma.resource.findMany()
    return NextResponse.json({ data: items })
  } catch (error) {
    return NextResponse.json({ error: 'Internal Server Error' }, { status: 500 })
  }
}

export async function POST(req: NextRequest) {
  try {
    const body = await req.json()
    const validated = CreateSchema.parse(body)
    const item = await prisma.resource.create({ data: validated })
    return NextResponse.json({ data: item }, { status: 201 })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({ error: error.errors }, { status: 400 })
    }
    return NextResponse.json({ error: 'Internal Server Error' }, { status: 500 })
  }
}
```
