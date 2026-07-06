https://github.com/zainnab-sparq/cc-self-train

/branch 命令, 可以从当前会话, fork 出一个新对话, 直接从新对话开始内容

记忆中, 分为 CLAUDE.md, CLAUDE.local.md 以及 .claude/rules, 可以根据 指定不同paths 的 frontmatter, 来生效在不同路径下的 rules

skills 的使用很有讲究, 可以设定如下几种:
- context: fork 表示当 agent 通过 skill 调用时, 走后台启动, 而不占用主会话
- effort: 设置推理强度
- skillOverrides : 控制哪些 skill 对 Claude 可见。值：off（完全隐藏）、user-invocable-only（仅你手动调用可见）、name-only（仅名称可见，省描述占用的 token）。
  
- disable-model-invocation: true, 设置只能手动触发不能模型自动触发

hook:
在配置 hook 的时候, 可以配置 type:prompt, command等, 来表明当前 hook 是怎么使用的, 示例中的 hook, 可以告诉 read 哪个文件的时候, 注入什么上下文(必须读什么文件), 也可以 hook 一下每个文件头, 加入一些 metadata

task: 此功能主要区分于 todo-list 的点是可以描述出 dependency 依赖, 比如 taskitem2 实际上依赖于 taskitem1, taskitem3 和 taskitem4 可以并行
使用 ctrl+T 可以看到目前有哪些 task