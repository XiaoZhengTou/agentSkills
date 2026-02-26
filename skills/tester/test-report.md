---
name: test-report
description: 汇总单元测试和集成测试结果，生成结构化测试报告，标注覆盖率、失败用例和风险点
agent: tester
inputs:
  - .claude/project-context.json
outputs:
  - .claude/handoffs/test-report.md
---

# test-report

## description
汇总单元测试和集成测试结果，生成结构化测试报告，标注覆盖率、失败用例和风险点。

## steps
1. 运行所有测试套件，收集原始结果
2. 统计通过/失败/跳过用例数量
3. 生成代码覆盖率报告（按文件和分支）
4. 标注失败用例的错误信息和堆栈
5. 识别覆盖率低于阈值的模块
6. 输出 Markdown 格式测试报告

## output
```markdown
# 测试报告 — {日期}

## 总览
| 指标 | 数值 |
|------|------|
| 总用例数 | 42 |
| 通过 | 40 |
| 失败 | 2 |
| 跳过 | 0 |
| 覆盖率 | 83% |

## 覆盖率详情
| 文件 | 语句 | 分支 | 函数 | 行 |
|------|------|------|------|-----|
| app/api/resources/route.ts | 95% | 88% | 100% | 95% |
| lib/chains/example-chain.ts | 72% | 65% | 80% | 72% |

## 失败用例
### T023: POST /api/resources — 重复邮箱应返回 409
- 期望: status 409
- 实际: status 500
- 原因: 未处理 Prisma P2002 唯一约束错误

## 风险模块
- lib/chains/example-chain.ts 覆盖率低于 80% 阈值，需补充测试
```
