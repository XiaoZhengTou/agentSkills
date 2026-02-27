---
name: requirements-parse
description: 解析用户输入的原始需求，提取关键信息，输出结构化需求文档
agent: analyst
inputs:
  - .claude/project-context.json
outputs:
  - .claude/handoffs/requirements.json
---

# requirements-parse

## description
解析用户输入的原始需求，提取关键信息，输出结构化需求文档。识别功能需求、非功能需求、约束条件和验收标准。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取用户提供的原始需求文本
3. 识别并分类需求类型（功能/非功能/约束）
4. 提取核心实体、操作和业务规则，结合 project-context 中的技术栈补充约束
5. 识别边界条件和异常场景
6. 输出结构化 JSON 格式需求文档（在 `tech_context` 字段中记录当前技术栈）

## output
```json
{
  "title": "需求标题",
  "functional": [
    { "id": "F001", "description": "功能描述", "priority": "high|medium|low" }
  ],
  "non_functional": [
    { "id": "NF001", "type": "performance|security|scalability", "description": "描述" }
  ],
  "constraints": ["约束条件列表"],
  "acceptance_criteria": ["验收标准列表"]
}
```
