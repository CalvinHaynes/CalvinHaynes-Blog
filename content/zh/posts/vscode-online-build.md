---
keywords:
- "Vscode"
- "Ipad"
- "code-server"
- "阿里云"
title: "上课摸鱼必备 -- Vscode网页版的搭建教程"
date: 2021-08-05T20:41:23+08:00
lastmod: 2021-08-05T20:41:23+08:00
description: "使用code-server搭建在线运行的Vscode"
draft: false 
author: Calvin Haynes
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
tags:
- "Vscode"
categories: "geek-tools"
img: "https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image-20210610155903032.3hg5z4x2jgw0.png"
---
## 前言

​		上课想练练数据结构与算法？或者就是想玩玩儿Vscode？或者有一个自己的服务器，但是觉得没有利用到极致？那么这篇文章将带你搭建一个在线版的Vscode，利用浏览器实现全平台使用Vscode，管你什么手机，Pad，电脑，板砖，咳咳，整就完了！！！

​		文章中所有的超链接都是很不错的资源，建议都要仔细看看，为了不让文章那么太长，所以我用了不少超链接。

> 本文搭建环境：开源项目[code-server](https://github.com/cdr/code-server)，一台服务器（至少一核2G才能有比较流畅的效果）

**如果本文对你有帮助的话，还望关注，点赞，转发，收藏，谢谢咯。**

## （一）运行效果

![image-20210610155903032](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image-20210610155903032.3hg5z4x2jgw0.png)

> 这个Vscode在线版是运行在我买的阿里云学生机的9999端口的，毕竟9.9一月，对于学生党很友好，我的个人博客也搭在上面的，性能一般，但是也很够用了。

## （二）基础配置

#### **1 - 下载code-server到服务器上**

> 进到服务器的SSH中，这个只要你买了服务器应该都可以用SSH的，服务器还没买的，也不会用服务器的，看以下几篇文章（其实不限制与阿里云的，不是推广阿里云哈，其他云怎么用大家自行选购，因为我用的是阿里云，所以这几篇文章也都是阿里云的一些使用教程）：
>
> - https://zhuanlan.zhihu.com/p/368487727
> - https://www.zhihu.com/search?type=content&q=%E9%98%BF%E9%87%8C%E4%BA%91%E5%AD%A6%E7%94%9F%E6%9C%BA%E6%95%99%E7%A8%8B（自己挑着看）
> - https://blog.csdn.net/u011002997/article/details/83933365
>
> 官网上应该也还有比较完善的使用手册啥的，深入玩一下的话，建议自己多研究研究，上面这几篇文章也是我大体看上去不错的，要想明白究竟怎么用的还是要自己用好搜索引擎。

```linux
wget https://github.com/cdr/code-server/releases/download/v3.10.2/code-server-3.10.2-linux-amd64.tar.gz
```

> 这一步下载速度可能会很慢，甚至中途失败，可以考虑挂代理，不会Linux下挂代理的，看我下面的骚操作

当然，你最好有一个梯子，这样总归是要更快和更稳定的。

下面我将演示如何在Windows下下载code-server再传到服务器上：

1. 首先我想介绍一下我使用过那么多的SSH最好用的一款软件：**[Termius](https://www.termius.com/)**

   ​	这个软件是真正的全平台，而且简直是我这种颜值控福利，终端各种皮肤，贼好看，如果你有幸申请到[Github学生包](https://blog.csdn.net/u012195214/article/details/87214085)的话，还有其他不少福利。

   ​	[关于Termius的使用教程](https://www.duangvps.com/archives/417)

2. 在Windows下载code-server的压缩包

   [点击这个链接](https://github.com/cdr/code-server/releases)

   再点这个，就开始下载了

   ![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.6m7u4m3ya700.png)

3. 下载完压缩包之后，找到下载的位置，然后就要介绍Termius的[SFTP](https://cloud.tencent.com/developer/article/1506251#:~:text=%E4%BB%80%E4%B9%88%E6%98%AFSFTP%EF%BC%9F,SFTP%E6%98%AF%E4%B8%80%E7%A7%8D%E5%AE%89%E5%85%A8%E7%9A%84%E6%96%87%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE%EF%BC%8C%E4%B8%80%E7%A7%8D%E9%80%9A%E8%BF%87%E7%BD%91%E7%BB%9C%E4%BC%A0%E8%BE%93%E6%96%87%E4%BB%B6%E7%9A%84%E5%AE%89%E5%85%A8%E6%96%B9%E6%B3%95%EF%BC%9B%E5%AE%83%E7%A1%AE%E4%BF%9D%E4%BD%BF%E7%94%A8%E7%A7%81%E6%9C%89%E5%92%8C%E5%AE%89%E5%85%A8%E7%9A%84%E6%95%B0%E6%8D%AE%E6%B5%81%E6%9D%A5%E5%AE%89%E5%85%A8%E5%9C%B0%E4%BC%A0%E8%BE%93%E6%95%B0%E6%8D%AE%E3%80%82%20SFTP%E8%A6%81%E6%B1%82%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%94%A8%E6%88%B7%E5%BF%85%E9%A1%BB%E7%94%B1%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%BF%9B%E8%A1%8C%E8%BA%AB%E4%BB%BD%E9%AA%8C%E8%AF%81%EF%BC%8C%E5%B9%B6%E4%B8%94%E6%95%B0%E6%8D%AE%E4%BC%A0%E8%BE%93%E5%BF%85%E9%A1%BB%E9%80%9A%E8%BF%87%E5%AE%89%E5%85%A8%E9%80%9A%E9%81%93%EF%BC%88SSH%EF%BC%89%E8%BF%9B%E8%A1%8C%EF%BC%8C%E5%8D%B3%E4%B8%8D%E4%BC%A0%E8%BE%93%E6%98%8E%E6%96%87%E5%AF%86%E7%A0%81%E6%88%96%E6%96%87%E4%BB%B6%E6%95%B0%E6%8D%AE%E3%80%82)功能

   ​	选中你的服务器

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.6fubf7xwqu80.png)

​		先找到你本地压缩包的网址，选中你本地的压缩包，直接拖到服务器上就行（哎，真不错，我就是玩儿）

![GIF 2021-6-10 16-34-08](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/GIF 2021-6-10 16-34-08.1n12dp0c16ow.gif)

4. 传过去之后现在你可以到你的服务器中ls -a一下，看看它在不在

​	![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.2e00phmhdyck.png)

那么以上就是下载的全部内容了

#### 2 - 解压安装试运行（运行部分可以先不弄，下一步的更好用）

- 解压

```linux
tar -xvzf code-server-3.10.2-linux-amd64.tar.gz
```

- 可以改个名

```linux
mv code-server-3.10.2-linux-amd64 code-server
```

- 运行试下（建议先看下参数列表）

PS：得确保你开了9999端口，下面是我的服务器防火墙配置

> 为啥不用8080端口？[戳这](https://baike.baidu.com/item/8080%E7%AB%AF%E5%8F%A3#:~:text=8080%E7%AB%AF%E5%8F%A3%E6%98%AF%E8%A2%AB%E7%94%A8%E4%BA%8EWWW%E4%BB%A3%E7%90%86%E6%9C%8D%E5%8A%A1%E7%9A%84%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%AE%9E%E7%8E%B0%E7%BD%91%E9%A1%B5%E6%B5%8F%E8%A7%88%EF%BC%8C%E7%BB%8F%E5%B8%B8%E5%9C%A8%E8%AE%BF%E9%97%AE%E6%9F%90%E4%B8%AA%E7%BD%91%E7%AB%99%E6%88%96%E4%BD%BF%E7%94%A8%20%E4%BB%A3%E7%90%86%E6%9C%8D%E5%8A%A1%E5%99%A8%20%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E4%BC%9A%E5%8A%A0%E4%B8%8A%E2%80%9C%3A8080%E2%80%9D%20%E7%AB%AF%E5%8F%A3%E5%8F%B7,%E3%80%82%20%E5%8F%A6%E5%A4%96Apache%20Tomcat%20web%20server%E5%AE%89%E8%A3%85%E5%90%8E%EF%BC%8C%E9%BB%98%E8%AE%A4%E7%9A%84%E6%9C%8D%E5%8A%A1%E7%AB%AF%E5%8F%A3%E5%B0%B1%E6%98%AF8080%E3%80%82)

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.174au3x9jk80.png)

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.7jcmo9u92lw0.png)

```linux
cd code-server
export PASSWORD="你想设置的密码"
./code-server --port 9999 --host 0.0.0.0 --auth password
```

> - –port 9999 指定端口，缺省时为 8080
> - –host 0.0.0.0 允许公网访问，缺省时为 127.0.0.1，只能本地访问
> - –auth password 指定访问密码，可通过 export 命令设置，参数为 none 时不启用密码

- 可以看一下参数列表

```linux
./code-server --help
```

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.277j7h3h4x8g.png)

- 运行后，打开 Chrome 访问“服务器公网IP:端口”，效果图：

> 服务器公网IP去哪里查？  [戳这](https://blog.csdn.net/weixin_44226789/article/details/106396968)

![image-20210610164853020](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image-20210610164853020.4s0pnj2dndo0.png)



## （三）高级配置

​		我们都知道Linux是可以写shell脚本的，那么为了简化以上操作，也为了让其根据我们意愿后台运行或者终止，我们着手写两个脚本，start.sh和shut.sh(脚本是要写在code-server目录的奥)

脚本执行目标

- start.sh

  - 开启code-server，后台运行该进程
  - 记录当前进程的PID，也专门记录一个日志log文件便于以后查看
  - 将PID存到文件里面

  ```
  #start.sh
  export PASSWORD="412523"
  nohup ./code-server --port 9999 --host 0.0.0.0 --auth password > test.log 2>&1 &
  echo $! > save_pid.txt
  ```

- shut.sh

  - 读文件中的PID
  - 杀死进程

  ```
  #shut.sh
  kill -9 'cat save_pid.txt'
  ```


## （四）Ios端/IpadOS端的最佳使用方式

> 本来配置完以上，我们用任何设备，只要用浏览器就可以使用了，但是Ios端和IpadOS端有一个可以更加沉浸体验的软件，推荐给大家

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.41hexmd78n80.png)

- 以下是使用方法：

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.yi50np44r80.png)

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.jwjdy8ewin4.png)

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.1as8qmxwh0g0.png)

选好点**Save**就可以了，访问效果就如第一步运行效果的图

## （五）使用流程总结

- SSH登入服务器
- cd code-server
- ./start.sh
- 浏览器直接访问网址/Apple系列产品的serveditor
- 关了，免得一直占用我的服务器：./shut.sh

## （六）参考资料

[在Ipad上使用Vscode](https://blog.csdn.net/liteng607/article/details/106601569?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161529894516780269818215%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161529894516780269818215&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-106601569.first_rank_v2_pc_rank_v29&utm_term=ipad+Vscode)


