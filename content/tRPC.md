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