# Hexo中NexT主题下基于Fancybox的相册页面实现

最近想给博客写一个相册界面，在网络找了一些实现方式但都不太满意，然而自己的前端基础几乎为零，只能拿此前生成的博客页面里的HTML源代码来参考，正好Fancybox也非常合适。找学过前端的室友请教了一下之后，魔改了一番此前博文的HTML代码，效果还不错，故在此分享一下我的实现方式。
<!--more-->  
*本文所提出的方案中，图片的存储基于[阿里云OSS对象存储](https://oss.console.aliyun.com/)实现。*
# 效果图
![](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/Hexo%E7%9B%B8%E5%86%8C%E9%A1%B5%E9%9D%A2/GIF.gif)
# 缩略图生成
考虑到原图像的占用空间较大，直接在页面引入原图可能会严重拖慢内容加载速度，故对于原图像可以先进行适当的缩小以减少占用空间，而在点击图像后查看大图的情况下再加载原图。  
这里提供一个我个人实现的图片裁切程序，基于Python+OpenCV实现，也可以[点击此处直接下载](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/Hexo%E7%9B%B8%E5%86%8C%E9%A1%B5%E9%9D%A2/run.py)。  
```Python
import os
import cv2

def resize(img, t):
    h, w = img.shape[0], img.shape[1]

    if(h > w):
        img = cv2.copyMakeBorder(img, 0, 0, int((h - w) / 2), int((h - w) / 2), cv2.BORDER_CONSTANT, value=[255, 255, 255])
        img = cv2.resize(img, (t, t))

    else:
        img = cv2.copyMakeBorder(img, int((w - h) / 2), int((w - h) / 2), 0, 0, cv2.BORDER_CONSTANT, value=[255, 255, 255])
        img = cv2.resize(img, (t, t))
    
    return img


imgList = os.listdir(f"./raw")
for i in imgList:
    img = cv2.imread(f"./raw/{i}")
    cv2.imwrite(f"./images/{i}", img)
    img = resize(img, 512)
    cv2.imwrite(f"./images-s/{i}", img)
```
此段程序的实现逻辑是对于**raw文件夹**下的每一张图片，先在**images文件夹**下保存图片（*可以抹除图片中的地理位置等信息*），再对图片进行如下处理：
1. 获取图像的宽度、高度，并比较大小；
2. 根据比较的结果将较小的一边进行填充以成为一张正方形的图片；
3. 将图片缩放到目标宽度，并保存在**images-s文件夹**下。  

文件树如下：
```
./
├─images
├─images-s
├─raw
└─run.py
```
将需要处理的图片放在**raw文件夹**后执行**run.py**即可，会在images文件夹和images-s文件夹中分别生成**同名的原图和缩略图**。

# 图片存储
由于相关文档已经非常详细，故关于阿里云OSS对象存储的基本使用本文不再赘述。  
对于原图和缩略图的存储，我在Bucket中建立了两个文件夹：gallery和gallery-s，分别存储原图和缩略图。  
建立完成后，将上一步骤中的处理结果分别放在两个文件夹中即可。

# 页面生成
首先：
- 在Hexo中生成新的Page
```
hexo new page gallery
```
- 在Next的主题配置文件中的menu项加入相关链接
```
menu: 
  + gallery: /gallery/ || fa fa-camera
```
- （可选）languages下zh-CN.yml文件中加入对应中英文对照。

# 页面样式
在```root/source/_data/styles.styl```中加入如下代码：
```CSS
/*gallery*/

.gallery-page {
	margin-top: -50px;
}
.img-list,
.gallery-list {
	display: flex;
	flex-direction: row;
	flex-wrap: nowrap;
	align-items: flex-start;
}
.img-column {
	display: flex;
	flex-direction: column-reverse;
}
.img-column a,
.gallery-column a {
	border-bottom: 0px;
}
.gallery-item {
	margin-bottom: -50px
}
.gallery-item p {
	margin: -25px auto -10px;
	max-width: 50%;
	text-align: center;
	font-size: 15px;
	color: $black-deep;
	background: rgba(255,255,255,.3);
	border-radius: 7px;
	border: 1px solid $black-deep;
	box-shadow: 0 8px 20px -8px rgba(0,0,0,.3);
}
.posts-expand .post-body .gallery-column a img {
	height: 250px;
	width: 300px;
	object-fit: cover;
}
@media (max-width: 767px){
	.gallery-item p {
		min-width: 75px;
		font-size: 13px;
	}
}
```

## 为图片四周加上阴影
可在源代码中加入如下样式：
```CSS
.img {
    -webkit-box-shadow: 0 0 5px rgba(202, 203, 203, .5);
    -moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
}
```
*注：自从 NexT-7.3.0 开始，官方推荐采用数据文件将配置与主题分离，以在不修改主题源码的同时完成选项配置、自定义布局、自定义样式，便于后续 NexT 版本更新。*

# 页面内容
Fancybox文档中给出了一段基础代码示例：
```HTML
<a href="image.jpg" data-fancybox data-caption="My caption">
	<img src="thumbnail.jpg" alt="" />
</a>
```
其中image.jpg为原图，thumbnail.jpg为缩略图。  
下面给出我所修改的代码示例（可直接在对应的md文件中编写）：
```HTML
<link rel="stylesheet" href="//cdn.jsdelivr.net/gh/fancyapps/fancybox@3/dist/jquery.fancybox.min.css">
<script src="//cdn.jsdelivr.net/gh/fancyapps/fancybox@3/dist/jquery.fancybox.min.js"></script>
<script src="//cdn.jsdelivr.net/npm/jquery@3/dist/jquery.min.js"></script>

<div class="gallery-page">
  <!--2-->
  <div class="img-list">
    <a class="img" data-fancybox="gallery" href="6.jpg" data-caption="caption 6">
      <img src="6-s.jpg">
      <p class="image-caption">6</p>
    </a>
  	&nbsp;
    <a class="img" data-fancybox="gallery" href="5.jpg" data-caption="caption 5">
      <img src="5-s.jpg">
      <p class="image-caption">5</p>
    </a>
  	&nbsp;
    <a class="img" data-fancybox="gallery" href="4.jpg" data-caption="caption 4">
      <img src="4-s.jpg">
      <p class="image-caption">4</p>
    </a>
  </div>
  <br>
  
  <!--1-->
  <div class="img-list">
    <a class="img" data-fancybox="gallery" href="3.jpg" data-caption="caption 3">
      <img src="3-s.jpg"></a>
      <p class="image-caption">3</p>
  	&nbsp;
  	<a class="img" data-fancybox="gallery" href="2.jpg" data-caption="caption 2">
      <img src="2-s.jpg"></a>
      <p class="image-caption">2</p>
  	&nbsp;
  	<a class="img" data-fancybox="gallery" href="1.jpg" data-caption="caption 1">
      <img src="1-s.jpg"></a>
      <p class="image-caption">1</p>
  </div>
</div>
```
# 不足之处
- 图片的生成、上传及页面HTML代码的编辑较为繁琐；
- 图片无法根据页面宽度等自适应换行，在小屏幕设备上缩略图可能比较拥挤。

# 参考链接
- [Hexo-NexT 实现相册 | 小丁的个人博客](https://tding.top/archives/607c3b85.html)
