
Questions:
1. 不太理解 YaRN 的使用场景, 为什么要这样设计高低频
2. YaRN 和 RoPE 的代码是粘贴的, 需自己看懂
## 整体架构图

![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260308165430021.png)


## RMSNorm
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260305230437559.png)

RMSNorm 相比于 LayerNorm 少了一个计算均值并减去均值的部分，减少了计算量
下面有个 $\epsilon$ 是为了防止除以 0
$\gamma$ 是为了可以学习缩放的程度，而不是定死在 1

## 正余弦位置编码

==存在什么问题？==
>由于是和原本的词向量直接相加，因此可能会丢失掉原本词向量的语义，搞脏元数据

d 是固定值，比如 512，pos 是位置，比如我爱你中的我，pos 就是 0，然后他的第0维使用 sine 计算，第1维用 cos， 第二维用 sin 交替这样计算

![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260307135034668.png)

这里可以看到只有在低维度的时候数值变化较大，高纬度变化很小，高维度对远距离才比较敏感，近距离几乎不变

可以看到维度越小，周期越小，变化越快
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260307140521861.png)
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260307140904489.png)

> ==推导过程

![dc38b9549bc7ca8ba1e34a15b41cf9b5.jpg](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/dc38b9549bc7ca8ba1e34a15b41cf9b5.jpg)

## ROPE 旋转位置编码

这一块比较难理解, 给出几个煮包学习时看过的资料供参考
- https://www.bilibili.com/video/BV1F1421B7iv

可以看到旋转矩阵有几个比较好的性质, 第一是两个旋转矩阵相乘, 等同于角度相加(逆时针); 第二是转置矩阵就是顺时针旋转
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260308190127531.png)

通过数学推导可以看出, 加上旋转后的角度(即q旋转 m, n 旋转 n), 得到的点积, 既包含原先的 $qk^T$ 还包含了两者差的旋转角度(m-n)
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260308190353566.png)

这里的旋转矩阵就能看出, 节省了很多空间,这是一个正方形, 假设长为 n, 那么理应要存 n * n 个参数,但是现在只需要存  2n 个参数
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260308191331133.png)



==有没有办法不搞脏元数据的同时，又能保留位置信息？

>ROPE 就可以！

原本的词向量，乘上这个旋转矩阵，就会保持模长不变而仅改变角度
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260307142831260.png)

可以看到旋转的$\theta$取值和正余弦计算公式及其类似，因此机制也类似，在一个词的低纬度旋转的$\theta$较大，而在高纬度旋转的角度较小
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260307142949656.png)

然后两两维度一组进行旋转，那么为什么两两一组？因为旋转只能在二维平面上进行旋转，因此只能每两个维度进行一次旋转，这样，$d_{model}$只能设置为偶数

![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260308162201667.png)

## 正余弦 vs RoPE

正余弦的位置编码，在计算注意力分数中时，会出现无意义的中间项，给计算添加噪音
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260307142613359.png)
而 RoPE 不会，嘎嘎清爽
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260307142733177.png)
## YARN

YARN的出现是为了确保训练和推理时出现 token 长度不一致的情况, 比如训练的时候 token 长度都是 2048 的,但是推理的时候 token 长度是 4096 的,那么怎么进行处理呢?

- torch.outer(a, b)：**外积**。输入两个 1D 向量，输出一个 2D 矩阵。
    
- torch.dot(a, b)：**内积（点积）**。输入两个长度相同的 1D 向量，输出一个标量（单一数值）。
    
- torch.mul(a, b) 或 a * b：**逐元素相乘（Hadamard积）**。输入形状相同（或可广播）的张量，对应位置相乘。
    
- torch.matmul(A, B) 或 A @ B：**矩阵乘法**。
## GQA

由于多份 QKV 很占用显存,因此想办法让多个 Q 共享同一组 KV 
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260307214733922.png)


## Dataset

存储格式为 jsonl(即 json-line 格式), 为一行一行的 json, 方便代码读取和分割
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260307231406149.png)

## 预训练

动态学习率调整就是余弦退火学习率,cs 是 current step, ts 是 total step,当 cs 为 0 的时候,学习率为 lr,当 cs 为 ts 的时候,学习率为 0.1\*lr
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260307231003078.png)

==混合精度是指 32 位的时候占用显存大,因此可以通过 scaler,把后 16 位的数据一定程度反映到前 16 位进来以降低显存占用
