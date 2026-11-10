# OpenRouter Context Compression

* **工作方式**：当用户的历史对话、提示词和长文本的总 Token 数超过模型上限时，算法会**保留输入内容的最前端（如 System Prompt、全局指令、最早的上下文背景）和最尾端（最新的用户提问、近期的几轮对话）**，然后**砍掉中间部分的 Prompt 内容**。

* **设计理由**：在长文本处理中，大语言模型通常存在“迷失在中间”（Lost in the Middle）的现象，即模型对输入的最前和最后部分记忆最深刻。保留两端可以最大化维持当前的对话连续性与基本设定。

* 虽然上下文压缩能够保证请求不报错，但由于它“悄悄地”删除了中间的数据，在智能体（AI Agent）或复杂编程场景中会带来以下副作用：

* - **关键信息丢失与知识破坏**：采用“中间输出/截断”（Middle-Out Truncation）机制时，它会直接砍掉提示词中段的内容，可能导致底层逻辑、关键背景或低层次信号直接被破坏且不可恢复。
  - **AI 智能体陷入死循环**：当代理（Agent）或编码助手（如 Cline、Cursor 等）的对话被静默删减中间部分后，AI 会丢失对已检查文件或工具调用历史的记忆，导致**重复执行工具循环**。**代码与结构性数据损坏**：
    - **代码断层**：部分代码片段被截断后，函数声明、变量定义与上下文脱节，极易触发**语法错误或编译失败**。
    - **Schema 漂移**：结构化的 JSON Schema 或配置模板在中间被删后，会导致大模型输出格式错乱、校验失效。
  - https://github.com/openclaw/openclaw/issues/56317
  - 更像 Compaction
    - **Compression vs. Compaction:** **Compaction** is what an agent does natively when it runs out of memory (throwing away or crudely summarizing history). **Compression** is handled by dedicated gateway infrastructure (such as specialized engines like Headroom) to keep signal density high while protecting raw details.