# 工程纪律规范

> 来源：ECC (everything-claude-code) agentic-engineering + enterprise-agent-ops + 大衍天工开发规范
> 适用：所有使用本 multi-agent-dev 框架的项目

---

## 一、Eval-First 原则

每个任务开始前必须定义：
1. **capability eval** — 功能验收标准（具体、可测量）
2. **regression eval** — 不能破坏的现有行为

执行后对比 delta，不达标则重新实现（换方法，不重复相同失败动作）。

---

## 三、15 分钟单元规则

Analyst 拆分的每个任务单元必须满足：
- 可独立验证（有明确的 done condition）
- 单一主要风险
- 约 15 分钟可完成

不满足此规则的任务需继续拆分。

---

## 四、De-Sloppify 通道

实现完成后、测试开始前，必须执行清理 pass：

```
Backend/Frontend 实现完成
        ↓
  De-Sloppify Pass（独立 context）
  - 删除冗余防御性检查
  - 删除测试框架行为的测试（只保留业务逻辑测试）
  - 删除 console.log / 注释掉的代码
  - 运行 lint + type check 确认无误
        ↓
  Tester Agent
```

**原则**：不用负面指令约束 Implementer，而是加独立清理 pass。两个专注的 Agent 优于一个受约束的 Agent。

---

## 五、独立上下文原则

| 阶段 | 是否需要独立 context |
|------|-------------------|
| Analyst → Backend/Frontend | 可共享（同一实现链） |
| 实现 → De-Sloppify | 必须独立 |
| 实现 → Reviewer | 必须独立（消除作者偏见） |
| 主要阶段切换后 | 建议新开 session |

---

## 六、三次失败升级协议

```
第 1 次失败：诊断根因，针对性修复
第 2 次失败：换方法（不重复相同失败动作）
第 3 次失败：更广泛重新思考，搜索解决方案
3 次后仍失败：上报用户，说明已尝试的方法和具体错误
```

---

## 七、成本追踪

每个任务记录：

| 字段 | 说明 |
|------|------|
| token_estimate | token 估算 |
| retries | 重试次数 |
| duration | 耗时 |
| status | success / failure |

---

## 八、ADR（架构决策记录）

重大架构决策必须记录 ADR，存放在 `docs/adr/` 目录：

```markdown
# ADR-001: [决策标题]

## 背景
为什么需要做这个决策。

## 决策
选择了什么方案。

## 后果
### 正面
- ...
### 负面
- ...
### 备选方案
- 方案 A：...
- 方案 B：...

## 状态
Accepted / Deprecated / Superseded by ADR-XXX

## 日期
YYYY-MM-DD
```

---

## 九、可观测性基线（生产环境）

长期运行的 Agent 工作负载需要：
- 不可变部署产物
- 最小权限凭证
- 环境级 secret 注入（不硬编码）
- 硬超时和重试预算
- 高风险操作的审计日志

关键指标：
- 成功率
- 每任务平均重试次数
- 恢复时间
- 每成功任务成本
- 失败类型分布

---

## 十、反模式清单

| 不要做 | 应该做 |
|--------|--------|
| 无退出条件的无限循环 | 设置 max-runs / max-cost / 完成信号 |
| 重复相同的失败动作 | 追踪已尝试方法，换方法 |
| 用负面指令约束 Implementer | 加独立 de-sloppify pass |
| Reviewer 审查自己写的代码 | 独立 context window |
| 主观判断"完成了" | Eval-First，对比 delta |
| 任务粒度过大 | 15 分钟单元规则 |
| 实现后才写测试 | TDD：先写失败测试再实现 |
| 随机试错调试 | 7步系统化调试流程 |

---

## 十一、TDD 强制流程（Red/Green/Refactor）

Backend Agent 和 Frontend Agent 必须遵循 TDD，不可绕过：

```
1. Red   — 先写失败测试，明确验收标准
2. Green — 最小实现，让测试通过（不过度设计）
3. Refactor — 重构代码，保持测试绿色
```

**覆盖率强制要求**：

| 测试类型 | 最低覆盖率 |
|---------|-----------|
| 单元测试 | ≥ 80% |
| API/集成测试 | ≥ 90% |
| E2E（核心流程） | 100% |

**测试金字塔比例**：
- 单元测试 60%
- 集成测试 30%
- E2E 测试 10%

**测试命名规范**（AAA 模式）：
```
test_[功能]_[场景]_[预期结果]
例：test_create_order_invalid_customer_raises_value_error
```

---

## 十二、系统化调试流程（7步）

遇到 bug 或测试失败时，按此流程执行，禁止随机试错：

```
1. 复现问题   — 确认可稳定复现，记录复现步骤
2. 收集日志   — 获取完整错误信息、堆栈、上下文
3. 定位范围   — 缩小到具体模块/函数/行
4. 分析根因   — 找到真正原因（不是表象）
5. 制定方案   — 至少考虑两种修复方案
6. 实施修复   — 最小改动，不引入新问题
7. 验证结果   — 运行全量测试，确认修复且无回归
```

结合三次失败升级协议：每次失败记录已尝试方法，第3次后上报用户。

---

## 十三、Git 工作流规范

**Commit 消息格式**：
```
type: subject（不超过50字符）

body（可选，说明原因）
```

type 类型：
| type | 用途 |
|------|------|
| feat | 新功能 |
| fix | Bug 修复 |
| refactor | 重构（不改变行为） |
| test | 测试相关 |
| docs | 文档 |
| chore | 构建/工具链 |

**分支策略**：
```
main（生产）
  └── develop（集成）
        ├── feature/[task-id]-[描述]
        ├── bugfix/[issue-id]-[描述]
        └── hotfix/[描述]
```

**PR 前检查清单**：
- [ ] 所有测试通过
- [ ] 覆盖率达标
- [ ] lint + type check 无错误
- [ ] De-Sloppify 已执行
- [ ] commit 消息符合规范

---

## 十四、代码规范检查

每次实现完成后，De-Sloppify Pass 必须运行以下检查：

**TypeScript/JavaScript 项目**：
```bash
npx tsc --noEmit          # 类型检查
npx eslint src/           # lint
npx prettier --check src/ # 格式检查
```

**Python 项目**：
```bash
mypy app/                 # 类型检查
flake8 app/               # lint
black --check app/        # 格式检查
isort --check app/        # import 排序
```

所有检查必须零错误才能进入 Tester Agent 阶段。
