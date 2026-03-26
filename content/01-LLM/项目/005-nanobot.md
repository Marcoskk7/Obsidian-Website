对于 nanobot 必须搞清楚的点:
1. Memory 与 Rag 是怎么设计并使用的
	双层持久化记忆
	[memory.md](https://memory.md) 存储长期结构化的事实, 类似知识库
	history.md 是可 grep 的时间日志, 每条都会追加, 类似 short term memory
	怎么对 Memory 进行整合的?:
	当上下文 token 快满时, 用 LLM 自动压缩旧对话并写入 memory.md 和 history.md
	还有个 session 存储的是当前对话的上下文, 以 jsonl 格式存储
	
	
	![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260318171727933.png)
	什么时候触发压缩? 估算 prompt tokens>=context_window_tokens
	怎么估算 token?
	降级顺序：`provider.estimate_prompt_tokens()` → `tiktoken (cl100k_base)` → `len // 4 粗估` → 返回 0（跳过压缩）
	如果 provider 有提供预估 token 的函数, 那么使用 provider 的函数, 如果没有使用 tiktoken 进行计算, 实在不行就除以 4


2. Context 是怎么管理的
	对于 Anthropic 等支持 caching 的 provider, 会在 system prompt 和 tools 末尾自动注入 cache_control: ephemeral 
	对于 systemmessage 和 tools 这样的固定内容, 使用 Cache 可以节省大量 token

3. 设计了 sub-Agent 吗, 使用了什么通信协议
	有实现子代理架构, 用户无感知, 主 Agent 调用 spawn 工具创建子代理, 避免主Agent 阻塞
	
	当前的实现是 Queue 实现, 没有实现 MCP 或者 A2A 协议, 而是自定义的结构, 只是纯文本, sub-Agent 执行完成之后会返回文字结果, 塞回总线(bus)中
	没有实现子 Agent 之间的相互通讯, 不能互相通讯
4. 做了 RL 吗, 怎么做的?
	目前没有实现 RL
5. 怎么评估相关性能的?
	没有评估
6. 是不是可以设计 MultiAgent, 做一下 Agent 编排?

| 指标                 | 测量方法                                                                        | 工具/命令                                                                    |
| ------------------ | --------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| 记忆写入成功率            | 跑 20 轮对话，统计 `HISTORY.md` 新增条数 / consolidation 触发总次数                         | `grep -c "^\[20" HISTORY.md` + loguru 日志中 `Memory consolidation done` 次数 |
| 跨会话记忆延续率           | 新 session 中提问 10 个历史相关问题，人工判断正确率                                            | 手动测试                                                                     |
| Prompt Token 压缩比   | 50 轮对话中记录每轮 `response.usage.prompt_tokens`，对比有/无 consolidation              | 在 `_parse_response` 中加日志，绘制 token 增长曲线                                   |
| Prompt Caching 命中率 | 对比 Anthropic 账单中 `cache_creation_input_tokens` vs `cache_read_input_tokens` | Anthropic Dashboard 或 `response.usage` 中的 cache 字段                       |
| 会话隔离率              | 2 个 session 同时发 sub-agent 任务，检查结果是否串话                                       | 手动或脚本测试                                                                  |
| Provider 兼容覆盖      | 统计 `registry.py` 中注册的 provider spec 数量                                      | `grep -c "ProviderSpec" nanobot/providers/registry.py`                   |
| BFCL/GAIA 评测成绩     | `nanobot interview eval --suite all`                                        | 内置 CLI 命令                                                                |
| 核心代码行数             | `bash core_agent_lines.sh`                                                  |                                                                          |


遇到了什么困难:
经常遇到 context 爆满的问题, deepseek 不回复, 为什么会这样?:
因为原先的代码, 只会在每一轮对话开始的前后来估算中context 的长度来决定是否整合, 或者写入, 但是现在出现的问题是一次对话中, 多个 tool 调用直接卡死了


怎么实现各个 channels 之间的通讯的:
通过一个 InboundMessage 实现:
