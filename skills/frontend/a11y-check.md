---
name: a11y-check
description: 扫描前端组件和页面代码，检查无障碍合规问题，输出分级问题清单和修复建议
agent: frontend
inputs:
  - .claude/project-context.json
outputs:
  - .claude/handoffs/a11y-report.md
---

# a11y-check

## description
扫描指定的前端组件或页面文件，按 WCAG 2.1 AA 标准检查无障碍问题。根据 project-context 自动适配框架语法（JSX / Vue template / Angular template）。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 确认扫描目标：用户指定文件路径，或默认扫描 `src/` 下所有组件文件
3. 逐文件检查以下问题：
   - 图片缺少 `alt` 属性
   - 交互元素（button/a）缺少可读文本或 `aria-label`
   - 表单 input 缺少关联 `<label>` 或 `aria-labelledby`
   - 颜色对比度不足（需人工确认，标记为 warning）
   - 键盘焦点顺序混乱（`tabIndex` 使用不当）
   - 动态内容缺少 `aria-live` 区域
   - 模态框缺少焦点陷阱和 `role="dialog"`
4. 按严重程度分级：
   - `error`：直接违反 WCAG AA，必须修复
   - `warning`：可能影响体验，建议修复
   - `info`：最佳实践建议
5. 对每个问题给出具体修复代码示例

## output
```markdown
# a11y 检查报告

## 扫描范围
- 文件数：12
- 组件数：28

## 问题汇总
| 级别 | 数量 |
|------|------|
| error | 3 |
| warning | 5 |
| info | 2 |

## 详细问题

### [error] src/components/Avatar.tsx:14
图片缺少 alt 属性
```tsx
// 当前
<img src={user.avatar} />

// 修复
<img src={user.avatar} alt={`${user.name} 的头像`} />
```

### [error] src/components/IconButton.tsx:8
按钮仅包含图标，缺少可读文本
```tsx
// 当前
<button onClick={onClose}><XIcon /></button>

// 修复
<button onClick={onClose} aria-label="关闭"><XIcon /></button>
```

### [warning] src/components/SearchInput.tsx:22
input 缺少关联 label
```tsx
// 当前
<input type="text" placeholder="搜索..." />

// 修复
<label htmlFor="search" className="sr-only">搜索</label>
<input id="search" type="text" placeholder="搜索..." />
```
```
