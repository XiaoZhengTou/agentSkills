# 大衍天工开发规范分析报告 — 对 multi-agent-dev 的启示

> 来源：大衍天工-开发规范（BizOS AI驱动ERP系统）
> 目标：提炼工程化规范，强化本项目的 multi-agent 体系

---

## 一、大衍天工的核心工程思想

### 1. 三层 Skill 架构
大衍天工将 Skill 分为三层，职责清晰：

| 层级 | 内容 | 特点 |
|------|------|------|
| Core Skills | TDD、代码规范、Git、Agent协作、调试、Superpowers、Planning | 所有角色必须掌握的通用能力 |
| Role Skills | 前端包、后端包、测试包 | 角色专属技能集合，引用 Core Skills |
| Specialized Skills | 特定场景技能 | 按需使用 |

**对本项目的启示**：当前 multi-agent-dev 的 skills/ 目录按 Agent 角色分类，但缺少"通用 Core Skills"层。应将跨 Agent 通用的能力（如 detect-project、handoff 协议）提升为 Core Skills。

### 2. TDD 强制执行（Red/Green/Refactor）
大衍天工将 TDD 作为不可绕过的强制流程：
- 先写失败测试（Red）
- 最小实现通过测试（Green）
- 重构（Refactor）
- 覆盖率强制要求：单元测试 ≥ 80%，API 测试 ≥ 90%

**对本项目的启示**：当前 Tester Agent 是在实现后才介入。应将 TDD 嵌入 Backend/Frontend Agent 的工作流：先写测试，再实现。

### 3. Agent 间通讯协议（JSON 消息格式）
大衍天工定义了标准化的 Agent 间消息格式：
```json
{
  "from": "backend_engineer",
  "to": "frontend_engineer",
  "type": "notification",
  "content": {
    "message": "API接口已完成",
    "api_docs": "/docs/api-spec.yaml"
  }
}
```
消息类型：request / response / notification / question

**对本项目的启示**：当前 handoffs/ 目录只有文件交接，缺少结构化消息协议。应定义标准消息格式。

### 4. 需求驱动开发流程（完整闭环）
大衍天工的开发流程是完整闭环：
```
需求输入 → 架构设计 → TDD实现 → 代码审查 → 测试 → 部署 → 监控反馈
```
每个阶段有明确的输入/输出模板和检查清单。

**对本项目的启示**：缺少"监控反馈"环节。部署后应有验证和反馈机制。

### 5. 角色技能包（Role Pack）模式
每个角色有独立的技能包文件，包含：
- 引用的 Core Skills 列表
- 框架最佳实践（含代码示例）
- 工作流程（接收任务→创建测试→实现→协作）
- 常用命令
- 技术要求（必须/推荐）

**对本项目的启示**：当前 agent-roles.md 是一个文件描述所有角色。应拆分为独立的角色技能包文件，每个文件更详细、可独立使用。

### 6. 测试金字塔
```
E2E测试 (10%) — 核心流程100%覆盖
集成测试 (30%)
单元测试 (60%) — ≥80%覆盖率
```

**对本项目的启示**：Tester Agent 的 skills 应明确对应测试金字塔的三层，并规定各层比例。

### 7. 系统化调试（7步流程）
大衍天工定义了标准化的调试流程，避免随机试错：
1. 复现问题
2. 收集日志
3. 定位范围
4. 分析根因
5. 制定方案
6. 实施修复
7. 验证结果

**对本项目的启示**：当前三次失败升级协议可以结合此7步流程，使调试更系统化。

### 8. Git 工作流规范
- Commit 格式：`type: subject`（feat/fix/docs/refactor/test/chore）
- 分支策略：main → develop → feature/* / bugfix/* / hotfix/*
- PR 前检查清单
- 合并使用 `--no-ff`

**对本项目的启示**：缺少 Git 工作流规范。应在 engineering-standards.md 中补充。

---

## 二、差距对比（大衍天工 vs multi-agent-dev）

| 能力 | 大衍天工 | multi-agent-dev | 优先级 |
|------|---------|-----------------|--------|
| TDD 强制执行 | ✅ | ❌（测试在实现后） | 高 |
| Core Skills 层 | ✅ | ❌ | 高 |
| Agent 消息协议 | ✅ | 部分（文件交接） | 高 |
| 角色技能包文件 | ✅ | ❌（单文件） | 中 |
| 测试金字塔规范 | ✅ | ❌ | 中 |
| Git 工作流规范 | ✅ | ❌ | 中 |
| 系统化调试流程 | ✅ | 部分（3次升级） | 中 |
| 监控反馈环节 | ✅ | ❌ | 低 |
| 代码规范检查 | ✅ | ❌ | 中 |
| 开发时间估算 | ✅ | ❌ | 低 |

---

## 三、改进建议（已落地到其他文档）

1. **engineering-standards.md** — 补充 TDD 强制流程、Git 规范、代码规范、系统化调试
2. **agent-roles.md** — 补充 TDD 工作流到 Backend/Frontend Agent
3. **workflows/full-pipeline.md** — 将 TDD 嵌入实现阶段
4. **新增 skills/core/** — 提取通用 Core Skills
5. **新增 docs/agent-message-protocol.md** — 定义 Agent 间消息格式
