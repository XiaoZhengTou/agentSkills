---
name: detect-project
description: 自动探测当前项目的技术栈、框架、语言和工具链，输出标准化的 project-context.json
agent: common
inputs: []
outputs:
  - .claude/project-context.json
---

# detect-project

## description
自动探测当前项目的技术栈、框架、语言和工具链，输出标准化的 project-context.json，供所有其他 skill 读取使用。无需手动配置，直接适配任意项目。

## steps
1. 检查项目根目录，按以下优先级识别语言和框架：
   - 存在 `package.json` → 读取 dependencies/devDependencies，识别：
     - `next` → framework: nextjs, type: fullstack
     - `react` (无 next) → framework: react, type: frontend
     - `vue` → framework: vue, type: frontend
     - `express` / `fastify` / `koa` → framework: 对应值, type: backend
     - `@nestjs/core` → framework: nestjs, type: backend
   - 存在 `pom.xml` 或 `build.gradle` → language: java, framework: spring
   - 存在 `requirements.txt` 或 `pyproject.toml` → language: python，读取内容识别 fastapi/django/flask
   - 存在 `go.mod` → language: go，读取内容识别 gin/echo/fiber
   - 存在 `Cargo.toml` → language: rust
2. 识别 ORM / 数据库工具：
   - `prisma/schema.prisma` 存在 → orm: prisma
   - dependencies 含 `typeorm` → orm: typeorm
   - dependencies 含 `drizzle-orm` → orm: drizzle
   - `requirements.txt` 含 `sqlalchemy` → orm: sqlalchemy
   - 均无 → orm: none
3. 识别测试框架：
   - dependencies 含 `vitest` → test_framework: vitest
   - dependencies 含 `jest` → test_framework: jest
   - `requirements.txt` 含 `pytest` → test_framework: pytest
   - `pom.xml` 含 `junit` → test_framework: junit
4. 识别包管理器：
   - 存在 `pnpm-lock.yaml` → package_manager: pnpm
   - 存在 `yarn.lock` → package_manager: yarn
   - 存在 `package-lock.json` → package_manager: npm
   - 存在 `requirements.txt` → package_manager: pip
   - 存在 `pom.xml` → package_manager: maven
   - 存在 `build.gradle` → package_manager: gradle
5. 识别项目类型（若步骤1未确定）：
   - 同时有 `app/api/` 和 `app/` 页面目录 → type: fullstack
   - 只有 `src/components/` 或 `pages/` → type: frontend
   - 只有 `src/routes/` 或 `src/controllers/` → type: backend
6. 读取 `package.json` 的 name 字段作为 project_name
7. 将结果写入 `.claude/project-context.json`

## output
```json
{
  "project_name": "my-app",
  "type": "frontend | backend | fullstack",
  "language": "typescript | javascript | python | java | go | rust",
  "framework": "nextjs | react | vue | express | fastify | nestjs | fastapi | django | spring | gin",
  "orm": "prisma | typeorm | drizzle | sqlalchemy | hibernate | none",
  "test_framework": "vitest | jest | pytest | junit | none",
  "package_manager": "npm | yarn | pnpm | pip | maven | gradle",
  "detected_at": "2026-02-26T00:00:00Z"
}
```

## notes
- 每次执行 skill 前若 `.claude/project-context.json` 不存在或超过 24 小时，自动重新探测
- 探测结果可被 `.claude/project-context.json` 手动覆盖，手动配置优先级高于自动探测
- 多语言 monorepo 场景：以根目录主语言为准，子包信息记录在 `workspaces` 字段
