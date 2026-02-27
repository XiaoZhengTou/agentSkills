---
name: prisma-schema
description: 根据 PRD 数据模型和 API 设计，生成数据库 Schema 文件，自动适配 ORM 工具
agent: backend
inputs:
  - .claude/project-context.json
  - .claude/handoffs/prd.md
  - .claude/handoffs/api-spec.yaml
outputs:
  - .claude/handoffs/prisma-schema.md
---

# prisma-schema

## description
根据 PRD 数据模型和 API 设计，生成数据库 Schema 文件。根据 project-context 自动选择 ORM 工具：Prisma / TypeORM / SQLAlchemy / Hibernate 等。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 根据 project-context 的 orm 字段选择实现方式：
   - prisma → 生成 schema.prisma
   - typeorm → 生成 Entity 类（TypeScript）
   - drizzle → 生成 drizzle schema.ts
   - sqlalchemy → 生成 Python Model 类
   - hibernate → 生成 Java Entity 类
   - none → 生成原生 SQL DDL
3. 读取 PRD 实体关系定义，将实体映射为对应 ORM 的模型结构
4. 定义字段类型、约束和默认值
5. 建立模型间关联关系（一对多、多对多）
6. 添加索引和唯一约束
7. 输出对应 ORM 的 Schema 文件

## output
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  posts     Post[]
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  createdAt DateTime @default(now())
}
```
