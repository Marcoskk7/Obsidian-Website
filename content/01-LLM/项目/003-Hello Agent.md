https://github.com/datawhalechina/hello-agents
https://huggingface.co/docs/smolagents/index## 

`hello-agents` 简历部分怎么描述?
> Hello-Agents 智能体框架与多智能体应用开发｜AI 应用 / Agent 工程项目
> - 基于 `Python`、`FastAPI`、`Vue3`、`TypeScript`、`Pydantic` 和 `HelloAgents`，系统搭建并复现智能体开发全链路，完成 `16` 个章节、`82` 个 Python 示例和 `3` 个综合项目实践，覆盖单智能体、多智能体、工具调用、记忆检索、上下文工程与评估闭环。
> - 从零实现 `ReAct`、`Plan-and-Solve`、`Reflection` 等经典 Agent 范式，抽象统一的 `LLM / Tool / Memory` 能力层，支持外部工具调用、任务拆解、反思修正与多轮推理，提升复杂任务场景下的可扩展性与可维护性。
> - 构建基于 `Memory + RAG + Context Engineering` 的智能体增强方案，集成工作记忆、长期记忆、检索增强生成、结构化笔记和上下文压缩能力，改善长任务执行中的上下文丢失与信息噪声问题。
> - 集成 `MCP / A2A / ANP` 三类协议，打通智能体与外部工具、智能体与智能体、智能体网络之间的标准化通信，形成可扩展的多智能体协作架构。
> - 落地 `智能旅行助手`、`自动化深度研究助手`、`AI Town` 三个端到端应用：其中旅行助手实现行程规划、地图可视化、预算计算和 `PDF/图片` 导出；深度研究助手基于 `FastAPI + SSE` 实现流式研究报告生成，将传统 `1-2 小时` 的人工调研压缩至 `5-10 分钟`；AI Town 基于 `Godot + FastAPI` 实现 `3` 个 AI NPC、短期/长期记忆与 `5` 级好感度交互系统。
> - 引入 `BFCL`、`GAIA`、`LLM Judge`、`Win Rate` 等评估体系，对工具调用与综合任务能力进行量化验证；其中 `BFCL` 覆盖 `1120+` 样本，`GAIA` 覆盖 `466` 个真实任务，提升智能体系统的可评估性、可对比性和持续迭代效率。

## 必读指引

> [!abstract]- 必读指引
> 
> 必读顺序建议是：
> 
> `docs/chapter1/第一章 初识智能体.md`：理解 `Agent Loop`、`Thought/Action/Observation`，这是全书总纲。
> 
> `docs/chapter4/第四章 智能体经典范式构建.md`：必读，直接学会 `ReAct` / `Plan-and-Solve` / `Reflection` 三种经典范式。
> 
> `docs/chapter7/第七章 构建你的Agent框架.md`：必读，核心价值是“从会用框架，到会造框架”。
> 
> `docs/chapter8/第八章 记忆与检索.md`：必读，Agent 真正落地离不开 `Memory + RAG`。
> 
> `docs/chapter9/第九章 上下文工程.md`：必读，这一章非常贴近现在企业里的 Agent 工程实践。
> 
> `docs/chapter10/第十章 智能体通信协议.md`：必读，`MCP` / `A2A` / `ANP` 是现在面试里高频关键词。
> 
> `docs/chapter12/第十二章 智能体性能评估.md`：必读，很多人会做 Agent，但不会量化评估，这章能拉开差距。
> 
> `docs/chapter13/第十三章 智能旅行助手.md` 或 `docs/chapter14/第十四章 自动化深度研究智能体.md`：二选一至少精读一个。想偏产品落地读 `13`，想偏前沿 Agent 工作流读 `14`。
> 
> 补充：
> 
> `docs/chapter11/第十一章 Agentic-RL.md`：如果你投算法 / 训练岗，它也要升级为必读。
> 
> `docs/chapter15/第十五章 构建赛博小镇.md`：如果你想突出游戏 AI / NPC / 交互式智能体，再读它。
> 
> `docs/chapter2/3/5/6`：建议快读，不用花最多时间，但面试时能帮你补齐“历史、LLM基础、框架生态、低代码平台”的认知。
> 
> 必须自己手敲的代码
> 
> 别全敲，按“最小闭环”敲这些就够了：
> 
> 入门闭环
> 
> `code/chapter1/FirstAgentTest.py`：这份代码把最基础的 `Agent Loop` 跑通了，必须手敲一遍。
> 
> 三大范式
> 
> `code/chapter4/ReAct.py`
> 
> `code/chapter4/Plan_and_solve.py`
> 
> `code/chapter4/Reflection.py`
> 
> 自建框架核心
> 
> `code/chapter7/my_llm.py`
> 
> `code/chapter7/my_simple_agent.py`
> 
> `code/chapter7/my_react_agent.py`
> 
> `code/chapter7/my_calculator_tool.py`
> 
> `code/chapter7/my_advanced_search.py`
> 
> 记忆与检索
> 
> `code/chapter8/03_WorkingMemory_Implementation.py`
> 
> `code/chapter8/10_RAG_Pipeline_Complete.py`
> 
> 上下文工程
> 
> `code/chapter9/01_context_builder_basic.py`
> 
> `code/chapter9/03_note_tool_operations.py`
> 
> `code/chapter9/05_terminal_tool_examples.py`
> 
> 协议与多智能体
> 
> `code/chapter10/05_UseMCPToolInAgent.py`
> 
> `code/chapter10/07_SimpleA2AAgent.py`
> 
> 有余力再敲 `code/chapter10/11_ANPInit.py`
> 
> 评估闭环
> 
> `code/chapter12/02_bfcl_quick_start.py`
> 
> `code/chapter12/05_gaia_quick_start.py`
> 
> 最后挑一个完整项目复现
> 
> 产品向：`code/chapter13/helloagents-trip-planner`
> 
> 研究向：`code/chapter14/helloagents-deepresearch`
> 
> 一句话原则：先手敲 `chapter1 + chapter4 + chapter7`，再补 `chapter8/9/10/12`，最后完整复现 `chapter13` 或 `chapter14`。


