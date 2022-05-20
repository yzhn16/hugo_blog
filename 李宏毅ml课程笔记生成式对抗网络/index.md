# 李宏毅ML课程笔记——Generative Adversarial Networks

[李宏毅2021/2022春机器学习课程——生成式对抗网络](https://www.bilibili.com/video/BV1Wv411h7kN?p=58)

<!--more-->

# 生成式对抗网络（GAN）

## Generator

### Network as Generator

输入$x$和一个简单的分布$z$（不固定，从一个分布中采样得到，每次使用网络时都会随机生成。），经过网络输出一个复杂的分布$y$。这样的网络称为**Generator**。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-1/generator.png" style="zoom:50%;" />

### Why distribution？

当任务需要一些“创造力”时（同样的输入有多种可能的输出），需要预测分布。

- eg1. *Video Prediction*

  输入：吃豆人游戏的历史帧序列

  输出：吃豆人游戏新一帧的内容（吃豆人可能向不同的方向移动）

- eg2. *Drawing*

  输入：“Character with red eyes”

  输出1：酷拉皮卡

  输出2：辉夜

- eg3. *Chatbot*

  输入：“你知道辉夜是谁吗？”

  输出1：“她是秀知院学生会...”

  输出2：“她开创了忍者时代...”

## Generative Adversarial Network（GAN）

- [hindupuravinash/the-gan-zoo: A list of all named GANs! (github.com)](https://github.com/hindupuravinash/the-gan-zoo)

### Anime Face Generation

**Unconditional generation**

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-1/unconditional.png" style="zoom:50%;" />

- [GAN学习指南：从原理入门到制作生成Demo - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/24767059)

### Discriminator

Discriminator本身也是一个网络，Discriminator拿一张图片作为输入，输出一个数值。数字越大，表示图片越接近真实的图片。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-1/discriminator.png" style="zoom:50%;" />

### Basic Idea of GAN

> 写作敌人，念做朋友。

二者关系好比Generator造假钞，Discriminator是抓造假钞的警察，Generator越来越像，Discriminator的辨别能力越来越强。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-1/basic%20idea.png" style="zoom:50%;" />

### Algorithm

首先初始化generator $G$和discriminator $D$，在每一次训练中：

- Step 1：定住$G$，更新$D$

  $D$学习去给真实二次元人物赋予更高的分数，而为生成的二次元人物赋予更低的分数。

  <img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-1/step1.png" style="zoom:50%;" />

  $D$分辨真正的二次元人物和生成的二次元人物之间的差异，可以当作一个分类或回归任务处理。

- Step 2：定住$D$，更新$G$

  $G$学习去“骗过”$D$，经过调整后，使得生成的图片能在$D$中产生更高的分数。

  <img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-1/step2.png" style="zoom:50%;" />

## Theory behind GAN

### Objective

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-2/objective.png" style="zoom:50%;" />

$$
G^\ast=\arg\min_G Div(P_G,P_{data})
$$

其中$Div(P_G,P_{data})$是$P_G$与$P_{data}$之间的散度（Divergence），可以看作是两个分布之间某种距离，散度越大，代表两个分布越不像，散度越小，代表两个分布越相近。c.f. $w^\ast,b^\ast=\arg\min_{w,b}L$

### Sampling

虽然不清楚$P_G$和$P_{data}$的分布，但是可以从其中采样来计算散度。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-2/sampling.png" style="zoom:50%;" />

### Discriminator

- [[1406.2661\] Generative Adversarial Networks (arxiv.org)](https://arxiv.org/abs/1406.2661)

Discriminator的训练目标是看到真实数据就给出一个高的分数，看到生成数据就给出一个低的分数。

- **Training**：$D^\ast=\arg\max_DV(D,G)$

- **Objective Function** for $D$：$V(G,D)=E_{y\sim P_{data}}[\log D(y)]+E_{y\sim P_G}[\log(1-D(y))]$

$V(D,G)$是交叉熵的相反数，Discriminator可以等同于一个分类器，最小化交叉熵。而正好$\max_DV(D,G)$就和**JS散度**有关。

最开始，$P_G$和$P_{data}$混在一起，散度很小，Discriminator难以分辨哪些数据是生成数据而哪些数据是真实数据，即Discriminator难以区分小的$max_D{V(D,G)}$。

若$P_G$和$P_{data}$散度很大，$max_DV(D,G)$比较大，DIscriminator则很容易区分生成数据和真实数据。

此时，有：
$$
G^\ast=\arg\min_G\max_DV(G,D)
$$

### Why JS Divergence？

除了JS散度，也可以使用例如KL散度等其他散度。如何设计目标函数，得到不同的散度，在f-GAN论文中有详细的证明。

- [[1606.00709\] f-GAN: Training Generative Neural Samplers using Variational Divergence Minimization (arxiv.org)](https://arxiv.org/abs/1606.00709)

## Tips for GAN

### JS散度的问题

在大多数情况下，$P_G$和$P_{data}$重复的部分非常少。

- 数据本身的特性：数据是高维空间中的低维流形，重叠的部分可以忽略。

  > 流形学习的观点认为，我们所能观察到的数据实际上是由一个低维流形映射到高维空间上的。由于数据内部特征的限制，一些高维中的数据会产生维度上的冗余，实际上只需要比较低的维度就能唯一地表示。例如单位圆上有无穷多个点，无法用二唯坐标系上的点来表示圆上所有的点，而若使用极坐标，圆心在原点的圆只需一个参数——半径，就可以确定。

  <img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-2/manifold.png" style="zoom:50%;" />

- 采样：即使$P_G$和$P_{data}$有重叠，若采样的点不够多，对Discriminator来说也是没有重叠的。

而以上的问题会导致JS散度出现问题。

在两个分布完全不重叠时，无论两个分布的中心距离有多近，其JS散度都是一个常数$\log2$，无法判断哪个case更好，从而无法更新参数。

参考证明：[JS散度(Jensen–Shannon divergence) - MorStar - 博客园 (cnblogs.com)](https://www.cnblogs.com/MorStar/p/14882813.html)

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-2/js%20div%20problem.png" style="zoom:50%;" />

直观来看，如果两个分布不重叠，二分类的准确率几乎可以达到100%。

### Wasserstein distance

考虑两个分布$P$和$Q$，想象一台推土机，$P$是一堆土，$Q$是要堆放的目的地，把$P$挪动到$Q$的平均距离就是Wasserstein distance。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-2/wasserstein%20distance.png" style="zoom:50%;" />

考虑更复杂的情况，移动的方案可能有无穷多种。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-2/wasserstein%20distance2.png" style="zoom:50%;" />

穷举所有的移动方法，看哪一个移动方法可以让平均的距离最小，最小的值就是Wasserstein distance。但计算方法似乎比较复杂，要计算距离还需要求解这样一个最优化问题。

假设现在我们已经可以计算Wasserstein distance，就可以解决JS散度带来的问题。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-2/wasserstein%20distance3.png" style="zoom:50%;" />

### WGAN

WGAN使用Wasserstein distance取代JS散度。

省略过程，要得到Wasserstein distance，只需解以下这个式子：
$$
\max_{D\in 1-Lipschitz}\{E_{y\sim P_{data}}[D(y)]-E_{y\sim P_G}[D(y)]\}
$$
$D\in 1-Lipschitz$：$D$需要是一个足够平滑的函数，不能是变动很剧烈的函数，若没有这个限制，单纯让生成数据越小越好，真实数据越大越好，在生成数据和真实数据没有重叠的情况下，会给真实数据无穷大的正值而给生成数据无穷小的负值。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-2/wasserstein%20distance4.png" style="zoom:50%;" />

### $D\in 1-Lipschitz$

- Original WGAN：Weight

  强制参数$w$在$c$和$-c$之间，参数更新后若$w\gt c$，则$w=c$，若$w\lt -c$，则$w=-c$。

- Improved WGAN：Gradient Penalty

  在真实数据和生成数据中各取一个样本，两点连线中再取一个样本，要求这个点的梯度接近1。

  <img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-2/gradient%20penalty.png" style="zoom:50%;" />

  - [[1704.00028\] Improved Training of Wasserstein GANs (arxiv.org)](https://arxiv.org/abs/1704.00028)

- Spectral Normalization（SNGAN）：让梯度模长在任何地方都小于1

  - [[1802.05957\] Spectral Normalization for Generative Adversarial Networks (arxiv.org)](https://arxiv.org/abs/1802.05957)
  

### More Tips

- Tops from Soumith
  - [soumith/ganhacks: starter from "How to Train a GAN?" at NIPS2016 (github.com)](https://github.com/soumith/ganhacks)
- Tips in DCGAN：Guideline for network architecture design for image generation
  - [[1511.06434\] Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks (arxiv.org)](https://arxiv.org/abs/1511.06434)
- Improved techniques for training GANs
  - [[1606.03498\] Improved Techniques for Training GANs (arxiv.org)](https://arxiv.org/abs/1606.03498)
- Tips from BigGAN
  - [[1809.11096\] Large Scale GAN Training for High Fidelity Natural Image Synthesis (arxiv.org)](https://arxiv.org/abs/1809.11096)

GAN的训练需要Generator和Discriminator共同配合，若有其中一方不再进步，另一方也会停下来，*Generator和Discriminator需要棋逢敌手*。

## GAN for Sequence Generation

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/GAN%20seq.png" style="zoom:50%;" />

Decoder参数改变后，经过max输出的文字可能不会发生改变，就无法完成参数更新。可以用RL来训练。

- [[1905.09922\] Training language GANs from Scratch (arxiv.org)](https://arxiv.org/abs/1905.09922)

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/scratch.png" style="zoom:50%;" />

## More Generative Models

- [GAN（Full version）](https://www.youtube.com/playlist?list=PLJV_el3uVTsMq6JEFPW35BCiOQTsoqwNw)
- [Variational Autoencoder（VAE）](https://youtu.be/8zomhgKrsmQ)
- [FLOW-based Model](https://youtu.be/uXY18nzdSsM)

##  Evaluation of Generation

### Quality of Image

评价图像质量最直接的做法是找人来看，在Generator研究初期，有人会选几张图说“看，这个结果应该比目前的结果都要好，应该是SOTA。”，这显然不够客观，如何自动地评价生成图像的质量？

一个方法是使用图像分类器，输入一张图片$y$，输出图片属于各个类的概率分布$p(c|y)$，分布越集中，产生的图片可能就越好。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/quality%20of%20img.png" style="zoom:50%;" />

### Diversity - Mode Collapse

仅使用以上这种方法评估图像质量时可能会遇到Mode Collapse（模式坍塌）的问题。

训练GAN的过程可能会遇到以下问题：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/mode%20collapse.png" style="zoom:50%;" />

Generator生成出来的图片可能来来去去都是那几张：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/mode%20collapse2.png" style="zoom:50%;" />

直觉上看，这样的点是Discriminator的“盲点”，Discriminator没办法看出这样的图片是假的，当Generator学会产生这种图片后，就永远都可以骗过Discriminator。

### Diversity - Mode Dropping

Mode Dropping可能比Mode Collapse更难侦测出来，产生出来的数据可能只能贴近已有真实数据的分布，但真实的数据分布的多样性其实是更大的。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/mode%20dropping.png" style="zoom:50%;" />

一个人脸生成的例子：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/mode%20dropping2.png" style="zoom:50%;" />

### Diversity

过去判定多样性的做法是将一批图片输入到图片分类器中，计算所有图片的概率分布的均值，若平均的分布非常集中，则代表多样性不够。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/diversity.png" style="zoom:50%;" />

若输入这批图片产生的分布都非常不同，平均后的结果非常平坦，则代表多样性是足够的。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/diversity2.png" style="zoom:50%;" />

Diversity和Quality的评估方式相反，Diversity看的是一批图片的平均，而Quality看的是一张图片。

**Inception Score（IS）**：质量越高，多样性越大，则IS越高。

### Fréchet Inception Distance（FID）

- [[1706.08500\] GANs Trained by a Two Time-Scale Update Rule Converge to a Local Nash Equilibrium (arxiv.org)](https://arxiv.org/abs/1706.08500)

取Softmax前的输出向量，假设真实图像和生成图像都是高斯分布，计算这两个高斯分布之间的Fréchet distance，这个距离越小说明真实图像与生成图像越接近，生成图像品质越高。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/FID.png" style="zoom:50%;" />

可能需要大量的图片样本才能做到。

- [[1711.10337\] Are GANs Created Equal? A Large-Scale Study (arxiv.org)](https://arxiv.org/abs/1711.10337)

### We don't want memory GAN

生成出来的图片有可能和训练集一模一样，这种情况下FID非常小，也有可能仅仅把图片翻转，这样很难侦测出来。

- [[1511.01844\] A note on the evaluation of generative models (arxiv.org)](https://arxiv.org/abs/1511.01844)

### More about evaluation

- [[1802.03446\] Pros and Cons of GAN Evaluation Measures (arxiv.org)](https://arxiv.org/abs/1802.03446)

## Conditional Generation

前面所提到的Unconditional GAN的输入都是一个随机的分布。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/conditional%20generation.png" style="zoom:50%;" />

例如要做文本转图片，需要给模型一个文本输入$x$。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/text-to-img.png" style="zoom:50%;" />

### Conditional GAN

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/conditional%20GAN.png" style="zoom:50%;" />

在Unconditional GAN中，Discriminator接受一个图片$y$作为输入，输出一个数值，代表图片是真实的或是生成的，但这样的方法无法解Conditional GAN的问题，Generator可以产生非常接近真实的图片，但是忽略了输入的条件。

Conditional GAN中，需要成对的训练数据。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-3/conditional%20GAN2.png" style="zoom:50%;" />

Conditional GAN也可以用图像来生成图像，例如图像去雾，黑白转彩色，白天转黑夜，素描转实物。也叫**Image translation**或**pix2pix**。

通常可以将GAN和有监督学习结合，得到更好的结果。

### 其他应用

- Sound-to-image
- Talking Head Generation
  - [[1905.08233\] Few-Shot Adversarial Learning of Realistic Neural Talking Head Models (arxiv.org)](https://arxiv.org/abs/1905.08233)


## Learning from Unpaired Data

有一堆$x$和一堆$y$，但$x$和$y$不成对（未标注）。pseudo labeling（伪标签）和back translation（反向翻译）都需要一些成对的数据。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-4/unpaired%20data.png" style="zoom:50%;" />

例如在影像风格转换，将定义域$\mathcal{X}$真人头像转为定义域$\mathcal{Y}$二次元头像：

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-4/image%20style%20transfer.png" style="zoom:50%;" />

### Cycle GAN

输入一个Domain $\mathcal{X}$，输出Domain $\mathcal{Y}$。但如果仍然按照之前的方法学习，GAN无法判断生成的二次元图像是否与输入的真人图像是相似的，可能会将输入当作为高斯噪声，忽略输入的内容。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-4/cycle%20gan.png" style="zoom:50%;" />

Cycle GAN中会训练两个Generator，第一个Generator $G_{\mathcal{X}\rightarrow \mathcal{Y}}$将$\mathcal{X}$ domain的图变成$\mathcal{Y}$ domain的图，第二个Generator $G_{\mathcal{Y}\rightarrow \mathcal{X}}$将$\mathcal{Y}$ domain的图还原为$\mathcal{X}$ domain的图。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-4/cycle%20gan2.png" style="zoom:50%;" />

Cycle GAN能保证真实图片和生成图片有一些关系，但如何保证这种关系是我们想要的呢？例如输入一个戴眼镜的人，$G_{\mathcal{X}\rightarrow \mathcal{Y}}$将眼镜转成痣，但$G_{\mathcal{Y}\rightarrow \mathcal{X}}$又会把痣转成眼镜。理论上可能会出现这样的情况，不过在实际中这种情况往往不会出现。

类似地还有Disco GAN和Dual GAN，思想与Cycle GAN基本相同。

- [[1703.05192\] Learning to Discover Cross-Domain Relations with Generative Adversarial Networks (arxiv.org)](https://arxiv.org/abs/1703.05192)
- [[1704.02510\] DualGAN: Unsupervised Dual Learning for Image-to-Image Translation (arxiv.org)](https://arxiv.org/abs/1704.02510)

此外还有能够在多种风格之间转换的Star GAN：

- [[1704.02510\] DualGAN: Unsupervised Dual Learning for Image-to-Image Translation (arxiv.org)](https://arxiv.org/abs/1704.02510)

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-4/starGAN.png" style="zoom:50%;" />

### SELFIE2ANIME

- [Selfie2Anime](https://selfie2anime.com/)

- [[1907.10830\] U-GAT-IT: Unsupervised Generative Attentional Networks with Adaptive Layer-Instance Normalization for Image-to-Image Translation (arxiv.org)](https://arxiv.org/abs/1907.10830)

### Text Style Transfer

文字风格转换，例如将负面的句子转为正面的句子。和Cycle GAN的做法类似。

<img src="https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%9D%8E%E5%AE%8F%E6%AF%85%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E7%A8%8B2021%E6%98%A5/6/6-4/text%20style%20transfer.png" style="zoom:50%;" />

### More

- Unsupervised Abstractive Summarization
  - [[1810.02851\] Learning to Encode Text as Human-Readable Summaries using Generative Adversarial Networks (arxiv.org)](https://arxiv.org/abs/1810.02851)
- Unsupervised Translation
  - [[1710.04087\] Word Translation Without Parallel Data (arxiv.org)](https://arxiv.org/abs/1710.04087)
  - [[1710.11041\] Unsupervised Neural Machine Translation (arxiv.org)](https://arxiv.org/abs/1710.11041)
- Unsupervised ASR
  - [[1804.00316\] Completely Unsupervised Phoneme Recognition by Adversarially Learning Mapping Relationships from Audio Embeddings (arxiv.org)](https://arxiv.org/abs/1804.00316)
  - [[1812.09323\] Unsupervised Speech Recognition via Segmental Empirical Output Distribution Matching (arxiv.org)](https://arxiv.org/abs/1812.09323)
  - [[1904.04100\] Completely Unsupervised Speech Recognition By A Generative Adversarial Network Harmonized With Iteratively Refined Hidden Markov Models (arxiv.org)](https://arxiv.org/abs/1904.04100)
