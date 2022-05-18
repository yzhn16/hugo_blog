# 统计学习——感知机

李航《统计学习方法》第2章
<!--more-->

# 第2章 感知机

感知机是二类分类的线性分类模型。

感知机学习旨在求出将训练数据进行线性划分的分离超平面，为此，导入基于误分类的损失函数，利用梯度下降法对损失函数进行极小化，求得感知机模型。

## 感知机模型

> **定义**：假设输入空间是$\mathcal{X}\subseteq \mathbf{R}^n$，输出空间是$\mathcal{Y}=\{+1,-1\}$。输入$x\in \mathcal{X}$表示实例的特征向量，对应于输入空间的点；输出$y\in\mathcal{Y}$表示实例的类别。感知机是由输入空间到输出空间的如下函数：
> $$
> f(x)=\text{sign}(w\cdot x+b)
> $$
> 其中$w$和$b$是参数，$w\in \mathbf{R}^n$叫做权值（weight），$b\in \mathbf{R}$叫做偏置（bias）。$\text{sign}$是符号函数。

线性方程$w\cdot x+b=0$对应于特征空间$\mathbf{R}^n$中的一个超平面$S$，这个超平面将特征空间划分为正、负两类。

## 感知机学习策略

假设训练数据集是**线性可分**的，感知机的学习目标是求得一个能够将训练集正、负实例点**完全正确分开**的分离超平面。

为了便于优化，损失函数的选择是**误分类点到超平面$S$的总距离**：
$$
\frac{1}{\|{w}\|}|w\cdot x_0+b|
$$
其中，$\|w\|$是$w$的$L_2$范数。

对于**误分类点**有$-y_i(w\cdot x_i+b)\gt 0$，则所有误分类点到超平面$S$的总距离为
$$
-\frac{1}{\|w\|}\sum_{x_i\in M}y_i(w\cdot x_i+b)
$$
其中$M$为误分类点的集合，不考虑$\frac{1}{\|w\|}$即得到感知机学习的损失函数
$$
L(w,b)=-\sum_{x_i\in M}y_i(w\cdot x_i+b)
$$

## 感知机学习算法

### 感知机学习算法的原始形式

求参数$w,b$，使其为损失函数极小化问题的解
$$
\min_{w,b}L(w,b)=-\sum_{x_i\in M}y_i(w\cdot x_i+b)
$$
感知机学习算法采用**随机梯度下降法**（Stochastic GD），每次**随机选取一个**误分类点使其梯度下降。

损失函数的梯度由下式给出：
$$
\begin{aligned}
\nabla_wL(w,b)&=-\sum_{x_i\in M}y_ix_i \\
\nabla_bL(w,b)&=-\sum_{x_i\in M}y_i
\end{aligned}
$$
随机选取一个误分类点$(x_i,y_i)$对$w,b$进行更新（沿着梯度的**反**方向）：
$$
\begin{aligned}
w&\leftarrow w+\eta y_ix_i \\
b&\leftarrow b+\eta y_i
\end{aligned}
$$

> **算法（感知机学习算法的原始形式）**:
>
> 输入：训练数据集$T=\{(x_1,y_1),(x_2,y_2),\cdots,(x_N,y_N)\}$，其中$x_i\in \mathcal{X}=\mathbf{R}^n$，$y_i\in \mathcal{Y}=\{-1,+1\}$，$i=1,2,\cdots,N$；学习率$\eta(0\lt \eta\leqslant 1)$；
>
> 输出：$w,b$；感知机模型$f(x)=\text{sign}(w\cdot x+b)$。
>
> （1）选取初值$w_0,b_0$；
>
> （2）在训练集中选取数据$(x_i,y_i)$；
>
> （3）如果$y_i(w\cdot x_i+b)\leqslant 0$，
> $$
> \begin{aligned}
> w&\leftarrow w+\eta y_ix_i \\
> b&\leftarrow b+\eta y_i
> \end{aligned}
> $$
> （4）转至（2），直至训练集中没有误分类点。

### 算法的收敛性

现在证明，对于线性可分数据集，感知机学习算法原始形式收敛，即经过有限次迭代可以得到一个将训练数据集完全正确划分的分离超平面及感知机模型。

