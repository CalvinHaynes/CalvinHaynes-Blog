---
keywords: 
- 
title: "Hugo部署个人博客教程"
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

# Hugo部署个人博客教程（究极详细了)

## 前言

经过种种尝试，还是决定选择了Hugo作为了搭建个人博客的框架，Hugo是目前可以搭建个人博客的框架中部署最快的，而且坑也相对很少，不过还是有的，近几天折腾了不少东西，也终于算是初步搭建完了Hugo+Zoo主题的[个人博客](https://blog.calvinhaynes.top/)

在本文中我会讲解部署的详细过程，也会科普一些部署中的原理，毕竟我也刚刚接触，所以有些说的不对的地方也希望评论区的大佬来勘误，谢谢啦。

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

#### 2、建站过程（利用GithubPages进行部署）

- 在Github中建立一个仓库，仓库名为`用户名.github.io`

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.1l0hxny3q7b4.png)

- 将仓库clone到本地一个你想存放网站文件的文件夹

```shell
git clone <YOUR-REPOSITORY_URL>
```

- 创建网站：在你想存放网站文件的文件夹中敲入

```shell
//注意site后面是一个点，不能忽略啊，.代表当前目录
hugo new site .
```

- 添加主题（以我博客的主题Zzo为例子）：

```shell
git init
git submodule add https://github.com/zzossig/hugo-theme-zzo.git themes/zzo
```

