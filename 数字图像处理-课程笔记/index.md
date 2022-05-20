# 数字图像处理 课程笔记

参考视频是广东海洋大学的课程录像，参考书目为冈萨雷斯等著，阮秋琦等译的《数字图像处理》（第四版）。
<!--more-->


# 图像的空域增强技术

## 概述

### 空域的概念

- **空域**：像素组成的空间。
- **空域增强技术**：直接作用于图像像素的增强技术。

$$
像素的空间坐标(x,y) \rightarrow 像素的灰度值f(x,y)
$$

### 空域增强的模型

$$
g(x,y)=E_H[f(x,y)]
$$

### 分类

#### 基于像素的空域增强

$E_H$定义在每个**像素**$(x,y)$上。

- 像素点操作：$g(x,y)=P_{xy}[f(x,y)]$

- 几何操作：$(x',y')=M(x,y)$

#### 基于模板的空域增强

$E_H$定义在像素$(x,y)$的某个**邻域**上。
$$
t=E_H[s,n(s)]
$$

## 图像间运算

### 算数运算

两幅图像对应位置像素$p、q$：$p+q、p-q、p\times q、p\div q$

### 应用

- 图像去噪

  对原始图像$f(x,y)$和随机噪声$e(x,y)$：
  $$
  \begin{aligned}
  g(x,y)&=f(x,y)+e(x,y) \\
  \overline{g}(x,y)&=\frac{1}{M} \sum_{i=1}^{M}g_i(x,y)
  \end{aligned}
  $$

- 医学图像的数字减影  

- 图像局部显示

  二值模板图像与原图像做**乘法**，进行图像的局部显示。

## 直接灰度映射

### 原理

$$
灰度值\xrightarrow{f}另一灰度值
$$

### 典型灰度映射

#### 图像求反

斜率为-1的线性灰度变换：
$$
t=(L-1)-s
$$

#### 对比度增强

基于像素的图像增强，即增强原图的各部分反差。

#### 分段线性增强

拉伸感兴趣的图像细节的灰度级。

eg.对于$0\sim 255$的灰度取值范围，划分为$0\sim 100、101\sim 200、201\sim 255$三个取值区间，只改变其中某段进行灰度变换。

分段线性增强的典型变换函数是三段线性变换函数：
$$
\large
t =
\begin{cases} 
\frac{t_1}{s_1}s,  & 0\le s \le s_1\\
\frac{t_2-t_1}{s_2-s_1}[s-s_1]+t_1, &s_1 \lt s \le s_2 \\
\frac{L-1-t_2}{L-1-s_2}[s-s_2]+t_2, &s2 \lt s \le L-1
\end{cases}
$$

经过**斜率小于1**的线性变换函数后：

- 压缩了灰度的动态范围
- 对比度下降

经过**斜率大于1**的线性变换函数后，反之。

#### 对数变换

原图动态范围太大，超出某些设备所允许的动态范围，需要压缩其动态范围，即$0 \sim L'(\gt L-1) \longrightarrow 0 \sim L-1$。

常采用对数变换实现动态范围的压缩：
$$
t=C\log(1+|s|)
$$
其中$C$为尺度比例常数。

可以使用对数变换来扩展图像中暗像素的值，同时压缩高灰度级的值。

#### 幂律（伽马）变换

$$
t=c\times s^\gamma
$$

其中$c$和$\gamma$为正常数。

对于$\gamma (\gamma \lt 1)$的幂律曲线，将较窄范围的暗色输入值映射为较宽范围的亮色输入值（*与对数变换类似*）；同时，将较宽范围的亮色输入值映射为较窄范围的输出值，$\gamma \gt$ 1的幂律变换与之效果完全相反。

可以采用幂律变换提升图像细节的质量。

#### 灰度切割（灰度级分层）

增强特定范围的对比度，用来突出图像中**特定灰度范围**的亮度。

- 方法一
  $$
  t =
  \begin{cases} 
  t_2,  & s_1\le s \le s_2\\
  t_1,  & \text{其他}
  \end{cases}
  $$

- 方法二
  $$
  t =
  \begin{cases} 
  t_2,  & s_1\le s \le s_2\\
  s,  & \text{其他}
  \end{cases}
  $$

#### 阈值化处理（二值化处理，灰度切割的特例）

$$
t =
\begin{cases} 
0,  & s\lt s_1\\
L-1,  & s\ge s_1
\end{cases}
$$

最终产生一个**黑白图像**。

#### 位图切割

$8$比特表示的图像看作$8$个单独的$1$比特平面（位图）组成，位面0表示最低位面，位面7表示最高位面。

每个位面均为**二值图像**，位面图像中像素的灰度值等于相应有效位的取值。

可实现以下应用：

- 操作特定位面增强图像
- 确定用于量化该图像的比特数的充分性
- 图像压缩

##### 实现方法

图像各像素的灰度值除以各有效位的权值$2^i$（$i$为有效位的序数，从$0$计数），商的整数部分为**奇数**，则该灰度值在相应位面中映射为1，若为**偶数**，则映射为$0$。（*可类比十进制转二进制的手算方法*）

例如8位灰度图，某一像素点灰度值121，则：
$$
\begin{aligned}
floor(121/2^7)=0 &\quad\Rightarrow\quad 位面\ 7\ 中取值为\ 0 \\
floor(121/2^6)=1 &\quad\Rightarrow\quad 位面\ 6\ 中取值为\ 1 \\
floor(121/2^5)=3 &\quad\Rightarrow\quad 位面\ 5\ 中取值为\ 1 \\
floor(121/2^4)=7 &\quad\Rightarrow\quad 位面\ 4\ 中取值为\ 1 \\
floor(121/2^3)=15 &\quad\Rightarrow\quad 位面\ 3\ 中取值为\ 1 \\
floor(121/2^2)=30 &\quad\Rightarrow\quad 位面\ 2\ 中取值为\ 0 \\
floor(121/2^1)=60 &\quad\Rightarrow\quad 位面\ 1\ 中取值为\ 0 \\
floor(121/2^0)=121 &\quad\Rightarrow\quad 位面\ 0\ 中取值为\ 1
\end{aligned}
$$

##### MATLAB实现代码

```matlab
im=imread('fractal_iris.bmp');
rowcnt=size(im,1);
columncnt=size(im,2);
subplot(3,3,1);imshow(im);title('源图像');
for i=7:-1:0
	for x=1:rowcnt
		for y=1:columncnt
			if mod(floor(double(im(x,y))/(2^i)),2)==0
				bitmap(x,y)=0;
			else
				bitmap(x,y)=1;
			end
		end
	end
	subplot(3,3,7-i+2);imshow(bitmap);title(strcat('位平面',num2str(i)));
end
```



## 直方图修正——直方图均衡化

### 直方图和累计直方图

#### 直方图

$$
\begin{aligned}
h(k)=n_k \quad k=0,1,\cdots,L-1
\end{aligned}
$$

其中，$n_k$是图像$f(x,y)$中具有灰度值$k$的像素的个数。

是图像的一种统计表达，反映了图像中像素的灰度值的分布情况。

若某图像的灰度直方图具有**二峰性**，则表明这个图像较亮区域和较暗区域可以较好的分离。

#### 归一化直方图

$$
\begin{aligned}
p(s_k)&=\frac{n_k}{n} \quad s_k=\frac{k}{L-1},0\le s_k\le1
\end{aligned}
$$

其中，$n$为图像所有像素的数量。

#### 累计直方图

$$
H(k)=\sum_{i=0}^{k}n_i
$$

其中，$n_i$表示图像中灰度级等于$i$的像素点数量。

#### 归一化累计直方图

$$
\begin{aligned}
P(s_k)=\sum_{i=0}^kp(s_i)
\end{aligned}
$$

### 直方图均衡化原理

把图像的直方图变换为**均匀分布**的形式，以此增强动态范围偏小的图像的反差，从而实现**对比度增强**。

实质是选用合适的变换函数来**修正图像灰度级的归一化直方图**$p(s_k)$，为了能从图像中获得尽量多的信息量（图像熵尽可能大），要求$p(s_k)$为**常数**。

增强函数$E_H(s)$需要满足：

- $E_H(s)$为单值单增函数（*保持原有排列次序*）
- $0\le E_H(s) \le L-1$（*灰度级动态范围一致*）

反变换也应该满足上述条件。

**累积分布函数（CDF）**满足以上条件：
$$
t_k=E_H(s_k)=\sum_{i=0}^{k}\frac{n_i}{n}=\sum_{i=0}^{k}p(s_i)
$$
例如一幅图像$64\times 64(n=4096)$，每个像素点用3比特表示（8个灰度级），像素点的灰度值分布如下：
$$
\begin{array}{c|cccccccc}
\text{灰度级}& 0 & 1 & 2 & 3 & 4 & 5 & 6 & 7  \\
\hline 
\text{像素数量} & 790 & 1023 & 850 & 656 & 329 & 245 & 122 & 81
\end{array}
$$
实现**直方图均衡化**步骤如下：

1. 计算源图像的**归一化直方图**；
   $$
   \begin{array}{c|cccccccc}
   灰度级k & 0 & 1 & 2 & 3 & 4 & 5 & 6 & 7  \\
   \hline 
   归一化灰度级s_k & \frac{0}{7} & \frac{1}{7} & \frac{2}{7} & \frac{3}{7} & \frac{4}{7} & \frac{5}{7} & \frac{6}{7} & \frac{7}{7}  \\
   \hline 
   像素数量n_k & 790 & 1023 & 850 & 656 & 329 & 245 & 122 & 81 \\
   \hline
   \textbf{归一化直方图}p(s_k) & 0.19 & 0.25 & 0.21 & 0.16 & 0.08 & 0.06 & 0.03 & 0.02 
   \end{array}
   $$

2. 计算源图像的**归一化累积直方图**；
   $$
   \begin{array}{c|cccccccc}
   灰度级k & 0 & 1 & 2 & 3 & 4 & 5 & 6 & 7  \\
   \hline 
   归一化灰度级s_k & \frac{0}{7} & \frac{1}{7} & \frac{2}{7} & \frac{3}{7} & \frac{4}{7} & \frac{5}{7} & \frac{6}{7} & \frac{7}{7}  \\
   \hline 
   像素数量n_k & 790 & 1023 & 850 & 656 & 329 & 245 & 122 & 81 \\
   \hline
   归一化直方图p(s_k) & 0.19 & 0.25 & 0.21 & 0.16 & 0.08 & 0.06 & 0.03 & 0.02  \\
   \hline
   \textbf{归一化累积直方图}t_k & 0.19 & 0.44 & 0.65 & 0.81 & 0.89 & 0.95 & 0.98 & 1.00 
   \end{array}
   $$

3. $t_k'=int[(L-1)t_k+0.5]$（小数部分四舍五入），将$t_k$**扩展**到$[0, L-1]$范围内，得到直方图均衡化后的灰度值。
   $$
   \begin{array}{c|cccccccc}
   灰度级k & 0 & 1 & 2 & 3 & 4 & 5 & 6 & 7  \\
   \hline 
   归一化灰度级s_k & \frac{0}{7} & \frac{1}{7} & \frac{2}{7} & \frac{3}{7} & \frac{4}{7} & \frac{5}{7} & \frac{6}{7} & \frac{7}{7}  \\
   \hline 
   像素数量n_k & 790 & 1023 & 850 & 656 & 329 & 245 & 122 & 81 \\
   \hline
   归一化直方图p(s_k) & 0.19 & 0.25 & 0.21 & 0.16 & 0.08 & 0.06 & 0.03 & 0.02  \\
   \hline
   归一化累积直方图t_k & 0.19 & 0.44 & 0.65 & 0.81 & 0.89 & 0.95 & 0.98 & 1.00 \\
   \hline
   \textbf{扩展}t_k' & 1 & 3 & 5 & 6 & 6 & 7 & 7 & 7
   \end{array}
   $$
   其中$t'_k$的选取：选择最靠近的一个灰度级的值，例如$0.19$离$\frac{1}{7}$最近，则修正的灰度级为$1$，以此类推。
   
