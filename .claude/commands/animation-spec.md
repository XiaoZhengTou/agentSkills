---
name: animation-spec
description: 为页面交互定义过渡动画规范，生成 Framer Motion / CSS Transition / Tailwind animate 实现代码
agent: frontend
inputs:
  - .claude/project-context.json
outputs:
  - .claude/handoffs/animation-spec.md
---

# animation-spec

## description
分析页面交互场景，定义统一的动画规范（时长、缓动曲线、触发时机），输出可直接使用的动画实现代码。根据 project-context 自动适配动画库。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 用户指定目标组件或页面，读取现有实现
3. 识别需要动画的交互场景：
   - 页面/路由切换
   - 列表项进入/离开
   - 模态框/抽屉开关
   - 按钮反馈、加载状态
   - 数据更新（数字滚动、进度条）
4. 定义动画基准规范：
   - 微交互：100-150ms，ease-out
   - 组件进出：200-300ms，ease-in-out
   - 页面切换：300-400ms，ease-in-out
5. 根据 project-context 输出对应实现（framer-motion / css / tailwind）

## output
```tsx
// 基于 Framer Motion 的动画规范

// 1. 统一动画配置
export const transitions = {
  micro: { duration: 0.1, ease: 'easeOut' },
  component: { duration: 0.25, ease: 'easeInOut' },
  page: { duration: 0.35, ease: 'easeInOut' },
}

// 2. 列表项进入动画
export const listItemVariants = {
  hidden: { opacity: 0, y: 8 },
  visible: (i: number) => ({
    opacity: 1,
    y: 0,
    transition: { ...transitions.component, delay: i * 0.05 },
  }),
}

export function ProductList({ items }: { items: Product[] }) {
  return (
    <motion.ul>
      {items.map((item, i) => (
        <motion.li
          key={item.id}
          custom={i}
          variants={listItemVariants}
          initial="hidden"
          animate="visible"
        >
          <ProductCard product={item} />
        </motion.li>
      ))}
    </motion.ul>
  )
}

// 3. 模态框动画
export const modalVariants = {
  hidden: { opacity: 0, scale: 0.95 },
  visible: { opacity: 1, scale: 1, transition: transitions.component },
  exit: { opacity: 0, scale: 0.95, transition: transitions.micro },
}

// 4. 页面切换
export const pageVariants = {
  initial: { opacity: 0, x: 16 },
  animate: { opacity: 1, x: 0, transition: transitions.page },
  exit: { opacity: 0, x: -16, transition: transitions.component },
}
```
