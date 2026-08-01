
# S1

1. 教学使用 while True 的 loop, 会检查 stop.reason 是不是 tool_use, 而真实环境中会检查 needsFollowUp, 为什么这么设计, 因为要流式, 只要流式检查到有一个 tool_use, 就会设置 needsFollowUp 为 True, 表示本轮要继续进行工具使用
2. 当前的退出 reason 只有

