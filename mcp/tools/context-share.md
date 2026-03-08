# context-share

## description
定义跨 Agent 上下文共享规范，维护全局项目状态，确保所有 Agent 基于一致的信息工作。包含全局上下文、短期记忆和决策历史三层机制。

## 三层架构

### 1. 全局上下文（Global Context）
持久化项目级信息，所有 Agent 共享。

### 2. 短期记忆（短期记忆）
每个 Agent 执行期间的临时状态，任务完成后可选择归档或丢弃。

### 3. 决策历史（Decision Log）
所有关键决策的完整记录，支持追溯和检索。

## steps

### 全局上下文
1. 初始化项目上下文文件 `.claude/context.json`
2. 每个 Agent 执行前读取最新上下文
3. 执行过程中追加关键决策和发现
4. 任务完成后更新上下文状态
5. 冲突时以最新时间戳版本为准

### 短期记忆（Agent 级别）
1. Agent 启动时创建 `.claude/memory/{agent}-{task-id}.json`
2. 记录当前任务的中间状态、发现、疑问
3. 任务完成时可选择归档到长期记忆或丢弃
4. 支持跨任务继承关键记忆

### 决策历史
1. 每个 Agent 做出关键决策时记录到 `.claude/decisions/`
2. 按日期分文件存储：`decisions/2024-02-24.json`
3. 支持语义检索，回溯决策原因

## output

### 全局上下文
```json
{
  "project": {
    "name": "项目名称",
    "version": "0.1.0",
    "tech_stack": ["Next.js", "Prisma", "LangChain", "TypeScript"]
  },
  "shared_state": {
    "prd_path": "docs/prd.md",
    "api_spec_path": "docs/api-spec.yaml",
    "schema_path": "prisma/schema.prisma",
    "task_list_path": ".claude/tasks.json"
  },
  "decisions": [
    {
      "id": "D001",
      "agent": "analyst",
      "timestamp": "2024-02-24T10:00:00Z",
      "decision": "使用 JWT 认证方案",
      "rationale": "无状态、易于水平扩展",
      "stakeholders": ["backend", "frontend"]
    }
  ],
  "agent_status": {
    "analyst": "completed",
    "backend": "in_progress",
    "frontend": "pending",
    "tester": "pending",
    "reviewer": "pending"
  },
  "last_updated": "2024-02-24T10:30:00Z"
}
```

### 短期记忆（Agent 任务级）
```json
{
  "memory_id": "MEM-backend-T001",
  "agent": "backend",
  "task_id": "T001",
  "created_at": "2024-02-24T10:00:00Z",
  "current_progress": "已完成 3/5 个 API 端点",
  "discoveries": [
    "发现用户认证模块可复用现有库",
    "订单接口需要拆分为 3 个以提升性能"
  ],
  "open_questions": [
    "支付回调是否需要支持异步？"
  ],
  "key_context": {
    "entities_defined": ["User", "Order", "Product"],
    "apis_planned": ["POST /auth/login", "GET /orders/:id"],
    "db_tables_needed": 5
  },
  "will_archive": true
}
```

### 决策历史记录
```json
{
  "decision_id": "D001",
  "timestamp": "2024-02-24T10:00:00Z",
  "agent": "analyst",
  "category": "architecture|api|db|auth|feature",
  "decision": "使用 JWT 认证方案",
  "alternatives_considered": ["Session", "OAuth2", "API Key"],
  "rationale": "无状态、易于水平扩展、与前后端分离架构匹配",
  "stakeholders": ["backend", "frontend"],
  "approval_required": false,
  "approved_by": null,
  "linked_tasks": ["T001", "T002"],
  "impact": "high",
  "reversible": true
}
```

## 冲突解决策略

| 冲突类型 | 解决方式 |
|---------|---------|
| 状态不一致 | 以最新时间戳版本为准 |
| 决策冲突 | 资深 Agent 裁定（如 Reviewer） |
| 接口冲突 | Backend 主导，Frontend 适配 |
| 优先级冲突 | 由 Analyst 重新评估 |

## 目录结构

```
.claude/
├── context.json              # 全局上下文
├── memory/
│   ├── backend-T001.json     # Agent 短期记忆
│   └── frontend-T002.json
├── decisions/
│   ├── 2024-02-24.json       # 按日期归档
│   └── 2024-02-25.json
└── handoffs/                 # Agent 间交接文件
```

## 配置项

```json
{
  "context_share": {
    "enabled": true,
    "context_file": ".claude/context.json",
    "memory_ttl_hours": 24,
    "auto_archive_memories": true,
    "decision_retention_days": 30,
    "semantic_search": false,
    "vector_store_path": ".claude/vector-db/"
  }
}
```