## 空间滤波机理

组成：

- 一个**邻域**
- 对领域内像素执行的**预定义操作**

滤波在邻域中心坐标产生一个新的像素，其值是滤波操作的结果。滤波器的中心访问图像中的每个像素后生成滤波后的图像。

可根据执行的操作分为**线性空间滤波器**和**非线性空间滤波器**。

## 线性滤波

### 技术分类和实现原理

#### 技术分类

##### 平滑滤波

平滑线性空间滤波器使用滤波器模板确定的领域像素的平均灰度值代替邻域中心像素的值。降低了图像灰度的“尖锐”变化。

应用：降低噪声、模糊处理...

影响：边缘模糊的负面效应

##### 锐化滤波

削弱图像中灰度缓慢变化的区域，同时使图像中灰度值发生突变的区域得到增强（或不变）。（即消除图像中的低频分量，同时增强（或不影响）高频分量。）

效果：增强被模糊的细节或目标的边缘

*与平滑滤波互逆。凸显细节，弱化背景。*

#### 实现原理（模板卷积）

大小为$m\times n$的滤波器对大小为$M\times N$的图像$f(x,y)$进行线性空间滤波，对于图像中的任意一点$(x,y)$，**滤波器的响应$g(x,y)$是滤波器的系数与该滤波器所覆盖像素点的像素值的乘积之和**，即：
$$
\begin{aligned}
g(x,y)=\sum_{s=-a}^{a}\sum_{t=-b}^bw(s,t)f(x+s,y+t)
\end{aligned}
$$
其中：

- $w(s,t)$为滤波器系数，滤波器中心系数$w(0,0)$对准位置为$(x,y)$的像素；
- $m=2a+1$，$n=2b+1$，且$a$、$b$为正整数。

*使用奇数尺寸的滤波器可更简化索引且更为直观，因为其滤波器的中心落在整数值上。*

**算法步骤**：

1. 将滤波器在图像中漫游，并将滤波器中心与图像中某个像素位置重合；
2. 将滤波器中的各个系数与滤波器所覆盖的各对应像素的灰度值相乘；
3. 将2中的所有成绩结果进行相加，并将加法运算的结果赋给图像中对应滤波器中心位置的像素（滤波器的输出响应）。

大小为$3\times 3$滤波器的线性空间滤波为例，滤波下方的的图像部分像素和滤波器系数入以下所示：
$$
\begin{bmatrix}
{ } & {\vdots} & {\vdots} & {\vdots} & { } \\
{\cdots} & {f(x-1,y-1)} & {f(x-1,y)} & {f(x-1,y+1)} & {\cdots} \\
{\cdots} & {f(x,y-1)} & {f(x,y)} & {f(x,y+1)} & {\cdots}\\
{\cdots} & {f(x+1,y-1)} & {f(x+1,y)} & {f(x+1,y+1)} & {\cdots}\\
{ } & {\vdots} & {\vdots} & {\vdots} & { }
\end{bmatrix}
$$

$$
\begin{bmatrix}
{w(-1,-1)} & {w(-1,0)} & {w(-1,1)} \\
{w(0,-1)} & {w(0,0)} & {w(0,1)} \\
{w(1,-1)} & {w(1,0)} & {w(1,1)}\\
\end{bmatrix}
$$

$$
\begin{aligned}
g(x,y) &= w(-1,-1)f(x-1,y-1) &&+ w(-1,0)f(x-1,y) &&+ w(-1,1)f(x-1,y+1) \\
		&+ w(0,-1)f(x,y-1) &&+ w(0,0)f(x,y) &&+ w(0,1)f(x,y+1) \\
		&+ w(1,-1)f(x+1,y-1) &&+ w(1,0)f(x+1,y) &&+ w(1,1)f(x+1,y+1)
\end{aligned}
$$

***对于每一个滤波的结果，其参与运算的邻域灰度值均为原始图像对应邻域的灰度值，相当于在把结果存放在一个新的空白矩阵上，而非在原始图像上就地修改。***

### 线性平滑滤波器

#### 邻域平均

$$
g(x,y)=\frac{1}{mn}\sum_{(x,y)\in S}f(x,y)
$$

其中：

- $S$为滤波器模板覆盖的像素邻域

- $mn$为邻域$S$中像素点数

算法简单，但会使图像产生模糊，且邻域越大，模糊越厉害。

#### 加权平均

滤波器模板中各个位置的系数采用不同的数值：

- 离模板中心近的像素权值大
- 离模板中心远的像素权值小
- 权值之和等于1

$$
\begin{aligned}
g(x,y)=\sum_{s=-a}^{a}\sum_{t=-b}^bw(s,t)f(x+s,y+t)
\end{aligned}
$$

$$
H_1 = 
\frac{1}{9}
\begin{bmatrix}
{1} & {1} & {1} \\
{1} & {1} & {1} \\
{1} & {1} & {1} \\
\end{bmatrix}
\quad
H_2 = 
\frac{1}{10}
\begin{bmatrix}
{1} & {1} & {1} \\
{1} & {2} & {1} \\
{1} & {1} & {1} \\
\end{bmatrix}
\quad
H_3 = 
\frac{1}{16}
\begin{bmatrix}
{1} & {2} & {1} \\
{2} & {4} & {2} \\
{1} & {2} & {1} \\
\end{bmatrix}
$$

## 非线性滤波

### 非线性平滑滤波器

**统计排序滤波器**的响应以滤波器所覆盖的图像区域中的所有像素的**排序**为基础，然后使用**统计排序的结果值**代替中心像素的值。其具备优秀的去噪能力，且比同尺寸的线性平滑滤波器的模糊程度明显要低。

#### 中值滤波器

使用像素邻域内灰度的**中值**代替中心像素的值。其主要功能是拥有不同灰度的像素点看起来更接近于它的相邻点（去除孤立像素）。

中值滤波器对处理**椒盐噪声**（*椒噪声：灰度值较低，偏暗；盐噪声：灰度值较高，偏亮。极端情况是黑色和白色噪声*）非常有效，因为这种噪声以黑白点的形式叠加在图像上。

##### 实现步骤

1. 将滤波器模板（无系数）在图像中漫游，并将模板中心与图像中某个像素位置重合；
2. 读取模板下各对应像素的灰度值；
3. 将灰度值按从小到大（或从大到小）的次序进行排序；
4. 确定排序结果的中值，将此中值赋予对应模板中心位置的像素。

$$
\begin{bmatrix}
{ } & {\vdots} & {\vdots} & {\vdots} & { } \\
{\cdots} & {10} & {20} & {20} & {\cdots} \\
{\cdots} & {20} & {15} & {20} & {\cdots}\\
{\cdots} & {20} & {25} & {100} & {\cdots}\\
{ } & {\vdots} & {\vdots} & {\vdots} & { }
\end{bmatrix}

\Rightarrow

\begin{bmatrix}
{10} & {15} & {20} & {20} & \textbf{20} & {20} & {20} & {25} & {100}
\end{bmatrix}
$$

##### 模板选择

去噪效果与以下两个因素有关：

- 模板形状
- 参与运算的像素数量

常用模板形状：方形、圆形、十字形等。
$$
\begin{bmatrix}
{\cdot} & {\cdot} & {\cdot} & {\cdot} & {\cdot} \\
{\cdot} & {\bullet} & {\bullet} & {\bullet} & {\cdot} \\
{\cdot} & {\bullet} & {\bullet} & {\bullet} & {\cdot} \\
{\cdot} & {\bullet} & {\bullet} & {\bullet} & {\cdot} \\
{\cdot} & {\cdot} & {\cdot} & {\cdot} & {\cdot} 
\end{bmatrix}
\quad
\begin{bmatrix}
{\cdot} & {\bullet} & {\bullet} & {\bullet} & {\cdot} \\
{\bullet} & {\bullet} & {\bullet} & {\bullet} & {\bullet}  \\
{\bullet} & {\bullet} & {\bullet} & {\bullet} & {\bullet} \\
{\bullet} & {\bullet} & {\bullet} & {\bullet} & {\bullet} \\
{\cdot} & {\bullet} & {\bullet} & {\bullet} & {\cdot} 
\end{bmatrix}
\quad
\begin{bmatrix}
{\cdot} & {\cdot} & {\bullet} & {\cdot} & {\cdot} \\
{\cdot} & {\cdot} & {\bullet} & {\cdot} & {\cdot} \\
{\bullet} & {\bullet} & {\bullet} & {\bullet} & {\bullet}\\
{\cdot} & {\cdot} & {\bullet} & {\cdot} & {\cdot} \\
{\cdot} & {\cdot} & {\bullet} & {\cdot} & {\cdot} 
\end{bmatrix}
$$

- 对于有缓变的较长轮廓线的图像，采用方形或圆形模板为宜；
- 对于包含有尖顶角物体的图像，采用十字形模板为宜，且模板大小则以不超过图像中最小有效物体的尺寸为宜；
- 对于包含点、线、尖细节较多的图像，则不适宜采用中值滤波。

#### 百分比滤波器

最大值（$max$）滤波器：
$$
g_{max}(x,y)=\mathop{max}\limits_{(s,t)\in N(x,y)}[f(s,t)]
$$
最小值（$min$）滤波器：
$$
g_{min}(x,y)=\mathop{min}\limits_{(s,t)\in N(x,y)}[f(s,t)]
$$
椒噪声有较低的灰度值，用最大值滤波器有较好的效果，而盐噪声反之。

*最大值滤波会细化黑色目标，最小值滤波会粗化黑色目标。*

#### 中点滤波器

$$
g_{mid}(x,y)=\frac{1}{2}[g_{max}(x,y)+g_{min}(x,y)]
$$

结合了排序统计和求平均，对于高斯和均匀分布随机噪声有较好效果。

*中点滤波器得到的结果图像会产生模糊。*

### 非线性锐化滤波器

锐化处理目的是突出图像中灰度的**过渡**部分。

锐化处理可以用**空间微分**来完成（微分算子的响应强度与像素的突变程度成正比）。即图像微分**增强**边缘与其他突变（噪声、线），并**削弱**灰度变化缓慢的区域。

常用滤波器：

- 基于一阶微分的锐化滤波器
- 基于二阶微分的锐化滤波器

#### 数字图像微分

1. 一阶微分
   - 在**恒定灰度区域**的一阶微分值**为零**；
   - 在**灰度台阶、灰度斜坡的起点处**一阶微分值**非零**；
   - **沿着灰度斜坡**的一阶微分值**非零**。

2. 二阶微分
   - 在**恒定灰度区域**的二阶微分值**为零**；
   - 在**灰度台阶、灰度斜坡的起点处**二阶微分值**非零**；
   - **沿着灰度斜坡**的二阶微分值**为零**。

