# 各 Agent 职责定义

## Analyst Agent — 需求分析

**职责**: 将用户的原始需求转化为结构化的开发规格。

| Skill | 输入 | 输出 |
|-------|------|------|
| `requirements-parse` | 原始需求文本 | 结构化需求 JSON |
| `prd-generate` | 结构化需求 JSON | PRD Markdown 文档 |
| `task-decompose` | PRD 文档 | 任务清单 JSON |

---

## Backend Agent — 后端开发

**职责**: 实现服务端 API、数据库模型和 AI 处理链。

| Skill | 输入 | 输出 |
|-------|------|------|
| `api-design` | PRD 数据模型 | OpenAPI 3.0 规范 |
| `prisma-schema` | 实体关系定义 | schema.prisma |
| `route-implement` | API 规范 + Schema | Next.js API Routes |
| `langchain-chain` | AI 功能需求 | LangChain 处理链 |

---

## Frontend Agent — 前端开发

**职责**: 实现用户界面、页面逻辑和前后端对接。

| Skill | 输入 | 输出 |
|-------|------|------|
| `component-design` | PRD 页面需求 | 组件树 + Props 接口 |
| `page-implement` | 组件设计文档 | Next.js 页面文件 |
| `api-integration` | API 规范 | 类型安全 API 客户端 |

---

## Tester Agent — 测试

**职责**: 验证代码正确性，生成覆盖率报告。

| Skill | 输入 | 输出 |
|-------|------|------|
| `unit-test` | 源码文件 | 单元测试文件 |
| `integration-test` | API 规范 + 源码 | 集成测试文件 |
| `test-report` | 测试运行结果 | 测试报告 Markdown |

---

## Reviewer Agent — 验收

**职责**: 代码审查、需求验收和部署验证。

| Skill | 输入 | 输出 |
|-------|------|------|
| `code-review` | 源码文件 | 审查意见报告 |
| `acceptance-check` | PRD + 实现代码 | 验收报告 |
| `deploy-verify` | 部署环境 | 部署验证报告 |
