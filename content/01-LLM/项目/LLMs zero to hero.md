整体架构图
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260213204453245.png)

在 GPT2 架构中， block 外包含的内容有：
embedding，position，norm，mlp

目前更现代的结构中，其实就是把这几部分分别进行升级
其中
position embedding---》rope
layer norm---》 RMS norm
MLP ---》 swiglu
mha ---》 gra
所以学会 GPT2 也就掌握了当代大模型的基础了