
# S1

1. 教学使用 while True 的 loop, 会检查 stop.reason 是不是 tool_use, 而真实环境中会检查 needsFollowUp, 为什么这么设计, 因为要流式, 只要流式检查到有一个 tool_use, 就会设置 needsFollowUp 为 True, 表示本轮要继续进行工具使用
2. 当前的退出 reason 只有当前轮不是工具调用, 而实际上的使用中, 会有多种退出路径, 包括但不限于 prompt 超限, 余额不足, hook 拦截, model error、abort、max turns、token budget continuation、reactive compact retry 等场景。每种场景都有对应的恢复或退出策略。

