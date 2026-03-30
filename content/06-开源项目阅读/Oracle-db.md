https://blogs.oracle.com/developers/agent-memory-why-your-ai-has-amnesia-and-how-to-fix-it?source=:ad:nw:op:awr:a_nas::RC_DEVT260124P00001:DevMktg__Alphasignal&SC=:ad:nw:op:awr:a_nas::RC_DEVT260124P00001:DevMktg__Alphasignal

为什么更大的 context 并非解决方案:
1. 首先, 虽然支持那么多, 但是还没到就会崩溃
2. 其次开启新的对话还是一样会清空
3. 成本呈线性增长
那么 Rag 有什么问题?:
知识引入了过去的一些文档和知识, 但是没有任何关联, 不知道之前的互动和用户身份等信息
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260327213704863.png)

热路径记忆:
- ChatGPT 采用, 在回应前决定记住某些信息, 会增加延迟, 但是保证即使
后台记忆: 
- 独立进程, 无延迟, 但不会立刻调用
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260327214405216.png)

记忆的实际运行机制:
1. 语义记忆
- 基于 embedding 语义搜索
- 能捕捉语义相似性, 但是不能捕捉结构关系
1. 知识图谱
- 有一连串关联, 知道你在什么时候做了什么事, 存储为实体和关系
1.  结构化数据库
- 存储事实记忆, 存储结构化数据：用户档案、访问控制、会话元数据、审计日志。