# multi-agent-dev

AI 驱动的全流程自动化开发体系，基于 Claude Code Skills 架构实现五个专职 Agent 的分工协作。
**支持任意技术栈**，通过自动探测项目环境，无需手动配置即可适配不同项目。

## 快速开始

### 适配新项目（3 步）

```bash
# 1. 将 .claude/ 目录复制到目标项目根目录
cp -r .claude/ /path/to/your-project/.claude/

# 2. 在目标项目中探测技术栈
/detect-project

# 3. 开始使用任意 skill
/requirements-parse
```

`/detect-project` 会自动识别框架、ORM、测试工具等，生成 `.claude/project-context.json`，
后续所有 skill 自动读取该文件，按当前项目技术栈执行。

### 执行完整流水线

按照 `workflows/full-pipeline.md` 中的阶段顺序，依次调用各 Agent 的 Skills。

## Agent 与 Skills

| Agent | Skills |
|-------|--------|
| Common | `/detect-project` |
| Analyst | `/requirements-parse` `/prd-generate` `/task-decompose` |
| Backend | `/api-design` `/prisma-schema` `/route-implement` `/langchain-chain` |
| Frontend | `/component-design` `/page-implement` `/api-integration` |
| Tester | `/unit-test` `/integration-test` `/test-report` |
| Reviewer | `/code-review` `/acceptance-check` `/deploy-verify` |

## 支持的技术栈

| 维度 | 支持范围 |
|------|---------|
| 前端框架 | Next.js / React / Vue / Angular |
| 后端框架 | Express / Fastify / NestJS / FastAPI / Django / Spring / Gin |
| ORM | Prisma / TypeORM / Drizzle / SQLAlchemy / Hibernate |
| 测试框架 | Vitest / Jest / Pytest / JUnit |
| 语言 | TypeScript / JavaScript / Python / Java / Go / Rust |

## 目录说明

```
skills/
  common/    通用 skill（detect-project）
  analyst/   需求分析相关 skill
  backend/   后端开发相关 skill
  frontend/  前端开发相关 skill
  tester/    测试相关 skill
  reviewer/  审查验收相关 skill
mcp/         MCP Server 配置和跨 Agent 工具协议
workflows/   Agent 间协作顺序和交接规范
docs/        架构说明和角色定义
.claude/     Claude Code 全局配置
```

## 工程纪律

本框架遵循 ECC (everything-claude-code) 工程规范：

- **Eval-First**：每个任务先定义验收标准，执行后对比 delta
- **TDD 强制**：Backend/Frontend Agent 先写失败测试，再实现（Red/Green/Refactor）
- **系统化调试**：7步调试流程，禁止随机试错
- **复杂度分级**：trivial/small/medium/large 走不同深度流水线
- **De-Sloppify**：实现后、测试前有独立清理通道
- **独立上下文**：Reviewer 必须在独立 session 中运行（消除作者偏见）
- **三次失败升级**：同一问题失败 3 次后上报用户

详见 [工程纪律规范](docs/engineering-standards.md)

## 文档

- [体系架构说明](docs/architecture.md)
- [各 Agent 职责定义](docs/agent-roles.md)
- [完整开发流水线](workflows/full-pipeline.md)
- [工程纪律规范](docs/engineering-standards.md)
- [ECC 分析与启示](docs/ecc-insights.md)
- [大衍天工分析与启示](docs/bizos-insights.md)
- [Agent 间消息协议](docs/agent-message-protocol.md)
- [团队使用规范](CLAUDE.md)
