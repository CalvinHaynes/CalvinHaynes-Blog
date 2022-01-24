---
keywords:
- "Github"
- "Git"
title: "一文教你玩转Git和Github"
date: 2022-01-21T00:27:09+08:00
lastmod: 2022-01-21T00:27:09+08:00
description: "对Git和Github的使用和历史讲解，旨在于让读者掌握Git的精髓和意义"
draft: false 
author: Calvin Haynes
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
tags:
- "Github"
categories: "程序猿技能"
img: "https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/Wallpaper/Git.34wbc2gdtre0.webp"
---

## 前言

> Git is a [free and open source](https://git-scm.com/about/free-and-open-source) distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

Git 是一个免费开源的分布式版本控制系统，最初由 Linux 之父 Linus 开发，它诞生的原因就是 Linus 希望有一个工具便于管理 Linux 内核代码的更新迭代，而且分布式的结构比集中式更加稳定并且更利于团队协作，因为每个用户主机都可以拥有一个完整的版本库，而且大家可以工作于不同分支最后合并为一个分支。

## 1 - Git的安装

- 官网安装：[Downloads](https://git-scm.com/downloads)

- Windows 系统包管理器 [Chocolatey](https://chocolatey.org/) 安装

```powershell
choco install git
```

- Linux 系统安装（ubuntu）

```bash
apt-get install git
```

## 2 - Git 中一些术语的解释和理解

> 下面会对Git中各种术语进行一些通俗易懂的解释，方便大家快速明白Git中各个术语的功能和含义，具体术语解释的官方手册地址我在每个术语的括号中的英文原词加上了超链接，大家想仔细了解的话可以点进去认真阅读。

- **远程仓库名称**（[remote name](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93%E7%9A%84%E4%BD%BF%E7%94%A8)）：简单来说它就是远程仓库（**URL链接**）映射在**本地**的一个别名/标签，纯粹是为了方便在本地可以方便的选择远端（**通常这个别名会自定义为远程仓库内容的概括**）进行代码提交和更新，默认是 **origin**，也可以在本地通过命令修改，通常一个本地的文件夹可以关联很多个远程仓库，然后可以同时使用和更新多个远程仓库的代码。
- **分支**（branch）：在项目开发中，通常对于新功能开发或者很多部门分工合作，**分支和合并**功能是一个非常好的方式，通常有一个主要用于发布项目的主分支，然后其他从主分支创建出来的子分支会在开发测试完毕后**合并**（merge）到主分支中。
- **Git的一些操作术语**
  - **克隆**（[clone](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%8E%B7%E5%8F%96-Git-%E4%BB%93%E5%BA%93)）：从远端服务器将一个git项目下载到本地。
  - **合并**（[merge](https://www.atlassian.com/git/tutorials/using-branches/git-merge)）：将一个分支的内容合并到另一个分支上（这个分支一般是发布项目的[**稳定项目分支**](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E5%BC%80%E5%8F%91%E5%B7%A5%E4%BD%9C%E6%B5%81#:~:text=%E8%AE%B8%E5%A4%9A%E4%BD%BF%E7%94%A8%20Git%20%E7%9A%84%E5%BC%80%E5%8F%91%E8%80%85%E9%83%BD%E5%96%9C%E6%AC%A2%E4%BD%BF%E7%94%A8%E8%BF%99%E7%A7%8D%E6%96%B9%E5%BC%8F%E6%9D%A5%E5%B7%A5%E4%BD%9C%EF%BC%8C%E6%AF%94%E5%A6%82%E5%8F%AA%E5%9C%A8%20master%20%E5%88%86%E6%94%AF%E4%B8%8A%E4%BF%9D%E7%95%99%E5%AE%8C%E5%85%A8%E7%A8%B3%E5%AE%9A%E7%9A%84%E4%BB%A3%E7%A0%81%E2%80%94%E2%80%94%E6%9C%89%E5%8F%AF%E8%83%BD%E4%BB%85%E4%BB%85%E6%98%AF%E5%B7%B2%E7%BB%8F%E5%8F%91%E5%B8%83%E6%88%96%E5%8D%B3%E5%B0%86%E5%8F%91%E5%B8%83%E7%9A%84%E4%BB%A3%E7%A0%81%E3%80%82%20%E4%BB%96%E4%BB%AC%E8%BF%98%E6%9C%89%E4%B8%80%E4%BA%9B%E5%90%8D%E4%B8%BA%20develop%20%E6%88%96%E8%80%85%20next%20%E7%9A%84%E5%B9%B3%E8%A1%8C%E5%88%86%E6%94%AF%EF%BC%8C%E8%A2%AB%E7%94%A8%E6%9D%A5%E5%81%9A%E5%90%8E%E7%BB%AD%E5%BC%80%E5%8F%91%E6%88%96%E8%80%85%E6%B5%8B%E8%AF%95%E7%A8%B3%E5%AE%9A%E6%80%A7%E2%80%94%E2%80%94%E8%BF%99%E4%BA%9B%E5%88%86%E6%94%AF%E4%B8%8D%E5%BF%85%E4%BF%9D%E6%8C%81%E7%BB%9D%E5%AF%B9%E7%A8%B3%E5%AE%9A%EF%BC%8C%E4%BD%86%E6%98%AF%E4%B8%80%E6%97%A6%E8%BE%BE%E5%88%B0%E7%A8%B3%E5%AE%9A%E7%8A%B6%E6%80%81%EF%BC%8C%E5%AE%83%E4%BB%AC%E5%B0%B1%E5%8F%AF%E4%BB%A5%E8%A2%AB%E5%90%88%E5%B9%B6%E5%85%A5%20master%20%E5%88%86%E6%94%AF%E4%BA%86%E3%80%82%20%E8%BF%99%E6%A0%B7%EF%BC%8C%E5%9C%A8%E7%A1%AE%E4%BF%9D%E8%BF%99%E4%BA%9B%E5%B7%B2%E5%AE%8C%E6%88%90%E7%9A%84%E4%B8%BB%E9%A2%98%E5%88%86%E6%94%AF%EF%BC%88%E7%9F%AD%E6%9C%9F%E5%88%86%E6%94%AF%EF%BC%8C%E6%AF%94%E5%A6%82%E4%B9%8B%E5%89%8D%E7%9A%84%20iss53%20%E5%88%86%E6%94%AF%EF%BC%89%E8%83%BD%E5%A4%9F%E9%80%9A%E8%BF%87%E6%89%80%E6%9C%89%E6%B5%8B%E8%AF%95%EF%BC%8C%E5%B9%B6%E4%B8%94%E4%B8%8D%E4%BC%9A%E5%BC%95%E5%85%A5%E6%9B%B4%E5%A4%9A%20bug%20%E4%B9%8B%E5%90%8E%EF%BC%8C%E5%B0%B1%E5%8F%AF%E4%BB%A5%E5%90%88%E5%B9%B6%E5%85%A5%E4%B8%BB%E5%B9%B2%E5%88%86%E6%94%AF%E4%B8%AD%EF%BC%8C%E7%AD%89%E5%BE%85%E4%B8%8B%E4%B8%80%E6%AC%A1%E7%9A%84%E5%8F%91%E5%B8%83%E3%80%82)），如果存在**分支冲突**（[conflicts](https://www.atlassian.com/git/tutorials/using-branches/merge-conflicts)）的话需要手动解决再将冲突的文件重新merge。
  - **获取**（[fetch](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E8%BF%9C%E7%A8%8B%E5%88%86%E6%94%AF#:~:text=git%20branch%20%2Dvv-,%E6%8B%89%E5%8F%96,%E5%BC%8F%E5%9C%B0%E4%BD%BF%E7%94%A8%20fetch%20%E4%B8%8E%20merge%20%E5%91%BD%E4%BB%A4%E4%BC%9A%E6%9B%B4%E5%A5%BD%E4%B8%80%E4%BA%9B%E3%80%82,-%E5%88%A0%E9%99%A4%E8%BF%9C%E7%A8%8B%E5%88%86%E6%94%AF)）：fetch只从远端获取了远端相对于本地更新的数据，但并不会合并到本地的分支上，需要用户手动合并到本地的分支上。
  - **拉取**（[pull](https://dev.to/lydiahallie/cs-visualized-useful-git-commands-37p1#pull)）：`git pull` = `git fetch` + `git merge`**，从远程仓库获取最新版本并直接合并到相对应的本地分支上，一般同步远程仓库和本地仓库就用pull一条指令就够了。
  - **索引更改**（add）：将任何变化的文件（增加、删除、修改）放入暂存区。
  - **提交**（commit）：**将暂存区里的改动提交到本地的版本库**。每次使用`git commit`命令我们都会在本地版本库生成一个40位的哈希值，这个哈希值也叫commit-id，commit-id在**版本回退**的时候是非常有用的，它相当于一个**快照**,可以在未来的任何时候通过与`git reset`的组合命令回到这里
  - **上传**（push）：将本地的分支版本库上传到远程分支并合并。

> 相信大家第一次看了上面的术语有可能也不是很理解，所以在后面我会写一个很长的例子方便大家理解和梳理Git使用的整个流程，继续看下去吧！
## 3 - 玩转 Git CLI

> 对于 CLI （Command Line Interface 命令行接口）本身确实是不如 GUI 图形化界面的软件使用起来方便的，但是由于大部分**服务器**的操作系统都是 **Linux**，在远程连接服务器我们往往在命令行界面是最为轻便和稳定的，所以学习 Git 的 CLI 操作是必不可少的，而且计算机程序的命令行版本一直是我觉得计算机领域最优雅的部分，少了很多视觉跟踪和鼠标的点击操作，我们只需要集中于屏幕光标位置利用键盘说出我们想让计算机做的事就可以了，非常的优雅！
> 还有重要的一点就是，Git 的 CLI 包含 Git 的全部内容/所有功能，而 GUI 软件往往不能发挥 Git 的全部作用。
> 以下仅仅介绍一些常用的 Git 命令，完整的 CLI命令大家可以参考[官方文档](https://git-scm.com/docs/git#_git_commands)

### git clone

```bash
git clone <远程仓库链接> <本地目录>
//example
git clone https://github.com/CalvinHaynes/MIT6.S081-2020Fall.git /usr/gittest
```

### git pull

```bash
git pull <远程主机名> <远程分支名>:<本地分支名>
//example：将远程主机origin的master分支拉取过来，与本地的main分支合并。
git pull origin master:main
//如果上述没有冒号，则表示将远程origin仓库的master分支拉取下来与本地当前分支合并

```

### git commit

```bash
//一般需要在 git commit 之前使用 git add 来暂存修改的文件
//但是也可以采用下面的方式简化这一步骤
git commit -am "关于提交内容的解释" 
//此时再使用 git status 来检查文件状态就会得到：nothing to commit, working tree clean
```

### git push

```bash
git push <远程主机名> <本地分支名>:<远程分支名>
//如果本地分支名与远程分支名相同，则可以省略冒号：
git push <远程主机名> <本地分支名>
//example
git push origin main:master
```

### git status

```bash
//查看文件状态（是否需要暂存、是否需要提交）
git status
```

### git remote

```bash
//查看和管理本地的远程仓库名称
//1、查看所有远程仓库
git remote
git remote -v
//2、重命名远程仓库
git remote rename <old> <new>
//3、新增一个远程仓库
git remote add <name> <url>
```

### git branch

```bash
//1、查看所有branch
git branch -a
//2、创建新分支，所有新分支都是基于默认分支
git branch <新分支名字>
//3、删除分支
git branch -d <删除分支的名字>
//强制删除
git branch -D <删除分支的名字> 
//4、将当前分支重命名为＜branch＞
git branch -m <branch>
```

### git switch

> 注意一定要保证分支是**干干净净**的再切换分支（即文件改动均已commit和push了）

```bash
//1、切换分支
git switch <要切换到的分支名>
//2、创建一个新分支并切换到该新分支
git switch -c <branchName>
```

### git merge & git mergetool（解决冲突）

```bash
//1、将分支合并到默认分支上
git merge <被合并的分支>
//2、解决合并冲突问题，利用mergetool，具体配置请看[**这篇文章**](https://panmenglin.github.io/satellite-log/notes/tools/tools-merge-tool.html)
git mergetool
//如果使用的是vimdiff，请戳[这里](https://kinboyw.github.io/2018/10/09/Use-Vimdiff-As-Git-Mergetool/)
```

### 小结（上述命令精简）

- `git clone <远程仓库链接> <本地目录>`
- `git pull <远程主机名> <远程分支名>:<本地分支名>`
- `git commit -am "关于提交内容的解释"`
- `git push <远程主机名> <本地分支名>:<远程分支名>`
- `git status`
- `git remote -v`
- `git switch <要切换到的分支名>`
- `git branch -a`
- `git branch -d <删除分支的名字>`
- `git branch <新分支名字>`
- `git merge <被合并的分支>`
- `git mergetool`

## 4 - 玩转 Git GUI

> 关于Git的GUI （Graphic User Interface） 图形化界面软件有很多，在官网上有一个所有GUI的[清单](https://git-scm.com/downloads/guis/)，但是这其中我个人最为推荐的还是github官方的GUI：[Github Desktop](https://desktop.github.com/) ，有关于其他程序猿的生产力软件推荐可以看我下面这篇文章：
>
>
> [属于每一个程序猿和学生的一份高效率软件清单--第一弹:程序猿高效开发工具（持续更新）](https://zhuanlan.zhihu.com/p/412903121)

- **主要操作界面分区如下：**

![githubdesktop](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/githubdesktop.4vj8lsgzxmo0.webp)

- 其他操作建议查看官网的手册，中文版的看起来也很舒服：[https://docs.github.com/cn/desktop](https://docs.github.com/cn/desktop)

## 5 - 玩转 GitHub

> Github是全球最大的开源代码服务器，利用Github可以很好的托管开源作者的代码，也可以方便其他人使用和借鉴，**下面我将介绍几个让你的Github用的更爽的插件和建议**，关于Github的使用我下面不会再讲解，如何使用可以看官方指南：
>
>
> [开始使用 GitHub - GitHub Docs](https://docs.github.com/cn/get-started)

### 1 - [Octotree浏览器插件](https://chrome.google.com/webstore/detail/octotree-github-code-tree/bkhaagjahfmjljalopjnoealnfndnagc?hl=zh-CN)

> 这个插件使得在Github上阅读代码更舒服，将Github项目的文件夹分层显示在左侧像IDE一样直观，使用效果如下：

![octotree](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/octotree.681skfj0s9c0.webp)

### 2 - [Github Web IDE 浏览器插件](https://chrome.google.com/webstore/detail/github-web-ide/adjiklnjodbiaioggfpbpkhbfcnhgkfe)

> 这个插件集成了多个云平台IDE厂商的产品，可以进入在线IDE查看代码，甚至可以搭建云环境进行在线运行，使用效果如下：

![GithubWebIDE](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/GithubWebIDE.2zdt7mpjkrs0.webp)

### 3 - [Github Gist](https://gist.github.com/)

> Github Gist可以在远端存储代码段，相当于自己有了一个云存储空间用于存一些常用的配置文件的代码段，或者是一些算法的代码段，十分的不戳！！

![GithubGist](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/GithubGist.3b44g4p8qrm0.webp)

### 4 - Github 个性个人简介

> 这是Github的一个彩蛋吧算是，只要你创建一个和自己Github账户名相同的Github仓库（一定要增加README.md文件，语法格式戳[这里](https://docs.github.com/cn/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)）就可以在你的Github主页上展示下面这种效果的自我介绍（大家也可以去[我的Github主页](https://github.com/CalvinHaynes)上看下效果）：

![GithubSelfIntroduction](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/GithubSelfIntroduction.ltrbkjuzkxc.webp)

**具体教程可以参考以下这篇文章：**

[来弄一个Github的个人介绍页吧~ - 掘金](https://juejin.cn/post/6886826772550123534)

## 6 - Git工作流讲解
### Git基本工作流图片

![GitWorkFlow](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/GitWorkFlow.pjkkod3vq7k.webp)
### 通过一个故事理解Git工作流
小明是一个刚刚毕业的大学生，被一家知名的互联网企业签下了，由于第一次参与企业级大型项目，所以对于复杂的Git工作流十分的不熟悉，于是公司的老员工们纷纷来帮助他。

小明先是打开了公司开源的Git仓库地址，并利用Git工具clone下来了项目的代码，但是一看到上百条分支就头晕了，于是老员工Jack拿出了上面那张Git基本工作流的图片来一一为他讲解，Jack说：其实呢，之所以创建这么多的分支的原因就是为了方便基于老代码进行的优化和更新的新代码能够根据不同的变化占有一个分支，将一大个复杂的问题分类成一个个小的问题，从而下派给各个部门进行工作，这样不同的部门各自管理一条各自的分支，工作就井然有序啦！

小明说：哦，原来是这样，那这个图上这些分支的名字都代表什么含义呢？

这时另一名老员工Smith说：这张图呢是我们企业最初一张基本的Git工作流的图咯，现在已经非常复杂了，不过对于你初步理解整个流程是非常好的，那么这个**Master**分支，顾名思义就是我们的**主分支**啦，是我们所有部门整合起来的一个主分支，代表了我们项目整体的走向，每次一个**子分支**完成测试后就将其 **merge** 到 **Master** 分支然后再进行整体的测试，如果一个版本的功能已经完成全部测试，那么就将其发放到 **Release** 分支啦，也就是正式发行给大众的分支，就像咱们平时使用的某一款软件的一个版本更新一样（类比上图就相当于基于 **Master** 分支创建了一个 **Develop** 分支，然后 Develop 分支开发到一定程度后将其 **merge** 到 **Release** 分支和 **Master** 分支），那么图中的 **HotFix** 分支顾名思义就是处理一些突发bug的分支，进行快速修改再重新 **Merge** 到 **Master** 分支，**Feature** 分支就是代表一个个新功能的分支。

小明说：哦，原来如此，明白了，那我就利用手上的Git工具进行一下实践试试看咯！

另一名老员工小姐姐Mary说：嗯嗯，有什么问题可以继续问我们哦，哦对了，你有没有考虑过这样一种情况呢，如果在 **Master** 分支的某一个节点创建了一个子分支，并在子分支对文件1进行了修改，与此同时 **Master** 分支也对文件1进行了修改，然后子分支 **Merge** 到了 **Master** 分支，这时候怎么处理呐！

小明说：哦，这不是合并冲突嘛！通常我们就手动的查看冲突的内容，通过一些解决冲突的工具，比如 **mergetool** 或者是 IDE 中的一些插件，然后选择自己想要留下的那个文件的内容就可以啦！解决冲突之后再进行 **Merge**

大家露出了欣慰的表情，都回到了自己的工位。。。。。
## 结语

创作不易，整理这些内容花费了很大精力，希望大家能有所收获，能尽快熟练使用Git，毕竟coding的产品必然是要走向大众的，一个代码产品的诞生和管理需要一个好的版本控制工具，Git在众多的版本控制工具中无疑是每个程序猿的不二之选，无论是独立开发者亦或是团队开发，Git都是一大利器！

---

感谢您看到最后，如果本文对您有所帮助的话，还希望给我一个一键三连（狗头保命），如果对于我和我的文章感兴趣的话，欢迎点一个关注，您会收到我回答和文章的更新通知，也欢迎加入我建立的技术交流群QQ：[725133797](https://jq.qq.com/?_wv=1027&k=dD4NZkUt) 讨论交流。

最后附上我的个人博客站：[https://blog.calvinhaynes.top/](https://blog.calvinhaynes.top/)，欢迎访问和交流