# agent-handoff

## description
定义 Agent 间任务交接的标准协议，确保上下文、产出物和状态信息的完整传递。

## steps
1. 发送方 Agent 完成当前任务，整理产出物
2. 构造标准交接包（HandoffPackage）
3. 将交接包写入 `.claude/handoffs/{task-id}.json`
4. 通知接收方 Agent 任务就绪
5. 接收方读取交接包，验证完整性
6. 接收方确认接收，开始执行后续任务

## output
```json
{
  "handoff_id": "HO-{timestamp}",
  "from_agent": "analyst|backend|frontend|tester|reviewer",
  "to_agent": "analyst|backend|frontend|tester|reviewer",
  "task_id": "T001",
  "status": "ready",
  "artifacts": [
    {
      "type": "document|code|schema|test|report",
      "path": "相对文件路径",
      "description": "产出物描述"
    }
  ],
  "context": {
    "summary": "本阶段工作摘要",
    "decisions": ["关键决策记录"],
    "blockers": ["已知阻塞项"],
    "notes": "给接收方的备注"
  },
  "next_steps": ["接收方需要执行的具体步骤"]
}
```
