---
name: langchain-chain
description: 构建 AI 处理链，集成 Claude 模型，实现结构化输出、RAG 检索增强和 Agent 工具调用
agent: backend
inputs:
  - .claude/project-context.json
outputs:
  - .claude/handoffs/langchain-chain.md
---

# langchain-chain

## description
构建 AI 处理链，集成 Claude 模型，实现结构化输出、RAG 检索增强和 Agent 工具调用。根据 project-context 的 language 选择 LangChain.js 或 LangChain Python。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 读取业务需求，确定 AI 处理场景
3. 根据 project-context 的 language 选择实现：
   - typescript/javascript → LangChain.js (@langchain/anthropic)
   - python → LangChain Python (langchain-anthropic)
4. 选择合适的 Chain 类型（LLMChain/RetrievalQA/AgentExecutor）
5. 定义 Prompt Template 和输出解析器
6. 配置向量存储（如需 RAG）
7. 注册工具函数（如需 Agent）
8. 根据 project-context 的 framework 封装为对应的 API 路由

## output
```typescript
// lib/chains/example-chain.ts
import { ChatAnthropic } from '@langchain/anthropic'
import { PromptTemplate } from '@langchain/core/prompts'
import { StringOutputParser } from '@langchain/core/output_parsers'
import { RunnableSequence } from '@langchain/core/runnables'

const model = new ChatAnthropic({
  modelName: 'claude-3-5-sonnet-20241022',
  temperature: 0,
})

const prompt = PromptTemplate.fromTemplate(
  `You are a helpful assistant. Answer the following question: {question}`
)

export const exampleChain = RunnableSequence.from([
  prompt,
  model,
  new StringOutputParser(),
])

// Usage in API route:
// const result = await exampleChain.invoke({ question: userInput })
```
