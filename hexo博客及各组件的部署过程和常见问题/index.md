# Hexo博客及各组件的部署过程和常见问题

![](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/cover/hexo.png)
自从使用Hexo搭建个人博客以来，在建站、迁移以及服务器等方面都遇到了不少问题，本篇博文对所遇到的问题进行总结，会持续更新。
<!--more-->
# Linux设置定时任务
环境：CentOS 8
## 执行内容
　　新建文件_crond.sh，作为定时执行的内容。
  ```
  #!/bin/bash
  cd /www/blog/hexo
  git pull git@github.com:egname/egrepo.git

  #echo pull successfully > /home/gitpull.log
  ```

## crontab服务
　　启动crontab服务，CentOS版本不同，具体命令可能有所差异。
  ```
  systemctl start crond 
  ```
　　启动服务

  ```
  systemctl stop crond            # 关闭服务
  systemctl restart crond         # 重启服务     
  systemctl reload crond          # 重新载入配置
  systemctl status crond          # 状态
  ```
## 设置计时器
　　crontab 选项 参数  
　　选项:  
　　　　-e：编辑该用户的计时器设置；  
　　　　-l：列出该用户的计时器设置；  
　　　　-r：删除该用户的计时器设置；  
　　　　-u：指定要设定计时器的用户名称。
  ```
  crontab -e
  ```

 　　进入insert插入模式，以每五秒钟执行一次为例。ESC后输入wq保存并退出。
  ```
  */5 * * * * /root/_crond.sh
  ```

# node版本过高
　　具体Warning内容如下：
```
(node:15748) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:15748) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
(node:15748) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(node:15748) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:15748) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
```
　　解决方式：找到博客目录下*node_modules\stylus\lib\nodes\index.js*，在*index.js*下加入以下代码：
```
exports.lineno = null;
exports.column = null;
exports.filename = null;
```

# 博客迁移
## 1 环境安装
### git
#### 1 配置git
```
git config --global user.name "yourname"
git config --global user.email "youremail"
```
#### 2 生成、配置密钥
```
ssh-keygen -t rsa -C "youremail"
```
　　获取密钥内容后在GitHub上配置SSH Keys，此处略。
### npm、hexo、node.js
　　使用常规方式安装即可。
```
sudo apt install nodejs

sudo apt-get install npm

sudo npm install -g hexo-cli
```

## 2 文件迁移
　　从原设备上将Hexo博客文件夹迁移至新设备。
## 3 重新部署
### 1 安装插件
```
cd HexoBlog
npm install
```
### 2 清除缓存并重新生成
```
hexo clean
hexo g
hexo d
```

# 阿里云ECS连接出错
　　具体报错如下：
```
访问公网IP地址需要在实例安全组白名单中增加Workbench的服务器白名单:
xxx.xxx.xxx.xxx/24
xxx.xxx.xxx.xxx/24
```
　　尝试多种方案未果，使用WinSCP，可以正常连接。

# Valine评论系统
# 部分页面隐藏TOC栏
在对应界面的Markdown文件中设置：
```
toc: 
  enable: false
```
