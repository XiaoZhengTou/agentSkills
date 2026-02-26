---
name: deploy-verify
description: 验证应用在目标环境的部署状态，检查服务健康、环境变量、数据库连接和关键功能可用性
agent: reviewer
inputs:
  - .claude/project-context.json
outputs:
  - .claude/handoffs/deploy-report.md
---

# deploy-verify

## description
验证应用在目标环境的部署状态，检查服务健康、环境变量、数据库连接和关键功能的可用性。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 根据 project-context 的 framework 检查对应的环境变量配置完整性
3. 验证数据库连接和迁移状态（根据 orm 字段选择检查方式）
4. 调用健康检查端点（/api/health 或 /health）
5. 执行冒烟测试（核心业务流程）
6. 检查日志中的错误和警告
7. 输出部署验证报告

## output
```markdown
# 部署验证报告 — {环境} — {时间}

## 环境变量
| 变量 | 状态 |
|------|------|
| DATABASE_URL | ✅ 已配置 |
| NEXTAUTH_SECRET | ✅ 已配置 |
| ANTHROPIC_API_KEY | ✅ 已配置 |
| NEXT_PUBLIC_API_URL | ✅ 已配置 |

## 服务健康
- API 服务: ✅ 正常 (响应时间: 45ms)
- 数据库连接: ✅ 正常
- Prisma 迁移: ✅ 最新 (migration_20240224)

## 冒烟测试
| 场景 | 结果 |
|------|------|
| GET /api/health | ✅ 200 OK |
| POST /api/auth/login | ✅ 200 OK |
| GET /api/resources | ✅ 200 OK |

## 结论: ✅ 部署验证通过，可对外开放
```
