# 各 Agent 职责定义

## Analyst Agent — 需求分析

**职责**: 将用户的原始需求转化为结构化的开发规格。

| Skill | 输入 | 输出 | 验收标准 |
|-------|------|------|---------|
| `requirements-parse` | 原始需求文本 | 结构化需求 JSON | 所有业务实体已识别，边界条件已列出 |
| `prd-generate` | 结构化需求 JSON | PRD Markdown 文档 | 包含用户故事、数据模型、页面路由 |
| `task-decompose` | PRD 文档 | 任务清单 JSON | 每条任务含复杂度级别、完成条件、依赖关系 |

**任务清单格式要求**（符合 15 分钟单元规则）：
```json
{
  "id": "task-001",
  "title": "实现用户登录 API",
  "complexity": "small",
  "acceptance": ["POST /api/auth/login 返回 JWT", "密码错误返回 401"],
  "deps": [],
  "agent": "backend"
}
```

---

## Backend Agent — 后端开发

**职责**: 实现服务端 API、数据库模型和 AI 处理链。
**TDD 强制**：先写失败测试（Red），再最小实现（Green），再重构（Refactor）。

**工作流程**：
```
1. api-design     — 设计接口规范
2. [Red]          — 先写 API 集成测试（此时测试必须失败）
3. prisma-schema  — 实现数据模型
4. route-implement — 最小实现，让测试通过
5. langchain-chain — 实现 AI 处理链（如需要）
6. [Refactor]     — 重构，保持测试绿色
```

| Skill | 输入 | 输出 | 验收标准 |
|-------|------|------|---------|
| `api-design` | PRD 数据模型 | OpenAPI 3.0 规范 | 所有端点有请求/响应 schema，错误码已定义 |
| `prisma-schema` | 实体关系定义 | schema.prisma | 关系正确，索引已添加，迁移可执行 |
| `route-implement` | API 规范 + Schema | Next.js API Routes | 通过 integration-test，无 lint 错误 |
| `langchain-chain` | AI 功能需求 | LangChain 处理链 | 链可独立运行，有错误处理 |

---

## Frontend Agent — 前端开发

**职责**: 实现用户界面、页面逻辑和前后端对接。
**TDD 强制**：先写失败测试（Red），再最小实现（Green），再重构（Refactor）。

**工作流程**：
```
1. component-design — 设计组件树和 Props 接口
2. [Red]            — 先写 Widget/组件测试（此时测试必须失败）
3. page-implement   — 最小实现，让测试通过
4. api-integration  — 对接后端 API
5. [Refactor]       — 重构，保持测试绿色
```

| Skill | 输入 | 输出 | 验收标准 |
|-------|------|------|---------|
| `component-design` | PRD 页面需求 | 组件树 + Props 接口 | 组件职责单一，Props 类型完整 |
| `page-implement` | 组件设计文档 | Next.js 页面文件 | 通过 unit-test，无 TypeScript 错误 |
| `api-integration` | API 规范 | 类型安全 API 客户端 | 所有端点有类型，错误状态已处理 |

---

## Tester Agent — 测试

**职责**: 验证代码正确性，生成覆盖率报告。
**前置条件**: 在 De-Sloppify Pass 之后执行（确保测试针对业务逻辑，非框架行为）

| Skill | 输入 | 输出 | 验收标准 |
|-------|------|------|---------|
| `unit-test` | 源码文件 | 单元测试文件 | 覆盖率 ≥ 80%，测试只验证业务逻辑 |
| `integration-test` | API 规范 + 源码 | 集成测试文件 | 覆盖所有 happy path 和主要错误路径 |
| `test-report` | 测试运行结果 | 测试报告 Markdown | 包含通过率、覆盖率、失败原因 |

---

## Reviewer Agent — 验收

**职责**: 代码审查、需求验收和部署验证。
**关键约束**: 必须在独立 context window 中运行，不得与实现阶段共享会话（消除作者偏见）

| Skill | 输入 | 输出 | 验收标准 |
|-------|------|------|---------|
| `code-review` | 源码文件 | 审查意见报告 | 覆盖安全、边界条件、隐藏耦合 |
| `acceptance-check` | PRD + 实现代码 | 验收报告 | 每条 acceptance criteria 有明确通过/失败状态 |
| `deploy-verify` | 部署环境 | 部署验证报告 | 所有端点可访问，关键流程可执行 |

**审查重点**（来自 ECC agentic-engineering）：
- 不变量和边界条件
- 错误边界
- 安全和认证假设
- 隐藏耦合和上线风险
- 不在风格问题上浪费审查周期（lint 已覆盖）
