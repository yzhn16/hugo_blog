# 李宏毅ML课程笔记——各式各样的Attention

[李宏毅2021/2022春机器学习课程——各式各样的神奇的自注意力](https://www.bilibili.com/video/BV1Wv411h7kN?p=51)

<!--more-->

# Self-attention变型

Sequence length=$N$，产生的$N$个key向量和$N$个query向量两两之间做Dot-product，共$N^2$平方次计算，得到一个$N\times N$的矩阵Attention Matrix，根据该矩阵对value向量加权求和。Self-attention往往是模型里面的一个小模块，当$N$很大时，模型的主要计算量都集中在Self-attention上，对于计算速度的优化往往都是用在影像上。

## Human knowledge

### Local Attention / Truncated Attention

某些问题不用看完整的序列，只用看左右邻居的信息即可，将其他位置的信息设为0。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/local%20attention.png" style="zoom:50%;" />

可以加快运算速度，但每次做Attention只能看到某个小范围的信息，和CNN的差别就不大了。

### Stride Attention

与Local Attention类似，看更远的邻居，例如看三个位置之前和三个位置之后的信息。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/stride%20attention.png" style="zoom:50%;" />

### Global Attention

在原始序列中加入一些特殊的token，代表该位置要做Global Attention，Global Attention会从序列中的每一个token去收集信息。

- Attend to every token -> 收集所有的信息
- Attended by every token -> 能获取全局的信息

Global Attention有两种做法，可以从原始序列中选择一些已有的字符（例如BERT中的[CLS]标志、句号等）作为token，或外加额外的token。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/global%20attention.png" style="zoom:50%;" />

### Papers

- [[2004.05150\] Longformer: The Long-Document Transformer (arxiv.org)](https://arxiv.org/abs/2004.05150)
- [[2007.14062\] Big Bird: Transformers for Longer Sequences (arxiv.org)](https://arxiv.org/abs/2007.14062)

## Clustering

在Attention矩阵，可能有些值很大，有些值特别小，可以直接把较小的权值置0，问题在于如何快速估计哪些地方的Attention值较高，而哪些地方的Attention值较低。

### Reformer

- [Reformer: The Efficient Transformer | OpenReview](https://openreview.net/forum?id=rkgNKkHtvB)

- [[2003.05997\] Efficient Content-Based Sparse Attention with Routing Transformers (arxiv.org)](https://arxiv.org/abs/2003.05997)

#### 步骤

- Step 1：对query和key做聚类

  <img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/clustering1.png" style="zoom:50%;" />

  聚类有很多可以加速的方法，对query和key做聚类时，会采取精度相对较低但速度很快的方法。

- Step 2 对同一Cluster的query和key计算Attention分数

  <img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/clustering2.png" style="zoom:50%;" />

  不属于同一类的直接将Attention值设为0。

## Learnable Patterns

### Sinkhorn Sorting Network

- [[2002.11296\] Sparse Sinkhorn Attention (arxiv.org)](https://arxiv.org/abs/2002.11296)

让机器去学习两个位置的向量要不要做Attention。

Sinkhorn Sorting Network里面有一个额外需要学习的矩阵，来决定哪些地方需要计算Attention。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/sinkhorn%20sorting%20network.png" style="zoom:50%;" />

多个向量会共享一个矩阵以加快计算速度。（例如对于长度为100的输入，会分成10组，每组都是同一个矩阵。）

## Representative key

Attention矩阵中有很多冗余列，往往无需$N\times N$的Attention矩阵。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/Linformer.png" style="zoom:50%;" />

### Linformer

- [[2006.04768\] Linformer: Self-Attention with Linear Complexity (arxiv.org)](https://arxiv.org/abs/2006.04768)

从$N$个key中选出最有代表性的$K$个key，只需算$N\times K$的Attention矩阵。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/Linformer2.png" style="zoom:50%;" />

#### Reduce Number of Keys

**Linformer**中，对$N$的向量做**线性组合**：
$$
M_{d\times N}\times M_{N\times K}=M_{d\times K}
$$
<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/Linformer3.png" style="zoom:50%;" />

在**Compressed Attention**中的处理方式是对输入的较长序列用CNN去处理，得到一个较长的序列。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/compressed%20attention.png" style="zoom:50%;" />

- [[1801.10198\] Generating Wikipedia by Summarizing Long Sequences (arxiv.org)](https://arxiv.org/abs/1801.10198)

## $k,q$ first $\rightarrow$ $v,k$ first

### 忽略Softmax的情况

首先**不考虑Softmax的计算**，Attention的计算式为：
$$
O\approx VK^TQ
$$
<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/attention.png" style="zoom:50%;" />

调整计算顺序，有：
$$
O\approx V[K^TQ]\rightarrow O\approx [VK^T]Q
$$

- 对于计算方法$O\approx V[K^TQ]$：
  - $A=K^TQ$：$N\times d\times N$
  - $O=VA$：$d'\times N\times N$
  - 求和：$(d+d')N^2$
- 而对于计算方法$O\approx [VK^T]Q$：
  - $M_1=VK^T$：$d'\times N\times d$
  - $M_2=M_1Q$：$d'\times d\times N$
  - 求和：$2d'dN$

### 加回Softmax

已知存在一个$\phi$，使得有[以下式子](#实现)成立：

> $$
> \exp(q\cdot k)\approx \phi(q)\cdot\phi(k)
> $$

则对于原始的Self-attention计算，有：
$$
\begin{aligned}
b^1=\sum_{i=1}^Na'_{1,i}v^i&=\sum_{i=1}^N\frac{\exp{(q^1\cdot k^i)}}{\sum_{j=1}^{N}\exp{(q^1\cdot k^j)}}v^i \\
&=\sum_{i=1}^N\frac{\phi(q^1)\cdot\phi(k^i)}{\sum_{j=1}^{N}\phi(q^1)\cdot\phi(k^j)}v^i \\
&=\frac{\sum_{i=1}^{N}[\phi(q^1)\cdot\phi(k^i)]v^i}{\sum_{j=1}^{N}\phi(q^1)\cdot\phi(k^j)}
\end{aligned}
$$
其中，对于分母部分：
$$
\sum_{j=1}^{N}\phi(q^1)\cdot\phi(k^j)=\phi(q^1)\cdot\sum_{j=1}^{N}\phi(k^j)
$$
对于分子部分，由于：
$$
\phi(q^1)=
\begin{bmatrix}
q_1^1 \\
q_2^1 \\
\vdots
\end{bmatrix}
\quad
\phi(k^1)=
\begin{bmatrix}
k_1^1 \\
k_2^1 \\
\vdots
\end{bmatrix}
$$
则有：
$$
\begin{aligned}
&\quad\sum_{i=1}^N[\phi(q^1)\cdot \phi(k^i)]v^i \\
&=[\phi(q^1)\cdot\phi(k^1)]v^1+[\phi(q^1)\cdot\phi(k^2)]v^2+\cdots \\
&=(q_1^1k_1^1+q_2^1k_2^1+\cdots)v^1+(q_1^1k_1^2+q_2^1k_2^2+\cdots)v^2+\cdots \\
&=(q_1^1k_1^1v^1+q_2^1k_2^1v^1+\cdots)+(q_1^1k_1^2v^2+q_2^1k_2^2v^2+\cdots)+\cdots \\
&=q_1^1(k_1^1v^1+k_1^2v^2+\cdots)+q_2^1(k_2^1v^1+k_2^2v^2+\cdots)+\cdots \\

\end{aligned}
$$
设$\phi(q^1)$的维度为$M$，则：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/res1.png" style="zoom:50%;" />

即在分子中的$M$个向量中，每一个向量都是通过，拿出$\phi(k^1)$、$\phi(k^2)$、...、$\phi(k^N)$中的第$i$个分量，对$v^1$、$v^2$、...、$v^N$做加权和。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/res2.png" style="zoom:50%;" />

可以看出，每次计算$b^i$时，除了$\phi(q^i)$以外，其他部分没有发生变化，这部分内容无需**再重复计算**。

### Self-attention中的$q$、$k$、$v$

计算$b^1$：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/res3.png" style="zoom:50%;" />

产生的$M$个向量以及$\sum_{j=1}^{N}\phi(k^j)$在后面$b^2$、$b^3$、$b^4$的计算中无需再进行计算。

### 实现

关于$\phi$的实现：

- [[1812.01243\] Efficient Attention: Attention with Linear Complexities (arxiv.org)](https://arxiv.org/abs/1812.01243)
- [Linear Transformers (linear-transformers.com)](https://linear-transformers.com/)
- [[2103.02143\] Random Feature Attention (arxiv.org)](https://arxiv.org/abs/2103.02143)
- [[2009.14794\] Rethinking Attention with Performers (arxiv.org)](https://arxiv.org/abs/2009.14794)

## New framework

### 无需$q,k$产生Attention——Synthesizer

将Attention矩阵作为网络的参数。
$$
\begin{bmatrix}
\alpha_{1,1} & \alpha_{1,2} & \alpha_{1,3} & \alpha_{1,4} \\
\alpha_{1,2} & \alpha_{2,2} & \alpha_{2,3} & \alpha_{2,4} \\
\alpha_{1,3} & \alpha_{2,3} & \alpha_{3,3} & \alpha_{3,4} \\
\alpha_{1,4} & \alpha_{2,4} & \alpha_{3,4} & \alpha_{4,4} \\
\end{bmatrix}
$$

## Attention-free

- [[2105.03824\] FNet: Mixing Tokens with Fourier Transforms (arxiv.org)](https://arxiv.org/abs/2105.03824)
- [[2105.08050\] Pay Attention to MLPs (arxiv.org)](https://arxiv.org/abs/2105.08050)
- [[2105.01601\] MLP-Mixer: An all-MLP Architecture for Vision (arxiv.org)](https://arxiv.org/abs/2105.01601)

## 总结

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/summary.png" style="zoom:50%;" />