对于一维离散函数$f(x)$，采用差分计算其微分如下：

- 一阶微分
  $$
  \frac{\partial f}{\partial x}=f(x+1)-f(x)
  $$

- 二阶微分
  $$
  \begin{aligned}
  \frac{\partial^2 f}{\partial x^2}&=[f(x+1)-f(x)]-[f(x)-f(x-1)] \\
  &= f(x+1)+f(x-1)-2f(x)
  \end{aligned}
  $$

对于二维的数字图$f(x,y)$，可以沿着两个空间轴处理偏微分。

例如灰度扫描值及一阶微分、二阶微分：
$$
\begin{bmatrix}
6 & 6 & 6 & 5 & 4 & 3 & 2 & 1 & 1 & 1 & 6 & 6 & 6 \\
恒定灰度 & & 斜坡起点 &  &  & 斜坡 &  &  &  & 台 & 阶 &  &  \\
0 & 0 & -1 & -1 & -1 & -1 & 0 & 0 & 0 & 5 & 0 & 0 & 0 \\
0 & 0 & -1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & -5 & 0 & 0
\end{bmatrix}
$$

#### 基于一阶微分的锐化滤波器——梯度算子

基于一阶微分的锐化滤波常用**梯度幅值**来实现。

对于图像$f$，在任意坐标$(x.y)$上的**梯度**$\nabla f$定义为**二维列向量**：
$$
\nabla f=
\begin{bmatrix}
Gx & Gy
\end{bmatrix}^T
=
\begin{bmatrix}
\frac{\partial f}{\partial x} & \frac{\partial f}{\partial y}
\end{bmatrix}^T
$$
梯度的**幅值**$|\nabla f|$：
$$
|\nabla f|=\sqrt{G_x^2+G_y^2}=\sqrt{(\frac{\partial f}{\partial x})^2 + (\frac{\partial f}{\partial y})^2}
$$
实际应用中，一般把梯度的幅值称为梯度，并采用绝对值近似求梯度幅值：
$$
|\nabla f|=|G_x|+|G_y|=|\frac{\partial f}{\partial x}| + |\frac{\partial f}{\partial y}|
$$

##### 梯度（一阶微分）的近似计算方法（滤波模板）:

###### 直接差分
$$
   \begin{aligned}
   G_x=f(x+1,y)-f(x,y) \\
   G_y=f(x,y+1)-f(x,y)
   \end{aligned}
$$
   直接差分算子：
$$
   垂直方向
   \begin{bmatrix}
   \underline{-1} & 0 \\
   1 & 0 
   \end{bmatrix}
   \quad
   水平方向
   \begin{bmatrix}
   \underline{-1}& 1 \\
   0 & 0 
   \end{bmatrix}
$$

###### 交叉差分
$$
   \begin{aligned}
   G_x=f(x+1,y+1)-f(x,y) \\
   G_y=f(x+1,y)-f(x,y+1)
   \end{aligned}
$$
   交叉差分（Roberts）算子：
$$
   垂直方向
   \begin{bmatrix}
   \underline{-1} & 0 \\
   0 & 1 
   \end{bmatrix}
   \quad
   水平方向
   \begin{bmatrix}
   \underline{0} & -1 \\
   1 & 0 
   \end{bmatrix}
$$

###### Sobel算子
$$
   \begin{aligned}
   G_x = &f(x+1,y-1)+2f(x+1,y)+f(x+1,y+1)\\
   &-f(x-1,y-1)-2f(x-1,y)-f(x-1,y+1) \\
   G_y = &f(x-1,y+1)+2f(x,y+1)+f(x+1,y+1)\\
   &- f(x-1,y-1)-2f(x,y-1)-f(x+1,y-1)
   \end{aligned}
$$
   Sobel算子：
$$
   垂直方向
   \begin{bmatrix}
   -1 & -2 & -1 \\
   0 & \underline{0} & 0 \\
   1 & 2 & 1 
   \end{bmatrix}
   \quad
   水平方向
   \begin{bmatrix}
   -1 & 0 & 1 \\
   -2 & \underline{0} & 2 \\
   -1 & 0 & 1 
   \end{bmatrix}
$$

*下划线标出元素为滤波器模板的原点。*

*可以看出，实现**平滑**的滤波器系数之和为**1**，实现**锐化**的滤波器系数之和为**0**。*

##### 应用

工业检测、辅助人工检测缺陷，或更为通用的自动检测的预处理。

#### 基于二阶微分的锐化滤波器——拉普拉斯算子

数字图像$f(x,y)$的拉普拉斯变换定义为：
$$
\nabla ^2f=\frac{\partial ^2f}{\partial x^2}+\frac{\partial ^2f}{\partial y^2}
$$
其中：
$$
\begin{aligned}
\frac{\partial ^2f}{\partial x^2}=f(x+1,y)+f(x-1,y)-2f(x,y)\\
\frac{\partial ^2f}{\partial y^2}=f(x,y+1)+f(x,y-1)-2f(x,y)
\end{aligned}
$$
即：
$$
\nabla ^2f=f(x-1,y)-2f(x,y)+f(x,y+1)+f(x,y-1)]-4f(x,y)
$$
执行上式定义的离散拉普拉斯变化所用的滤波器模板：
$$
(a)
\begin{bmatrix}
0 & 1 & 0 \\
1 & -4 & 1 \\
0 & 1 & 0
\end{bmatrix}
$$

$$
(b)
\begin{bmatrix}
1 & 1 & 1 \\
1 & -8 & 1 \\
1 & 1 & 1
\end{bmatrix}
\quad
(c)
\begin{bmatrix}
0 & -1 & 0 \\
-1 & 4 & -1 \\
0 & -1 & 0
\end{bmatrix}
\quad
(d)
\begin{bmatrix}
-1 & -1 & -1 \\
-1 & 8 & -1 \\
-1 & -1 & -1
\end{bmatrix}
$$

$(b)$为执行离散拉普拉斯变换的扩展模板，包括了对角方向的的领域像素;$(c)、(d)$为其他两种拉普拉斯变换的实现，仅符号相反，结果等效。

**使用拉普拉斯变换对图像进行增强的基本方法**可表示为：
$$
g(x,y)=
\begin{cases}
f(x,y)-\nabla ^2f,若拉普拉斯模板中心系数为负 \\
f(x,y)+\nabla ^2f,若拉普拉斯模板中心系数为正
\end{cases}
$$
将原始图像和拉普拉斯图像**叠加**在一起，以增强细节。

#### 混合空间增强法

若原始图像的灰度动态范围很窄并且伴随着很高的噪声，则采用单一的图像增强算法很难对其进行增强。

# 傅里叶变换

## 傅里叶变换及其反变换

空域$\stackrel{正变换}{\longrightarrow}$其他空间$\stackrel{反变换/逆变换}{\longrightarrow}$空域。

### 一维连续傅里叶变换及反变换

$$
\begin{aligned}
F(\mu)=\int_{-\infty}^{\infty}f(t)e^{-j2\pi \mu t}dt \\
f(t)=\int_{-\infty}^{\infty}F(\mu)e^{j2\pi \mu t}d\mu
\end{aligned}
$$

其中，$j=\sqrt{-1}$

### 二维连续傅里叶变换及反变换

$$
\begin{aligned}
F(\mu,v)=\int_{-\infty}^{\infty}\int_{-\infty}^{\infty}f(t,z)e^{-j2\pi(\mu t+vz)}dtdz \\
f(t,z)=\int_{-\infty}^{\infty}\int_{-\infty}^{\infty}F(\mu,v)e^{j2\pi(\mu t+vz)}d\mu dv
\end{aligned}
$$

### 一维DFT及IDFT


$$
\begin{aligned}
F_m=\frac{1}{M}\sum_{n=0}^{M-1}f_ne^{-j2\pi mn/M},\quad m=0,1,2,\cdots,M-1 \\
f_n=\sum_{m=0}^{M-1}F_me^{j2\pi mn/M},\quad n=0,1,2,\cdots,M-1
\end{aligned}
$$

$x$和$y$表示图像坐标变量（空域变量）并使用$u$和$v$表示频率变量，上式变为：
$$
\begin{aligned}
F(\mu)=\frac{1}{M}\sum_{x=0}^{M-1}f(x)e^{-j2\pi ux/M},\quad u=0,1,2,\cdots,M-1\\
f(x)=\sum_{u=0}^{M-1}F(u)e^{j2\pi ux/M},\quad x=0,1,2,\cdots,M-1
\end{aligned}
$$
由欧拉公式
$$
e^{j\theta}=\cos\theta+j\sin\theta
$$
有：
$$
\begin{aligned}
F(u)&=\frac{1}{M}\sum_{x=0}^{M-1}f(x)e^{-j2\pi ux/M} \\
&=\frac{1}{M}\sum_{x=0}^{M-1}f(x)(\cos\frac{2\pi ux}{M}-j\sin\frac{2\pi ux}{M})
\end{aligned}
$$

#### 傅里叶变换$F(u)$的极坐标表示

$$
F(u)=|F(u)|e^{-j\varphi(u)}
$$

其中，

相角或相位谱：
$$
\varphi(u)=\arctan[\frac{I(u)}{R(u)}]
$$
$R(u)$和$I(u)$分别是$F(u)$的实部和虚部。

幅度谱（频谱）：
$$
|F(u)|=\sqrt{R(u)^2+I(u)^2}
$$
功率谱：
$$
P(u)=|F(u)|^2=R(u)^2+I(u)^2
$$

### 二维DFT及IDFT

对大小为$M\times N$的图像$f(x,y)$：
$$
F(u,v)=\frac{1}{MN}\sum_{x=0}^{M-1}\sum_{y=0}^{N-1}f(x,y)e^{-j2\pi(ux/M+vy/N)}
$$
其中，
$$
u=0,1,2,\cdots,M-1 \\
v=0,1,2,\cdots,N-1
$$

$$
f(x,y)=\sum_{x=0}^{M-1}\sum_{y=0}^{N-1}F(u,v)e^{-j2\pi(ux/M+vy/N)}
$$

其中，
$$
x=0,1,2,\cdots,M-1 \\
y=0,1,2,\cdots,N-1
$$

*在有些文献中，常数$1/MN$通常出现在DFT而非IDFT的前面。这时，这个常数的平方根应包含在正变换和反变换的前面，以便形成一个更为对称的变换对。只要使用一致，这种形式的任何表述就都是正确的。*

#### 二维DFT的极坐标表示

$$
F(u,v)=|F(u,v)|e^{-j\varphi(u,v)}
$$

其中，

相角或相位谱：
$$
\varphi(u,v)=\arctan[\frac{I(u,v)}{R(u,v)}]
$$
$R(u,v)$和$I(u,v)$分别是$F(u,v)$的实部和虚部。

幅度谱（频谱）：
$$
|F(u,v)|=\sqrt{R(u,v)^2+I(u,v)^2}
$$
功率谱：
$$
P(u,v)=|F(u,v)|^2=R(u,v)^2+I(u,v)^2
$$

### 关于频谱$|F(u,v)|$

- 频谱描述图像中某种频率的成分数量；
- 频谱中出现的明亮线反映了原始图像的灰度级变化方向。

## 傅里叶变换的性质

### 平移

### 可分离性

