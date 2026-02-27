---
name: task-decompose
description: 将 PRD 拆解为可执行的开发任务，按 Agent 角色分配，定义任务依赖关系和交接协议
agent: analyst
inputs:
  - .claude/project-context.json
  - .claude/handoffs/prd.md
outputs:
  - .claude/handoffs/tasks.json
---

# task-decompose

## description
将 PRD 拆解为可执行的开发任务，按 Agent 角色分配，定义任务依赖关系和交接协议。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取 PRD 文档，识别所有可开发单元
3. 根据 project-context 的 type 字段决定拆分维度：
   - fullstack → 后端/前端/测试/验收
   - backend → 后端/测试/验收
   - frontend → 前端/测试/验收
4. 为每个任务定义输入依赖和输出产物
5. 分析任务依赖图，自动推导并行分组：
   - 没有 depends_on 或依赖已在前一波完成的任务 → 可并行
   - 有未完成依赖的任务 → 必须顺序等待
   - 同一 wave 内的任务不能写入同一文件
   - 将可并行任务归入同一 wave，生成 execution_waves
6. 生成任务分配清单，指定负责 Agent

## output
```json
{
  "tasks": [
    {
      "id": "T001",
      "title": "任务标题",
      "agent": "backend|frontend|tester|reviewer",
      "depends_on": [],
      "inputs": ["所需输入文件或数据"],
      "outputs": ["产出物路径"],
      "priority": "high|medium|low",
      "skills": ["对应 skill 名称"],
      "can_parallel": true
    }
  ],
  "execution_waves": [
    {
      "wave": 1,
      "parallel": true,
      "tasks": ["T001", "T002"],
      "reason": "无依赖，输出文件不冲突"
    },
    {
      "wave": 2,
      "parallel": false,
      "tasks": ["T003"],
      "reason": "依赖 T001、T002 的输出"
    }
  ]
}
```
