https://github.com/zainnab-sparq/cc-self-train

/branch 命令, 可以从当前会话, fork 出一个新对话, 直接从新对话开始内容

记忆中, 分为 CLAUDE.md, CLAUDE.local.md 以及 .claude/rules, 可以根据 指定不同paths 的 frontmatter, 来生效在不同路径下的 rules

skills 的使用很有讲究, 可以设定如下几种:
- context: fork 表示当 agent 通过 skill 调用时, 走后台启动, 而不占用主会话
- effort: 设置推理强度
- skillOverrides : 控制哪些 skill 对 Claude 可见。值：off（完全隐藏）、user-invocable-only（仅你手动调用可见）、name-only（仅名称可见，省描述占用的 token）。
  
- disable-model-invocation: true, 设置只能手动触发不能模型自动触发