将偏置$b$并入权重向量$w$，记作$\hat{w}=(w^T,b)^T$，同样也将输入向量$x$加以扩充，记作$\hat{x}=(x^T,1)^T$，这样$\hat{x}\in \mathbf{R}^{n+1}$，$\hat{w}\in \mathbf{R}^{n+1}$，且$\hat{w}\cdot\hat{x}=w\cdot x+b$。

> **定理（Novikoff）**：设训练数据集$T=\{(x_1,y_1),(x_2,y_2),\cdots,(x_N,y_N)\}$是线性可分的，其中$x_i\in \mathcal{X}=\mathbf{R}^n$，$y_i\in \mathcal{Y}=\{-1,+1\}$，$i=1,2,\cdots,N$，则：
>
> （1）存在满足条件$\|\hat{w}_{opt}\|=1$的超平面$\hat{w}_{opt}\cdot\hat{x}=w_{opt}\cdot x+b_{opt}=0$将训练数据集完全正确分开；且存在$\gamma\gt 0$，对所有的$i=1,2,\cdots,N$
> $$
> y_i(\hat{w}_{opt}\cdot\hat{x}_i)\geqslant\gamma
> $$
> （2）令$R=\max_{1\le i\le N}\|\hat{x}_i\|$，则感知机学习算法的原始形式在训练数据集上的误分类次数$k$满足不等式
> $$
> k=\left(\frac{R}{\gamma}\right)^2
> $$

**证明**：

（1）由于训练数据集线性可分，故存在超平面可将训练数据集完全正确分开，取此超平面为$\hat{w}_{opt}\cdot\hat{x}=0$，使$\|\hat{w}_{opt}\|=1$。由于对于有限的$i=1,2,\cdots,N$，均有
$$
y_i(\hat{w}_{opt}\cdot\hat{x}_i)\gt 0
$$
所以存在
$$
\gamma=\min_i\{y_i(\hat{w}_{opt}\cdot\hat{x}_i)\}
$$
使
$$
y_i(\hat{w}_{opt}\cdot\hat{x}_i)\geqslant\gamma
$$
（2）感知机算法从$\hat{w}_0$开始，如果实例被误分类，则更新权重。令$\hat{w}_{k-1}$是第$k$个误分类实例之前的扩充向量权重，即
$$
\hat{w}_{k-1}=(w_{k-1}^T,b_{k-1})^T
$$
则第$k$个误分类实例的条件是
$$
y_i(\hat{w}_{k-1}\cdot\hat{x}_i)\leqslant 0
$$
若$(x_i,y_i)$是被$\hat{w}_{k-1}=(w_{k-1}^T,b_{k-1})^T$误分类的数据，则$w$和$b$的更新是
$$
\begin{aligned}
w_k&\leftarrow w_{k-1}+\eta y_ix_i \\
b_k&\leftarrow b_{k-1}+\eta y_i
\end{aligned}
$$
即
$$
\hat{w}_k=\hat{w}_{k-1}+\eta y_i\hat{x}_i
$$
下面推导两个不等式（1）和（2）：
$$
\hat{w}_k\cdot\hat{w}_{opt}\geqslant k\eta\gamma  \tag{1}
$$
由式$\hat{w}_k=\hat{w}_{k-1}+\eta y_i\hat{x}_i$及式$y_i(\hat{w}_{opt}\cdot\hat{x}_i)\geqslant\gamma$得
$$
\begin{aligned}
\hat{w}_k\cdot\hat{w}_{opt} &= \hat{w}_{k-1}\cdot\hat{w}_{opt}+\eta y_i\hat{w}_{opt}\cdot\hat{x}_i \\
&\geqslant \hat{w}_{k-1}\cdot\hat{w}_{opt}+\eta\gamma
\end{aligned}
$$

递推即得不等式（2）
$$
\hat{w}_k\cdot\hat{w}_{opt}\geqslant \hat{w}_{k-1}\cdot\hat{w}_{opt}+\eta\gamma \geqslant \hat{w}_{k-2}\cdot\hat{w}_{opt}+2\eta\gamma\geqslant \cdots\geqslant k\eta\gamma
$$

