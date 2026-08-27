
要定义 Agent Memory, 核心是弄懂核心的三个维度, 也就是怎么存, 怎么注入回上下文, 怎么管理:
1. What is it
		a. txt, md
		b.SQLite, 表格
		c.graph, nodes, 通过图的方式存储
2. How to find it
		1. 全量注入
		2. rag 检索, 选择性召回
		3. BM25 关键词匹配召回(sqlite)
		4. GraphRag
3. How to maintain it?
	1. 操作: 增删改查
	2. 生命周期: 过期时间, 新版内容
	3. 属性: trace_id, version 等
	4. 后台操作: dream, 负责审查事实, 更新内容 or 进行部分 merge



有四种 assets, 分别是
- chat memory
- llm-wiki
- code-graph
- skills

可了解的其余开源项目: 
- openviking
- headroom

分层记忆分为L0-L3

Q: 怎么分层的
A: 符号化短期记忆+结构化长期记忆 符号化短期记忆, 卸载到文件系统, 上下文仅保留 mermaid图

Q: 怎么保证 token 缩减, 做了什么测试:
![image.png](https://img.486597.xyz/img/20260822170821814.png)

Q: 分的是哪几层, 代表什么
A:
![image.png](https://img.486597.xyz/img/20260822170901574.png)


Q: rag 在 Memory 上会有什么问题
A: 首先, 召回的只是语义相似, 不一定确实需要, 像是在一堆便利贴中找线索, 第二个是我们召回了那么多内容, 但是我们也不清楚是否是否真的合适, 只能拿到对应的向量分数确实比较高而已, 但是查不出是哪一步出问题


真实带来的问题, 包括信息的捕获, 召回, 治理以及共享

捕获, 我们希望是用户无感的, 并且不能有太多噪声, 召回要防止召回无关上下文, 治理过严格, 可能会导致效率较低, 治理过松会导致越权的扩散, 所以后续治理是在相关性, 可信度, 权限, 成本和体验之间的平衡, 

![image.png](https://img.486597.xyz/img/20260823184355111.png)


那么 session init 的部分, 就是通过 team+agent+task 的部分来进行 init, 可以做资产绑定, 方便第二部进行资产召回,
> Proxy 管控的是 context 的组织, 而 core 管控的是核心的资产

高可用性, 说白了就是保证出错的时候, 能重新恢复, 重新回滚, 重新可用的

![image.png](https://img.486597.xyz/img/20260823200330053.png)
![image.png](https://img.486597.xyz/img/20260823200624112.png)
主要是为了有价值的团队经验, 以可信, 可控的方式, 去进入到下一次任务当中




代码阅读:
MemoryProxy:
1. server.ts:
	一切的入口, 负责注册路由, 实际业务逻辑处理交给 handler(不同的上游, 请求到不同handler), 提前连接状态存储, 包括COS, sqlite, fs, 进程内存, 以及检查 marker url:
	### Marker

例如普通请求：

```
/codebuddy/space-1/v1/chat/completions
```

带 Cost Guard marker：

```
/codebuddy/space-1/cost-guard/v1/chat/completions
```

带分析 marker：

```
/codebuddy/space-1/analyse/v1/chat/completions
```

其中：

```
cost-guard
analyse
```

就是 marker。

它不承载业务数据，只表示“本次请求启用某种特殊行为”。代码上, 我们可以拦截, 这些中间 marker, 然后在代码中做对应的处理, 
handler.ts
管理完整 HTTP 生命周期
        │
        │ 调用
        ▼
pipeline.ts
只修改请求 Body
        │
        │ 返回 Modified Body
        ▼
handler.ts
选择上游、发送请求、处理回执

> pipeline 说白了就是把不同协议转换成统一 `AgentContext`，按顺序执行所有资产 Hook，把结果放进 System、Tools 或 User Message，再转换回原协议。


MemoryCore:
普通对话：
CodeBuddy → Proxy /chat/completions
          → Session + Injector
          → 上游 LLM

Skill 查询：
模型执行 curl → Proxy /skill-bridge/v3/skill/search
             → 补入 Session 身份
             → Core /v3/skill/search
             → SkillCore / Store
在 memory core 中的 server.ts，它和在 memory proxy 中的 server.ts 最大的不同就是，它会负责更多内部业务逻辑上的内容。, 比方说，这个 skill 和 memory 都是在 proxy 侧注册的，是一个 bridge 路由。然后再从那里补充一些基础信息（如 Task team name 等），再转发到 core 这里来进行鉴权调用。

`tdai-core.ts` 是记忆系统的统一编排入口，负责召回与搜索记忆、捕获并存储 L0 原始对话，以及触发后台流水线生成 L1 记忆、L2 场景和 L3 用户画像。

memorycore/src/core/store 定义 L0/L1/L2/L3 级别存储什么信息, 
- L0 保存原始消息，核心字段是 Session、隔离身份、角色、消息正文和时间。
- L1 保存抽取后的结构化记忆，除正文外还包含类型、优先级、场景、版本和来源 Session。
- L2/L3 统一表示成 `ProfileRecord`：`type="l2"` 是场景资料，`type="l3"` 是更高层画像；二者按 Team 与 Agent 隔离，并带版本号。
- `IMemoryStore` 把写入、普通查询、向量检索、全文检索和 Profile 同步统一起来，上层不需要知道实际后端。

资产的重要字段:
source_type, source_ref, version, status, expires_at, confidence


检索资产进入上下文:
ACL 最先, 如果没权限, 那么根本不需要召回
召回相关性, 是两路召回算法, BM25+语义相似, BM25 是要精确关键词匹配的, 而语义相似是神经网络上的 Embedding
还有一个 token budget, 如果再引入新的




其余 Memory 框架:
 - Text2Mem —— 给所有记忆系统定义"操作指令集"，12 个原子操作 + 五元 JSON 契约 + 双层验证(这里在做json 操作前会做 dry-run, 即0 成本模拟跑一次, 不过我想知道这里 0 成本, 0 副作用, 跑一次跑的是什么, 可以验证什么信息) 项目亮点是想通过做一种 Memory 的 SQL 语言, 统一抽象 Memory 的规范
 - Mem0 —— 当下 Star 最多的记忆中间件，5 个工厂 / 双存储 / 三种记忆类型，含真实成本瓶颈分析    项目亮点是做了很多工程化的适配: 首先支持17+ LLM provider, 11+ Embedding 模型, 22+ VectorDB, 4 种图存储, 5 种 Reranker; 其次, 针对 uuid 可能会幻觉的问题, 会将 uuid 映射为 0, 1, 2, 3 这样的简单数字, 防止幻觉. 操作上看起来不复杂, 就是交给 LLM 决定增删改查(记忆划分, 语义记忆, 情境记忆, 程序记忆)
 - Letta —— 把 OS 虚拟内存思想搬进 Agent，Git 版本化记忆 + Sleeptime 异步后台学习      
 - ReMe —— 阿里 AgentScope 出品，"文件即记忆"，记忆对用户完全透明可直接编辑                     
 - memU —— 范式最激进：让记忆本身变成一个 24/7 后台主动 Agent              
 -  - MemOS v2.0.8：六层架构 + 三类记忆（文本 / 激活 / 参数），LoRA 记忆目前仍是 Placeholder 的真相                    
 - OpenViking：字节火山引擎的"上下文数据库"，文件系统隐喻 + L0/L1/L2 分层，Token 成本降低 92%~96%                   
 - Hindsight：LongMemEval SOTA，仿生三层记忆 + MPFP 图检索 + Consolidation 巩固引擎        
 - Second Me：本地训练"第二个你"，L0/L1/L2 知识蒸馏 + LoRA 微调，以及"100% 本地"的隐私悖论                       
 - MetaMem：不管存储只管"会不会用"，Learning to Learn 元记忆层 + Partial Correctness Filter   


实现思路:
首先每个资产可以有字段统计, 累计被召回的次数, 结合任务成功率进行测算, 就是召回的次数比上成功的次数, 可以一定程度上反馈是否真正有效

做 AB 实验, 消融实验, 控制召回的资产数量, 比如说去掉某一个zhao
                                                           