## Ch1 初识智能体

Agent 闭环主要包括三个内容, Thought-Action-Observation
Thought 是模型思考, Action 是模型打算调用的 tools, Observation 是模型清洗后的 tool 输出, 用于附加到 history 内容中
### workflow和agent 的区别

工作流是一种传统的自动化范式，其核心是**对一系列任务或步骤进行预先定义的、结构化的编排**。它本质上是一个精确的、静态的流程图，规定了在何种条件下、以何种顺序执行哪些操作。

## Ch4 智能体经典范式构建

一个良好定义的工具应包含以下三个核心要素：

1. **名称 (Name)**： 一个简洁、唯一的标识符，供智能体在 `Action` 中调用，例如 `Search`。
2. **描述 (Description)**： 一段清晰的自然语言描述，说明这个工具的用途。**这是整个机制中最关键的部分**，因为大语言模型会依赖这段描述来判断何时使用哪个工具。
3. **执行逻辑 (Execution Logic)**： 真正执行任务的函数或方法。

### ReAct 的优缺点

给的示例任务是搜索最新款华为手机, 需要调用 tools

最大优点是透明, 最大的局限性是依赖于 LLM 底层能力和执行效率
Thought+Action+Observation

ReAct 就是 Reasoning(推理)+Acting(行动)

核心代码包括:
1. 工具类(多个工具就多个类)
2. 工具注册类(所有工具都注册在这里)
3. Agent 类(包含 run ,\_parse_output 方法, 以及将 Observation 得到的内容不断附加到 history 中)
### Plan-and-Solve

示例是进行苹果的一个公式计算
```python
将字符串转为python 对象的列表
plan = ast.literal_eval(plan_str)
```

### Reflection

示例是一个找素数, 可以不断对算法进行优化, 反思迭代, 从O(n * sqrt(n))优化到 O(nloglogn)

核心代码包括:
1. memory, 设计短期记忆机制
2. 初始提示词, 反思提示词, 优化提示词

最大优点是鲁棒性提升和可靠性增强, 最大缺点是串行运行, 效率大大降低
### 三种范式总结

ReAct 是一边运行, 一边随时调整, 属于走一步, 看一步

Plan-and-Solve 是在出发前, 先锚定蓝图, 按照计划去执行

Reflection 是事后复盘反思, 对初稿进行优化后重新修正, 获得修订稿

![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260312174631829.png)

## Ch7 构建你的 Agent 框架

- 实现了auto 检测提供商(继承自框架, 但是将函数重写, 戒断 if
provider = "xxx" :)

- 发给 LLM 的一定是一个 List(Dict[str:Any])类型的列表

- Agent 类要包含 run, run_with_tools, \_parse_output 等方法
- 
## Ch8 记忆与检索

memory 负责存储和维护对话过程中的交互信息
rag 则负责从用户提供的知识库中检索相关信息作为上下文

```
HelloAgents记忆系统
├── 基础设施层 (Infrastructure Layer)
│   ├── MemoryManager - 记忆管理器（统一调度和协调）
│   ├── MemoryItem - 记忆数据结构（标准化记忆项）
│   ├── MemoryConfig - 配置管理（系统参数设置）
│   └── BaseMemory - 记忆基类（通用接口定义）
├── 记忆类型层 (Memory Types Layer)
│   ├── WorkingMemory - 工作记忆（临时信息，TTL管理）
│   ├── EpisodicMemory - 情景记忆（具体事件，时间序列）
│   ├── SemanticMemory - 语义记忆（抽象知识，图谱关系）
│   └── PerceptualMemory - 感知记忆（多模态数据）
├── 存储后端层 (Storage Backend Layer)
│   ├── QdrantVectorStore - 向量存储（高性能语义检索）
│   ├── Neo4jGraphStore - 图存储（知识图谱管理）
│   └── SQLiteDocumentStore - 文档存储（结构化持久化）
└── 嵌入服务层 (Embedding Service Layer)
    ├── DashScopeEmbedding - 通义千问嵌入（云端API）
    ├── LocalTransformerEmbedding - 本地嵌入（离线部署）
    └── TFIDFEmbedding - TFIDF嵌入（轻量级兜底）
```

