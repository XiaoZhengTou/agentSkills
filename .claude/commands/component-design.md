---
name: component-design
description: 基于 PRD 功能需求，设计前端组件树和交互规范，自动适配 React/Vue/Angular 等框架
agent: frontend
inputs:
  - .claude/project-context.json
  - .claude/handoffs/prd.md
outputs:
  - .claude/handoffs/component-design.md
---

# component-design

## description
基于 PRD 功能需求，设计前端组件树和交互规范。根据 project-context 自动适配 React/Vue/Angular 等框架。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取 PRD 功能需求和用户故事
3. 根据 project-context 的 framework 决定组件规范：
   - nextjs/react → 函数组件 + hooks，区分 Server/Client Component
   - vue → Composition API，SFC 单文件组件
   - angular → Component + Service 模式
4. 拆分页面为组件树，定义 props 接口和 emit 事件
5. 设计状态管理方案（本地 state / 全局 store）
6. 定义组件间数据流和交互事件
7. 输出组件设计文档（含组件树、接口定义、状态图）

## output
```typescript
// 组件树结构
// Page
//   └── Layout
//         ├── Header
//         ├── MainContent
//         │     ├── DataList
//         │     │     └── DataItem (×n)
//         │     └── DataForm
//         └── Footer

// 组件接口定义
interface DataItemProps {
  id: string
  title: string
  description?: string
  onEdit: (id: string) => void
  onDelete: (id: string) => void
}

interface DataFormProps {
  initialData?: Partial<DataItem>
  onSubmit: (data: DataItem) => Promise<void>
  onCancel: () => void
}

interface DataListProps {
  items: DataItem[]
  loading: boolean
  onRefresh: () => void
}
```