$$
\|\hat{w}_k\|^2\leqslant k\eta^2R^2\tag{2}
$$

由式$\hat{w}_k=\hat{w}_{k-1}+\eta y_i\hat{x}_i$及式$y_i(\hat{w}_{k-1}\cdot\hat{x}_i)\leqslant0 $得
$$
\begin{aligned}
\|\hat{w}_k\|^2&=\|\hat{w}_{k-1}\|^2+2\eta y_i\hat{w}_{k-1}\cdot\hat{x}_i+\eta^2\|\hat{x}_i\|^2 \\
&\leqslant \|\hat{w}_{k-1}\|^2+\eta^2\|\hat{x}_i\|^2 \\
&\leqslant \|\hat{w}_{k-1}\|^2+\eta^2R^2 \\
&\leqslant \|\hat{w}_{k-2}\|^2+2\eta^2R^2 \leqslant \cdots \\
&\leqslant k\eta^2R^2
\end{aligned}
$$
结合式$\hat{w}_k\cdot\hat{w}_{opt}\geqslant k\eta\gamma$及式$\|\hat{w}_k\|^2\leqslant k\eta^2R^2$即得
$$
k\eta\gamma \leqslant\hat{w}_k\cdot\hat{w}_{opt}\leqslant\|\hat{w}_k\|\|\hat{w}_{opt}\|\leqslant \sqrt{k}\eta R
$$
则有
$$
k^2\gamma^2\leqslant kR^2
$$
于是
$$
k\leqslant \left(\frac{R}{\gamma}\right)^2
$$


### **感知机学习算法的对偶形式**

对偶形式的基本想法是，将$w$和$b$表示为实例$x_i$和标记$y_i$的线性组合的形式，通过求解其系数而求得$w$和$b$。

在感知机学习算法的原始形式中，假设初始值$w_0,b_0$均为0。假设样本点$(x_i,y_i)$在更新过程中被使用了$n_i$次，最后学习到的$w,b$可以分别表示为
$$
\begin{aligned}
w&=\sum_{i=1}^Nn_i\eta y_ix_i \\
b&=\sum_{i=1}^Nn_i\eta y_i
\end{aligned}
$$
考虑$n_i$，如果$n_i$的值很大，说明这个样本点经常被误分类，这意味着它离超平面距离很近，超平面稍微移动，这个点就可能被误分类，并且这样的点很可能就是*支持向量*。

将$w,b$代入感知机学习算法的原始形式：
$$
f(x)=\text{sign}(w\cdot x+b)=\text{sign}\left(\sum_{j=1}^Nn_j\eta y_jx_j\cdot x+\sum_{j=1}^Nn_j\eta y_i\right)
$$
此时的学习目标从$w,b$变为$n_i$。

> **算法（感知机学习算法的对偶形式）**：
>
> 输入：训练数据集$T=\{(x_1,y_1),(x_2,y_2),\cdots,(x_N,y_N)\}$，其中$x_i\in \mathcal{X}=\mathbf{R}^n$，$y_i\in \mathcal{Y}=\{-1,+1\}$，$i=1,2,\cdots,N$；学习率$\eta(0\lt \eta\leqslant 1)$；
>
> 输出：$n$；感知机模型$f(x)=\text{sign}\left(\sum_{j=1}^Nn_j\eta y_jx_j\cdot x+\sum_{j=1}^Nn_j\eta y_i\right)$，其中$n=(n_1,n_2,\cdots,n_N)^T$。
>
> （1）$n\leftarrow 0$；
>
> （2）在训练集中选取数据$(x_i,y_i)$；
>
> （3）如果$y_i\left(\sum_{j=1}^Nn_j\eta y_jx_j\cdot x_i+\sum_{j=1}^Nn_j\eta y_i\right)\leqslant0$，更新参数，将误分类次数加一：
> $$
> n_i\leftarrow n_i+1
> $$
> （4）转至（2）直至没有误分类数据。

对偶形式中训练实例仅以内积的形式出现，可以预先将训练集中实例间的内积计算出来存在矩阵中，此矩阵为Gram矩阵
$$
\symbfit{G}=[x_i\cdot x_j]_{N\times N}
$$

