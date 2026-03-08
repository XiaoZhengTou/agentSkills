# Agent 间消息协议

> 来源：大衍天工-开发规范 agent-teams-collaboration
> 适用：所有 Agent 间的交接和通讯

---

## 消息格式

所有 Agent 间通讯使用统一的 JSON 消息格式：

```json
{
  "from": "backend",
  "to": "frontend",
  "type": "notification",
  "content": {
    "message": "API 接口已完成",
    "artifacts": ["docs/api-spec.yaml"],
    "notes": "POST /api/orders 需要 JWT token"
  }
}
```

## 消息类型

| type | 用途 | 期望响应 |
|------|------|---------|
| `request` | 请求另一个 Agent 提供信息或执行操作 | 必须回复 response |
| `response` | 回复 request | 无需回复 |
| `notification` | 通知阶段完成或状态变更 | 无需回复 |
| `question` | 询问澄清或确认 | 必须回复 response |

## 标准交接消息

### Analyst → Backend/Frontend
```json
{
  "from": "analyst",
  "to": "backend",
  "type": "notification",
  "content": {
    "message": "需求分析完成，可以开始开发",
    "artifacts": ["docs/prd.md", ".claude/handoffs/tasks.json"],
    "notes": "task-003 依赖 task-001 完成后才能开始"
  }
}
```

### Backend → Frontend
```json
{
  "from": "backend",
  "to": "frontend",
  "type": "notification",
  "content": {
    "message": "API 接口已完成并通过测试",
    "artifacts": ["docs/api-spec.yaml"],
    "notes": "所有端点需要 Authorization: Bearer <token> header"
  }
}
```

### Backend/Frontend → Tester
```json
{
  "from": "backend",
  "to": "tester",
  "type": "notification",
  "content": {
    "message": "实现完成，De-Sloppify 已执行，可以开始测试",
    "artifacts": ["app/api/**", "prisma/schema.prisma"],
    "coverage": "TDD 阶段已有单元测试，需补充集成测试和 E2E"
  }
}
```

### Tester → Reviewer
```json
{
  "from": "tester",
  "to": "reviewer",
  "type": "notification",
  "content": {
    "message": "测试完成",
    "artifacts": ["docs/test-report.md"],
    "coverage": "单元测试 85%，集成测试 92%",
    "issues": []
  }
}
```

### Reviewer → [发起方]
```json
{
  "from": "reviewer",
  "to": "backend",
  "type": "request",
  "content": {
    "message": "发现问题，需要修复",
    "artifacts": ["docs/review-report.md"],
    "issues": [
      {
        "severity": "high",
        "location": "app/api/orders/route.ts:45",
        "description": "缺少权限检查，任何用户可访问他人订单"
      }
    ]
  }
}
```

## 并行 vs 串行任务原则

**可并行执行**（无数据依赖）：
- Backend API 开发 + Frontend 组件设计
- 多个独立模块的单元测试

**必须串行执行**（有数据依赖）：
- Analyst 完成 → Backend/Frontend 才能开始
- Backend API 完成 → Frontend API 对接
- De-Sloppify 完成 → Tester 开始
- Tester 完成 → Reviewer 开始

## 冲突解决

| 冲突类型 | 解决方式 |
|---------|---------|
| 接口不一致 | Backend 发 `notification` 更新 api-spec.yaml，Frontend 重新对接 |
| 数据格式冲突 | 双方发 `question` 给 Analyst，由 Analyst 裁决 |
| 时序问题 | 后置 Agent 发 `request` 等待前置 Agent 完成 |
