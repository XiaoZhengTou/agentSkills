---
name: code-review
description: 对提交的代码进行全面审查，检查代码质量、安全漏洞、性能问题和规范符合度
agent: reviewer
inputs:
  - .claude/project-context.json
outputs:
  - .claude/handoffs/code-review.md
---

# code-review

## description
对提交的代码进行全面审查，检查代码质量、安全漏洞、性能问题和规范符合度，输出带行号的审查意见。

## steps
1. 读取待审查的代码文件
2. 检查代码规范（命名、格式、注释）
3. 识别安全风险（SQL注入、XSS、未授权访问）
4. 评估性能问题（N+1查询、未分页、内存泄漏）
5. 验证错误处理完整性
6. 输出分级审查意见（blocker/major/minor）

## output
```markdown
# Code Review — {文件路径}

## Blocker（必须修复）
- [L42] `prisma.user.findMany()` 缺少分页，大数据量会导致内存溢出
  建议: 添加 `take` 和 `skip` 参数

## Major（强烈建议修复）
- [L18] 直接将用户输入拼接到查询条件，存在注入风险
  建议: 使用 Zod 验证后再传入 Prisma

## Minor（建议优化）
- [L7] 函数命名 `getData` 语义不清
  建议: 重命名为 `fetchUserList`

## 通过项
- 错误处理完整，所有 async 函数均有 try/catch
- 响应格式统一，符合 API 设计规范
- TypeScript 类型定义完整，无 any 类型
```
