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

