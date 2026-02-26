# dev-to-review

## description
开发完成后进入测试和验收阶段的流程，定义 Tester 和 Reviewer Agent 的启动条件与交接规范。

## trigger
Backend Agent 和 Frontend Agent 均完成后触发 Tester Agent。

## phase-1: dev-to-tester

### checklist
- [ ] 后端所有 API Routes 实现完毕
- [ ] 前端所有页面和组件实现完毕
- [ ] 代码已提交到功能分支
- [ ] 交接包写入 `.claude/handoffs/dev-to-tester.json`

### tester-start-steps
1. 读取 `docs/api-spec.yaml` 和 `docs/prd.md`
2. 执行 `unit-test` skill（后端 API + 前端组件）
3. 执行 `integration-test` skill（端到端流程）
4. 执行 `test-report` skill，生成 `docs/test-report.md`

## phase-2: tester-to-reviewer

### checklist
- [ ] 所有测试用例执行完毕
- [ ] `docs/test-report.md` 已生成
- [ ] 测试覆盖率 ≥ 80%（不满足需标注原因）
- [ ] 交接包写入 `.claude/handoffs/tester-to-reviewer.json`

### reviewer-start-steps
1. 读取 `docs/test-report.md`
2. 执行 `code-review` skill（重点审查失败用例涉及的代码）
3. 执行 `acceptance-check` skill（对照 PRD 验收标准）
4. 执行 `deploy-verify` skill
5. 输出最终验收结论

## outcome
- 通过: 触发部署流程，更新 `.claude/context.json` 所有 Agent 状态为 `completed`
- 不通过: 生成修复任务，分配回对应 Agent，重新进入流水线
