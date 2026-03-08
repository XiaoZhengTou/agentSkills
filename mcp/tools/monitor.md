# monitor

## description
提供流水线执行状态的实时监控面板，展示当前进度、耗时统计、成功率等关键指标。

## steps
1. 读取 `.claude/context.json` 获取全局状态
2. 读取 `.claude/handoffs/` 下所有任务文件
3. 读取 `.claude/memory/` 下 Agent 活动状态
4. 汇总生成监控报告
5. 可选：支持 WebUI / CLI / API 三种展示方式

## output

### 流水线概览
```json
{
  "pipeline": {
    "id": "P001",
    "name": "订单系统开发",
    "started_at": "2024-02-24T09:00:00Z",
    "estimated_duration_minutes": 45,
    "actual_duration_minutes": 32,
    "status": "in_progress"
  },
  "progress": {
    "total_tasks": 8,
    "completed": 3,
    "in_progress": 2,
    "pending": 3,
    "failed": 0,
    "percentage": 37.5
  },
  "agents": [
    {
      "name": "analyst",
      "status": "completed",
      "tasks_completed": 2,
      "total_tasks": 2,
      "last_active": "2024-02-24T09:30:00Z"
    },
    {
      "name": "backend",
      "status": "in_progress",
      "current_task": "api-design",
      "progress": "3/5 接口",
      "last_active": "2024-02-24T09:32:00Z"
    },
    {
      "name": "frontend",
      "status": "in_progress",
      "current_task": "component-design",
      "progress": "2/8 组件",
      "last_active": "2024-02-24T09:31:00Z"
    },
    {
      "name": "tester",
      "status": "pending",
      "tasks_completed": 0,
      "total_tasks": 2
    },
    {
      "name": "reviewer",
      "status": "pending",
      "tasks_completed": 0,
      "total_tasks": 2
    }
  ],
  "metrics": {
    "success_rate": 100,
    "avg_task_duration_minutes": 8,
    "error_count": 0,
    "retry_count": 0
  },
  "alerts": []
}
```

### CLI 展示示例

```
╔══════════════════════════════════════════════════════════════╗
║                    🚀 流水线监控面板                           ║
╠══════════════════════════════════════════════════════════════╣
║ 项目：订单系统开发                     状态：进行中           ║
║ 耗时：32/45 分钟                       进度：3/8 任务 (37%)   ║
╠══════════════════════════════════════════════════════════════╣
║ Agent     状态         当前任务         进度                  ║
║ ─────────────────────────────────────────────────────────────║
║ ✅ analyst   完成         requirements     2/2                ║
║ 🔄 backend   进行中       api-design       3/5 接口           ║
║ 🔄 frontend  进行中       component-des...
║ ⏳ tester    等待         -                 0/2                ║
║ ⏳ reviewer  等待         -                 0/2                ║
╠══════════════════════════════════════════════════════════════╣
║ 成功率：100%   平均任务耗时：8分钟   错误：0   重试：0       ║
╚══════════════════════════════════════════════════════════════╝
```

### API 端点（可选）

```bash
# 获取流水线状态
GET /api/pipeline/status

# 获取某个 Agent 状态
GET /api/agents/backend

# 获取任务历史
GET /api/tasks/history?limit=20
```

## 配置项

```json
{
  "monitor": {
    "enabled": true,
    "refresh_interval_seconds": 30,
    "display_mode": "cli",  // cli | web | api
    "web_port": 3456,
    "metrics_retention_days": 7,
    "show_estimated_time": true,
    "alert_on_failure": true,
    "notify_channels": ["webchat"]
  }
}
```

## Alert 告警示例

```json
{
  "alert_id": "ALT-001",
  "type": "task_failed|timeout|low_success_rate",
  "severity": "warning|critical",
  "message": "Backend Agent 任务 T003 失败",
  "details": {
    "task_id": "T003",
    "error": "API rate limit exceeded",
    "retry_count": 3
  },
  "timestamp": "2024-02-24T10:15:00Z",
  "actions": ["retry", "skip", "notify_user"]
}
```