# ECC + System 分析报告 — 对 multi-agent-dev 的启示

> 来源：everything-claude-code (ECC) + system (StyleAI 电商工作流)
> 目标：提炼可落地的工程规范，强化本项目的 multi-agent 体系

---

## 一、ECC 的核心思想提炼

### 1. Eval-First（评估优先）
ECC 的 `agentic-engineering` skill 强调：**先定义完成标准，再执行**。
- 每个任务开始前定义 capability eval + regression eval
- 执行后对比 delta，而不是主观判断"好像完成了"
- **对本项目的启示**：每个 Agent 的输出物应有明确的验收标准（acceptance criteria），Reviewer Agent 的 `acceptance-check` skill 正是这个思路，但需要更结构化。

### 2. 模型分级路由（Model Routing）
ECC 建议按复杂度路由到不同模型以控制成本。
**本项目不采用**：统一使用 claude-sonnet-4-6，无需路由逻辑。

### 3. 15分钟单元规则（Task Decomposition）
每个工作单元应满足：
- 可独立验证
- 单一主要风险
- 有明确的完成条件

**对本项目的启示**：Analyst Agent 的 `task-decompose` 输出的任务清单，每条任务应符合此规则。

### 4. De-Sloppify 模式（清理通道）
不要用负面指令约束 Implementer，而是加一个独立的清理 pass：
```
实现 → 清理（去除冗余测试/防御代码/console.log）→ 验证
```
**对本项目的启示**：在 Backend/Frontend Agent 完成实现后，Tester Agent 之前，应有一个 de-sloppify 步骤。

### 5. 独立上下文窗口（Separate Context Windows）
ECC Ralphinho 模式的关键设计：**Reviewer 绝不是代码的作者**。
每个阶段独立的 context window，消除作者偏见。
**对本项目的启示**：Reviewer Agent 不应在同一会话中审查自己参与实现的代码。

### 6. 复杂度分级流水线（Complexity Tiers）
| 级别 | 流水线深度 |
|------|-----------|
| trivial | implement → test |
| small | implement → test → code-review |
| medium | research → plan → implement → test → PRD-review + code-review |
| large | + final-review |

**对本项目的启示**：当前 full-pipeline 对所有任务一视同仁，应根据任务复杂度裁剪流水线。

### 7. 成本纪律（Cost Discipline）
每个任务追踪：模型、token 估算、重试次数、耗时、成功/失败。
只在低级模型明确失败时才升级模型。

---

## 二、system (StyleAI) 的结构优势

system 目录的亮点是**三层模型的清晰分离**：
- Workflows 层：端到端业务流程
- Agents 层：角色职责
- Skills 层：原子化能力

以及 Agent 定义的**输入/输出表格**格式，清晰规范。

本项目 `docs/agent-roles.md` 已采用类似格式，这是正确的。

system 的不足：缺乏 ECC 的工程纪律（模型路由、eval-first、成本追踪）。

---

## 三、multi-agent-dev 现状差距分析

| 能力 | ECC | system | multi-agent-dev | 优先级 |
|------|-----|--------|-----------------|--------|
| 模型路由 | ✅ | ❌ | 不采用 | — |
| Eval-First | ✅ | ❌ | ❌ | 高 |
| 复杂度分级 | ✅ | ❌ | ❌ | 高 |
| De-Sloppify | ✅ | ❌ | ❌ | 中 |
| 独立上下文 | ✅ | ❌ | 部分 | 中 |
| 成本追踪 | ✅ | ❌ | ❌ | 中 |
| 自主循环模式 | ✅ | ❌ | ❌ | 低 |
| 可观测性 | ✅ | ❌ | ❌ | 低 |
| ADR 记录 | ✅ | ❌ | ❌ | 中 |
| 输入/输出规范 | 部分 | ✅ | ✅ | 已有 |
| 工作流定义 | ✅ | ✅ | ✅ | 已有 |

---

## 四、改进建议（已落地到其他文档）

1. **agent-roles.md** — 新增模型分配、验收标准格式
2. **architecture.md** — 新增复杂度分级、eval-first 循环、de-sloppify 通道
3. **workflows/full-pipeline.md** — 新增分级路由逻辑
4. **新增 engineering-standards.md** — 工程纪律规范

---

## 五、自主循环模式参考（备用）

当项目需要无人值守执行时，可参考 ECC 的 6 种模式：

| 模式 | 复杂度 | 适用场景 |
|------|--------|---------|
| Sequential Pipeline | 低 | 日常开发步骤 |
| NanoClaw REPL | 低 | 交互式持久会话 |
| Infinite Agentic Loop | 中 | 并行内容生成 |
| Continuous Claude PR Loop | 中 | 多日迭代项目 |
| De-Sloppify Pattern | 附加 | 任何实现步骤后 |
| Ralphinho DAG | 高 | 大型特性、多单元并行 |

本项目当前规模适合 **Sequential Pipeline + De-Sloppify**，未来可升级到 Ralphinho。