$$
\begin{aligned}
F(u,v)&=\frac{1}{MN}\sum_{x=0}^{M-1}\sum_{y=0}^{N-1}f(x,y)e^{-j2\pi (ux/M+vy/N)} \\
&= \frac{1}{M}\sum_{x=0}^{M-1}e^{-j2\pi ux/M}\frac{1}{N}\sum_{y=0}^{N-1}f(x,y)e^{-j2\pi vy/N} \\
&= \frac{1}{M}\sum_{x=0}^{M-1}e^{-j2\pi ux/M}F(x,v)
\end{aligned}
$$

$F(x,v)$是沿着$f(x,y)$的**一行**进行傅里叶变换的结果，当$x=0,1,\cdots,M-1$，则沿着$f(x,y)$的所有行计算傅里叶变换。
$$
f(x,y) \stackrel{一维行变换}{\longrightarrow} F(x,v) \stackrel{一维列变换}{\longrightarrow} F(u,v)
$$
二维IDFT与上述过程类似。

### 平均值

图像$f(x,y)$在**原点**处的傅里叶变换等于图像的**平均灰度级**。
$$
F(0,0)=\frac{1}{MN}\sum_{x=0}^{M-1}\sum_{y=0}^{N-1}f(x,y)
$$

## 快速傅里叶变换（FFT）


# 频率域图像增强

## 频率域滤波基础

在**频率域**研究图像增强：

- 可以利用频率成分和图像外表之间的对应关系；（一些在空间域表述困难的增强任务，在频率域中变得非常普通。）
- 滤波在频率域更为直观，它可以解释空间域滤波的某些性质；（利用这些性质进行处理，再转换回图像空间，可以得到所需的效果。）
- 空间域和频率域中的滤波器组成了傅里叶变换对。（可以在频率域指定滤波器，并对其执行反变换，最后在空间域使用该反变换的结果作为空域滤波器。）

### 傅里叶变换的频率分量与图像空间特征

- 变化最慢的频率成份$(u=v=0)$对应图像的平均灰度级：
  $$
  F(0,0)=\frac{1}{MN}\sum_{x=0}^{M-1}\sum_{y=0}^{N-1}f(x,y)
  $$

- 从变换的原点移开时，低频成分对应着图像中灰度慢变化的分量（图像的平滑部分）；

- 进一步偏离原点时，较高的频率成分对应图像中变化越来越快的灰度（边缘或噪声等尖锐部分）。

### 频率域滤波的基本步骤

1. 用$(-1)^{x+y}$乘输入图像$f(x,y)$，使其**原点中心化**；
2. 对步骤1的结果执行**DFT**，得到关于中心对称的频谱$F(u,v)$；
3. 生成一个**实的、中心对称的频域滤波器**$H(u,v)$；
4. 对滤波器$H(u,v)$、频谱$F(u,v)$执行**阵列相乘**（对应元素逐个进行相乘),形成乘积$G(u,v)=H(u,v)F(u,v)$，其中$G(m,n)=H(m,n)F(m,n)$，且$0\le m \le M-1,0\le n \le N-1$；
5. 对步骤4的结果$G(u,v)$执行**反DFT**，并取其结果的**实部**；
6. 用$(-1)^{x+y}$乘步骤5的反DFT结果的实部，得到**滤波结果**$g(x,y)$。

### 频域滤波器如何作用于图像

#### 低通滤波器

使频谱的**低频成分通过**，同时使其**高频成分衰减**。

- 被低通滤波的图像比原始图像减少了尖锐的细节部分，突出了平滑过渡部分；
- 对应于空间域滤波的平滑处理，如均值滤波器。

#### 高通滤波器

使频谱的**高频成分通过**，同时使其**低频成分衰减**。

- 被高通滤波的图像比原始图像少了灰度级的平滑过渡，突出了边缘等细节部分；
- 对应于空间域滤波的锐化处理，如梯度算子、拉普拉斯算子。

## 频率域低通（平滑）滤波器

低通滤波器的作用：用于截断频谱中所有处于指定距离$D_0$之外的高频成分。

### 理想低通滤波器（ILPF）

假设频谱中心在$(M/2,N/2)$处，则任意频谱成分$(u,v)$到中心（原点）的距离$D(u,v)$定义为：
$$
D(u,v)=\sqrt{(u-\frac{M}{2})^2+(v-\frac{N}{2})^2}
$$
理想低通滤波器$H(u,v)$定义为：
$$
H(u,v)=
\begin{cases}
1\quad D(u,v)\le D_0 \\
0\quad D(u,v)\gt D_0 \\
\end{cases}
$$
 *在半径为$D_0$的圆内，所有频率没有衰减地完全通过滤波器，而在此半径的圆之外的所有频率*完全被衰减掉。

总图像功率值$P_T$：
$$
P_T=\sum_{u=0}^{M-1}\sum_{v=0}^{N-1}P(u,v)
$$
其中：
$$
P(u,v)=|F(u,v)|^2=R(u,v)^2+I(u,v)^2
$$
原点位于频谱中心处，半径为$D_0$的圆包含$\alpha\%$的总功率，

其中：
$$
\alpha=100[\sum_u\sum_vP(u,v)/P_T]
$$

- *随着滤波器**半径的增大**，**滤除的功率**越来越**少**，导致的**模糊**也越来越**弱**。*

- 理想低通滤波器产生**模糊**和**振铃**现象，且模糊和振铃现象**反比于截断频率**（即半径$D_0$）。

### 巴特沃斯低通滤波器（BLPF）

$n$阶巴特沃斯低通滤波器定义如下：
$$
H(u,v)=\frac{1}{1+{[D(u,v)/D_0]}^{2n}}
$$

- **低阶**滤波器没有明显振铃现象（滤波器在低频和高频之间平滑过渡）。*且1阶BLPF核即没有振铃效应又没有负值。*

### 高斯低通滤波器（GLPF）

二维高斯低通滤波器定义如下：
$$
H(u,v)=e^{-D(u,v)^2/2\sigma ^2}
$$
$\sigma$是关于频谱中心的扩展度的度量。

令$\sigma=D_0$，则二维高斯低通滤波器表示为：
$$
H(u,v)=e^{-D(u,v)^2/2D_0 ^2}
$$

- 平滑效果稍差于相同截止频率的二阶BLPF；
- **没有出现振铃现象**，优于BLPF。

### 应用实例

- 用于机器识别系统识别字符的预处理；
- 减少人脸图像的皮肤细纹核小斑点；
- 消除卫星、航空图像中的不重要特征。


## 频率域高通（锐化）滤波器

高通滤波器的作用：用于截断频谱中所有处于指定距离$D_0$之内的低频成分。

### 理想高通滤波器（IHPF）

截止频率距原点的距离为$D_0$的IHPF定义为：
$$
H(u,v)=
\begin{cases}
0\quad D(u,v)\le D_0 \\
1\quad D(u,v)\gt D_0
\end{cases}
$$

- 振铃现象明显。

### 巴特沃斯高通滤波器（BHPF）

$n$阶且截止频率距原点的距离为$D_0$的BHPF定义为：
$$
H(u,v)=\frac{1}{1+{[D_0/D(u,v)]}^{2n}}
$$

- BHPF的结果比IHPF的结果尖锐得多，边缘失真也小得多。

### 高斯高通滤波器（GHPF）

截止频率距原点的距离为$D_0$的GHPF定义为：
$$
H(u,v)=1-e^{-D(u,v)^2/2D_0^2}
$$

- GHPF的结果比BHPF和IHPF的结果更尖锐，即使是对微小物体和细线条的滤波也是较清晰的。

### 高通滤波器与低通滤波器的关系

$$
H_{HP}(u,v)=1-H_{LP}(u,v)
$$

其中：

$H_{LP}(u,v)$：低通滤波器函数；$H_{HP}(u,v)$：高通滤波器函数。

被低通滤波器衰减的频率成分能通过高通滤波器，反之亦然。

### 高频提升和高频加强

高通滤波效果等同于用原始图像的频谱减去低通滤波的结果图像频谱。

图像经过高通滤波后，由于高通滤波器除去了傅里叶变换的零频率，其背景的平均强度减小到接近黑色。

原始图像加到滤波后的结果图像，即**高频提升滤波**或**高频加强滤波**。

#### 高频提升滤波

将原始图像按一定比例加到滤波后的结果中，以保留原始图像的背景。

设在空域中，原始图像为$f(x,y)$，高通滤波后的结果图像为$f_{HP}(x,y)$，低通滤波后的结果图像为$f_{LP}(x,y)$，高通提升滤波的结果图像为$f_{HB}(x,y)$，则**高频提升滤波的空域形式**如下：
$$
f_{HB}(x,y)=A\times f(x,y)-f_{LP}(x,y),\quad A\ge1
$$
亦即：
$$
\begin{aligned}
f_{HB}(x,y)&=(A-1)\times f(x,y)+f(x,y)-f_{LP}(x,y) \\
&=(A-1)\times f(x,y)+f_{HP}(x,y)
\end{aligned}
$$
对于高频提升滤波的空域形式$f_{HB}(x,y)$和频域形式$F_{HB}(u,v)$：
$$
\begin{gather*}
f_{HB}(x,y)=(A-1)\times f(x,y)+f_{HP}(x,y) \\
\downarrow \\
F_{HB}(u,v)=(A-1)\times F(u,v)+F_{HP}(u,v) \\
\downarrow \\
F_{HB}(u,v)=(A-1)\times F(u,v)+H_{HP}(u,v)\times F(u,v) \\
\downarrow \\
F_{HB}(u,v)=[(A-1)+H_{HP}(u,v)]\times F(u,v)
\end{gather*}
$$
**高频提升滤波器**：
$$
H_{HB}(u,v)=(A-1)+H_{HP}(u,v),\quad A\ge1,A=1时普通高通
$$
**高频提升滤波**：
$$
F_{HB}(u,v)=H_{HB}(u,v)\times F(u,v)
$$

#### 高频加强滤波

加强增强图像的高频成分。

在高通滤波器函数前乘一个常数，再增加一个偏移量以便使零频率不被滤波器滤除掉。

高通滤波：
$$
G(u,v)=H_{HP}(u,v)\times F(u,v)
$$
高频加强滤波器：
$$
H_E(u,v)=k\times H_{HP}(u,v)+c
$$
$k\ge 0$且$k\gt c$，$k$的典型值在$1.5$到$2.0$之间，$c$的典型值在$0.25$到$0.5$之间。

高频加强滤波：
$$
\begin{aligned}
G_E(u,v)&=H_E(u,v)\times F(u,v) \\
&= [k\times H_{HP}(u,v)+c]\times F(u,v) \\
&=k\times H_{HP}(u,v)\times F(u,v) + c\times F(u,v) \\
&=k\times G(u,v)+c\times F(u,v)
\end{aligned}
$$

# 图像复原

## 图像退化/复原过程的模型

### 图像退化与图像复原

**图像退化**是指图像在形式、存储、处理和传输过程中，由于成像系统、存储设备、处理方法和传输介质的不完善，从而导致的**图像质量下降**。

引起图像退化的原因有：

- 成像系统的散焦；
- 成像设备与物体的相对运动；
- 成像器材的固有缺陷；
- 外部干扰；
- ......

**图像复原**（图像恢复）指的是对退化的图像进行处理，试图恢复降质的图像。

二者关系：

- 图像复原可以看作是图像退化的**逆过程**；
- 实际情况中，退化过程往往并不知晓，这种复原称为**盲目复原**；
- 图像模糊的同时，噪声和干扰也会同时存在。

### 图像退化/复原模型

