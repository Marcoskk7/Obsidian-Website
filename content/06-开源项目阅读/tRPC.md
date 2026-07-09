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