
# S1(Loop)

1. 教学使用 while True 的 loop, 会检查 stop.reason 是不是 tool_use, 而真实环境中会检查 needsFollowUp, 为什么这么设计, 因为要流式, 只要流式检查到有一个 tool_use, 就会设置 needsFollowUp 为 True, 表示本轮要继续进行工具使用
2. 当前的退出 reason 只有当前轮不是工具调用, 而实际上的使用中, 会有多种退出路径, 包括但不限于 prompt 超限, 余额不足, hook 拦截, model error、abort、max turns、token budget continuation、reactive compact retry 等场景。每种场景都有对应的恢复或退出策略。


# S2

工具执行使用分区算法, 只读的会划分为同一个 batch, batch 之间严格串行, 同一 batch 内可以并行执行, 且同一 batch 内会有并发限制

每个工具结果, 会有一个最大字符限制, 如果超过限制, 就会将内容进行落盘
, `maxResultSizeChars` 字段。结果超过这个值就落盘，模型看到的是预览 + 文件路径。FileRead 特殊——设为 `Infinity`，防止读文件的输出又被当成文件落盘。具体来说，如果 FileRead 的结果超过阈值被落盘，模型下次读那个落盘文件时又会触发落盘 → 无限循环, 而这样会导致占用大量磁盘的存储.

工具执行的权限审计:
首先是提供的参数类型是否正确->
参数是否合理(比如 file_read)的路径, 是否真实可访问, 而不仅仅是保证为一个字符串->
pretool_hook, 自定义一些 hook->
权限审查, 用户的 deny/approve/ask->
-执行

# S3

工具的审计包括四种, deny, ask, allow, passthrough(这里是交给工具自己定义批准, 检查规则)

S4 (Hook)
CC 的 Stop hooks 有一个防无限循环机制（`query.ts:212,1300`）：`stopHookActive` 状态字段。当 stop hooks 产生 blockingError 时，循环带 `stopHookActive: true` 重入下一轮。后续迭代中 stop hooks 看到这个标志就不会再次触发。这防止了一个永不停机的 bug：模型自纠后 stop hook 再次报错 → 模型再自纠 → stop hook 再报错...