# LLM张老师短视频系列

tokenization 就是通过词表，将文字转化为数字的过程
position encoding 就是不仅要有数字化的文字，还要有对应的位置，否则无法学到对应关系，使用的是 sin 和 cos，关于模型为什么能学习到里面的信息，没有统一的说法，只能说只要 epoch 够多，模型就是能学到对应的趋势
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260205235952815.png)

这里的每一行都是一个文字，每一列是一个 dimension
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260206000304725.png)


- 矩阵相乘是怎么做的
