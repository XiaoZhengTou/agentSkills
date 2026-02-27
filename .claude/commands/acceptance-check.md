---
name: acceptance-check
description: 对照 PRD 验收标准，逐条核查功能实现是否完整，输出验收结论和待修复清单
agent: reviewer
inputs:
  - .claude/project-context.json
  - .claude/handoffs/prd.md
  - .claude/handoffs/tasks.json
outputs:
  - .claude/handoffs/acceptance-report.md
---

# acceptance-check

## description
对照 PRD 验收标准，逐条核查功能实现是否完整，输出验收结论和待修复清单。

## steps
1. 读取 PRD 中的验收标准列表
2. 读取 task-decompose.md 中的任务产出物
3. 逐条验证功能需求是否实现
4. 验证非功能需求（性能、安全、可用性）
5. 检查 API 契约与实现的一致性
6. 输出验收结论（通过/条件通过/不通过）

## output
```markdown
# 验收报告 — {项目名称} v{版本}

## 验收结论: 条件通过

## 功能需求核查
| ID | 需求描述 | 状态 | 备注 |
|----|----------|------|------|
| F001 | 用户注册功能 | ✅ 通过 | |
| F002 | 用户登录（JWT） | ✅ 通过 | |
| F003 | 资源 CRUD | ⚠️ 部分通过 | 删除缺少软删除 |
| F004 | 分页列表 | ❌ 未实现 | |

## 非功能需求核查
| 类型 | 标准 | 状态 |
|------|------|------|
| 性能 | API 响应 < 200ms | ✅ 通过 |
| 安全 | 所有端点需认证 | ⚠️ /api/public 未保护 |

## 待修复项
1. F004: 实现资源列表分页（`page` 和 `pageSize` 参数）
2. F003: 添加软删除字段 `deletedAt`
3. 安全: 为 /api/public 添加速率限制
```
