# heartbeat

## description
定义 Agent 心跳检测机制，监控 Agent 任务执行状态，支持超时自动升级和任务恢复。

## steps
1. 任务启动时记录 `start_time` 到 `.claude/handoffs/{task-id}.json`
2. 心跳进程定期检查所有活跃任务的执行时间
3. 超过 `timeout_threshold`（默认 10 分钟）未更新的任务标记为 `hung`
4. hung 状态持续超过 `max_hung_duration`（默认 3 次心跳）自动升级
5. 升级时记录 `upgrade_reason`，通知用户介入
6. 恢复任务时检查 `last_valid_state`，尝试恢复或重新执行

## output
```json
{
  "heartbeat_id": "HB-{timestamp}",
  "task_id": "T001",
  "agent": "backend",
  "status": "active|hung|upgraded|completed",
  "start_time": "2024-02-24T10:00:00Z",
  "last_update": "2024-02-24T10:05:00Z",
  "check_interval_seconds": 120,
  "timeout_threshold_seconds": 600,
  "max_hung_cycles": 3,
  "hung_cycles_count": 0,
  "upgrade_reason": null,
  "last_valid_state": {
    "skill": "api-design",
    "progress": "已分析 3/5 个接口",
    "artifacts": ["docs/api-spec.yaml:lines 1-50"]
  }
}
```

## 配置项

在 `.claude/settings.json` 中可配置：

```json
{
  "heartbeat": {
    "enabled": true,
    "check_interval_seconds": 120,
    "timeout_threshold_seconds": 600,
    "max_hung_cycles": 3,
    "auto_upgrade": true,
    "notify_on_upgrade": true
  }
}
```

## 升级通知消息示例

```json
{
  "from": "system",
  "to": "user",
  "type": "notification",
  "content": {
    "severity": "warning",
    "message": "Backend Agent 任务超时",
    "task_id": "T001",
    "hung_duration": "15 分钟",
    "last_progress": "已分析 3/5 个接口",
    "actions": ["继续等待", "手动干预", "重置任务"]
  }
}
```