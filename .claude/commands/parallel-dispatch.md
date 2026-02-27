---
name: parallel-dispatch
description: 读取 tasks.json 的 execution_waves，自动判断并行可行性，按波次派发 subAgent 执行任务
agent: common
inputs:
  - .claude/handoffs/tasks.json
outputs:
  - .claude/handoffs/dispatch-report.md
---

# parallel-dispatch

## description
读取 task-decompose 生成的 execution_waves，逐波执行：
- parallel: true 的 wave → 同一消息内并发派发多个 subAgent
- parallel: false 的 wave → 等待上一波完成后顺序执行
无需人工判断，完全由依赖图驱动。

## steps

1. 读取 `.claude/handoffs/tasks.json`，若不存在则先执行 `/task-decompose`

2. 遍历 `execution_waves`，按 wave 序号顺序处理：

   **wave.parallel === true：**
   - 在同一条消息内，为 wave.tasks 中每个任务各派发一个 subAgent
   - 每个 subAgent 的 prompt 包含：任务标题、agent 角色、skills 列表、inputs/outputs 路径
   - 等待所有 subAgent 返回后再进入下一波

   **wave.parallel === false：**
   - 按 wave.tasks 顺序逐个执行，每个任务完成后再执行下一个
   - 适用于有强依赖的任务（如 prisma-schema 必须在 route-implement 之前）

3. 每个 subAgent 执行完成后记录结果：
   - 成功：记录产出物路径
   - 失败：记录错误原因，暂停后续 wave，提示用户处理

4. 所有 wave 执行完毕后，生成 dispatch-report.md

## subAgent prompt 模板

```
你是一个 {agent} agent，负责执行以下任务：

任务：{title}
需要执行的 skills：{skills}
输入文件：{inputs}
输出文件：{outputs}

执行步骤：
1. 读取所有 inputs 文件
2. 按顺序执行 skills 列表中的每个 skill
3. 将结果写入 outputs 指定的路径
4. 返回：执行摘要 + 产出物路径 + 遇到的问题（如有）

约束：只修改 outputs 中列出的文件，不改动其他文件。
```

## output

`.claude/handoffs/dispatch-report.md` 格式：

```markdown
# Dispatch Report

## Wave 1（并行）
- T001 backend/api-design ✅ → .claude/handoffs/api-spec.yaml
- T002 frontend/wireframe-generate ✅ → .claude/handoffs/wireframe.md

## Wave 2（顺序）
- T003 backend/route-implement ✅ → src/routes/
- T004 frontend/page-implement ✅ → src/pages/

## Wave 3（并行）
- T005 tester/unit-test ✅ → .claude/handoffs/test-report.md
- T006 tester/integration-test ✅ → .claude/handoffs/test-report.md

## 总结
- 完成任务：6/6
- 失败任务：0
- 产出物：见各 wave 记录
```
