# 李宏毅ML课程笔记——Transformer

[李宏毅2021/2022春机器学习课程——Transformer](https://www.bilibili.com/video/BV1Wv411h7kN?p=49)
<!--more-->

# Transformer

Transformer是一个Sequence-to-sequence（Seq2seq）的模型，输出的长度由模型自己来决定。

## Sequence-to-sequence

### 应用

- Maching Translation

- Speech Translation

- Text-to-Speech（TTS）


#### Questions Answering（QA）

大部分自然语言处理问题可以看作是QA（Question：这个句子的翻译是什么？；Context：一个句子；Answer：翻译结果）。
$$
\text{question}, \text{context} \stackrel{Seq2seq}{\longrightarrow} \text{answer}
$$

#### Multi-label Classification

$$
\text{data} \stackrel{Seq2seq}{\longrightarrow} \text{class 7, class 9, class 13}
$$

- [[1909.03434\] Order-free Learning Alleviating Exposure Bias in Multi-label Classification (arxiv.org)](https://arxiv.org/abs/1909.03434)
- [[1707.05495\] Order-Free RNN with Visual Attention for Multi-Label Classification (arxiv.org)](https://arxiv.org/abs/1707.05495)

#### Object Detection

- [[2005.12872\] End-to-End Object Detection with Transformers (arxiv.org)](https://arxiv.org/abs/2005.12872)

### 结构

$$
\text{Encoder}\longrightarrow \text{Decoder}
$$

- [[1409.3215\] Sequence to Sequence Learning with Neural Networks (arxiv.org)](https://arxiv.org/abs/1409.3215)
- [[1706.03762\] Attention Is All You Need (arxiv.org)](https://arxiv.org/abs/1706.03762)

## Encoder

- 输入一排向量：$\{x^1,x^2,x^3,x^4\}$
- 输出一排向量：$\{h^1,h^2,h^3,h^4\}$

Self-attention、RNN、CNN...均可用来作为Encoder。

### Transformer Encoder

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/seq2seq.png" style="zoom:50%;" />

在Transformer Encoder中，加入了Residual Connection，经过Self-attention输出的向量加上原输入的向量后当作新的输出向量。

得到Residual的结果以后，进行Normalization，但此处使用的是Layer Normalization而非Batch Normalization。对于输入的向量，Layer Norm会计算它的均值$m$和标准差$\sigma$，与Batch Norm不同点在于，Batch Norm是对不同特征、样本的同一个维度计算均值和标准差，而Layer Norm是对同一个特征、样本的不同维度去计算均值和标准差。

Layer Norm的结果将作为FC的输入，经过FC Network得到新的向量，在FC层也同样地加入了Residual Connection，得到的结果再做一次Layer Norm，则得到了此Block的输出。

即一个Block的结构如下：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/encoder.png" style="zoom:50%;" />

此Block会重复N次，组成Transformer的Encoder，BERT与Transformer Encoder采用了相同的结构。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/encoder2.png" style="zoom:50%;" />

### More

- [[2002.04745\] On Layer Normalization in the Transformer Architecture (arxiv.org)](https://arxiv.org/abs/2002.04745)

- [[2003.07845\] PowerNorm: Rethinking Batch Normalization in Transformers (arxiv.org)](https://arxiv.org/abs/2003.07845)

## Decoder——Autoregressive（AT）

### Decoder的运作方式

除了Encoder产生的输出以外，Decoder中还会加入一个BOS（Begin of Sequence）符号（token），用来表示开始，BOS token是一个One-hot表示的向量。

输出一个向量，向量的长度应该和Vocabulary Size相等来表示所有的汉字（对于中文，Vocabulary Size就是所有汉字的数量），每一个中文对应向量中的一个数值，这个向量是经过Soft-max得到的，取最大的作为输出的文字，Decoder会将这个输出的文字的One-hot向量作为新的输入。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/decoder.png" style="zoom:50%;" />

但是按照这样的运作方式，后面会产生无穷尽的文字，像文字接龙一样一直不能停下来，所以，还应该加一个EOS（End of Sequence） token来表示文字的结束，一般情况下，EOS token和BOS token都用一个相同的向量来表示，故向量的长度应该为Vocabulary Size + 1。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/decoder2.png" style="zoom:50%;" />

在这种运作方式下，某步的错误预测可能影响后面的预测（“一步错，步步错。”），具体参考最后一节[Scheduled Sampling](#Scheduled-Sampling)。

### Transformer Decoder

Transformer中的Decoder和Encoder结构类似，除中间的Multi-Head Attention和Add & Norm结构外，在第一次Multi-Head Attention计算中加入了Mask。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/encodervsdecoder.png" style="zoom:50%;" />

#### Masked Self-attention

在产生$b^1$的时候，只能考虑$a^1$的信息，而不能考虑$a^2$、$a^3$、$a^4$的信息；在产生$b^2$的时候，只能考虑$a^1$、$a^2$的信息，而不能考虑$a^3$、$a^4$的信息。

具体来说，在产生$b^2$时，只会拿第二个位置的query去跟第一个位置的key和第二个位置的key来计算Attention，而不管第三、四个位置的key。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/mask.png" style="zoom:50%;" />

对于Decoder而言，先有$a^1$才有$a^2$，才有接下来的$a^3$、$a^4$，计算$b^2$的时候无法考虑$a^3$、$a^4$。

## Decoder——Non-autoregressive（NAT）

### AT v.s. NAT

AT分别输入BOS token、$w_1$、$w_2$、$w_3$、EOS token，而NAT一次输入一整排BOS token。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/ATvsNAT.png" style="zoom:50%;" />

优点：

- 并行
- 输出长度可控

问题：模型如何知道输出的长度，从而确定输入的BOS token的数量？

- 用一个模型来预测输出长度；
- 输出一个很长的句子，忽略EOS token以后的内容。

NAT的表现往往比AT要差（Multi-modality）。

## Encoder-Decoder

Transformer中由**Cross Attention**模块来连接Encoder和Decoder。该模块接收Encoder的两个输出和Decoder的一个输出作为输入。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/cross%20attention.png" style="zoom:50%;" />

将BOS token输入到Decoder的Masked Self-attention模块后，将输出的向量进行线性变换得到query $q$，再将$q$与Encoder中的key $k^1$、$k^2$、$k^3$计算Attention分数，与value $v^1$、$v^2$、$v^3$相乘加权求和得到$v$。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/cross%20attention2.png" style="zoom:50%;" />

Cross Attention比Self-Attention出现要更早。

- [Listen, attend and spell: A neural network for large vocabulary conversational speech recognition | IEEE Conference Publication | IEEE Xplore](https://ieeexplore.ieee.org/document/7472621)

Transformer中，Decoder中每一层都与Encoder的最后一层做Cross Attention，也有论文的工作中尝试了与其他层的不同的连接方式。

- [[2005.08081\] Layer-Wise Multi-View Decoding for Improved Natural Language Generation (arxiv.org)](https://arxiv.org/abs/2005.08081)

## Training

以下以一段标签为“机器学习”的语音数据为例。

在Decoder中输入BOS token后，输出的向量应该和“机”对应的向量越接近越好，即通过计算两个向量的交叉熵，交叉熵越小越好，这个过程和分类非常相似。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/training.png" style="zoom:50%;" />

包括最后一个EOS token，模型希望最后一个字所输出的内容与EOS token的One-hot向量是接近的。

在训练过程中，Decoder的输入是真实标签，训练过程中会给Decoder看正确答案，即给Decoder输入BOS token和“机”以后，希望模型的输出是“器”，给Decoder输入BOS token、“机”和“器”之后，希望模型输出的是“学”。这种方法叫做**Teacher Forcing**，将正确答案作为输入。

训练时Decoder可以看到完全正确的信息，而测试的时候Decoder可能会看到一些错误的信息，可能会导致“一步错，步步错。”，训练与测试不一致的现象叫做**exposure bias**，方法是在学习时，给Decoder的输入加入一些错误的信息，具体参考最后一节[Scheduled Sampling](#Scheduled-Sampling)。

## Tips

### Copy Mechanism

某些信息并不需要机器来学习，可能是从输入信息中复制出来，例如聊天机器人中：

- eg1

  > User：你好，我是*库洛洛*。
  >
  > Machine：*库洛洛*你好，很高兴认识你。


- eg2

  > User：小杰*不能使用念能力了*！
  >
  > Machine：你所谓的*「不能使用念能力」*是什么意思？




又例如从文章中提取摘要这一任务，从文章中复制一些信息是很模型很关键的能力。

- [[1704.04368\] Get To The Point: Summarization with Pointer-Generator Networks (arxiv.org)](https://arxiv.org/abs/1704.04368)
- [[1603.06393\] Incorporating Copying Mechanism in Sequence-to-Sequence Learning (arxiv.org)](https://arxiv.org/abs/1603.06393)

### Guided Attention

在一些任务中（例如语音辨识、TTS等），对于输入的每一个内容都要看到，不能漏掉某些信息。

Guided Attention要求机器以特定的方式完成Attention的计算，应该由左向右分别产生输出。例如在TTS中，应该先看最左边的文字产生输出，最后看最右的文字产生输出。

- Monotonic Attention Location-aware attention

### Beam Search

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/5/beam%20search.png" style="zoom:50%;" />

要找到最优解，暴力搜索难以计算。通过Beam Search找一个不是完全精准的解。

- [[1904.09751\] The Curious Case of Neural Text Degeneration (arxiv.org)](https://arxiv.org/abs/1904.09751)

假设一个任务的答案非常明确，Beam Search会比较有帮助，但对于一些答案不唯一的任务（例如文本补全），分数最高的路径可能结果并不是很好，往往需要在Decoder中加入随机性（noise）。

## Optimizing Evaluation Metrics

训练时使用交叉熵，在评估时使用BLEU。BLEU不可微分，无法作为Loss。不过对于无法优化的Loss，可以将其当作Reinforcement Learning（RL）的reward，Decoder作为Agent，将其看作是RL问题来解决。

## Scheduled Sampling

Schedule可能会影响计算的并行化，对于Transformer的Scheduled Sampling另有方法。

- [[1506.03099\] Scheduled Sampling for Sequence Prediction with Recurrent Neural Networks (arxiv.org)](https://arxiv.org/abs/1506.03099)
- [[1906.07651\] Scheduled Sampling for Transformers (arxiv.org)](https://arxiv.org/abs/1906.07651)
- [[1906.04331\] Parallel Scheduled Sampling (arxiv.org)](https://arxiv.org/abs/1906.04331)
