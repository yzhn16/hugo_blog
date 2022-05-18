# 李宏毅ML课程笔记——Self-attention

[李宏毅2021/2022春机器学习课程——自注意力机制（Self-attention）](https://www.bilibili.com/video/BV1Wv411h7kN?p=38)
<!--more-->  

# 自注意力机制（Self-Attention）

考虑两种不同的输入：

- 输入是一个向量
- **输入是一排向量**（输入的向量个数可能会改变）

## 将一排向量作为输入

### 方法

- One-hot Encoding
- Word Embedding

### 输入

- 一段语音窗口
- 一张图
- 分子结构
- ...

### 输出

#### N个向量对应N个标签

- 句子中每个词的词性：I saw a saw -> N V DET N
- 社交网络中每个人的购买意向：甲 -> buy;乙 -> not

#### N个向量对应一个标签

- 情感分析：this is good -> positive
- 分子属性分析：一个分子图 -> 亲水性

#### N个向量对应多个标签（seq2seq）

- 机器翻译

## Fully-connected Network

使用全连接网络，设置窗口大小，每次输入邻近的多个词。但限制于窗口大小，无法考虑整个句子的影响，且窗口覆盖整个句子比较困难。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/fc.png" style="zoom:50%;" />

## Self-attention

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/self-attention.png" style="zoom:50%;" />

例：

- 输入：一排向量$\{a^1,a^2,a^3,a^4\}$
- 输出：一排向量$\{b^1,b^2,b^3,b^4\}$

*$b^1$、$b^2$、$b^3$、$b^4$分别都是考虑了$a^1$、$a^2$、$a^3$、$a^4$而产生的。*

### 如何生成$b^1$？

#### $a^1$与其他向量的相关性$\alpha$的计算方法

##### **Dot-product**

输入两个向量，分别与矩阵$W^q$和$W^k$相乘，再将得到的向量$q$和$k$做点乘。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/dot-product.png" style="zoom:50%;" />
$$
\begin{aligned}
q&=a^1\times W^q \\
k&=a^2\times W^k \\
\alpha &= q\cdot k
\end{aligned}
$$

##### Additive

输入两个向量，分别与矩阵$W^q$和$W^k$相乘，将得到的向量$q$和$k$串起来并通过激活函数，最后通过一个变换得到$alpha$。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/additive.png" style="zoom:50%;" />

#### $b^1$的计算

本节中的例子中，对于$b^1$，计算步骤如下：

- step1.计算$q^1$

$$
q^1=W^qa^1
$$

- step2.计算$k^1$、$k^2$、$k^3$、$k^4$

$$
k^i=W^ka^i
$$

- step3.计算$\alpha_{1,1}$、$\alpha_{1,2}$、$\alpha_{1,3}$、$\alpha_{1,4}$

$$
\alpha_{1,i}=q^1\cdot k^i \\
$$

- step4.通过Soft-max（也可使用ReLU等）计算$\alpha_{1,1}'$、$\alpha_{1,2}'$、$\alpha_{1,3}'$、$\alpha_{1,4}'$

$$
\alpha_{1,i}'=\frac{\exp(\alpha_{1,i})}{\sum_j\exp(\alpha_{1,j})}
$$

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/self-attention-1.png" style="zoom:50%;" />

- step5.计算$v^1$、$v^2$、$v^3$、$v^4$
  $$
  v^i=W^va^i
  $$

- step6.计算$b^1$（根据Attenion分数抽取重要信息）
  $$
  b^1=\sum_j\alpha_{1,j}'v^j
  $$

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/self-attention-2.png" style="zoom:50%;" />

### 输出向量组$\{b^1,b^2,b^3,b^4\}$的完整计算过程

整理以上过程，$Q$、$K$、$V$和Attention分数的计算过程如下：

#### $Q$、$K$、$V$的计算

由：
$$
\begin{aligned}
q^i &=W^qa^i \\
k^i &=W^ka^i \\
v^i &=W^va^i 
\end{aligned}
$$
有：
$$
\begin{aligned}
\begin{bmatrix}
q^1 & q^2 & q^3 & q^4
\end{bmatrix}
&=W^q
\begin{bmatrix}
a^1 & a^2 & a^3 & a^4
\end{bmatrix}
\\
\begin{bmatrix}
k^1 & k^2 & k^3 & k^4
\end{bmatrix}
&=W^k
\begin{bmatrix}
a^1 & a^2 & a^3 & a^4
\end{bmatrix}
\\
\begin{bmatrix}
v^1 & v^2 & v^3 & v^4
\end{bmatrix}
&=W^v
\begin{bmatrix}
a^1 & a^2 & a^3 & a^4
\end{bmatrix}
\end{aligned}
$$
即：
$$
\begin{aligned}
Q&=W^qI \\
K&=W^kI \\
V&=W^vI
\end{aligned}
$$
<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/self-attention-3.png" style="zoom:50%;" />

#### Attention分数的计算

由：
$$
\alpha_{1,i}=k^i\cdot q^1 \\
$$
有：
$$
\begin{bmatrix}
\alpha_{1,1} \\
\alpha_{1,2} \\
\alpha_{1,3} \\
\alpha_{1,4}
\end{bmatrix}
=
\begin{bmatrix}
k^1 \\
k^2 \\
k^3 \\
k^4 
\end{bmatrix}
\cdot
q^1
$$
进一步有：
$$
\begin{bmatrix}
\alpha_{1,1} & \alpha_{2,1} & \alpha_{3,1} &\alpha_{4,1} \\
\alpha_{1,2} & \alpha_{2,2} & \alpha_{3,2} &\alpha_{4,2} \\
\alpha_{1,3} & \alpha_{2,3} & \alpha_{3,3} &\alpha_{4,3} \\
\alpha_{1,4} & \alpha_{2,4} & \alpha_{3,4} &\alpha_{4,4} \\
\end{bmatrix}
=
\begin{bmatrix}
k^1 \\
k^2 \\
k^3 \\
k^4 
\end{bmatrix}
\cdot
\begin{bmatrix}
q^1 & q^2 & q^3 &q^4
\end{bmatrix}
$$
即：
$$
A=K^TQ
$$
对$A$进行Soft-max得到$A'$：
$$
A'=
\begin{bmatrix}
\alpha_{1,1}' & \alpha_{2,1}' & \alpha_{3,1}' &\alpha_{4,1}' \\
\alpha_{1,2}' & \alpha_{2,2}' & \alpha_{3,2}' &\alpha_{4,2}' \\
\alpha_{1,3}' & \alpha_{2,3}' & \alpha_{3,3}' &\alpha_{4,3}' \\
\alpha_{1,4}' & \alpha_{2,4}' & \alpha_{3,4}' &\alpha_{4,4}' \\
\end{bmatrix}
$$
<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/self-attention-4.png" style="zoom:50%;" />

最后将$V$与$A'$相乘，得到$b^1$、$b^2$、$b^3$、$b^4$：
$$
\begin{bmatrix}
b^1 & b^2 & b^3 & b^4
\end{bmatrix}
=
\begin{bmatrix}
v^1 & v^2 & v^3 & v^4
\end{bmatrix}
\cdot
\begin{bmatrix}
\alpha_{1,1}' & \alpha_{2,1}' & \alpha_{3,1}' &\alpha_{4,1}' \\
\alpha_{1,2}' & \alpha_{2,2}' & \alpha_{3,2}' &\alpha_{4,2}' \\
\alpha_{1,3}' & \alpha_{2,3}' & \alpha_{3,3}' &\alpha_{4,3}' \\
\alpha_{1,4}' & \alpha_{2,4}' & \alpha_{3,4}' &\alpha_{4,4}' \\
\end{bmatrix}
$$
即得到Self-attention的输出：
$$
O=VA'
$$

#### 总结

$$
\begin{aligned}
&Q=W^qI \\
&K=W^kI \\
&V=W^vI \\
&A=K^TQ \\
&A \rightarrow A' \\
&O=VA'
\end{aligned}
$$

$W^q$、$W^k$、$W^v$是需要学习的参数。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/self-attention-5.png" style="zoom:50%;" />

## Multi-head Self-attention

多个$q$，对应不同种类的相关性。

例如对于2 head的情况，$a^i$对应的$q^i$、$k^i$、$v^i$具体有$q^{i,1}$、$k^{i,1}$、$v^{i,1}$：
$$
\begin{aligned}
&q^{i,1}=W^{q,1}q^i \\
&q^{i,2}=W^{q,2}q^i
\end{aligned}
$$
类似的，$a^j$对应的$q^j$、$k^j$、$v^j$具体有$q^{j,1}$、$k^{j,1}$、$v^{j,1}$：

在使用Dot-product计算Attention分数时，使用对应的$q$和$k$进行计算，$q^{i,1}$分别和$k^{i,1}$、$k^{j,1}$进行计算，在将Attention分数分别和$v^{i,1}$、$v^{j,1}$计算，得到$b^{i,1}$，类似可得到$b^{i,2}$。
$$
b^i=W^O
\begin{bmatrix}
b^{i,1} \\
b^{i,2}
\end{bmatrix}
$$
<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/multi-head%20self-attention.png" style="zoom:50%;" />

## 位置编码

Self-attention中，对输入的几个向量所进行的操作是相同的，与位置无关，可能丢失了位置信息。

为每一个位置都设定一个向量$e^i$，将该向量加到$a^i$上。
$$
e^i+a^i\rightarrow q^i,k^i,v^i
$$
向量$e^i$可以通过一个规则设定（人工）或从训练数据中学习出来。

有各种不同的方法产生位置编码：

- Sinusoidal
- Position embedding
- FLOATER
- RNN
- ...

## Self-attention用于语音

把声音讯号表示为一排向量，一般一个向量只有10ms的长度，会导致向量个数过多，$A'$计算的复杂度是$O(L^2)$，一般使用**Truncated Self-attention**，不看一整句话，只看一小部分。

## Self-attention用于图像

一张图片可以看成是一排向量。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/self-attention%20for%20img.png" style="zoom:50%;" />

- Self-Attention GAN
- Detection Transformer(DETR)

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/self-attention%20for%20img2.png" style="zoom:50%;" />

## Self-attention v.s. CNN

Self-attention可以看作复杂化的CNN，Self-attention考虑全局。CNN是Self-attention的特例，Self-attention可以通过设定合适的参数，达到和CNN同样的效果。

- [[1911.03584\] On the Relationship between Self-Attention and Convolutional Layers (arxiv.org)](https://arxiv.org/abs/1911.03584)

## Self-attention v.s. RNN

RNN（Recurrent Neural Network）：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/4/self-attention%20vs%20RNN.png" style="zoom:50%;" />

RNN的缺点很明显，很难去考虑到输入位置较远的向量，并且无法并行计算。

- [[2006.16236\] Transformers are RNNs: Fast Autoregressive Transformers with Linear Attention (arxiv.org)](https://arxiv.org/abs/2006.16236)

## Self-attention用于图

图中每个结点可以表示为一个向量，边可以用来考虑结点之间的关联性，计算Attention分数时，只需计算相连的结点。

Self-attention用在图上面，是某一种类型的GNN（Graph Neural Network）。

## More

Self-Attention的计算量较大，优化效率是一个研究方向。

- [[2011.04006\] Long Range Arena: A Benchmark for Efficient Transformers (arxiv.org)](https://arxiv.org/abs/2011.04006)
- [[2009.06732\] Efficient Transformers: A Survey (arxiv.org)](https://arxiv.org/abs/2009.06732)
