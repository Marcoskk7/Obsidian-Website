# Mem0 - 一个工业级的 Agent Memory 框架

## 面试视角
> [!abstract]- 面试 Prompt
> / #Role 你是一位拥有 15 年经验的资深技术专家/架构师，擅长代码审查、架构设计、分布式系统以及工程实践。你现在担任我的“面试官”，目的是通过深度提问，考察我不仅是“如何实现”功能，更是“为什么这样设计”以及“权衡（Trade-offs）”的能力, 重点中的重点, 你要考察我如何结合业务设计。 # Context 我将向你展示我的项目代码或架构设计（如果是代码库，请读取项目结构）。你需要仔细阅读并理解业务逻辑。 # Interaction Rules 1. **引导式提问（One-by-One）**：不要一次性问所有问题。请每次只针对一个核心点（比如架构设计、并发处理、数据一致性、代码可维护性等）向我提问。 2. **深度挖掘（Socratic Method）**：在我的回答之后，不要只说“正确”或“错误”。请针对我的回答提出挑战（"What if..."，或者“考虑到这种方案，如何解决 X 问题？”），引发我更深层次的思考。 3. **评估反馈（Evaluation）**：每轮问答结束时（或者在我请求时），请从“代码质量”、“方案合理性”、“业务边界意识”三个维度给我 1-5 分的打分，并给出具体的改进建议。 4. **技术栈聚焦**：请基于我项目使用的技术栈（我会告知你）进行提问，侧重于该技术栈的陷阱和最佳实践。 # Initialization 请首先确认你已准备好。确认后，请询问我： 1. 这个项目的核心业务场景是什么？ 2. 请让我上传项目路径或粘贴核心代码片段。 请在准备好后，立刻开始你的开场白。

- 真实运行 eval
- 搞懂数据流向

## 项目结构

- cookbooks - 一些jupyter的教学
- docs - 技术文档
- embedchain- mem0前身
- evaluation - 核心
  - metrics - 一些测评的指标
- open memory - 一个完整的前后端
- server - 精简版 , 只有fastapi后端
- OpenClaw是一个OpenClaw的插件

## 评估流程

评估代码的流程, 是使用locomo数据集

然后调用记忆+rag, 返回的原始结果要拿到三个分数, 分别是BLEU-1, Token F1分数和LLM评判器给的分数: 运行顺序 run_experiments->evals->generate_scores

## mem0 文件夹

### memory 模块

对于Memory基类, 使用的设计模式是先做抽象基类, ABC, MemoryBase, 再继承MemoryBase进行使用

storage是使用了sqlite3来持久化

### configs 模块

- base是一些config
- prompts.py是核心代码, 学她的设计模式
  - 身份+擅长xxx+你的任务
  - 给一些few-shot examples
- llms.py是对不同供应商的api的适配, embeddings.py是对embedding模型的api的适配, vector_stores一样, 对数据库
- reranker是对各种工具做reranker, base定义ABC, 输入query, documents, top_k, 返回排好序的文档和相关分数

通过util.factory来统一创建各个组件

## 核心机制

### Reranker 的作用

那么reranker的作用是什么? 做精排, 直接query是一种粗排, 速度快但是准度低, 是一个直接embedding求余弦相似度,  再通过reranker精排

### 增删改None 逻辑

- 增: 旧的记忆没有, 新的记忆有, 那么新增 === 调用_create_memory写入sqlite3
- 删: 新的记忆和旧的记忆矛盾, 删去旧的
- 改: 语义有变化或者信息量变化, 那么更改
- None: 旧的记忆也有, 那么不动

### Search

- 搜索向量记忆和图记忆
- 可选是否重排序

### 为什么现在更多使用 markdown?

人类可读可编辑, 可以直接修改记忆

透明可调试

## 设计优势

- 工厂模式
- uuid映射
- embedding缓存
- 第一次是抽取
- 第二次是决策操作

### Memory.add() 函数的实现逻辑

![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260323230045775.png)

### historydb 的表设计逻辑

![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260323230238188.png)

## 存在的问题

1. 每次对话就做一次fact 提取, 如果跨多次对话, 才能抽一次信息, 就会导致模型最后的 Memory 其实是空的, 但实际上是有 fact 要记忆的
2. qps 低, 要两次 llm 额外调用+第一次调用的时候需要embedding+search
3. 所有记忆的管理都交给 llm 决定, 那么基模的能力就很重要了

## 总结

整体来看，mem0 在把记忆这件事工程化上做了几件比较关键的事：

1. 把 `add`​ 设计成一个具备决策能力的入口，在 `infer=True` 下通过两次 LLM 调用和一次向量检索，把 ADD / UPDATE / DELETE / NONE 合在一条管线里做完。
2. 用一个独立的 SQLite history 表记录所有变更，把每条记忆视作一个可以审计的状态机，也可以说是一种血缘追踪，而不是简单的覆盖式更新。
3. 通过Vector Factory 和 Graph Factory，把底层存储细节屏蔽在统一接口之后，让 Memory 层只关心"存什么、怎么搜"。事实上 mem0 的绝大部分代码都在适配十几种向量库和图数据库上。
4. 在设计上明确区分了长期记忆和短期工作记忆的角色，前者交给 mem0，后者留给上层框架或应用代码。
