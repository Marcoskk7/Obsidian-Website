
拓展问答:
1. 你的 functioncall 怎么保证准确率?
答: 做了动态路由+prompt 调优(给出正反例), badcase 收敛入memory, 失败重试和参数校验, 以及我们提供了规范化的字段来给模型进行 observation，让它更好地 thinking 下一步。