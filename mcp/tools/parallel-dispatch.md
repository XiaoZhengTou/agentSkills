# parallel-dispatch

## description
定义 Agent 任务并行调度机制，基于 DAG 依赖图实现智能并发，提升流水线效率。

## steps
1. 解析任务列表，构建 DAG 依赖图
2. 识别可并行的任务节点（无依赖或依赖已满足）
3. 将任务分配到多个 Agent 并行执行
4. 监听各分支完成事件，触发下游任务
5. 汇总结果，处理跨分支依赖

## DAG 示例

```
Analyst (PRD)
    │
    ├──▶ Backend ──▶ Tester ──▶ Reviewer
    │      │
    │      └─▶ (并行) Frontend ──▶ Tester
    │
    └─▶ (等待) Backend + Frontend 都完成后聚合
```

## 并行任务识别规则

| 场景 | 是否可并行 |
|------|-----------|
| Backend API 开发 + Frontend 组件设计 | ✅ 可并行 |
| 多个独立的 API 端点实现 | ✅ 可并行 |
| 多个独立组件开发 | ✅ 可并行 |
| Analyst 输出 → Backend/Frontend | ❌ 必须等 Analyst |
| Backend API 完成 → Frontend 对接 | ❌ 必须等 API |
| De-Sloppify → Tester | ❌ 必须等清理 |

## output

### DAG 配置格式
```json
{
  "pipeline_id": "P001",
  "nodes": [
    {
      "id": "analyst-1",
      "agent": "analyst",
      "skill": "requirements-parse",
      "depends_on": []
    },
    {
      "id": "backend-1",
      "agent": "backend",
      "skill": "api-design",
      "depends_on": ["analyst-1"]
    },
    {
      "id": "frontend-1",
      "agent": "frontend",
      "skill": "component-design",
      "depends_on": ["analyst-1"]
    }
  ],
  "execution_plan": {
    "stage_1": ["analyst-1"],
    "stage_2": ["backend-1", "frontend-1"],
    "stage_3": ["tester-1"],
    "stage_4": ["reviewer-1"]
  },
  "estimated_duration_minutes": 45
}
```

### 并行执行状态
```json
{
  "running": [
    {
      "task_id": "backend-1",
      "agent": "backend",
      "started_at": "2024-02-24T10:00:00Z"
    },
    {
      "task_id": "frontend-1",
      "agent": "frontend",
      "started_at": "2024-02-24T10:00:00Z"
    }
  ],
  "waiting": ["tester-1"],
  "completed": ["analyst-1"]
}
```

## 配置项

```json
{
  "parallel_dispatch": {
    "enabled": true,
    "max_parallel_tasks": 3,
    "auto_parallel_detection": true,
    "conflict_resolution": "priority_based",
    "load_balance": "round_robin"
  }
}
```

## 使用示例

```bash
# 手动指定并行
/parallel-dispatch --tasks T001,T002,T003

# 自动分析依赖并调度
/detect-dependencies --pipeline full-pipeline
/execute-pipeline --mode parallel
```