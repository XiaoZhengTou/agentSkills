---
name: page-implement
description: 基于组件设计文档，实现前端页面，集成数据获取和交互逻辑，自动适配框架
agent: frontend
inputs:
  - .claude/project-context.json
  - .claude/handoffs/prd.md
  - .claude/handoffs/api-spec.yaml
outputs:
  - .claude/handoffs/page-implement.md
---

# page-implement

## description
基于组件设计文档，实现前端页面，集成数据获取和交互逻辑。根据 project-context 自动适配框架。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取 component-design.md 组件树和接口定义
3. 根据 project-context 的 framework 选择实现方式：
   - nextjs → app/[feature]/page.tsx，区分 Server/Client Component，使用 Tailwind CSS
   - react → src/pages/[feature].tsx 或 src/views/[feature].tsx
   - vue → src/views/[feature].vue，Composition API
   - angular → src/app/[feature]/[feature].component.ts
4. 实现数据获取逻辑：
   - nextjs Server Component → 直接调用 prisma/db，禁止用相对路径 fetch（服务端无法解析）
   - nextjs Client Component → 使用 SWR 或 React Query 调用 API
   - 其他框架 → SSR/CSR 视框架而定
5. 添加 loading 和 error 边界处理
6. 实现响应式布局

## output
```typescript
// app/[feature]/page.tsx
import { Suspense } from 'react'
import { DataList } from '@/components/DataList'
import { fetchItems } from '@/lib/api'

export default async function FeaturePage() {
  const items = await fetchItems()

  return (
    <main className="container mx-auto px-4 py-8">
      <h1 className="text-2xl font-bold mb-6">功能页面</h1>
      <Suspense fallback={<div>Loading...</div>}>
        <DataList items={items} />
      </Suspense>
    </main>
  )
}

// app/[feature]/loading.tsx
export default function Loading() {
  return <div className="animate-pulse">Loading...</div>
}

// app/[feature]/error.tsx
'use client'
export default function Error({ error, reset }: {
  error: Error
  reset: () => void
}) {
  return (
    <div>
      <p>Something went wrong: {error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  )
}
```
