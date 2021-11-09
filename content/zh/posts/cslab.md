---
keywords: 
- "MIT6.S081"
- "Docker"
- "code-server"  
title: "一个玩转国外CSLab的通用环境搭建方案（附我的搭建好的MIT6.S081环境）"
date: 2021-10-30T10:45:08+08:00
lastmod: 2021-10-30T10:45:08+08:00
description: "一个玩转国外CSLab的通用环境搭建方案（附我的搭建好的MIT6.S081环境）"
draft: false 
author: Calvin Haynes
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
tags: 
- "OS操作系统"
- "Docker"
- "Vscode"
categories: "CS基础知识"
img: "https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/Wallpaper/wallpaper-12-type.jpg"
---
## 前言

自从学习了Docker，我就无处施展自己的才华了，突然想起之前学习MIT6.S081课程的时候环境搭建完后就一直在吃灰，再加上我突然联想到之前搭建的code-server（点[这里](https://zhuanlan.zhihu.com/p/379632978)），所以我突然有了个大胆的想法，搭建一个Docker的MIT6.S081实验环境，并且采用code-server的形式，达成一个开箱即用的效果，甚至可以挂到云服务器（阿里云学生机）上，实现随时随地使用code-server进行实验，有灵感的时候随时掏出IPAD或者手机就可以实验搞起，实在是太香了，而且以后其他CSLab也可以采用相似的方案，那么下面我们就开始吧。

**没有了解过Docker的小伙伴我给出以下几个链接供大家学习和参考**

- [Docker从入门到实践文档（非常详尽）](https://yeasy.gitbook.io/docker_practice/)
- [油管视频列表（科学上网）](https://www.youtube.com/playlist?list=PL2WlOKghhsn0OR0zqmnqwHBlb5e0gwVK9)
- Dockerfile参考文档
  - `Dockerfie` 官方文档：https://docs.docker.com/engine/reference/builder/
  - `Dockerfile` 最佳实践文档：https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
  - `Docker` 官方镜像 `Dockerfile`：https://github.com/docker-library/docs

**如何按需正确食用本文**

- 仅仅想直接使用我搭建好的环境的童鞋请直接移步到使用篇，利用Docker可以直接`pull`我的`Regisry`（环境比较大，国外服务器很慢，后面我会说其他解决方案），然后直接进行MIT6.S081的实验
- 想学习Docker搭建环境的童鞋可以慢慢看环境搭建步骤

## 效果演示

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.5l840ak5vw00.png)

## 环境搭建步骤

通过不断的尝试，我觉得这种方案可以作为做国外CSLab的通用方案，只要写好相应的`Dockerfile`就可以了，code-server和其他一些常见的比如镜像源的设置、使用的linux发行版等等是通用的，这些保留，然后剩下的部分就专注于在`Dockerfile`中搭建相应国外CSLab的环境就可以了，基于此思路，我们进行下面的搭建步骤。（下面一步步按照我的思路来走，之后你想搭建自己的其他实验环境，就按照这样思路向下进行就可以了）

### 1 - 首先准备好各种搭建环境的工具

- 首先我是Windows系统，所以下载了[Docker Desktop](https://www.docker.com/products/docker-desktop)

- 简单配一些国内的`Docker Registry`镜像源（点开设置中`Docker Engine`就行了）

  ![Docker国内镜像设置](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/Docker国内镜像设置.6eeiwtzqy180.png)

```json
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://xzahxrmt.mirror.aliyuncs.com"
  ],
  "insecure-registries": [],
  "debug": false,
  "experimental": false,
  "features": {
    "buildkit": true
  }
}
```

- `windows`原生的命令行终端太拉了，所以下载他们新出的[Windows Terminal](https://www.microsoft.com/zh-cn/p/windows-terminal/9n0dx20hk701?activetab=pivot:overviewtab)，样子好看了不少，还能开多个`Tab`标签，还能定制化样式，确实好用（附上`youtube`上的[美化教程](https://www.youtube.com/watch?v=235G6X5EAvM&t=462s)）

  ![windows-terminal](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/windows-terminal.uv30qstxow0.png)

### 2 - 然后进行MIT6.S081环境搭建的调研和实践

- 首先肯定是官网的环境搭建说明书是第一位要看的：https://pdos.csail.mit.edu/6.828/2020/tools.html
- 结果发现搭建问题很多，我使用的是`ubuntu20.04LTS`，官方教程对应ubuntu的部分搭建起来一大堆问题。
- 然后我就开始互联网冲浪寻找其他人分享的环境搭建的教程，自己先简单开个`Docker`测试每个教程的环境搭建是否有效，最终结合多家教程和自己的思考，拿到了正确的环境配置方式，并写了对应的`Dockerfile`，具体内容如下（没一步操作我都写好注释了，可以先看完上面的Docker从入门到实践中的[`Dockerfile`](https://yeasy.gitbook.io/docker_practice/image/build)部分然后再看这里进行理解）：

```dockerfile
FROM ubuntu:20.04
ARG arch_name=amd64

# 前几步因为针对本实验依赖不全的原因问题百出，所以放弃了，还是ubuntu20.04官方镜像源对于MIT6.S081所需依赖最全面
# 1.备份源列表
# RUN cp /etc/apt/sources.list /etc/apt/sources.backup.list
# 2.把本目录下的sources.list中的镜像源添加到Docker中，下载速度起飞
# COPY sources.list /etc/apt/sources.list
# 3.更新源
# RUN apt-get update 

# 创建一个mit6s081的用户和其home目录
RUN useradd -m mit6s081 && \
    echo "root ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers && \
    echo "mit6s081 ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers 
    
# 设置一些环境变量
ENV TZ=Asia/Shanghai \
    LANG=en_US.utf8 \
    LANGUAGE=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    DEBIAN_FRONTEND=noninteractive


# MIT6.S081 Lab所用依赖
# 1.安装RISC-V交叉编译工具和一些其他的常用工具
RUN apt-get update && \
    apt-get install -y sudo locales git wget vim build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu libpixman-1-dev gcc-riscv64-unknown-elf libglib2.0-dev pkg-config
# 2.安装QEMU和配置QEMU
RUN wget https://download.qemu.org/qemu-5.1.0.tar.xz 
RUN tar xf qemu-5.1.0.tar.xz 
RUN cd qemu-5.1.0 && \
    ./configure --disable-kvm --disable-werror --prefix=/usr/local --target-list=riscv64-softmmu && \
    make && \
    make install

# 下载code-server并安装
RUN apt-get install -y aria2 && \
    aria2c https://github.com.cnpmjs.org/cdr/code-server/releases/download/v3.12.0/code-server_3.12.0_${arch_name}.deb && \
    dpkg -i code-server_3.12.0_${arch_name}.deb

# 切换用户mit6s081
USER mit6s081
# 下载一些code-server的插件
RUN mkdir /home/mit6s081/extensions
# 1.Markdown Extension
RUN code-server --install-extension yzhang.markdown-all-in-one
# 2.Cpp Extension
ADD cpptools-linux.vsix /home/mit6s081/extensions
RUN code-server --install-extension /home/mit6s081/extensions/cpptools-linux.vsix
# 3.Material Theme Extension
RUN code-server --install-extension equinusocio.vsc-material-theme 

# 切换回Root用户，拥有最高权限
USER root
RUN apt-get update

# 暴露8848端口，用于code-server本地运行的端口
EXPOSE 8848
# 设置code-server密码
ENV PASSWORD=mit6s081

USER mit6s081
CMD [ "code-server", "--bind-addr", "0.0.0.0:8848", "--auth", "password" ]
```

- 写好`Dockerfile`之后，直接`docker build -t <镜像名>:<版本号/标签> .`就构建好镜像了，注意后面那个`.`可不能省略，具体原因看[这里](https://yeasy.gitbook.io/docker_practice/image/build#jing-xiang-gou-jian-shang-xia-wen-context)

> 之所以创建了两个用户也是为了大家在使用环境的时候专注于使用当前环境，而不给大家`root`用户修改环境的权限，所以如果你遇到了使用`apt-get update`等指令时给出`denied`的报错正是说明了这点。
>
> 当然如果你想自己增加一些功能，也可以本地使用命令`docker exec -u root -it <Container名字> /bin/bash`进入你创建的`container`中，然后你可以自定义这个`Container`的环境，然后`commit`到镜像上，如果功能不错提高实验幸福感的话，欢迎知乎私信或者Github上提Issue。

### 3 - 一些环境搭建中遇到的坑

1. 被选择时区的环节卡住

```bash
Please select the geographic area in which you live. Subsequent configuration
questions will narrow this down by presenting a list of cities, representing
the time zones in which they are located.

  1. Africa      4. Australia  7. Atlantic  10. Pacific  13. Etc
  2. America     5. Arctic     8. Europe    11. SystemV
  3. Antarctica  6. Asia       9. Indian    12. US
Geographic area:
```

**可以通过配置环境变量，来跳过这个步骤**

```dockerfile
ENV DEBIAN_FRONTEND=noninteractive
```

2. RUN的使用不当，导致环境搭建过程中一直出错

主要是最初不理解Dockerfile，顺着指令说明书就开始莽了，后来才发现`RUN`指令每次执行会在当前`image`之上的新的一层中执行后面的命令并提交结果，这样就导致前后有关联的指令执行总是报错（原因就是他们本就应该是一条流程下来的，现在拆分成不同层了当然会出错），所以我才学会用 `&&`和`\`（反斜杠）将单个 RUN 指令延续到下一行。

比如下面这几行例子：

```dockerfile
RUN cd qemu-5.1.0 && \
    ./configure --disable-kvm --disable-werror --prefix=/usr/local --target-list=riscv64-softmmu && \
    make && \
    make install
```

如果不用`&&`和`\`关联各行的话，比如第一行`cd`到那个**qemu-5.1.0**目录了，下一行**./configure**执行的时候在新的一层了，就一定会报错不存在**./configure**（因为这个**configure**配置**qemu**的可执行文件就在**qemu-5.1.0**目录下）

## 使用篇

### 1 - 方式一：DockerHub

#### **首先pull下来我的Docker镜像**：

- 由于DockerHub用的是国外的服务器，所以很慢很慢，但是就一行命令，比较容易
- 镜像地址在这里：https://hub.docker.com/repository/docker/calvinhaynes412/mit6.s081/tags?page=1&ordering=last_updated
- 最新版本的pull镜像的命令如下：

```bash
docker pull calvinhaynes412/mit6.s081:v1.3.1
```

#### **然后去Github上Fork我上传的MIT6.S081原版纯净的实验环境**

> 这样你就能自己git管理自己的实验代码了，其实我就是把官方的git仓库自己上传了，主要是由于MIT pdos官方没有放出来这个2020实验的源代码不方便Fork之后直接自己使用，所以我转载了官方的2020实验源代码，是方便大家直接Fork下来自己做实验，并且我加了一个MIT的开源许可协议。

- 项目地址：https://github.com/CalvinHaynes/MIT6.S081-2020-labs（）
- **Fork完之后clone到你本地的一个文件夹里面**（重点，之后创建`Docker`的`Container`的时候需要使用）

> **想在这里顺便介绍一下我自己学习此Lab整理笔记和实验解决方案的一个仓库，供大家学习参考，如果有帮助的话希望给我点一个STAR**
>
> - GitHub项目地址：https://github.com/CalvinHaynes/MIT6.S081-2020Fall
>
> - 码云版本：https://gitee.com/CalvinHaynes/MIT6.S081-2020Fall
>
> 在项目中涉及了每个实验（如果我能坚持做完的话，我相信我可以）的实验解决代码、实验笔记、课堂笔记、课堂小练习、一些学习操作系统优质的资料等等。

#### **创建一个Container**

- 点击`RUN`

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.1je3xe1ues0w.png)

- 配置`Container`

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.sdofgdy3chc.png)

> 这里的`Volumes`配置你可以理解为将`Host Path`中的文件目录挂载在此`Container`中，挂载在`Container`的具体位置就是这里配置的`Container Path`（这里就直接配置成我图中写的路径，因为我已经在`.gdbinit`设置过了，如果你修改为其他位置的话，就修改`/home/mit6s081/.gdbinit`中的内容为`add-auto-load-safe-path <你设置的Container Path>/.gdbinit`）
>
> **Volumes简单理解就是主机目录挂载到容器上**

### 2 - 方式二：阿里云镜像仓库

- 由于DockerHub服务器在国外，实在难以忍受它的速度，有一天偶然发现，阿里云竟然有容器镜像服务，NICE！冲！！！
- 这个速度简直飞快，实测大约不到一分钟就pull下来了
- 先在Docker Engine中加入阿里云的Docker镜像，前面的Docker Engine设置中的`"https://xzahxrmt.mirror.aliyuncs.com"`就是阿里云的镜像加速器
- 运行指令：

```bash
docker pull registry.cn-beijing.aliyuncs.com/calvin_haynes/mit6.s081:release-v1.0.0
```

- 之后**创建一个Container**的方法和上述一致

### 3 - 开始做实验吧！

如果你也是配置的`8848`端口，那么直接点击http://localhost:8848/**(密码是==mit6s081==哦**)，就可以进入在线的`Vscode`进行愉快（狗头保命）的`MIT6.S081`实验了！！

## 完整的命令行使用教程(服务器端和本地Linux系统用户看这里)

1. [安装docker](https://yeasy.gitbook.io/docker_practice/install)
2. 配置阿里云镜像加速器（**以下命令针对ubuntu**）

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xzahxrmt.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

3. pull镜像（最新版本，之后版本更新的通知我会在评论区说明，记得偶尔来看看）

```bash
docker pull registry.cn-beijing.aliyuncs.com/calvin_haynes/mit6.s081:release-v1.3.1
```

4. clone实验源代码(**我还会慢慢在这个项目中增加我学习过程中的笔记和实验代码等资料,如果对你有帮助的话,希望帮我点个`STAR`哦**)

```bash
git clone https://github.com/CalvinHaynes/MIT6.S081-2020-labs <放实验文件的任何你想要放的位置>
```

5. 创建Container（`-u 0:0`是为了保证`Container`中的`root`用户和本机的`root`用户权限一致,不过其实不建议用`root`权限，所以如果本机和`Container`的其他非`root`用户有相同的`uid`的话最好 `-u uid:uid`）

```bash
docker run -p 8848:8848 --name=mit6.s081 -u 0:0 -v "<刚才clone的项目的位置>:<设置一个Container中的位置映射到前面的clone的项目的位置>" registry.cn-beijing.aliyuncs.com/calvin_haynes/mit6.s081:release-v1.3.1
```

6. 本地用户浏览器访问 **http://localhost:8848/**  密码： **mit6s081**.
7. 服务器端用户浏览器访问**http://<服务器公网ip>:8848/** 密码： **mit6s081**.

## 结语

这虽然只是一个MIT6.S081的Docker环境搭建教程，但是同时也是以后搭建其他实验环境的思路，如果学会了这一套思路和手段，那么以后后搭建任何环境都是唾手可得的！！

关于`MIT6.S081`的实验解析和课程笔记未来都会逐渐在我的[项目](https://github.com/CalvinHaynes/MIT6.S081-2020Fall)中更新，欢迎大家关注这个项目，关于此`Docker`，还有很多可以优化的地方，我会同步更新`Dockerfile`，`Docker`阿里云镜像/官方镜像，敬请期待。

-------

感谢您看到最后，如果本文对您有所帮助的话，还希望给我一个一键三连（狗头保命），如果对于我和我的文章感兴趣的话，欢迎点一个关注，您会收到我回答和文章的更新通知，也欢迎加入我建立的技术交流群QQ：725133797 讨论交流。

最后附上我的个人博客站：https://blog.calvinhaynes.top/，欢迎访问和交流