$$
\begin{aligned}
f(x,y)\rightarrow 退化函数H \rightarrow &\sum \stackrel{退化图像g(x,y)}{\longrightarrow} 复原滤波器 \rightarrow \hat{f}(x,y) \\
&\uparrow 噪声n(x,y) \\ 
\end{aligned}
$$

$$
g(x,y)=H[f(x,y)]+n(x,y)
$$

**图像复原**：在给定$g(x,y)$和$H$的基础上得到对$f(x,y)$的某个近似，通常采用线性的、空间不变的复原技术。

如果退化系统（函数）$H$是**线性空间不变系统**：

1. 线性：
   $$
   \begin{aligned}
   & H[k_1f_1(x,y)+k_2f_2(x,y)]=k_1H[f_1(x,y)]+k_2H[f_2(x,y)] \\
   \end{aligned}
   $$
   齐次性：$H[kf(x,y)]=kH[f(x,y)]$

   叠加性：$H[f_1(x,y)+f_2(x,y)]=H[f_1(x,y)]+H[f_2(x,y)]$

2. 空间不变性：
   $$
   H[f(x-a,y-b)]=g(x-a,y-b)\quad H[f(x,y)]=g(x,y)
   $$
   即图像中任一像素点通过退化系统时的响应只取决于该点的输入值，而与该点的位置无关。

则退化图像可以表示为：

**空域模型**：
$$
g(x,y)=h(x,y)*f(x,y)+n(x,y)
$$
**频域模型**：
$$
G(u,v)=H(u,v)F(u,v)+N(u,v)
$$

## 噪声模型

图像中的噪声是**随机**的，其**灰度值的统计特征**可以用**概率密度函数**（PDF）或相应的**累积分布函数**（CDF）进行表征。

对于退化图像中的噪声$n(x,y)$（*噪声的灰度值，非位置*），有多种不同的统计模型：

- 均匀（Uniform）噪声
- 指数（Exponential）噪声
- 高斯（Gaussian）噪声
- 瑞利（Rayleigh）噪声
- 伽马（爱尔兰）噪声
- 脉冲（椒盐）噪声
- 周期噪声

### 均匀噪声

$$
\Large
p(z)=
\begin{cases}
\frac{1}{b-a}\quad &a\le z\le b \\
0\quad &其他
\end{cases}
$$

其中，$z$表示噪声灰度值。
$$
z=a+(b-a)\times U(0,1)
$$
$U(0,1)$表示区间$[0,1]$内的均匀随机数。
$$
\begin{aligned}
\mu &= \frac{a+b}{2} \\
\sigma^2 &= \frac{(b-a)^2}{12}
\end{aligned}
$$
实例（MATLAB）：

```matlab
a=2;
b=5;
noise=a+(b-1)*rand(100,100);
blackIm=zeros(100,100);
noisedIm=noise+blackIm;
```

### 指数噪声

$$
\Large
p(z)=
\begin{cases}
ae^{-az}\quad & z\ge 0\\
0\quad & z\lt0
\end{cases}
$$

其中，$a\gt 0$。
$$
z=-\frac{1}{a}\times ln[1-U(0,1)]
$$

$$
\begin{aligned}
\mu&=\frac{1}{a} \\
\sigma^2 &= \frac{1}{a^2}

\end{aligned}
$$

实例（MATLAB）：

```matlab
a=2;
noise=(-1/a)*log(1-rand(100,100));
blackIm=zeros(100,100);
noiseIm=noise+blackIm;
```

### 高斯噪声

$$
\Large
p(z)=\frac{1}{\sqrt{2\pi}\sigma}\exp[{-\frac{(z-\mu)^2}{2\sigma^2}}]
$$

$$
z=\mu+\sigma\times N(0,1)
$$

$N(0,1)$表示标准正态分布的随机数。

灰度值有$70\%$落在$[\mu-\sigma,\mu+\sigma]$范围内。

实例（MATLAB）：

```matlab
mu=0;
sigma=0.1;
noise=mu+sigma*randn(100,100);
blackIm=zeros(100,100);
noiseIm=noise+blackIm;
```

### 瑞利噪声

$$
\Large
p(z)=
\begin{cases}
\frac{2}{b}(z-a)\exp[-\frac{(z-a)^2}{b}] &z\ge a\\
0 &z\lt a
\end{cases}
$$

$$
z=a+\sqrt{-b\times ln[1-U(0,1)]}
$$

$$
\begin{aligned}
\mu &= a + \sqrt{\frac{\pi b}{4}} \\
\sigma^2&=\frac{b(4-\pi)}{4}
\end{aligned}
$$

实例（MATLAB）：

```matlab
a=0;
b=0.1;
noise=a+(-b*log(1-rand(100,100))).^2;
blackIm=zeros(100,100);
noiseIm=noise+blackIm;
```

### 伽马噪声

$$
\Large
p(z)=
\begin{cases}
\frac{a^bz^{b-1}}{(b-1)!}e^{-az} \quad &z\gt0 \\
0 &z\lt0
\end{cases}
$$

其中，$a\gt0$，$b$为正整数。
$$
z=E_1+E_2+\cdots+E_b
$$
$E_i$是具有参数$a$的指数随机数。
$$
\begin{aligned}
\mu &= \frac{b}{a} \\
\sigma^2 &= \frac{b}{a^2}
\end{aligned}
$$
实例（MATLAB）：

```matlab
a=2;
b=5;
noise=zeros(100,100);
for j=1:b
	noise=noise+(-1/a)*log(1-rand(100,100));
end
blackIm=zeros(100,100);
noiseIm=noise+blackIm;
```

### 脉冲噪声

$$
\Large
p(z)=
\begin{cases}
P_a \quad &z=a \\
P_b &z=b \\
0 &其他
\end{cases}
$$

若$P_a$或$P_b$为零，则脉冲噪声称为单极脉冲；若$P_a$或$P_b$均不为零，则脉冲噪声成为双脉冲噪声或椒盐噪声。

通常，$a$、$b$等于所允许的最小值和最大值。

```matlab
d=0.5;
noise=rand(100,100);
noise(noise<d/2)=0;
noise(noise>=d/2 & noise<d)=1;
blackIm=zeros(100,100);
noiseIm=noise+blackIm;
```

## 空间域滤波复原

当一幅图像中存在的唯一退化因素是噪声时，其退化模型如下：

空域模型：
$$
g(x,y)=f(x,y)+n(x,y)
$$
频域模型：
$$
G(u,v)=F(u,v)+N(u,v)
$$
可以选择空域滤波的方法来复原图像。

- 均值滤波器、中点滤波器适合处理高斯或均匀分布等随机噪声；
- 中值滤波器适合处理椒盐噪声；
- 最大值滤波器适合处理“椒”噪声；
- 最小值滤波器适合处理“盐”噪声。

### 自适应滤波器

自适应滤波行为基于由$m\times n$矩形窗口$S_{xy}$定义的区域内图像的统计特征。

该类滤波器的响应基于：

- $g(x,y)$：图像$g$任意像素点的灰度值
- $\sigma_n^2$：被污染图像$g$的方差
- $m_L$：区域$S_{xy}$上像素点的灰度局部均值
- $\sigma_L^2$：区域$S_{xy}$上像素点的灰度局部方差

预期性能：

- 若$\sigma_n^2=0$（零噪声），滤波器返回$g(x,y)$；
- 若$\sigma_L^2$与$\sigma_n^2$高相关，滤波器返回$g(x,y)$的近似值；
- 若$\sigma_L^2=\sigma_n^2$（局部性质和整个图像的性质相同），滤波器返回区域$S_{xy}$上像素的局部均值$m_L$。

假设噪声是加性和位置无关的，$\sigma_n^2 \le \sigma_L^2$。

表达式如下：
$$
\hat{f}(x,y)=g(x,y)-\frac{\sigma_n^2}{\sigma_L^2}[g(x,y)-m_L]
$$
$\sigma_n^2$是唯一事先需要知道的量。

## 退化函数的估计

频域退化模型：
$$
G(u,v)=H(u,v)F(u,v)+N(u,v)
$$

### 图像观察估计法

- 寻找简单结构、受噪声影响小的子图像$g_s(x,y)$；

- 构造一个估计图像$\hat{f}_s(x,y)$，它和观察的子图像$g_s(x,y)$有相同大小和特性；

- 根据位置不变的假设：
  $$
  H_s(u,v)=\frac{G_s(u,v)}{\hat{F}_s(u,v)}
  $$

### 试验估计法

$$
  H(u,v)=\frac{G(u,v)}{A}
$$

  其中，$A$为常量，表示脉冲强度。

### 模型估计法

#### 散焦模糊（Disk Blur）

$$
\large
h(x,y)=
\begin{cases}
\frac{1}{\pi R^2} \quad &x^2+y^2\le R^2 \\
0 &others
\end{cases}
$$

$$
\Downarrow{DFT}
$$

$$
H(u,v)=2\pi R \frac{J_1(R\sqrt{u^2+v^2})}{\sqrt{u^2+v^2}}
$$

其中：

- $R$是散焦半径；

- $J_1(\cdot)$是一阶第一类贝塞尔（Bessel）函数；

- $H(u,v)$是圆对称的。

#### 运动模糊（Motion Deblur）

$$
H(u,v)=\frac{T}{\pi (ua+vb)}sin[\pi (ua+vb)]e^{-j\pi (ua+vb)}
$$

其中：

- $T$为采集时间长度（曝光时间）；
- $a$、$b$分别为垂直、水平方向的运动距离。

#### 大气湍流模糊

$$
H(u,v)=e^{-k(u^2+v^2)^{5/6}}
$$

其中，常数$k$与湍流的性质有关，$k$越大，湍流越剧烈。

## 图像复原方法——逆滤波

频域退化模型：
$$
G(u,v)=H(u,v)F(u,v)+N(u,v)
$$
原始图像傅里叶变换结果的估计：
$$
\hat{F}(u,v)=\frac{G(u,v)}{H(u,v)}
$$
没有考虑噪声的处理。

## 图像复原方法——维纳滤波

综合了退化函数和噪声统计特性，引入最小二乘约束条件，使得$\hat{f}(x,y)$域原始的未退化图像$f(x,y)$之间的均方误差最小。
$$
\min{MSE}=\min{\frac{1}{MN}\sum_{x=0}^{M-1}\sum_{y=0}^{N-1}[\hat{f}(x,y)-f(x,y)]^2}
$$
维纳滤波器：
$$
H_w(u,v)=\frac{1}{H(u,v)}\frac{|H(u,v)|^2}{|H(u,v)|^2+s\frac{|N(u,v)|^2}{|F(u,v)|^2}}
$$
其中：

- $H(u,v)$为退化函数；
- $|H(u,v)|^2$为$H(u,v)$的功率谱；
- $s$为最小二乘约束条件的拉格朗日常数；
- $|N(u,v)|^2$为噪声的功率谱；
- $|F(u,v)|^2$为未退化图像的功率谱。
- $\frac{|N(u,v)|^2}{|F(u,v)|^2}$为噪信功率比。

若退化图像具有较低的噪信功率比，则维纳滤波器$H_w(u,v)$近似为逆滤波器$\frac{1}{H(u,v)}$。如果噪声为0，则维纳滤波器退化为逆滤波。

如果噪信功率比未知或不能估计，则维纳滤波器可近似为：
$$
H_w(u,v)=\frac{1}{H(u,v)}\frac{|H(u,v)|^2}{|H(u,v)|^2+K}
$$

