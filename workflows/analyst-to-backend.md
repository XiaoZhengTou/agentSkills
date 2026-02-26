# analyst-to-backend

## description
需求分析 Agent 到后端开发 Agent 的交接流程，定义数据传递格式和启动条件。

## trigger
Analyst Agent 完成 `task-decompose` skill，生成任务清单后触发。

## handoff-checklist
- [ ] `docs/prd.md` 已生成，包含用户故事、数据模型、API 契约草稿
- [ ] `.claude/tasks.json` 已生成，后端任务已标注 `agent: "backend"`
- [ ] `.claude/context.json` 中 `analyst` 状态更新为 `completed`
- [ ] 交接包写入 `.claude/handoffs/analyst-to-backend.json`

## context-passed
```json
{
  "from_agent": "analyst",
  "to_agent": "backend",
  "key_artifacts": [
    { "type": "document", "path": "docs/prd.md" },
    { "type": "document", "path": ".claude/tasks.json" }
  ],
  "focus_areas": [
    "数据模型实体及关系（见 PRD 第3节）",
    "API 端点列表（见 PRD 第4节）",
    "认证方案决策（见 .claude/context.json decisions）"
  ],
  "constraints": [
    "使用 PostgreSQL 作为数据库",
    "所有 API 需要 JWT 认证",
    "遵循 RESTful 设计规范"
  ]
}
```

## backend-start-steps
1. 读取 `.claude/handoffs/analyst-to-backend.json`
2. 读取 `docs/prd.md` 中的数据模型章节
3. 执行 `api-design` skill
4. 执行 `prisma-schema` skill
5. 执行 `route-implement` skill
6. 如有 AI 功能需求，执行 `langchain-chain` skill
