# Claude Code Explore Agent Research

来源核对时间：2026-05-15

- Anthropic Claude Code subagents docs: https://code.claude.com/docs/en/sub-agents
- Anthropic Claude Code common workflows: https://code.claude.com/docs/en/tutorials
- Anthropic blog, "How and when to use subagents in Claude Code": https://claude.com/blog/subagents-in-claude-code

有用的设计要点：

- 子代理运行在独立的上下文窗口中，只把相关结果返回给主对话。
- Claude Code 把 Explore agent 定位为快速、只读的代码搜索代理，用于文件发现、代码搜索和代码库探索。
- Anthropic 建议在研究密集型工作、大量工具输出、彼此独立的并行调查、重新评审和分阶段工作流中使用子代理。
- 一个强烈的委派信号是：任务需要大约 10 个或更多文件，或者大约 3 个或更多彼此独立的工作块。
- 有效的提示词会定义范围，只在任务彼此独立时才要求并行执行，并明确期望的输出格式。
- 自定义代理路由很依赖 description/trigger 字段，所以 skill 元数据应明确写出哪些情形会触发这个工作流。
- 常见工作流文档明确建议把大规模代码库探索交给子代理，这样主上下文只接收发现结果。

对这个 skill 的启示：

- 主代理不应该先做大范围文件读取。
- Explorer 提示词应该窄、只读，并且输出形状明确。
- 主交接产物应该是一张简洁的 key-files 表，包含理由和置信度。
- 主代理在编辑前只应验证最小必要文件集。
