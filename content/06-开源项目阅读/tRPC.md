# GO基础

module是一个公司, package是一个小部分, 一个 go.mod 会声明一个 module, 而一个 module 下会包含很多 package, ./... 表示递归当前 module 下的所有 package, 如果下面有子 module, 那么不会递归子 module 下的内容
go.mod 中的 indirect 模块, 表示的是项目直接依赖的依赖, 因此叫间接依赖
replace 关键字会用本地的路径代替远端路径, 这样本地的修改, 会直接应用

CI 脚本测试仓库核心 module，排除 examples/docs/test/resource

![image.png](https://img.486597.xyz/img/20260708230340243.png)
 这里的 json 意思是, 怎么处理序列化和反序列化, 表明在 json 时叫什么内容, omitempty 是表示支持空传
 
EvalSetResult：一整个评测集的结果
EvalCaseResult：一条评测用例的结果
EvalMetricResult：某个 metric 的评测结果

输入是一组 cases + metrics
输出是一组 case results + metric results

Metric 定义的是成功的标准, 比如这里我们定义两个, 一个是最后结果正确, 一个是工具调用路径正确, 或
```
EvalMetric{
    MetricName: "final_response_match",
    EvaluatorName: "finalresponse",
    Threshold: 1.0,
}

EvalMetric{
    MetricName: "tool_trajectory_match",
    EvaluatorName: "tooltrajectory",
    Threshold: 1.0,
}
```

真实使用的: 
final_response_avg_score
tool_trajectory_avg_score
llm_rubric_response
llm_rubric_critic
llm_rubric_knowledge_recall
llm_hallucinations

> EvalCase 表示一条评测样例。

EvalID：这条 case 的唯一 ID
EvalMode：评测模式
ContextMessages：额外上下文
Conversation：期望对话内容
ActualConversation：实际运行出来的对话内容
SessionInput：初始化 session 的数据
Rubrics：这条 case 自己的评分细则

>Invocation 表示一个 turn

Invocation
  UserContent: "北京今天温度多少？"
  Tools:
    - Name: "weather_tool"
      Arguments: {"city": "北京"}
      Result: {"temperature": "28°C"}
  FinalResponse: "北京今天 28°C"
  
> Tool 表示一次工具调用

type Tool struct {
    ID        string `json:"id,omitempty"`
    Name      string `json:"name,omitempty"`
    Arguments any    `json:"arguments,omitempty"`
    Result    any    `json:"result,omitempty"`
}

EvalCase = 一道题
Invocation = 这道题里的一轮对话/调用
Tool = 这一轮里发生的工具调用
EvalMetric = 判卷规则
EvalCaseResult = 一道题的判卷结果
EvaluationResult = Evaluate 返回的总报告

evaluation_test.go展示了未来我们可能要用到的测试套路:
1. 用 inmemory manager 准备数据
2. 用 fakeService 预设评测结果
3. 调用 Evaluate
4. 断言聚合结果、调用次数、状态、分数

per-case delta, 表示改变前后 ,我们的result的变化, 是 fail->success 还是 succeess->fail
acceptance gate 表示是不是接受, 超过阈值吗

fakeService 对 #2003 重要，是因为它支持无 API Key、无随机性、可重复地测试baseline/candidate 对比、逐 case 差异、接受门禁和报告生成这些工程逻辑。

`ctx context.Context` 主要解决这些问题：

```
取消执行
超时控制
跨调用传递请求级信息
```

agent.Invocation 存这些信息:
```
type Invocation struct {
    Agent Agent
    AgentName string
    InvocationID string
    Branch string
    ParentMetadata *ParentInvocationMetadata
    EndInvocation bool

    Session *session.Session
    SessionService session.Service

    Model model.Model
    Message model.Message
    RunOptions RunOptions
    TransferInfo *TransferInfo

    Plugins PluginManager

    StructuredOutput *model.StructuredOutput
    StructuredOutputType reflect.Type

    MemoryService memory.Service
    ArtifactService artifact.Service

    state map[string]any

    MaxLLMCalls int
    MaxToolIterations int

    llmCallCount int
    toolIterationCount int
}

当前 Agent 是谁：Agent / AgentName
这次调用 ID：InvocationID
多 Agent 调用链：Branch / ParentMetadata
当前会话：Session / SessionService
当前输入消息：Message
当前模型：Model
运行配置：RunOptions
插件：Plugins
结构化输出配置：StructuredOutput
长期记忆服务：MemoryService
Artifact 服务：ArtifactService
本次 invocation 临时状态：state
调用限制：MaxLLMCalls / MaxToolIterations
```

```
Runner.Run 是用户入口和生命周期管理器；
Agent.Run 是具体智能体的一次执行逻辑。
```

`生命周期` 这个词这里指：一次请求从开始到结束，谁创建资源、谁传递上下文、谁处理事件、谁清理资源. 比如说 Runner.Run 这里就管理了:
- 根据 appName/userID/sessionID 查 session
- 不存在就创建 session
- 把当前 user message 写入 session
- 运行后把 assistant/tool 事件写回 session
- 管理 requestID / cancel / tracing / plugins

怎么比较期望的 Tooltrajectory 和真实运行的 Tooltrajectory 的?
首先定义好 case, set 和 metric, 然后我们跑 tool 进行 match 比较, 在 examples 中的比较, 首先检查工具数量, 一定要>=预期数量(因为可以比预期的多), 其次是比较每个 tool 的 metric 是否符合要求, 在这个场景中, 设定的 metric 是 name, arguments 和 result

匹配的 strategy 涉及到比如说是否忽略工具调用顺序, 是否包含 subset 等
toolMock 适合 mock 这些: 天气、新闻、搜索、时间、票务、数据库、远程 API、模型调用,
有两种匹配结果 mock 方式, 一个是根据 name, 一个是根据 name+arguments, 必须要 name 和 arguments 都对, 才 mock Toolcall result, 那么, 为什么要做这么一个操作, 明明 name 和 arguments 都对了, 那么 result 不一定是一样的吗? 这里就要区分了, 有的 toolcall 是一样的, 比如计算器, 但是如果查天气, 使用同样的 city , 在不同的时间查, 得到的 result 就有可能不一致了


Session Memory & Knowledge

其中 session 的隔离维度是 appName+userID+sessionID
memory 的隔离维度是 appName+userID

EvalSet
└── EvalCases[]
    └── Conversation[]
        ├── UserContent
        ├── FinalResponse  // 预期回答
        └── Tools[]        // 预期工具调用

如果要查看预期工具调用, 就在 EvalSet.EvalCases[i].Conversation[j].Tools 字段中


EvalCaseResults[]
└── EvalMetricResultPerInvocation[]
    └── ActualInvocation
        └── Tools[]
            ├── Name
            ├── Arguments
            └── Result




type Service interface {
    AddMemory(...)
    UpdateMemory(...)
    DeleteMemory(...)
    ClearMemories(...)
    ReadMemories(...)
    SearchMemories(...)
    Tools()
    EnqueueAutoMemoryJob(...)
    Close()
}

其中 clear 是删除一个用户的全部相关 memory, 而不是一条一条进行清理, close 是用于清理资源的


在 simple memory 里，为什么既要 `WithTools(memoryService.Tools())`，又要 `WithMemoryService(memoryService)`？
WithTools = 给 LLM 看见工具
WithMemoryService = 给工具运行时提供后端

测评模式存储位置, 普通模式需要真实运行 Agent, 而 trace 模式使用的是旧的已经生成好的对话, 适合重放历史线上事故的对话, 而不用重新调用 Agent

actuals   ← Runner 的输出 (普通模式)
actuals   ← EvalCase.ActualConversation(trace 模式)

expecteds ← EvalCase.Conversation

### 测评流程

```
evaluation.New()
    ↓
AgentEvaluator.Evaluate("math-basic")
    ↓
MetricManager 读取评分规则
    ↓
Service.Inference()
    ├── Runner 执行 Agent
    ├── 产生 actual Invocation
    └── 收集 Trace
    ↓
Service.Evaluate()
    ├── actuals = 推理结果
    ├── expecteds = EvalSet.Conversation
    ├── Registry 选择 Evaluator
    └── Evaluator 比较并评分
    ↓
聚合多次运行
    ↓
EvalResultManager.Save()
    ↓
返回 EvaluationResult
```


issue 2003 必须覆盖的几种错误:

profile 存的是不同 surface 被替换为什么
text gradient 是自然语言描述怎么修改
aggregation 是把多个 case 对于同一 surface 的修改建议进行合并
patch 是真正可用的最终文本, 为 optimizer 生成的修改

### 提示词优化的流程
```
InputProfile
    ↓
Train evaluation
    ↓
Losses
    ↓
Backward
    ↓
Aggregation
    ↓
Patches
    ↓
OutputProfile
    ↓
Validation evaluation
    ↓
Acceptance
    ↓
Stop decision
```
真正的E