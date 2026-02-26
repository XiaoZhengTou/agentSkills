# Contributing to multi-agent-dev

欢迎贡献！以下是参与本项目的方式。

## 贡献类型

- **新增 Skill**：为某个 Agent 添加新的 skill 文件
- **改进现有 Skill**：完善 steps、补充 output 示例、修复错误
- **文档改进**：完善 README、CLAUDE.md 或 docs/ 下的文档
- **Bug 报告**：通过 Issue 报告问题

## 新增 Skill 步骤

1. Fork 本仓库
2. 在 `skills/<agent>/` 下创建 `<skill-name>.md`
3. 按规范编写 frontmatter 和主体内容（参考 CLAUDE.md 的 Skill 编写规范）
4. 在 `.claude/settings.json` 对应 agent 的 `skills` 数组中注册
5. 提交 PR，描述该 skill 的用途和适用场景

## Skill 文件规范

每个 skill 必须包含：

```markdown
---
name: skill-name
description: 一句话说明
agent: analyst | backend | frontend | tester | reviewer | common
inputs:
  - .claude/project-context.json
outputs:
  - .claude/handoffs/output-file
---

## description
## steps
## output
```

详细规范见 [CLAUDE.md](./CLAUDE.md#skill-编写规范)。

## 提交规范

```
feat(skill): 添加 xxx skill
fix(skill): 修复 xxx 的 outputs 字段
docs: 完善 README 快速开始章节
```

## 本地测试

将 `.claude/` 目录复制到任意前端/后端项目，执行 `/detect-project` 后测试你的 skill。

## 问题反馈

通过 [GitHub Issues](../../issues) 提交，请注明：
- 使用的技术栈
- 执行的 skill 名称
- 期望行为 vs 实际行为
