## Cha6 提示词工程

1. 提示词不是一次写好的, 一般是先一版, 然后不断迭代改进
2. 越清晰效果越好, 把模型当孩子
3. 最好有评价标准, 不然改了怎么知道是好的呢
- 给 few shot
- 使用 XML tags 是很好的
- 给模型一个 Role, 让他 act as a role, 更容易代入
- 可以直接从<think>开始, 不使用 response 的 api 而是使用 generate
- 长上下文, 把 doc 放到最前面最好, 因为最贴近的时候效果最好
- 

## Cha7 LangChain and LangGraph

1. langchain 调用 init model+invoke, 使用 prompt template
	1. 统一的用 init_chat_model 或着 ChatxxModel 去初始化一个 chat model
	2. 统一的用 invoke(messages) or invoke({'xx':'xx'})
	3. 可用 Prompttemplate 构建结构化的模板
	4. | 语法糖
	5. 结构化输出, parser
2. langgraph memory
3. langgraph 的 create_react_agent



