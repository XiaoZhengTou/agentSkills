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
   [De-Sloppify Pass]   ← 独立清理通道（去除冗余代码/测试）
        ↓
   Tester Agent (测试报告)
        ↓
   Reviewer Agent (验收报告)  ← 独立上下文，非代码作者
        ↓
     部署上线
```

## 复杂度分级流水线

不同复杂度的任务走不同深度的流水线，避免对简单任务过度处理：

| 级别 | 判断标准 | 流水线 |
|------|---------|--------|
| trivial | 单文件、无业务逻辑变更 | implement → test |
| small | 单模块、逻辑清晰 | implement → test → code-review |
| medium | 跨模块、有 API 变更 | research → plan → implement → test → PRD-review + code-review |
| large | 跨层、架构级变更 | medium 全流程 + final-review |

Analyst Agent 在 `task-decompose` 阶段为每个任务标注复杂度级别。

## Eval-First 循环

每个任务执行前必须定义验收标准，执行后对比结果：

```
1. 定义 capability eval（功能验收标准）
2. 定义 regression eval（回归检查点）
3. 执行实现
4. 运行 eval，对比 delta
5. delta 不达标 → 重新实现（不重复相同失败动作）
```

## 独立上下文原则

Reviewer Agent 必须在独立的 context window 中运行，不得与实现阶段共享会话。
这消除作者偏见，是发现隐藏问题的关键保障。

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
