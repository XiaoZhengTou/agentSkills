# error-recovery

## description
定义任务失败的错误处理与回滚机制，支持自动重试、降级处理和状态回滚。

## steps
1. 任务执行前保存 `checkpoint` 到 `.claude/checkpoints/{task-id}.json`
2. 执行过程中捕获错误，分类为 `retryable` / `non-retryable` / `requires-user`
3. **可重试错误**（网络超时、LLM 调用失败）：按指数退避重试 3 次
4. **不可重试错误**（语法错误、配置缺失）：记录错误信息，跳过或降级处理
5. 需要人工介入的错误：暂停流水线，通知用户决策
6. 回滚时读取最近 checkpoint，恢复到执行前状态

## 错误分类

| 错误类型 | 例子 | 处理策略 |
|---------|------|---------|
| `retryable` | 网络超时、API 限流、LLM 调用失败 | 指数退避重试 3 次 |
| `non-retryable` | 语法错误、文件不存在、配置错误 | 跳过或降级，标记错误继续 |
| `requires-user` | 需要人工审批、权限不足、冲突 | 暂停等待用户决策 |

## output

### Checkpoint 格式
```json
{
  "checkpoint_id": "CP-{timestamp}",
  "task_id": "T001",
  "skill": "api-design",
  "created_at": "2024-02-24T10:00:00Z",
  "artifacts_snapshot": {
    "docs/api-spec.yaml": "file-content-hash"
  },
  "context_snapshot": {
    "analyst": "completed",
    "backend": "in_progress"
  },
  "retry_count": 0
}
```

### 错误报告格式
```json
{
  "error_id": "ERR-{timestamp}",
  "task_id": "T001",
  "agent": "backend",
  "skill": "api-design",
  "error_type": "retryable|non-retryable|requires-user",
  "error_message": "API rate limit exceeded",
  "retry_count": 2,
  "last_retry_at": "2024-02-24T10:05:00Z",
  "suggested_actions": ["retry_with_backoff", "skip_task", "notify_user"],
  "user_decision": null
}
```

## 配置项

在 `.claude/settings.json` 中配置：

```json
{
  "error_recovery": {
    "enabled": true,
    "max_retries": 3,
    "base_delay_ms": 1000,
    "exponential_backoff": true,
    "auto_skip_non_retryable": false,
    "checkpoint_before_each_skill": true,
    "notify_on_requires_user": true
  }
}
```

## 重试策略示例

```
第 1 次重试：等待 1 秒
第 2 次重试：等待 2 秒
第 3 次重试：等待 4 秒
然后放弃，进入错误处理流程
```