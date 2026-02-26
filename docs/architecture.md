# 体系架构说明

## 概述

本体系基于 Claude Code Skills 架构，构建了一套 AI 驱动的全流程自动化开发体系。五个专职 Agent 通过标准化的交接协议协同工作，覆盖从需求分析到部署验证的完整开发生命周期。

## 核心组件

### Skills 层
每个 `.md` 文件是一个独立的 Skill，通过 `/skill-name` 调用。Skill 包含三段结构：
- `description` — 功能说明
- `steps` — 执行步骤
- `output` — 产出物格式

### MCP 层
通过 MCP 协议连接外部工具：
- `filesystem` — 文件读写
- `database` — Prisma 数据库操作
- `github` — 代码仓库管理

### Workflow 层
定义 Agent 间的数据流转顺序和交接规范，存储在 `.claude/handoffs/` 目录。

## 数据流

```
用户需求
   ↓
Analyst Agent (PRD + 任务清单)
   ↓              ↓
Backend Agent  Frontend Agent
   ↓              ↓
        Tester Agent (测试报告)
              ↓
        Reviewer Agent (验收报告)
              ↓
           部署上线
```

## 技术栈

| 层次 | 技术 |
|------|------|
| AI 框架 | Claude Code + Skills |
| 工具协议 | MCP (Model Context Protocol) |
| 后端框架 | Next.js App Router |
| 数据库 ORM | Prisma |
| AI 处理链 | LangChain.js |
| 测试框架 | Vitest + Testing Library |
| 类型系统 | TypeScript |