# 形态学图像处理

## 概述

作用：**简化图像数据**，去除图像中不重要的结构，仅保持图像的基本形状特性。

基本思想：使用具有一定形态的**结构元素**去**度量和提取**图像中的对应**形状**，以达到对图像进行处理和分析的目的。

数学基础和所用语言：集合论

基本运算：**膨胀**、**腐蚀**、**开启**、**闭合**。

## 集合论基础

### 并、交、补、差

$$
\begin{gather*}
&C=A\cup B \\
&D=A\cap B \\
&A^c=\{w|w\notin A\} \\
&A-B=\{w|w\in A, w\notin B\}=A\cap B^c
\end{gather*}
$$

### 反射与平移

#### 反射

一个集合$B$的反射表示为$\hat{B}$：
$$
\hat{B}=\{w|w=-b,b\in B\}
$$
$\hat{B}$关于原集合$B$**原点对称**。
$$
B:(x,y)\rightarrow \hat{B}:(-x,-y)
$$

#### 平移

一个集合$B$的平移表示为$(B)_z$：
$$
(B)_z=\{c|c=b+z,b\in B\}
$$
其中，$z=(z_1,z_2)$。
$$
B:(x,y)\rightarrow (B)_z:(x+z_1,y+z_2)
$$

### 二值图像的逻辑运算

$$
\begin{gather*}
NOT(A) \\
(A) AND (B) \\
(A)OR(B) \\
(A)XOR(B)
\end{gather*}
$$

##  二值图像形态学处理

设$A$：像素集合，$B$：结构元素（成员是感兴趣目标的像素的集合），处理过程是用$B$对$A$进行操作。

通过让$B$在$A$上平移，以便$B$的**原点**访问$A$的每一个像素，以此得到一个新的像素集合。

结构元素的**原点**是形态学运算的**参考点**。

*原点可以包含在结构元素中，也可以不包含在结构元素中。*

转换为矩阵阵列的结构元素示例：
$$
\begin{bmatrix}
& \cdot &  \\
\cdot & \bullet & \cdot \\
 & \cdot &  \\
\end{bmatrix}
\quad
\begin{bmatrix}
\cdot & \cdot & \cdot \\
\cdot & \bullet & \cdot \\
\cdot & \cdot & \cdot \\
\end{bmatrix}
\quad
\begin{bmatrix}
\cdot \\
\cdot \\
\bullet \\
\cdot \\
\cdot 
\end{bmatrix}
\quad
\begin{bmatrix}
& & & \cdot & & & \\
 &  & \cdot & \cdot & \cdot &  & \\
 & \cdot & \cdot & \cdot & \cdot & \cdot &   \\
\cdot & \cdot & \cdot & \bullet & \cdot & \cdot & \cdot \\
 & \cdot & \cdot & \cdot & \cdot & \cdot &   \\
 &  & \cdot & \cdot & \cdot &  & \\
& & & \cdot & & & \\
\end{bmatrix}
$$

## 膨胀和腐蚀

### 膨胀

效果：扩大图像中的物体。

设$A$：原始二值图像，$B$：结构元素，则$A$被$B$膨胀定义为：
$$
A\oplus B=\{z|(\hat{B})_z\cap A \ne \emptyset\}
$$
或：
$$
A\oplus B = \{z|[(\hat{B})_z\cap A]\subseteq A \}
$$
即$A$被$B$膨胀的结果是满足上式的所有位移$z$的点（前景像素点）的集合。

膨胀应用实例：桥接裂缝

```matlab
A=imread("broken_text.tif");
B=[0 1 0;1 1 1;0 1 0];
result=imdilate(A,B);
```

其中结构元素$B:\begin{bmatrix}
0 & 1 & 0 \\
1 & 1 & 1 \\
0 & 1 & 0
\end{bmatrix}$



### 腐蚀

效果：缩小图像中的物体。

设$A$：原始二值图像，$B$：结构元素，则$A$被$B$腐蚀定义为：
$$
A \ominus B = \{z|(B)_z \subseteq A\}
$$
即，将结构元素$B$相对于集合$A$进行平移，只要平移后的结构元素都包含在集合$A$中，则这些位移$z$的点的集合（前景像素点）为腐蚀结果。
如果结构元素取$\begin{bmatrix}
1 & 1 & 1 \\
1 & 1 & 1 \\
1 & 1 & 1
\end{bmatrix}$，腐蚀将使物体的边界沿周边减少一个像素。

腐蚀可以去除**小于结构元素**的物体。

腐蚀应用实例（MATLAB）：

```matlab
A=imread('wirebond.tif');
B=strel('square',11); %边长为11的方形结构元素
result=imerode(A,B);
```

## 开启和闭合

### 开启

效果：平滑物体的轮廓、断开较窄的狭颈、消除细的突出物。
$$
A \circ B=(A\ominus B)\oplus B
$$
即先用$B$对$A$腐蚀，然后用$B$对腐蚀结果进行膨胀。

性质：

- $A\circ B$是$A$的子集
- 若$C \subseteq D$，则$C\circ B\subseteq D\circ B$
- $(A\circ B)\circ B=A \circ B$

### 闭合

效果：同样平滑物体的轮廓，但弥合狭窄的断裂和细长的沟壑，消除小孔，并填补轮廓中的缝隙。
$$
A\bullet B=(A\oplus B)\ominus B
$$
即先用$B$对$A$膨胀，然后用$B$对腐蚀结果进行腐蚀。

性质：

- $A\bullet B$是$A$的子集
- 若$C \subseteq D$，则$C\bullet B\subseteq D\bullet B$
- $(A\bullet B)\bullet B=A \bullet B$

### 实例（MATLAB）

#### 实例1

```matlab
A=imread('shapes.tif');
B=strel('square',20);
result_open=imopen(A,B);
result_close=imclose(A,B);
result_open_close(result_open,B);
```

#### 实例2（去除指纹图像上的杂散点）

```
A=imread('noisy-fingerprint.tif');
B=strel('square',3);
result_open=imopen(A,B);
result_open_close=imclose(result_open,B);
```

## 形态学的主要应用

### 边界提取

图像$A$的边界$b(A)$定义为：
$$
b(A)=A-(A\ominus B)
$$
其中，$B$是适当的结构元素。

边界提取实例（MATLAB）：

```
A=imread('people.jpg');
B=strel('square',3);
result=A-imerode(A,B);
```

### 孔洞填充

孔洞：被**前景像素**连成的边框所包围的**背景区域**。

- 令$A$表示一个集合：其元素是$8$连通的边界，且每个边界包围一个孔洞；

- 令$X_0$表示一个与包含$A$的相同大小的阵列，其初始状态为：

  - 包含每个孔洞中的一个指定位置处的前景像素点；
  - 除上述的前景像素点外，其余元素均为背景像素点。

- 在给定$A$和$X_0$的前提下，采用前景像素填充$A$的所有空洞的过程如下：
  $$
  X_k=(X_{k-1}\oplus B)\cap A^c\quad k=1,2,3,\cdots
  $$

  - 其中，$B$是对称结构元素，$B=\begin{bmatrix}0 & 1 & 0 \\ 1 & 1 & 1 \\ 0 & 1 & 0\end{bmatrix}$；
  - 若$X_k=X_{k-1}$，则算法在迭代的第$k$步结束；
  - 集合$X_k$包含所有被填充的孔洞，$X_k$和$A$的并集则包含被填充的孔洞及其边界。

每一步运算中，膨胀结果与$A^c$的交集操作实现了将膨胀结果限制在感兴趣区域内，即条件膨胀。

*B对图像X的膨胀是B对X的**前景元素**的膨胀。*


# 图像缩放

## 图像缩放的变换公式

直接坐标公式：
$$
\begin{aligned}
x&=c_xx_0 \\
y&=c_yy_0
\end{aligned}
$$
齐次坐标公式：
$$
\begin{bmatrix}
x & y & 1
\end{bmatrix}
=
\begin{bmatrix}
x_0 & y_0 & 1
\end{bmatrix}
\begin{bmatrix}
c_x & 0 & 0\\
0 & c_y & 0\\
0 & 0 & 1
\end{bmatrix}
=
\begin{bmatrix}
c_xx_0 & c_yy_0 & 1
\end{bmatrix}
$$

## 图像的缩小

### 图像缩小的实现方法

一个简单方法是等间隔地选取样本（重采样）。

以采样间隔$2$为例：
$$
\begin{bmatrix}
0 & 0 & 0 & 0 & 0 & 0 \\
0 & 1 & 0 & 2 & 0 & 3 \\
0 & 0 & 0 & 0 & 0 & 0 \\
0 & 4 & 0 & 5 & 0 & 6  \\
0 & 0 & 0 & 0 & 0 & 0 \\
0 & 7 & 0 & 8 & 0 & 9
\end{bmatrix}_{6\times 6}
\longrightarrow
\begin{bmatrix}
1 & 2 & 3\\
4 & 5 & 6\\
7 & 8 & 9
\end{bmatrix}_{3\times 3}
$$
**算法步骤**：

1. 确定重采样的行和列（采样间隔）

   $M\times N\rightarrow c_xM\times c_yN$，采样间隔：
   $$
   k_x=\frac{1}{c_x}\quad k_y=\frac{1}{c_y}
   $$

2. 重采样
   $$
   G(x,y)=F(int(k_x\times x),int(k_y\times y))
   $$

## 图像的放大

### 图像放大的实现方法

当放大倍数$k$为整数时，可以采取如下例（放大$3$倍）所示的简单方法：
$$
\begin{bmatrix}
1 & 2\\
3 & 4
\end{bmatrix}_{2\times 2}
\longrightarrow
\begin{bmatrix}
1 & 1 & 1 & 2 & 2 & 2 \\
1 & 1 & 1 & 2 & 2 & 2 \\
1 & 1 & 1 & 2 & 2 & 2 \\
3 & 3 & 3 & 4 & 4 & 4 \\
3 & 3 & 3 & 4 & 4 & 4 \\
3 & 3 & 3 & 4 & 4 & 4 \\
\end{bmatrix}_{6\times 6}
$$
问题：容易出现马赛克效应。

**算法步骤**：

1. 计算放大后图像的大小

   $M\times N \rightarrow c_xM\times C_yN$

2. 求出放大的新图像像素值
   $$
   G(x,y)=F(\frac{x}{c_x},\frac{y}{c_y})
   $$

采用**插值法**解决映射坐标$(x/c_x,y/c_y)$在原图像空间中不存在的情况。
$$
\begin{bmatrix}
11 & 12 & 13 \\
21 & 22 & 23 \\
31 & 32 & 33
\end{bmatrix}_{3\times 3}
\longrightarrow
\begin{bmatrix}
11 & ? & 12 & ? & 13 & ? \\
21 & ? & 22 & ? & 23 & ?\\
31 & ? & 32 & ? & 33 & ?
\end{bmatrix}_{3\times 6}
$$

$$
\begin{aligned}
G(0,0)=F(0,0) \quad G(0,1)=F(0,0.5)=?\\
G(1,0)=F(1,0) \quad G(1,1)=F(1,0.5)=?\\
G(2,0)=F(2,0) \quad G(2,1)=F(2,0.5)=?
\end{aligned}
$$

### 最近邻插值

将放大后未知的像素点坐标换算到原始图像，与原始图像上邻近的$4$个像素点比较，最靠近邻近点的像素值即为该未知像素点的像素值。

