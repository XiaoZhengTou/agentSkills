---
name: prd-generate
description: 基于结构化需求文档，生成完整的 PRD，包含用户故事、功能规格、数据流图和 API 契约草稿
agent: analyst
inputs:
  - .claude/project-context.json
  - .claude/handoffs/requirements.json
outputs:
  - .claude/handoffs/prd.md
---

# prd-generate

## description
基于结构化需求文档，生成完整的 PRD（产品需求文档），包含用户故事、功能规格、数据流图和 API 契约草稿。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取 requirements-parse 输出的结构化需求
3. 为每个功能需求编写用户故事（As a... I want... So that...）
4. 定义数据模型和实体关系，字段类型参考 project-context 中的 orm 和 language
5. 设计 API 接口契约（端点、请求/响应格式），风格参考 project-context 中的 framework
6. 绘制核心业务流程图（Mermaid 格式）
7. 输出完整 PRD Markdown 文档

## output
```markdown
# PRD: {项目名称}

## 用户故事
- As a {角色}, I want {功能}, so that {价值}

## 数据模型
- 实体定义及关系

## API 契约
- 端点列表及请求/响应规格

## 业务流程
- Mermaid 流程图

## 验收标准
- 可量化的验收条件
```
