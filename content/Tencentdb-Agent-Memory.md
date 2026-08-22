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