```
HelloAgents RAG系统
├── 文档处理层 (Document Processing Layer)
│   ├── DocumentProcessor - 文档处理器（多格式解析）
│   ├── Document - 文档对象（元数据管理）
│   └── Pipeline - RAG管道（端到端处理）
├── 嵌入表示层 (Embedding Layer)
│   └── 统一嵌入接口 - 复用记忆系统的嵌入服务
├── 向量存储层 (Vector Storage Layer)
│   └── QdrantVectorStore - 向量数据库（命名空间隔离）
└── 智能问答层 (Intelligent Q&A Layer)
    ├── 多策略检索 - 向量检索 + MQE + HyDE
    ├── 上下文构建 - 智能片段合并与截断
    └── LLM增强生成 - 基于上下文的准确问答
```

### MemoryTool
- add: 会话 id 自动管理
	- 多模态数据的智能处理
	- 上下文信息的自动补充
	- importance 参数, 标记记忆重要程度
- search: 查找最相关的内容
	- 包含参数 min_importance, 过滤低质量结果
	-  memory_type指定记忆类型搜索
- forget: 删除不重要的记忆
	- 删除重要性低于阈值的记忆
	- 删除时间长远的记忆
	- 删除容量超限时的记忆
- consolidate: 整合记忆, 将重要短期记忆整合为长期记忆
	- 识别重要性超过 0.7 的进行整合保存
### 四种记忆类型
- 工作记忆:
	- 最活跃, 纯内存存储, TTL 机制进行自动清理, 访问速度快
	- 检索使用 TF-IDF 向量化进行语义检索, 没有则退化关键词匹配
- 情景记忆:
	- 要包含时间信息和事件的完整性
	- 持久化, 包含 metadata
- 语义记忆
	- 记录下百科内容, 比如一些常识内容
	- 使用 vector db 进行存储
- 感知记忆:
	- 存储的是多模态信息, 一般是原始的输入的信息
### Rag系统

朴素 rag: 使用 TF-IDF 和 BM25 进行传统关键词匹配, 难以理解语义相似性
高级 rag: 基于稠密嵌入的语义检索, 将文本转化为高级向量, 模型能理解语义相似性
模块化 rag: 混合检索
```
任意格式文档 → MarkItDown转换 → Markdown文本 → 智能分块 → 向量化 → 存储检索
```

存储包括段落层级的分块, 让他知道每块内容属于哪个大标题
- 高级检索策略: MQE, HyDE
MQE 是用 LLM 生成了多个等价的 Query, HyDE, 是先回答, 再基于回答进行回归

对于实战系统, 使用了 MQE 后, 通过LLM生成语义等价但表述不同的查询，从多个角度理解用户意图，提升召回率30%-50%。

chunk 的策略是, 先对段落进行划分, 然后包含一些 metadata, 比如对应的标题, 最后再贪心合并, 直到最接近 max_chunk_tokens

### 思考题
1. 在8.3节的RAG系统中，我们使用MarkItDown将各种格式文档统一转换为Markdown。请深入思考：
    
    > **提示**：这是一道动手实践题，建议实际操作
    
    - 当前的智能分块策略基于Markdown的标题层次（#、##、###）进行分割。如果处理的是没有明确标题结构的文档（如小说、法律条文），应该如何优化分块策略？请尝试实现一个基于"语义边界"的分块算法。
	    - 针对每一个法律条文名称进行划分, 对于小说, 可以根据自然段落进行划分, 比如第 x 条, 或者\n 来划分


    - 在8.3.5节中介绍了MQE（多查询扩展）和HyDE（假设文档嵌入）两种高级检索策略。请选择一个实际场景（如技术文档问答、医疗知识检索），对比基础检索、MQE和HyDE三种方法的效果差异，并分析各自的适用场景。
	    - 基础检索直接 query, 适合精准度高的提问
	    - MQE 适合模糊的提问, 多个等价提问能更好找到关键词
	    - HyDE 适用于专业的, 如法律, 医疗等领域
    - RAG系统的检索质量很大程度上取决于嵌入模型的选择。请对比本章提到的三种嵌入方案（百炼API、本地Transformer、TF-IDF），从准确性、速度、成本、离线部署等维度进行评估，并给出选型建议。
	    - 做混合检索, 结合 BM25 粗排, 再使用本地 Transformer 或者云端 API 进行精排
## Ch9 上下文工程

- **压缩整合**：适合需要长对话连续性的任务，强调上下文的“接力”。
- **结构化笔记**：适合有里程碑/阶段性成果的迭代式开发与研究。
- **子代理架构**：适合复杂研究与分析，能从并行探索中获益。

可以从图中看出, prompt 只包含文字的上下文, 但是 context 是多模态的, 需要筛选之前的有效信息重新构建
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260318134106353.png)

