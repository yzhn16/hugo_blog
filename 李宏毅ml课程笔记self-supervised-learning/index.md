# 李宏毅ML课程笔记——Self-Supervised Learning


[李宏毅2021/2022春机器学习课程——Self-Supervised Learning](https://www.bilibili.com/video/BV1Wv411h7kN?p=71)

<!--more-->

# 自监督式学习（Self-Supervised Learning）

## 芝麻街与进击的巨人

### 芝麻街

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-1/stavreal.png" style="zoom:50%;" />

### 进击的巨人：Bertolt Hoover

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-1/%E8%B4%9D%E7%89%B9%E9%9C%8D%E5%B0%94%E5%BE%B7%E8%83%A1%E4%BD%9B.png" style="zoom:50%;" />

### 主流模型参数量

|       Model        | Parameters |
| :----------------: | :--------: |
|        ELMO        |    94M     |
|        BERT        |    340M    |
|       GPT-2        |   1542M    |
|      Megatron      |     8B     |
|         T5         |    11B     |
|     Turing NLG     |    17B     |
|       GPT-3        |    175B    |
| Switch Transformer |    1.6T    |

## BERT

### Self-supervised Learning

对于数据$x$，监督学习需要知道数据的标签$\hat{y}$，来让模型输出我们想要的$y$。而自监督学习没有标注，将$x$分为两部分，一部分$x'$输入到模型得到$y$，另一部分$x''$作为标签，然后让$y$和$x''$越接近越好。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/self-supervised%20learning.png" style="zoom:50%;" />

自监督学习可以看作是一种无监督学习的方法，无监督学习的范围很大，里面有很多不同的方法，为了明确说明现在说做的工作，就称为自监督学习。

### Masked token prediction

- [[1810.04805\] BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding (arxiv.org)](https://arxiv.org/abs/1810.04805)

BERT的架构和Transfromer Encoder相同，输入一排向量，输出另一排向量，一般用在文字处理上。

输入一串token（token是处理一段文字的单位，在中文里一般把一个方块字当作一个token。），随机盖住一些token，盖住token有两种方法：

- 变为某个特殊的token
- 随机换为另一个token

对BERT的输出序列分别做线性变换（乘矩阵），再做Softmax就得到了一个分布。  

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/masking%20input.png" style="zoom:50%;" />

BERT不知道被盖住的部分是什么内容，但我们知道这部分内容，BERT学习的目标是输出和盖住的部分越接近越好。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/masking%20input2.png" style="zoom:50%;" />

### Next Sentence Prediction

从资料库中拿出两个句子，两个句子之间加入特殊的分隔符号[SEP]，再整个序号的最前面加[CLS]符号，整个序列输入BERT，看[CLS]对应的输出，[CLS]经过BERT的输出再经过线性变换后输出为Yes/No，代表这两个句子是不是相接的。

*例：[CLS] I like cat. [SEP] He likes dog*

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/next%20sentence%20prediction.png" style="zoom:50%;" />

- Next Sentence Prediction对于BERT接下来要做的事情可能无用

  - [[1907.11692\] RoBERTa: A Robustly Optimized BERT Pretraining Approach (arxiv.org)](https://arxiv.org/abs/1907.11692)

    Next Sentence Prediction这个任务可能比较简单，BERT可能学习不到太多有用的东西。

- **SOP**：Sentence order prediction Used in ALBERT

  - [[1909.11942\] ALBERT: A Lite BERT for Self-supervised Learning of Language Representations (arxiv.org)](https://arxiv.org/abs/1909.11942)

  两个句子本来就连在一起，人为拆分开，本来放在前面的句子作为Sentence 1，本来放在后面的句子作为Sentence 2，或本来放在前面的句子作为Sentence 2，本来放在后面的句子作为Sentence 1。然后让BERT去回答是哪一种顺序。

### Downstream Tasks

在训练BERT时，给了BERT两个任务：

- Masked token prediction
- Next sentence prediction

在训练BERT时，似乎仅仅在教BERT如何去做“填空题”，但BERT可以用在其他地方，BERT真正在下游任务（Downstream Tasks）中被使用，但需要少量有标注的数据。BERT经过微调（Fine-tune）可以去完成各种其他的任务。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/Fine-tune.png" style="zoom:50%;" />

### GLUE

任务集GLUE（General Language Understanding Evaluation）共有9个任务，

- Corpus of Linguistic Acceptability（CoLA）
- Stanford Sentiment Treebank（SST-2）
- Microsoft Research Paraphrase Corpus（MRPC）
- Quora Question Pairs（QQP）
- Semantic Textual Similarity Benchmark（STS-B）
- Multi-Genre Natural Language Inference（MNLI）
- Question-answering NLI（QNLI）
- Recognizing Textual Entailment（RTE）
- Winograd NLI（WNLI）

可以在这9个任务上分别微调模型得到9个模型，通过结果数值来判断模型的好坏。

中文版本的GLUE：

- [CLUE中文语言理解基准测评 (cluebenchmarks.com)](https://www.cluebenchmarks.com/)

### How to use BERT

#### Case 1

> - Input：sequence
> - Output：class
> - Example：Sentiment analysis

给BERT输入一个句子，前面放[CLS] token，对[CLS]输出的向量做线性变换，Softmax后输出class。需要提供大量的已标注的训练资料。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/case1.png" style="zoom:50%;" />

Linear部分是随机初始化，而BERT部分将学会了做“填空题”的BERT模型的参数拿来初始化。

#### Case 2

> - Input：sequence
> - Output：sequence
> - Example：POS tagging（词性标注）

给BERT输入一个句子，前面放[CLS] token，对句子里面每一个token输出的向量做线性变换，Softmax后输出每一个token的类别。需要提供已标注的训练资料。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/case2.png" style="zoom:50%;" />

#### Case 3

> - Input：two sequences
>
> - Output：a class
>
> - Example：Natural Language Inference（NLI）
>
>   前提：一个人骑马越过了一架坏掉的飞机，假设：这个人在一个小餐馆里面，输出：矛盾。
>
>   <img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/case3_eg.png" style="zoom:50%;" />

输入两个句子，两个句子之间放[SEP] token，第一个句子前放[CLS] token，整串内容输入BERT，对[CLS]输出的向量做线性变换，Softmax后输出class。需要提供已标注的训练资料。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/case3.png" style="zoom:50%;" />

#### Case 4

> - **Extraction-based Question Answering**（有限制的QA，答案一定能在文章中找到。）
>
> - Input：
>
>   - Document：$D=\{d_1,d_2,\cdots,d_N\}$
>
>   - Query：$Q=\{q_1,q_2,\cdots,q_M\}$
>
>   *对于中文，$d_i$和$q_i$都是汉字。*
>
> - Output：two integers$(s,e)$
>
>   - Answer：$A=\{d_s,\cdots,d_e\}$
>
>   *输出两个正整数，代表答案的范围。*
>
> <img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/case4_eg.png" style="zoom:50%;" />

输入问题和文章，问题和文章之间放[SEP] token，问题前放[CLS] token，整串内容输入BERT。

文章的各个token输出的向量先和一个向量（橙）做内积，再对结果做Softmax，得到答案起始的位置。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/case4_1.png" style="zoom:50%;" />

文章的各个token输出的向量再和一个向量（蓝）做内积，再对结果做Softmax，得到答案结束的位置。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/case4_2.png" style="zoom:50%;" />

以上只有和BERT输出做内积的两个向量是随机初始化的，即这两个向量是重头开始学习的。

### BERT Embryology（胚胎学）

- [[2010.02480\] Pretrained Language Model Embryology: The Birth of ALBERT (arxiv.org)](https://arxiv.org/abs/2010.02480)

BERT的训练需要耗费大量的资源，BERT的训练资料大概是30亿个词，是哈利波特全集的3000倍。有没有什么方法去节省计算资源？

从观察BERT的训练过程开始，BERT在什么时候学会填什么样的词汇？他的填空能力是怎么增进的？

### Pre-training a seq2seq model

BERT只有预训练的Encoder。

Encoder和Decoder间通过Cross Atention连接起来，在Encoder的输入中故意加一些扰动，希望Decoder输出的句子和弄坏前的句子是一样的。

#### MASS / BART

- [[1905.02450\] MASS: Masked Sequence to Sequence Pre-training for Language Generation (arxiv.org)](https://arxiv.org/abs/1905.02450)
- [[1910.13461\] BART: Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension (arxiv.org)](https://arxiv.org/abs/1910.13461)

对Encoder的输入加一些扰动来弄坏原本的内容：

- 盖住一些词
- 删掉一些词
- 打乱词顺序
- 词顺序旋转
- 混合

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/MASS_BART.png" style="zoom:50%;" />

#### T5 - Comparison

- Transfer Text-to-Text Transformer（T5）

T5在Colossal Clean Crawled Corpus（C4）进行训练，对比了多种弄坏的方法。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-2/T5.png" style="zoom:50%;" />

### Why does BERT work?

将文字输入到BERT中，得到的输出向量称为**embedding**，代表了各个token的意思，有相似意思的token有着非常相似的embeddng。 

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-3/embedding.png" style="zoom:50%;" />

在语言中常常有一词多义的情况，例如“吃苹果”的“果”和“苹果手机”的“果”的含义可能相差较大。通过Cosine Similarity计算“吃苹果”和“苹果手机”的Embedding的相似度，可以发现它们之间的相似度较低。

> "You shall know a word by the company it keeps."

一个词的意思可以从上下文看出来，BERT在做“填空题”的过程中所学习的内容也许就是根据上下文来预测当前被盖住的词汇。事实上，BERT之前已经有这样的方法：word embedding，word embedding中的CBOW就是把中间挖空然后预测内容。BERT所抽取出来的向量也叫做**Contextualized word embedding**。

现在尝试将BERT拿来做蛋白质分类、DNA分类、音乐分类。

- [[2103.07162\] Is BERT a Cross-Disciplinary Knowledge Learner? A Surprising Finding of Pre-trained Models' Transferability (arxiv.org)](https://arxiv.org/abs/2103.07162)

以DNA分类为例：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-3/DNA.png" style="zoom:50%;" />

将DNA中的序列替换成文字，输入到BERT中，输出分类，当作是文章分类的任务来处理。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-3/DNA_2.png" style="zoom:50%;" />

类似地，对于蛋白质，随意给各个氨基酸映射到词汇上，对于音乐，将各个音符映射到词汇上，得到了下面的结果：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-3/Protein_DNA_music_res.png" style="zoom:50%;" />

BERT的表现是比较好的，就算给BERT乱七八糟的句子，它可能也能把任务完成的比较好，这也说明BERT的表现可能并不完全来自于它“看得懂”文章这件事，关于BERT到底为什么好的问题可能还有很大的研究空间。

### More

- [[DLHLP 2020\] BERT and its family - Introduction and Fine-tune - YouTube](https://www.youtube.com/watch?v=1_gRK9EIQpc)
- [[DLHLP 2020\] BERT and its family - ELMo, BERT, GPT, XLNet, MASS, BART, UniLM, ELECTRA, and more - YouTube](https://www.youtube.com/watch?v=Bywo7m6ySlk)

### Multi-lingual BERT

在训练的时候，会拿各种各样的语言来给BERT做“填空题”，Multi-BERT使用了104种语言来训练。拿英文的QA资料去训练，Multi-BERT就会做中文的QA问题。

#### Zero-shot Reading Comprehension

在英文数据集SQuAD和中文数据集DRCD上：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-3/zero-shot.png" style="zoom:50%;" />

- [[1909.09587\] Zero-shot Reading Comprehension by Cross-lingual Transfer Learning with Multi-lingual Language Representation Model (arxiv.org)](https://arxiv.org/abs/1909.09587)

#### Cross-lingual Alignment

也许对Multi-lingual BERT来说，不同语言间没有什么差别。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-3/cross-lingual.png" style="zoom:50%;" />

#### Mean Reciprocal Rank（MRR）

MRR值越高，两个不同语言align的越好（同样意思但不同语言的词汇的向量比较接近）。

在1000k资料量下，相比200k资料量下的效果显著提升：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-3/mrr1.png" style="zoom:50%;" />

### 一个神奇的实验

BERT可以让同样意思但不同语言的词汇的向量很接近，但在训练Multi-lingual BERT时，还是给BERT喂中文，它能够做中文填空，喂英文能够做英文填空，不会混在一起，给他喂英文他并没有填中文进去。说明来自不同语言的符号终究还是不一样，并没有完全抹掉语言的资讯。

把所有英文的embedding平均起来，再把所有中文的embedding平均起来，两者相减得到的向量就是中文和英文之间的差距。给Multi-lingual BERT一句英文，得到一串embedding，将embedding加上相减得到的向量，这些向量对于Multi-lingual BERT就变成了中文的句子，再让BERT去做“填空题”，就填出了中文的答案。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-3/wired.png" style="zoom:50%;" />

语言的资讯还是藏在Multi-lingual BERT中：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-3/unsupervised%20learning.png" style="zoom:50%;" />

## GPT

### Predict Next Token

GPT修改了BERT中模型的任务，GPT的任务是预测接下来的句子是什么。

对于训练资料“台湾大学”，在最前面加上[BOS] token，对于[BOS] token，GPT输出一个embedding，接下来用这个embedding预测下一个应该出现的“台”这个token。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-4/predict%20next%20sentence.png" style="zoom:50%;" />

GPT与拿掉Cross attention后的Transformer Decoder结构类似。

GPT要预测下一个token，有生成的能力，GPT最知名的例子就是用GPT写了一篇关于独角兽的假新闻。

- [Demo – InferKit](https://app.inferkit.com/demo)

### How to use GPT?

GPT有一个更“狂“的使用方式，和人类更接近。

#### “Few-shot” Learning

例如在进行外语考试时，首先会看题目的说明（*“...从A、B、C、D四个选项中选出最佳选项...”*），再会看一个例子（*“...衬衫的价格是9镑15便士，所以你选择...”*）。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-4/few-shot%20learning.png" style="zoom:50%;" />

“Few-shot” Learning中完全没有Gradient Descent，GPT文献中将这种训练称为“In-context Learning”。

类似地，还有“One-shot” Learning、甚至“Zero-shot” Learning。

#### “One-shot” Learning

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-4/one-shot%20learning.png" style="zoom:50%;" />

#### “Zero-shot” Learning

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-4/zero-shot%20learning.png" style="zoom:50%;" />

第三代GPT测试了42个任务：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-4/GPT_benchmarks.png" style="zoom:50%;" />

### More

- [[DLHLP 2020\] 來自獵人暗黑大陸的模型 GPT-3 - YouTube](https://www.youtube.com/watch?v=DOG1L9lvsDY)

## Beyond Text

不止NLP，在语音、图像上都可以用Self-Supervised Learning的技术。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-4/beyond%20text.png" style="zoom:50%;" />

### Image - SimCLR

- [[2002.05709\] A Simple Framework for Contrastive Learning of Visual Representations (arxiv.org)](https://arxiv.org/abs/2002.05709)
- [google-research/simclr: SimCLRv2 - Big Self-Supervised Models are Strong Semi-Supervised Learners (github.com)](https://github.com/google-research/simclr)

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-4/SimCLR.png" style="zoom:50%;" />



### Image - BYOL

- [[2006.07733\] Bootstrap your own latent: A new approach to self-supervised Learning (arxiv.org)](https://arxiv.org/abs/2006.07733)

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-4/BYOL.png" style="zoom:50%;" />

### Speech

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-4/speech.png" style="zoom:50%;" />

### Speech GLUE - SUPERB

**S**peech processing **U**niversal **PER**formance **B**enchmark，包含了十多个下游任务，包含内容、说话的人、情感、语义等。

- Toolkit：[s3prl/s3prl: Self-Supervised Speech Pre-training and Representation Learning Toolkit. (github.com)](https://github.com/s3prl/s3prl/)

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/7/7-4/toolkit.png" style="zoom:50%;" />

- https://github.com/andi611/Self-Supervised-Speech-Pretraining-and-Representation-Learning
