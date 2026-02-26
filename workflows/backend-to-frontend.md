# backend-to-frontend

## description
后端开发 Agent 到前端开发 Agent 的交接流程，确保前端基于稳定的 API 契约进行开发。

## trigger
Backend Agent 完成 `route-implement` skill，所有 API Routes 实现后触发。

## handoff-checklist
- [ ] `docs/api-spec.yaml` 已生成（OpenAPI 3.0）
- [ ] `prisma/schema.prisma` 已生成并通过 `prisma validate`
- [ ] 所有 API Routes 已实现，本地测试通过
- [ ] `.claude/context.json` 中 `backend` 状态更新为 `completed`
- [ ] 交接包写入 `.claude/handoffs/backend-to-frontend.json`

## context-passed
```json
{
  "from_agent": "backend",
  "to_agent": "frontend",
  "key_artifacts": [
    { "type": "schema", "path": "docs/api-spec.yaml" },
    { "type": "schema", "path": "prisma/schema.prisma" },
    { "type": "code", "path": "app/api/**" }
  ],
  "api_base_url": "/api",
  "auth": {
    "type": "JWT",
    "header": "Authorization: Bearer {token}",
    "login_endpoint": "POST /api/auth/login"
  },
  "notes": "所有列表接口支持 ?page=1&pageSize=20 分页参数"
}
```

## frontend-start-steps
1. 读取 `.claude/handoffs/backend-to-frontend.json`
2. 读取 `docs/api-spec.yaml` 了解所有可用端点
3. 执行 `component-design` skill
4. 执行 `page-implement` skill
5. 执行 `api-integration` skill
