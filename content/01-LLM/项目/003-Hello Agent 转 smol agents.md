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


## CH1 初识智能体

Agent 闭环主要包括三个内容, Thought-Action-Observation
Thought 是模型思考, Action 是模型打算调用的 tools, Observation 是模型清洗后的 tool 输出, 用于附加到 history 内容中

## ReAct

一个良好定义的工具应包含以下三个核心要素：

1. **名称 (Name)**： 一个简洁、唯一的标识符，供智能体在 `Action` 中调用，例如 `Search`。
2. **描述 (Description)**： 一段清晰的自然语言描述，说明这个工具的用途。**这是整个机制中最关键的部分**，因为大语言模型会依赖这段描述来判断何时使用哪个工具。
3. **执行逻辑 (Execution Logic)**： 真正执行任务的函数或方法。