**算法步骤**：

1. $(u,v)(G)\rightarrow(x+\Delta{x},y+\Delta{y})$；
2. 计算$(x+\Delta{x},y+\Delta{y})$与$(x,y)、(x,y+1)、(x+1,y)、(x+1,y+1)$之间的距离，取距离最短的点的像素值作为$(u,v)$的像素值。

### 双线性插值

将放大后未知的像素点坐标换算到原始图像，计算原始图像上$4$个邻近像素点$A、B、C、D$对$P$点的影响，$P$点灰度值由$4$个邻近点灰度值加权求和得到（权值可以用距离进行度量）。

**算法步骤**：

- 根据$(x+\Delta{x},y+\Delta{y})$确定：
  $$
  (x,y)、(x,y+1)、(x+1,y)、(x+1,y+1)
  $$
  
- 由$A、B$两点插值计算出$e$点的灰度值的$F(x,y+\Delta{y})$；

$$
\begin{aligned}
F(x,y+\Delta{y})&=\frac{\sqrt{(x-x)^2+((y+1)-(y+\Delta{y}))^2}}{\sqrt{(x-x)^2+((y+1)-y)^2}}F(x,y)\\
&\quad\ +\frac{\sqrt{(x-x)^2+((y+\Delta{y})-y)^2}}{\sqrt{(x-x)^2+((y+1)-y)^2}}F(x,y+1) \\
&= (1-\Delta{y})\times F(x,y)+\Delta{y}\times F(x,y+1)
\end{aligned}
$$

$$
\begin{bmatrix}
B(x,y+1) & \cdot & \cdot \\
e(x,y+\Delta{y}) & \cdot & \cdot\\
A(x,y) & \cdot & \cdot
\end{bmatrix}
$$



- 由$C、D$两点插值计算出$f$点的灰度值$F(x+1,y+\Delta{y})$；

$$
\begin{aligned}
F(x+1,y+\Delta{y})&=\frac{\sqrt{((x+1)-(x+1))^2+((y+1)-(y+\Delta{y}))^2}}{\sqrt{((x+1)-(x+1))^2+((y+1)-y)^2}}F(x+1,y)\\
&\quad\ +\frac{\sqrt{((x+1)-(x+1))^2+((y+\Delta{y})-y)^2}}{\sqrt{((x+1)-(x+1))^2+((y+1)-y)^2}}F(x+1,y+1) \\
&= (1-\Delta{y})\times F(x+1,y)+\Delta{y}\times F(x+1,y+1)
\end{aligned}
$$

$$
\begin{bmatrix}
\cdot & \cdot & D(x+1,y+1) \\
\cdot & \cdot & f(x+1,y+\Delta{y})\\
\cdot & \cdot & C(x+1,y)
\end{bmatrix}
$$



- 由$e、f$两点插值计算出$P$点的灰度值$F(x+\Delta{x},y+\Delta{y})$。

$$
\begin{aligned}
F(x+\Delta{x},y+\Delta{y})&=\frac{\sqrt{((x+1)-(x+\Delta{x}))^2+((y+\Delta{y})-(y+\Delta{y}))^2}}{\sqrt{((x+1)-x)^2+((y+\Delta{y})-(y+\Delta{y}))^2}}F(x,y+\Delta{y})\\
&\quad\ +\frac{\sqrt{((x+\Delta{x})-x)^2+((y+\Delta{y})-(y+\Delta{y}))^2}}{\sqrt{((x+1)-x)^2+((y+\Delta{y})-(y+\Delta{y}))^2}}F(x+1,y+\Delta{y}) \\
&= (1-\Delta{x})\times F(x,y+\Delta{y})+\Delta{x}\times F(x+1,y+\Delta{y})
\end{aligned}
$$

$$
\begin{bmatrix}
\cdot & \cdot & \cdot \\
e(x,y+\Delta{y}) & P(x+\Delta{x},y+\Delta{y}) & f(x+1,y+\Delta{y})\\
\cdot & \cdot & \cdot
\end{bmatrix}
$$



示例：
$$
\begin{bmatrix}
11 & 12 & 13 \\
21 & 22 & 23 \\
31 & 32 & 33
\end{bmatrix}_{3\times 3}
\longrightarrow
\begin{bmatrix}
? & ? & ? & ? & ? & ? \\
? & ? & ? & ? & ? & ?\\
? & ? & ? & ? & ? & ?
\end{bmatrix}_{3\times 6}
$$
放大比例$c_x=1,c_y=2$，新图像$G$的像素坐标：
$$
\begin{aligned}
X&=
\begin{bmatrix}
0 & 1 & 2
\end{bmatrix}
\\
Y&=
\begin{bmatrix}
0 & 1 & 2 & 3 & 4 & 5
\end{bmatrix}
\end{aligned}
$$
则在原图像$F$中的映射坐标：
$$
\begin{aligned}
X&=
\begin{bmatrix}
0 & 1 & 2
\end{bmatrix}
\\
Y&=
\begin{bmatrix}
0 & 0.5 & 1 & 1.5 & 2 & 2.5
\end{bmatrix}
\end{aligned}
$$
则可得$G$中部分点的像素值：
$$
\begin{bmatrix}
11 & ? & 12 & ? & 13 & ? \\
21 & ? & 22 & ? & 23 & ?\\
31 & ? & 32 & ? & 33 & ?
\end{bmatrix}_{3\times 6}
$$
对于$G(0,1)=F(0,0.5)$，$F(0,0.5)$的邻近像素：
$$
\begin{bmatrix}
F(0,0) & F(0,1) \\
F(1,0) & F(1,1)
\end{bmatrix}
$$
$\cdots\cdots$

### 双三次插值

**算法原理**：

- 未知像素点$P(u,v)(G)\rightarrow$ 原始图像空间$(x,y)$；

- 确定原始图像上的$16$个邻近像素点；

- 采用下式计算$P$点的灰度值$F(x,y)$：

$$
F(x,y)=\sum_{i=0}^3\sum_{j=0}^{3}a_{ij}x^iy^j
$$

其中，$16$个未知系数$a_{ij}$可由原始图像$(x,y)$处的$16$个邻近像素所确定的方程组进行求解。

**算法步骤**：

1. 坐标映射，确定原图像中的16个邻近点；
   $$
   \begin{bmatrix}
   \cdot &  & \cdot &  & \cdot &  & \cdot \\
   & & & & & & \\
   \cdot &  & \cdot &  & \cdot &  & \cdot \\
   & & & \bullet& & & \\
   \cdot &  & \cdot &  & \cdot &  & \cdot \\
   & & & & & & \\
   \cdot &  & \cdot &  & \cdot &  & \cdot \\
   \end{bmatrix}
   $$

2. 在$4$条**水平直线**上分别用三次多项式插值，计算点$A、B、C、D$处的灰度值；
   $$
   \begin{bmatrix}
   \cdot &  & \cdot & A\cdot & \cdot &  & \cdot \\
   & & & & & & \\
   \cdot &  & \cdot & B\cdot & \cdot &  & \cdot \\
   & & & \bullet& & & \\
   \cdot &  & \cdot & C\cdot & \cdot &  & \cdot \\
   & & & & & & \\
   \cdot &  & \cdot & D\cdot & \cdot &  & \cdot 
   \end{bmatrix}
   $$

   $$
   F(x,y)=\sum_{j=0}^{3}a_jy^j
   $$

   

3. 对$A、B、C、D$四点在**垂直方向**上再做三次多项式插值。
   $$
   \begin{bmatrix}
   \cdot &  & \cdot & A\cdot & \cdot &  & \cdot \\
   & & & & & & \\
   \cdot &  & \cdot & B\cdot & \cdot &  & \cdot \\
   & & & P\bullet& & & \\
   \cdot &  & \cdot & C\cdot & \cdot &  & \cdot \\
   & & & & & & \\
   \cdot &  & \cdot & D\cdot & \cdot &  & \cdot 
   \end{bmatrix}
   $$

   $$
   F(x,y)=\sum_{i=0}^{3}b_ix^i
   $$


# 图像边缘检测

## 概述

物体边界、表面方向的改变、不同的颜色、光照明暗的变化...

图像边缘是一组相连的像素集合，这些像素位于两个不同区域的边界上。边缘检测是一种典型的图像预处理过程。

### 图像的边缘模型

1. 台阶边缘

   在**1个像素**的距离上发生两个灰度级间理想的过渡。
   $$
   \begin{bmatrix}
   0 & 0 & 0 & 3 & 3 & 3
   \end{bmatrix}
   $$

2. 斜坡边缘

   边缘的宽度**不只1个像素宽**。
   $$
   \begin{bmatrix}
   0 & 0 & 0 & 1 & 2 & 3 & 3 & 3
   \end{bmatrix}
   $$

3. 屋顶边缘

   通过一个区域的线的模型，边缘的宽度**由该线的宽度和尖锐度决定**。
   $$
   \begin{bmatrix}
   0 & 0 & 1 & 3 & 1 & 0 & 0
   \end{bmatrix}
   $$

### 无噪图像的导数与边缘的关系

$$
\begin{bmatrix}
f & &0 & 0 & 1 & 2 & 3 & 3 & 3 \\
f' & &0 & 0 & 1 & 1 & 1 & 0 & 0 \\
f'' & &0 & 0 & 1 & 0 & -1 & 0 & 0
\end{bmatrix}
$$

- 一阶导数的幅值可检测图像中某个点处是否存在一个边缘（峰值为边缘的位置）；

- 二阶导数的符号可用于确定一个边缘像素位于该边缘偏暗的一侧还是偏亮的一侧；
- 对于图像中的每条边缘，二阶导数生成两个值，同时二阶导数的零交叉点可用于定位粗边缘的中心。

## 基本的边缘检测技术

### 图像梯度及其性质

梯度$\nabla f$、梯度的幅值$|\nabla f|$、梯度的方向$\alpha(x,y)$：
$$
\begin{gather*}
\nabla f=
\begin{bmatrix}
g_x & g_y
\end{bmatrix}^T
=
\begin{bmatrix}
\frac{\partial f}{\partial x} & \frac{\partial f}{\partial y}
\end{bmatrix}^T
\\
|\nabla f|=\sqrt{g_x^2+g_y^2}=\sqrt{(\frac{\partial f}{\partial x})^2 + (\frac{\partial f}{\partial y})^2}
\\
\alpha(x,y)=\arctan[\frac{g_y}{g_x}]
\end{gather*}
$$

任意点$(x,y)$处边缘的方向与该点处梯度的方向$\alpha(x,y)$正交。

### 梯度算子——直接差分算子

$$
\begin{aligned}
g_x=f(x+1,y)-f(x,y) \\
g_y=f(x,y+1)-f(x,y)
\end{aligned}
$$

直接差分模板：
$$
\begin{bmatrix}
\underline{-1} & 0\\
1 & 0
\end{bmatrix}
\quad
\begin{bmatrix}
\underline{-1} & 1\\
0 & 0
\end{bmatrix}
$$
直接差分算子仅能检测**水平、垂直方向**的边缘。

### 梯度算子——Roberts算子

$$
\begin{aligned}
g_x=f(x+1,y+1)-f(x,y) \\
g_y=f(x+1,y)-f(x,y+1)
\end{aligned}
$$

