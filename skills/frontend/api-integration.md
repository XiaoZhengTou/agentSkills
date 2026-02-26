---
name: api-integration
description: 实现前端与后端 API 的对接层，封装请求、处理认证、统一错误处理，生成类型安全的 API 客户端
agent: frontend
inputs:
  - .claude/project-context.json
  - .claude/handoffs/api-spec.yaml
outputs:
  - .claude/handoffs/api-integration.md
---

# api-integration

## description
实现前端与后端 API 的对接层，封装请求、处理认证、统一错误处理。根据 project-context 自动适配框架和语言。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取 api-design.md 中的端点规范
3. 根据 project-context 的 framework 选择实现方式：
   - nextjs/react → fetch 封装 + SWR 或 React Query
   - vue → axios 封装 + Pinia store 或 VueQuery
   - angular → HttpClient + Service 注入
4. 实现请求拦截（添加 Authorization header）
5. 封装每个端点为类型安全的函数（TypeScript）或模块（Python/Java）
6. 处理网络错误和业务错误的统一展示

## output
```typescript
// lib/api/client.ts
const BASE_URL = process.env.NEXT_PUBLIC_API_URL || '/api'

async function request<T>(path: string, options?: RequestInit): Promise<T> {
  const res = await fetch(`${BASE_URL}${path}`, {
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
    ...options,
  })
  if (!res.ok) {
    const error = await res.json()
    throw new Error(error.message || 'Request failed')
  }
  return res.json()
}

// lib/api/resources.ts
export const resourceApi = {
  list: () => request<Resource[]>('/resources'),
  get: (id: string) => request<Resource>(`/resources/${id}`),
  create: (data: CreateResourceDto) =>
    request<Resource>('/resources', { method: 'POST', body: JSON.stringify(data) }),
  update: (id: string, data: UpdateResourceDto) =>
    request<Resource>(`/resources/${id}`, { method: 'PATCH', body: JSON.stringify(data) }),
  delete: (id: string) =>
    request<void>(`/resources/${id}`, { method: 'DELETE' }),
}

// hooks/useResources.ts
import useSWR from 'swr'
export function useResources() {
  const { data, error, isLoading, mutate } = useSWR('/resources', resourceApi.list)
  return { resources: data, error, isLoading, refresh: mutate }
}
```
