---
keywords: 
- 
title: "Hugo部署个人博客教程（GithubPages + 阿里云）"
date: 2021-08-05T00:22:49+08:00
lastmod: 2021-08-05T00:22:49+08:00
description: "Hugo搭建个人博客系列"
draft: false
author: Calvin Haynes
hideToc: false
enableToc: true
enableTocContent: true
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
tags:
-
categories: "hugo"
img: "https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/Wallpaper/35af5124880511ebb6edd017c2d2eca2.6mpycsk390k0.jpg"
---

# Hugo部署个人博客教程（GithubPages + 阿里云）

## 前言

经过种种尝试，还是决定选择了Hugo作为了搭建个人博客的框架，Hugo是目前可以搭建个人博客的框架中部署最快的，而且坑也相对很少，不过还是有的，近几天折腾了不少东西，也终于算是初步搭建完了Hugo+Zoo主题的[个人博客](https://blog.calvinhaynes.top/)

在本文中我会讲解部署的详细过程，也会科普一些部署中的原理，毕竟我也刚刚接触，所以有些说的不对的地方也希望评论区的大佬来勘误，谢谢啦。

如果遇到部署问题，欢迎在评论区打出你的问题，我和知乎这个优秀社区的所有人都有可能回答你的疑问哦，本文如果对你有帮助的话，还希望多多点赞收藏转发，谢谢啦。

## 1 - 使用Hugo创建静态网站实战



#### 1、安装Hugo

博主用的是Windows系统，所以安装过程中我会基于Windows进行讲解，有关其他操作系统的安装方法可以参考官方的[文档](https://gohugo.io/getting-started/installing/)

Windows下的Hugo安装我推荐使用chocolatey包管理器进行安装，接触过Linux的朋友们都知道，Linux的各种包管理器，利用命令行就可以实现包的更新，删除等操作，Windows下也有类似的包管理器就是chocolatey了。



**安装过程详解**

- 安装chocolatey包管理器：在[官网](https://chocolatey.org/)点击 `Install Now` 即可下载

- 命令行中敲入`choco --version`，如果显示版本号证明你的 chocolatey 已经安装完毕，如果有误请检查环境变量

- 命令行中安装hugo：
  
  ```shell
  choco install hugo -confirm
  ```
  
  安装hugo-extended（扩展版本）：
  
  ```shell
  choco install hugo-extended -confirm
  ```
  
- 检查hugo是否安装成功：命令行中敲入`hugo version`



#### 2、初步建站实战（利用GithubPages进行部署）

- 在Github中建立一个仓库，仓库名为`用户名.github.io`

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.1l0hxny3q7b4.png)

- 将仓库clone到本地一个你想存放网站文件的文件夹

```shell
git clone <YOUR-REPOSITORY_URL>
```

- 创建网站：在存放你博客的根目录 <YOUR-REPOSITORY_URL> 中敲入以下命令

```shell
//注意site后面是一个点，不能忽略啊，.代表当前目录
hugo new site .
```

- 添加主题（以我博客的主题Zzo为例子）：

```shell
git init
git submodule add https://github.com/zzossig/hugo-theme-zzo.git themes/zzo
```

- 把 themes/zzo 下的 exampleSite文件夹的内容复制到你博客的根目录
- 在博客根目录中敲命令：（启动hugo服务器）

```shell
hugo server
```

- 点击[这里](http://localhost:1313/)查看测试（主题和示例网站中的markdown博客都正确显示了）



#### 3、GithubPages服务 + 云托管



##### 1 - 配置GithubPages服务进行个人博客的部署，使得所有人都可以访问你的博客站

- 在 config.toml 文件中设置 baseURL 为 Github Pages 服务的域名（用户名.github.io）

```toml
#config/config.toml

baseURL = "https://blog.calvinhaynes.top/" #未设置阿里云托管之前应该是 "https://用户名.github.io"
title = "Calvin Haynes's Blog"
theme = "zzo"
```

- 将更改过后的 github 仓库文件夹（就是你的博客根目录）推送到远端
- 在Github上设置GithubPages服务的一些参数

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.1ssfj1nz0jkw.png)

（Ps：有关 Custom domain 在以下的云托管中进行解释）



##### 2 - 云托管（阿里云）

​		博主买了阿里云的学生机，一个1核2GB内存的轻量应用服务器，所以将博客站云托管在这个服务器上，以下就是基于阿里云的教程，其他服务器也都差不太多，可以自行google，有关阿里云服务器购买和初步配置自行到官网查看文档吧。

- 拥有一个阿里云服务器还不够，你需要[购买注册](https://wanwang.aliyun.com/?spm=5176.19720258.J_8058803260.53.55d32c4axFU59f)一个你自己的专属域名。
- 拥有自己的专属域名之后，进入域名解析的[工作台](https://dns.console.aliyun.com/?spm=5176.12818093.ProductAndService--ali--widget-home-product-recent.dre0.5adc16d05LdZIH#/dns/domainList)，就可以看见你的域名了

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.7jhy38mugak0.png)

- 解析设置：（点击修改后各种参数的设置都有相关说明，其他问题也都可以在阿里云官网找到文档解释）

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.37z6o0rdyam0.png)

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.5faxpze1aqc0.png)

- 服务器绑定域名：

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.5u53qpc5aao0.png)

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.3992hz7hv360.png)

- 解析设置完后在你的博客根目录中的 static （静态文件存放的地方）中创建一个 CNMAE 文件（注意没有任何后缀）

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.q0epou32e1s.png)

- CNAME 文件中写入你设置的域名，我的就是 [blog.calvinhaynes.top](https://blog.calvinhaynes.top/) ，写完记得更新github库（CNAME文件就是给你这个Github库绑定域名用的，CNAME的全名就是 Canonical Name，意思是别名）

- 最后更改 Github pages 配置中的 Custom domain 为你设置的新域名就好咯

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.1ssfj1nz0jkw.png)

- 现在就可以输入你设置的域名进行访问咯，阿里云服务器在中国，所以访问速度快的一批！！！（别忘了config.toml文件的baseURL参数设置）



## 2 - 主题参数设置和个性化

​	这部分主要是看主题作者写的详细文档进行自己的设计，在Hugo官网中找到的比较热门的主题的Github库中都有相关的文档，建议小伙伴们自行查看学习，每个主题都有不同之处，我使用的是Zzo主题，它的官方文档在[这里](https://zzo-docs.vercel.app/)

​	我的Github仓库在[这里](https://github.com/CalvinHaynes/CalvinHaynes.github.io)，欢迎 fork 和查看配置文件



## 3 - 资源整理

  Zzo主题官方文档：https://zzo-docs.vercel.app/

  阿里云云解析DNS文档：https://help.aliyun.com/product/29697.html?spm=a2c4g.11186623.6.540.d0044e82AEgtJN

  

