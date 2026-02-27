---
name: component-variant
description: 为指定组件生成所有状态变体（default/hover/disabled/loading/error/empty），输出完整的变体实现代码
agent: frontend
inputs:
  - .claude/project-context.json
outputs:
  - .claude/handoffs/component-variant.md
---

# component-variant

## description
为指定组件补全所有 UI 状态变体，确保交互完整性。根据 project-context 自动适配框架语法和样式方案（Tailwind / CSS Modules / styled-components）。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 用户指定目标组件文件路径，读取现有实现
3. 分析组件类型，确定需要覆盖的状态集合：
   - 按钮类：default / hover / active / disabled / loading
   - 输入类：default / focus / error / disabled / readonly
   - 列表类：default / empty / loading / error
   - 卡片类：default / hover / selected / skeleton
4. 对每个缺失状态生成实现代码
5. 若项目有 Storybook，同步生成对应 stories

## output
```tsx
// Button.tsx — 补全所有变体
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost' | 'danger'
  size?: 'sm' | 'md' | 'lg'
  loading?: boolean
  disabled?: boolean
  children: React.ReactNode
  onClick?: () => void
}

export function Button({
  variant = 'primary',
  size = 'md',
  loading = false,
  disabled = false,
  children,
  onClick,
}: ButtonProps) {
  const base = 'inline-flex items-center justify-center font-medium rounded-md transition-colors focus-visible:outline-none focus-visible:ring-2'
  const variants = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 active:bg-blue-800 disabled:bg-blue-300',
    secondary: 'bg-white text-gray-900 border border-gray-300 hover:bg-gray-50 disabled:opacity-50',
    ghost: 'text-gray-600 hover:bg-gray-100 disabled:opacity-50',
    danger: 'bg-red-600 text-white hover:bg-red-700 disabled:bg-red-300',
  }
  const sizes = {
    sm: 'h-8 px-3 text-sm',
    md: 'h-10 px-4 text-sm',
    lg: 'h-12 px-6 text-base',
  }

  return (
    <button
      className={`${base} ${variants[variant]} ${sizes[size]}`}
      disabled={disabled || loading}
      onClick={onClick}
      aria-busy={loading}
    >
      {loading && <Spinner className="mr-2 h-4 w-4 animate-spin" aria-hidden />}
      {children}
    </button>
  )
}
```
