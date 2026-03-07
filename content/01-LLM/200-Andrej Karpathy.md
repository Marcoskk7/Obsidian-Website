
# Andrej
## 基础概念
这个网站对 LLM 整体进行了可视化，[LLM 可视化](https://bbycroft.net/llm)
[# Deep Dive into LLMs like ChatGPT](https://www.youtube.com/@AndrejKarpathy)
GPT-2 有1.6b个参数，Llama3则有更多参数

## 训练部分

### Pre-Training
基模的训练数据全部来源于互联网，比如 wiki百科等等
对于base model，训练的结果仅仅是文字继续补全，对你上面的对话进行延续

### Post-Training
而到了 Post-Training 阶段，则相对便宜但更重要，这里才能将模型转变为你的助手
此时会将训练数据集全部替换为人工的上下文对话，来让 ai 学习如何进行对话，而这些好的上下文对话是由人类标注员进行实现的，手工标注希望模型预期的输出
但现在要手工标注这么多数据也不是特别现实，因此会用大语言模型生成很多对话，再加入人工对对话的编辑进行手动微调

你可以将 ai 的对话想象为和一个背后的人工标注员的对话，这极有可能是非常相似的

这一部分一般被称作 SFT，监督微调，是让模型学习标准的问答模式

### Reinforcement Learning

pre-train 相当于是上课看课本了解只是
Post-Train 相当于是课堂习题，老师带着你做，正确答案
Reinforcement-Learning 则是自己课下做课后习题进行提高

在这一部分没有人类标注员，而只有 model 自身，他只有问题和答案，但是没有过程，因此他会自己做很多种过程，假设图中能得到正确答案的只有四个，剩下的红色线表示做错了，那么模型会从这四个中选一个最好的，比如黄色，那么他会根据这个过程，将模型参数进行调整

![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260305161841942.png)

那么这个最好的答案是怎么得出的呢？是人类训练了一个模仿人类打分的 LLM，用于给这几个答案进行打分，因此得分高的一般是人类也认为做的好的

在 RLHF 阶段生成模型如果运行的步骤过长，那么几乎总会找到一种方法来骗过评判模型，这样得到的训练效果不是我们想要的，比如一个无意义的“the the the the the"可能可以得到一个很高的分数，但这很显然不对。
因此我们要做截断，上升到顶点好不再训练，可以直接发布

但是对于有确定答案的，比如围棋，AlphaGo，那么可以无限训练，不会被欺骗，赢就是赢，输就是输
## Hallucinations 幻觉

自信的回答源于训练时也是这样自信的回答数据，因此哪怕当前不认识，他也不会说不认识，而是模仿训练数据集进行自信的回答
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260304173300015.png)

> 解决方案：
> 对于一些事实，让一个 LLM 提出问题和标准答案，然后把问题重复给训练的模型，如果模型不能正确回答，那么新增一组数据，对话是问题+标准答案为“我不清楚/我不知道”，这样的数据多了，LLM 就会学会说不知道了

那么怎么进行 web search, 可以看到是有特殊的 token 进行启动的，比如这里的<SEARCH_START>就告诉我们现在可以进行内容的搜索了
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260304220009575.png)

## AI是怎么知道自己是谁的？

有两种方法，可以让 ai 清楚的知道自己是谁：
1. 通过加入一组专门训练名字对话的数据集在 SFT 中，比如“你是谁”“我是 xxx model，我的知识截止日期为 xxx"
2. 在用户对话前，加入一段 context 在你看不见的地方，也可以叫 System Prompt，这样 ai 可以快速拿取到身份信息，准确度更高

## 模型依赖 token 进行思考

这两个答案哪个更好？选择 2，因为答案 1 在一开始就回答了 3，所以所有计算都出现在一个 token“3”中，这是极不准确的，后面的语言都是对这个答案的补充说明，而右边则将 token 的计算拆分开，一步步慢慢计算，这是更符号 LLM 生成要求的
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260304222057189.png)

但是这一切都像是人类在进行心算，所以只要涉及复杂的问题，一定要 ask ai to use tools

## AI 不擅长拼写

对于单词的字符选取往往不尽人意，因为它是基于 token 来进行学习的，所以你问他 Hello 的第三个字母是什么，可能不容易精准得到，因为 Hello 在他眼里只是一堆数字 token

## DeepSeek-R1

别人都没怎么讲 reinforcement Learning publicly，只有 DeepSeek-R1 公开的进行谈论强化学习相关内容

##
# 3B1B

Chapter5 讲解基础概念，FFN 是前馈神经网络，直观理解了 embedding 的含义，越相似的东西在embedding 后也应该近似，举个例子，向量父亲减去向量儿子，应该和向量父子差不多
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260225184029883.png)

![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260225175739092.png)
