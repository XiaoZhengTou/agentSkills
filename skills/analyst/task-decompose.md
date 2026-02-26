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
5. 设置任务优先级和执行顺序
6. 生成任务分配清单，指定负责 Agent

## output
```json
{
  "tasks": [
    {
      "id": "T001",
      "title": "任务标题",
      "agent": "backend|frontend|tester|reviewer",
      "depends_on": ["T000"],
      "inputs": ["所需输入文件或数据"],
      "outputs": ["产出物描述"],
      "priority": "high|medium|low",
      "skills": ["对应 skill 名称"]
    }
  ],
  "execution_order": ["T001", "T002", "T003"]
}
```
