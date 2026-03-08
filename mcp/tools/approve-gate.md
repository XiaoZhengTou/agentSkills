# approve-gate

## description
定义关键节点的人工审批机制，确保关键决策需人工确认后才能继续流水线。

## steps
1. 流水线执行到配置了 `require_approval` 的节点时暂停
2. 生成审批请求，包含待确认内容
3. 通知用户等待审批
4. 用户批准/拒绝/修改后继续或回滚
5. 记录审批历史到 `.claude/approvals/`

## 审批节点配置

| 阶段 | 默认行为 | 可配置 |
|------|---------|--------|
| PRD 生成后 | 需审批 | 可跳过 |
| API 设计完成后 | 需审批 | 可跳过 |
| 开发完成后 | 必审批 | 不可跳过 |
| 测试通过后 | 需审批 | 可跳过 |
| 部署前 | 必审批 | 不可跳过 |
| Code Review 通过后 | 需审批 | 可跳过 |

## output

### 审批请求格式
```json
{
  "approval_id": "APR-{timestamp}",
  "gate": "prd_review|api_review|pre_deploy|post_test",
  "task_id": "T001",
  "status": "pending|approved|rejected|modified",
  "requested_at": "2024-02-24T10:00:00Z",
  "requested_by": "analyst",
  "content": {
    "summary": "需求分析完成，包含 5 个 API 端点设计",
    "artifacts": ["docs/prd.md", "docs/api-spec.yaml"],
    "changes_since_last": ["新增用户认证流程", "订单接口拆分为 3 个"]
  },
  "review_notes": null,
  "user_decision": null,
  "decided_at": null
}
```

### 用户决策消息示例
```json
{
  "from": "user",
  "to": "system",
  "type": "response",
  "content": {
    "decision": "approved|rejected|modified",
    "comment": "API 设计合理，可以继续",
    "modifications": null
  }
}
```

## 配置项

```json
{
  "approve_gates": {
    "enabled": true,
    "required_gates": ["pre_deploy"],
    "optional_gates": ["prd_review", "api_review", "post_test"],
    "timeout_minutes": 60,
    "auto_skip_on_timeout": false,
    "notify_channels": ["webchat", "telegram"]
  }
}
```

## 审批 UI 示例（用户视角）

```
🤖 审批请求

📋 节点：PRD 评审
📝 摘要：需求分析完成，包含 5 个 API 端点设计
📁 产出物：docs/prd.md, docs/api-spec.yaml
📅 请求时间：10:00

[ ✅ 批准 ] [ ❌ 拒绝 ] [ ✏️ 修改 ] [ ⏰ 稍后 ]
```

## 审批历史记录

所有审批记录保存在 `.claude/approvals/`：
```
.claude/approvals/
  2024-02-24.json   # 每日一个文件
```