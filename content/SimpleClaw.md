dispatch先进入 turn_processor.py, 先判断命令不是 help， exit 等内容，然后设置 tool 的上下文，

loop 中, 先从 MessageBus 中拉取对应的 InboundMessage,
InboundMessage 包含:
tenant_key, chat_id, channel, content

然后进入 runtime, 首先获取 runtime, 按 tenant_key 进行隔离
独立每个人的 sessions, memory 和 subagents, memory_consolidator
首次访问从