# CLAUDE.md — Multi-Agent Dev 团队规范

## 项目定位

这是一套通用的 AI 多 Agent 协作开发框架，适配任意技术栈的项目（前端/后端/全栈）。
通过自动探测项目技术栈，所有 skills 无需手动配置即可在不同项目中复用。

---

## 快速开始

### 1. 在目标项目中初始化

将 `.claude/settings.json` 复制到目标项目根目录，然后执行：

```
/detect-project
```

这会自动识别技术栈并生成 `.claude/project-context.json`。

### 2. 执行任意 skill

```
/requirements-parse   # 解析需求
/prd-generate         # 生成 PRD
/task-decompose       # 拆分任务
/api-design           # 设计 API
/route-implement      # 实现路由
/component-design     # 设计组件
/page-implement       # 实现页面
/unit-test            # 单元测试
/integration-test     # 集成测试
/code-review          # 代码审查
/deploy-verify        # 部署验证
```

每个 skill 会自动读取 `project-context.json`，按当前项目技术栈执行。

---

## 核心约定

### project-context.json 优先级
- 自动探测结果保存在 `.claude/project-context.json`
- 手动编辑该文件可覆盖自动探测结果，手动配置优先级更高
- 超过 24 小时自动重新探测

### handoff 文件规范
每个 skill 的输出存放在 `.claude/handoffs/` 目录：
```
.claude/handoffs/
  requirements.json     # requirements-parse 输出
  prd.md                # prd-generate 输出
  tasks.json            # task-decompose 输出
  api-spec.yaml         # api-design 输出
  test-report.md        # test-report 输出
```

### Agent 职责边界
- analyst → 需求分析、PRD、任务拆分（不写代码）
- backend → API 设计、数据库、路由实现
- frontend → 组件设计、页面实现、API 对接
- tester → 单元测试、集成测试、测试报告
- reviewer → 代码审查、验收检查、部署验证

---

## Skill 编写规范

### 核心必需文件

每个 skill 是 `.claude/commands/` 目录下的一个 Markdown 文件，文件名即 skill 名称（如 `unit-test.md`）。

### YAML 元数据（Frontmatter）

每个 skill 文件顶部必须包含 YAML frontmatter：

```markdown
---
name: skill-name
description: 一句话说明这个 skill 做什么
agent: analyst | backend | frontend | tester | reviewer | common
inputs:
  - .claude/project-context.json
  - .claude/handoffs/prd.md        # 按需列出依赖的 handoff 文件
outputs:
  - .claude/handoffs/output-file   # 该 skill 写入的 handoff 文件
---
```

字段说明：
- `name`：与文件名一致，对应 `/skill-name` 调用命令
- `agent`：归属的 agent 角色，需与 `settings.json` 中的分组对应
- `inputs`：执行前必须存在的文件，缺失时应先执行对应前置 skill
- `outputs`：执行后写入的文件，供下游 skill 读取

### 主体指令

frontmatter 之后是 skill 的执行指令，固定包含三个二级标题：

```markdown
## description
详细说明 skill 的目标和适用场景。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取 inputs 中列出的依赖文件
3. 执行核心逻辑（按技术栈分支处理）
4. 将结果写入 outputs 中列出的文件

## output
用代码块展示输出文件的格式示例（JSON / YAML / Markdown / 代码）。
```

### 可选辅助目录

```
.claude/commands/<skill-name>/
  SKILL.md          # 主文件（必需）
  scripts/          # 辅助脚本
  references/       # 参考资料
  assets/           # 静态资源
```

> 简单 skill 直接用单文件 `<skill-name>.md` 即可，只有逻辑复杂时才拆分为目录结构。

### 新增自定义 Skill 步骤

1. 在 `.claude/commands/` 下创建 `<skill-name>.md`
2. 写入 frontmatter 元数据（name / agent / inputs / outputs）
3. 编写 `## description`、`## steps`、`## output` 三节内容
4. 在 `.claude/settings.json` 的对应 agent 的 `skills` 数组中添加该 skill 名称
5. 执行 `/detect-project` 刷新上下文后即可使用 `/<skill-name>` 调用

---

## 适配新项目

1. 复制本仓库的 `.claude/` 目录到目标项目根目录
2. 执行 `/detect-project` 生成 `project-context.json`
3. 检查生成结果是否正确，如有误手动修正
4. 开始使用任意 skill

支持的技术栈：
- 前端：Next.js / React / Vue / Angular
- 后端：Express / Fastify / NestJS / FastAPI / Django / Spring / Gin
- ORM：Prisma / TypeORM / Drizzle / SQLAlchemy / Hibernate
- 测试：Vitest / Jest / Pytest / JUnit
- 语言：TypeScript / JavaScript / Python / Java / Go / Rust
