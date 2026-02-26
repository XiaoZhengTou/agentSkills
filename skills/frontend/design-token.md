---
name: design-token
description: 从设计稿或 PRD 中提取设计 token，生成颜色、字体、间距、圆角等变量，适配 Tailwind / CSS Variables / Style Dictionary
agent: frontend
inputs:
  - .claude/project-context.json
  - .claude/handoffs/prd.md
outputs:
  - .claude/handoffs/design-tokens.json
---

# design-token

## description
从 PRD 品牌规范或现有代码中提取设计 token，输出标准化的 token 文件。根据 project-context 自动适配 Tailwind config、CSS Variables 或 Style Dictionary 格式。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取 PRD 中的品牌色、字体规范；若无则扫描现有代码提取硬编码值
3. 按类别整理 token：color / typography / spacing / radius / shadow / z-index
4. 根据 project-context 的 styling 工具输出对应格式：
   - tailwind → 扩展 `tailwind.config.ts` 的 `theme.extend`
   - css → 生成 `:root {}` CSS Variables
   - style-dictionary → 输出 `tokens.json`
5. 写入 `.claude/handoffs/design-tokens.json` 供其他 skill 引用

## output
```json
// .claude/handoffs/design-tokens.json
{
  "color": {
    "primary": { "50": "#eff6ff", "500": "#3b82f6", "900": "#1e3a8a" },
    "neutral": { "50": "#f9fafb", "500": "#6b7280", "900": "#111827" },
    "success": "#22c55e",
    "warning": "#f59e0b",
    "error": "#ef4444"
  },
  "typography": {
    "fontFamily": { "sans": "Inter, system-ui", "mono": "JetBrains Mono" },
    "fontSize": { "sm": "0.875rem", "base": "1rem", "lg": "1.125rem", "xl": "1.25rem", "2xl": "1.5rem" },
    "fontWeight": { "normal": 400, "medium": 500, "semibold": 600, "bold": 700 }
  },
  "spacing": { "1": "0.25rem", "2": "0.5rem", "4": "1rem", "8": "2rem", "16": "4rem" },
  "radius": { "sm": "0.25rem", "md": "0.375rem", "lg": "0.5rem", "full": "9999px" },
  "shadow": {
    "sm": "0 1px 2px 0 rgb(0 0 0 / 0.05)",
    "md": "0 4px 6px -1px rgb(0 0 0 / 0.1)"
  }
}
```

```ts
// tailwind.config.ts 扩展示例
export default {
  theme: {
    extend: {
      colors: {
        primary: { 50: '#eff6ff', 500: '#3b82f6', 900: '#1e3a8a' }
      }
    }
  }
}
```
