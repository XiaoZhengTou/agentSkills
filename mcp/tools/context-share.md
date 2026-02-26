# context-share

## description
定义跨 Agent 上下文共享规范，维护全局项目状态，确保所有 Agent 基于一致的信息工作。

## steps
1. 初始化项目上下文文件 `.claude/context.json`
2. 每个 Agent 执行前读取最新上下文
3. 执行过程中追加关键决策和发现
4. 任务完成后更新上下文状态
5. 冲突时以最新时间戳版本为准

## output
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
      "rationale": "无状态、易于水平扩展"
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