Roberts模板：
$$
\begin{bmatrix}
\underline{-1} & 0\\
0 & 1
\end{bmatrix}
\quad
\begin{bmatrix}
\underline{0} & -1\\
1 & 0
\end{bmatrix}
$$
Roberts算子可用于检测**对角线方向**的边缘。

### 梯度算子——Prewitt算子

Prewitt模板：
$$
\begin{bmatrix}
-1 & -1 & -1 \\
0 & \underline{0} & 0\\
1 & 1 & 1
\end{bmatrix}
\quad
\begin{bmatrix}
-1 & 0 & 1 \\
-1 & \underline{0} & 1\\
-1 & 0 & 1
\end{bmatrix}
$$

### 梯度算子——Sobel算子

Sobel模板：
$$
\begin{bmatrix}
-1 & -2 & -1 \\
0 & \underline{0} & 0\\
1 & 2 & 1
\end{bmatrix}
\quad
\begin{bmatrix}
-1 & 0 & 1 \\
-2 & \underline{0} & 2\\
-1 & 0 & 1
\end{bmatrix}
$$

### 梯度算子——用于检测对角边缘的Prewitt、Sobel算子

对上述的Prewit模板和Sobel模板作出修改，以便它们沿对角线方向有最大的响应。

用于检测对角边缘的Prewitt模板：
$$
45\degree 方向梯度
\begin{bmatrix}
0 & 1 & 1 \\
-1 & \underline{0} & 1\\
-1 & -1 & 0
\end{bmatrix}
\quad
-45\degree 方向梯度
\begin{bmatrix}
-1 & -1 & 0 \\
-1 & \underline{0} & 1\\
0 & 1 & 1
\end{bmatrix}
$$
用于检测对角边缘的Sobel模板：
$$
45\degree 方向梯度
\begin{bmatrix}
0 & 1 & 2 \\
-1 & \underline{0} & 1\\
-2 & -1 & 0
\end{bmatrix}
\quad
-45\degree 方向梯度
\begin{bmatrix}
-2 & -1 & 0 \\
-1 & \underline{0} & 1\\
0 & 1 & 2
\end{bmatrix}
$$
一些对边缘检测不必要的细节往往表现为噪声，处理方法为：对图像**进行平滑处理后再进行边缘检测**。

参考程序（MATLAB）：

```matlab
im=im2double(imread('building.tif'));
im=filter2(fspecial('average',5),im);
template=[-1 -2 -1;0 0 0;1 2 1];
gx=abs(filter2(template,im));
gy=abs(filter2(template,im));
imGrad=gx+gy;
subplot(2,2,1);imshow(im);
subplot(2,2,2);imshow(gx);
subplot(2,2,3);imshow(gy);
subplot(2,2,4);imshow(imGrad);
```

```matlab
im=im2double(imread('building.tif'));
im=filter2(fspecial('average',5),im);
template45=[0 1 2;-1 0 1;-2 -1 0];
template135=[-2 -1 0;-1 0 1;0 1 2];
grad45=abs(filter2(template45,im));
grad135=abs(filter2(template135,im));
subplot(3,1,1);imshow(im);
subplot(3,1,2);imshow(grad45);
subplot(3,1,3);imshow(grad135);
```

## 先进的边缘检测技术

### Marr-Hildreth（马尔-希尔德雷斯）边缘检测器

#### 基于二阶微分（导数）的边缘检测技术——拉普拉斯算子

图像$f$在$(x,y)$处的拉普拉斯变换定义为：
$$
\nabla^2f=\frac{\partial^2f}{\partial x^2}+\frac{\partial^2f}{\partial y^2}
$$
其中：
$$
\begin{aligned}
\frac{\partial^2f}{\partial x^2}=f(x+1,y)+f(x-1,y)-2f(x,y) \\
\frac{\partial^2f}{\partial y^2}=f(x,y+1)+f(x,y-1)-2f(x,y)
\end{aligned}
$$
拉普拉斯模板：
$$
\begin{bmatrix}
0 & -1 & 0\\
-1 & \underline{4} & -1\\
0 & -1 & 0
\end{bmatrix}
$$

- 优点：

  - 可以利用零交叉的性质进行边缘定位；
    $$
    \begin{bmatrix}
    f'' & & 0 & 0 & -1 & 1 & 0 & 0
    \end{bmatrix}
    $$
    *连接$-1$和$1$，与轴线相交的点即为零交叉点。*

  - 可以确定一个像素是在边缘暗的一边还是亮的一边。

- 缺点：

  - 对噪声具有敏感性；
  - 幅值产生双边缘；
  - 不能检测边缘的方向。

#### Marr-Hildreth边缘检测器的提出及实现

Marr-Hildreth边缘检测算法由LoG滤波器（*高斯-拉普拉斯滤波器*）与输入图像$f(x,y)$卷积组成，即
$$
g(x,y)=[\nabla^2G(x,y)]*f(x,y)
$$
然后，寻找零交叉来确定$f(x,y)$中边缘的位置，由于微分、卷积运算均为线性操作，则上式表示为：
$$
g(x,y)=\nabla^2[G(x,y)*f(x,y)]
$$
即，先使用一个高斯平滑滤波器平滑图像，然后对该结果执行拉普拉斯变换，故Marr-Hildreth边缘检测算法**实现步骤**如下：

1. 使用高斯滤波器对输入图像进行平滑滤波；
2. 计算由第一步骤得到的图像的拉普拉斯变换；
3. 寻找第二步骤所的图像的零交叉（由此得到的边缘为一个像素宽）。

满足上述要求的算子是$\nabla^2G$（高斯拉普拉斯算子，简称LoG算子），其中：

1. $\nabla^2$是拉普拉斯算子：
   $$
   \frac{\partial^2}{\partial x^2}+\frac{\partial^2}{\partial y^2}
   $$

2. $G$是标准差为$\sigma$的二维高斯函数：
   $$
   G(x,y)=e^{-\frac{x^2+y^2}{2\sigma^2}}
   $$

即：
$$
\begin{aligned}
\nabla^2G(x,y)&=\frac{\partial^2G(x,y)}{\partial x^2}+\frac{\partial^2G(x,y)}{\partial y^2}\\
&=\frac{\partial}{\partial x}[\frac{-x}{\sigma^2}e^{-\frac{x^2+y^2}{2\sigma^2}}]+\frac{\partial}{\partial y}[\frac{-y}{\sigma^2}e^{-\frac{x^2+y^2}{2\sigma^2}}] \\
&=[\frac{x^2}{\sigma^4}-\frac{1}{\sigma^2}]e^{-\frac{x^2+y^2}{2\sigma^2}}+[\frac{y^2}{\sigma^4}-\frac{1}{\sigma^2}]e^{-\frac{x^2+y^2}{2\sigma^2}}\\
&=\frac{x^2+y^2-2\sigma^2}{\sigma^4}e^{-\frac{x^2+y^2}{2\sigma^2}}
\end{aligned}
$$
近似的$5\times 5$模板（该近似不唯一）：
$$
\begin{bmatrix}
0 & 0 & -1 & 0 & 0 \\
0 & -1 & -2 & -1 & 0 \\
-1 & -2 & 16 & -2 & -1 \\
0 & -1 & -2 & -1 & 0 \\
0 & 0 & -1 & 0 & 0
\end{bmatrix}
$$
通过该模板得到的图像来寻找零交叉点以进行图像的边缘检测。

实例（MATLAB）：

```matlab
edge_LoG=edge(inputImage,'log',T,sigma)
```

#### 寻找零交叉的方法

判定图像$g(x,y)$的任意像素$p$是否为零交叉点的一种方法如下：

1. 在图像$g(x,y)$中找到一个以$p$为中心的$3\times 3$邻域；
2. $p$像素为零交叉点意味着至少有**两个相对**的邻域像素的符号不同，有4种要测试的情况：左/右、上/下和两个对角；
3. 如果相对的两个邻域像素的符号不同，而且它们的像素值与$p$的像素值的绝对值差值超过指定的**阈值**。那么，$p$即为一个零交叉像素。

对于阈值为$0$的零交叉检测，会产生严重的意大利通心粉效应：所有的边缘都形成闭环，使用正阈值可避免闭环边缘。

- 优点：
  - 零交叉点图像中的边缘比梯度边缘细；
  - 抑制噪声能力和反干扰性能好。
- 缺点：
  - 边缘由零交叉点构成，而零交叉点计算比较复杂。

### Canny（坎尼）边缘检测器

坎尼边缘检测器是基于一阶微分的边缘检测方法。

**实现步骤**：

1. 用一个大小为$n\times n$的高斯滤波器平滑输入图像（**$n$的取值应为大于或等于$6$倍高斯滤波器的标准差的最小奇整数**）；
2. 计算滤波后图像的梯度幅值和方向角度；
3. 对梯度幅值执行**非极大值抑制**（剔除伪边缘点，保留候选边缘点）；
4. 对非极大值抑制的结果使用**双阈值**检测边缘（从候选边缘点中选择真实边缘点）；
5. 采用**连接分析**对双阈值边缘检测结果进行连接（得到连续完整的边缘）。

#### 非极大值抑制（Non-Maxima Suppression, NMS）

仅保留梯度幅值图像$M(x,y)$的极大值（严格上，保留梯度方向的极大值点），以实现边缘细化。

局部极值（$B$）周围存在相近数值的点（$A$、$C$），通过非极大值抑制选择适合的极值点作为边缘点.
$$
\begin{bmatrix}
& & & & & & \cdot B& & & & \\
& & & & & & & \cdot C& & & \\
& & & & & \cdot A& & & & & \\
& & & & & & & & & & \\
Th-&-&-&-&-&-&-&-&-&-&- \\
& & & & & & & & & & \\
& & & & & & & & & & \\
& & & & & & & & & & \\
& & & & & & & & & & \\
& & & &\cdot & & & & \cdot& & \\
&\cdot & &\cdot & & & & & &\cdot & \\
\cdot& &\cdot & & & & & & & & \cdot\\
\end{bmatrix}
$$
**实现步骤**：（假设仅保留梯度幅值极大值的结果为$N(x,y)$）

1. 将$N(x,y)$初始化为原始的梯度幅值图像$M(x,y)$；
2. 对于每个点$N(x,y)$，在梯度方向和反梯度方向各找$n$个像素点，若$N(x,y)$不是这些点中的最大点，则将$N(x,y)$置零，否则保持$N(x,y)$不变。

#### 对NMS结果使用双阈值检测边缘

**检测过程**：

1. 指定两个阈值$T_H、T_L$：$T_H\gt T_L$（建议高阈值与低阈值比率为$2:1$或$3:1$）；
2. 使用高阈值$T_H$检测边缘，得到高阈值边缘图$E_H(x,y)$（边缘点少但可靠）；
3. 使用低阈值$T_L$检测边缘，得到低阈值边缘图$E_L(x,y)$（边缘点多但错误检测率高）。

#### 对双阈值边缘检测结果进行边缘连接

**连接过程**：

1. 将高阈值边缘图$E_H(x,y)$中相连的边缘点输出为一副边缘图像$E(x,y)$；
2. 对于$E(x,y)$中每条边，从端点出发在低阈值边缘图$E_L(x,y)$中寻找其延长的部分，直至与$E(x,y)$中另外一条边的端点相连（8连通性），否则认为$E_L(x,y)$中没有它延长的部分；
3. 将$E(x,y)$作为结果输出。



Canny边缘检测实例（MATLAB）：

```matlab
edge_LoG=edge(inputImage,'canny',[T1 T2],sigma)
```
