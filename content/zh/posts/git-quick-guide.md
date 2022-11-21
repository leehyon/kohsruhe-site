---
title: "Git 快速上手"
date: 2022-11-20T16:59:39+08:00
draft: false
tags: ["git", "guide", "tool"]
categories: ["articles"]
authors:
- "Leehyon"
---

{{<audio src="audio/life_live.mp3" caption="♪ 超人 - 五月天" >}}

## 前言
本文主要内容参考自[Git教程 - 廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/896043488029600)，同时结合自己的使用习惯稍作修改。

对任何软件的使用，首先要**明确自己的需求**，虽然我之前就知道Git，但直到最近由于工作需要才开始接触命令行操作，一旦会了就基本告别GUI了。其次是**抓住软件的底层逻辑**，比如Git的核心是stage（暂存区）和指针，明白这两个概念，其他很多内容都不言自明了。

## Git是什么
Git是目前世界上最先进的**分布式**版本控制系统，即Distributed Version Control System (DVCS)，由大神Linus开发完成，他同时也是开源Linux创建者。

### 集中式和分布式
集中式版本控制系统，顾名思义是有一个中央服务器存储你的版本库。每次版本的更新和提交都需要你连接和上传到中央服务器。

而分布式版本控制系统没有中央服务器的概念，**每个人的本地电脑都是一个完整的版本库**，版本的更新和提交都发生在你自己的电脑上。

现在请思考：
- 如果每个人的电脑都有一个完整的版本库，那么该怎么实现多人协作？
- 相比集中式，分布式有什么优点？

## 安装和配置
在Windows上使用Git，可以从Git官网直接[下载安装程序](https://git-scm.com/downloads)，然后按默认选项安装即可。

安装完成后，右键开始菜单找到`Git Bash`，弹出一个带颜色的命令行窗口，输入`git -v`能查到版本信息就代表安装成功。

### 用户信息配置
安装完成后的第一步是配置你的个人信息，因为Git是分布式的，每台电脑需要自报家门。

在命令行输入以下内容：
```bash
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```
> 你的信息保存在全局配置文件`~/.gitconfig`中，你也可以通过`git config --list`命令确认你的信息。

命令中的`--global`参数，表示你这台电脑上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。
- 即在需要单独配置的仓库源目录下运行`git config`命令，省略`--global`

## 版本管理
> 版本库又名仓库，英文名repository，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

### 创建版本库
`git init`命令可以把一个目录变成Git可以管理的仓库。
- 创建后，该目录下会多一个`.git`的子目录，这个目录是Git来跟踪管理版本库的，该目录默认隐藏

### 添加和修改
`git status`命令可以让我们时刻掌握仓库当前的状态，而`git diff`命令可以对比修改的内容。

提交修改和提交新文件是一样的两步（从这里可以看出Git跟踪的是**修改**而不是文件），先`git add`，再`git commit`。
- `git add .`可以添加所有的修改

```bash
$ echo "# Git is fun" >> README.md
$ git add README.md
$ git commit -m "add readme"
```

### 版本控制
`git log`命令显示从最近到最远的提交日志。

```bash
leehyon@Kohs-MacBook kohsruhe % git log
commit fa9c94b19b0347a61c5e72b556947d9f86fefb21 (HEAD -> main, origin/main)
Author: Leehyon Koh <leehyon@live.com>
Date:   Sun Nov 20 16:43:58 2022 +0800

    test aliyun image hub

commit 7381263185e996491d84cde9f15520fe88afb379
Author: Leehyon Koh <leehyon@live.com>
Date:   Sun Nov 20 14:24:38 2022 +0800

    add first article
```
`commit`后面跟的一大串数字就是版本号，即`commit id`。
- 思考：为什么Git的版本号不像SVN一样以数字迭代的？

那么有了版本号是不是可以在不同版本间跳转了呢？那的确是的。不过Git必须首先知道当前是哪个版本。

在Git中，用`HEAD`表示当前版本，也就是最新的提交的`fa9c9...`，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本请写成`HEAD~100`。

使用`git reset`命令，并添加`Head`信息或指定版本号来实现版本回退：
```bash
$ git reset --hard HEAD^  # 回退到上一版本
$ git reset --hard 73812  # 或指定版本号
```
Git的版本回退速度非常快，因为它仅仅更改了`HEAD`的指向。这里是重点，后面提到的**分支管理**也是基于指针的操作。

思考：回到过去后，突然反悔了又想回到未来怎么办？
- Git提供了一个命令`git reflog`用来记录你的每一次命令，找到需要返回的`commit id`

## 核心逻辑一：暂存区
`.git`目录是Git的版本库，里面存了非常重要的两个东西：
- 一是被称为stage的暂存区
- 二是Git自动创建的第一个分支`master`，以及指向`master`的一个`HEAD`指针

![Stage](https://kohsruhe-image.oss-cn-shanghai.aliyuncs.com/images/stage.png)

前面讲了我们把文件往Git版本库里添加是分两步执行的：
1. `git add`把文件添加进去，实际上就是把文件修改添加到暂存区
2. `git commit`提交修改，实际上就是把暂存区的所有内容提交到当前分支

为什么Git比其他版本控制系统设计得优秀，因为Git跟踪并管理的是**修改**，而非文件。每次修改，如果不用`git add`到暂存区，那就不会加入到`git commit`中。

思考：什么是修改？什么又是文件？

### 撤销修改
先看下面一条命令：
```bash
$ git checkout -- README.md
```
意思是，把`README.md`文件在工作区的修改全部撤销，这里有两种情况：
1. `README.md`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态
2. `README.md`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态

`git checkout -- <file>`命令中的“`--`”很重要，没有“`--`”，就变成了“切换到另一个分支”。
- 一种可选的方法是：用`git reset HEAD <file>`可以把暂存区的修改撤销掉（unstage），重新放回工作区

注意：这里所指的撤销都是基于还没有`git commit`的。

每次在撤销或修改操作时，建议先用`git status`查看仓库状态。

思考：如果已经把暂存区提交到了版本库了，该怎么办？

## 远程仓库
GitHub是基于Git的一个面向开源及私有软件项目的**托管平台**。除了GitHub，其他的还有比如[GitLab](https://about.gitlab.com/)和[Gitee](https://gitee.com/)等。

### 配置SSH
由于本地仓库和GitHub仓库之间的传输是通过SSH协议加密的，所以需要首先配置SSH。配置过程参考[官方教程](https://docs.github.com/en/enterprise-server@3.7/authentication/connecting-to-github-with-ssh)。

配置完成后，可以在用户主目录里找到`.ssh`目录，里面有`id_rsa`和`id_rsa.pub`两个文件（如果使用默认名），这两个就是SSH的密钥对，`id_rsa`是私钥，不能泄漏，而`id_rsa.pub`是公钥，复制粘贴到GitHub设置里。

GitHub允许你添加多个SSH密钥对。如果你使用多台电脑工作，只要把每台电脑的公钥都添加到GitHub设置里，就可以在每台电脑上往GitHub远程仓库拉取和推送了。

### 配置GPG
我们查看仓库的`commit`历史时，发现有带`Verified`的绿色标记：
![Verified Commit](https://kohsruhe-image.oss-cn-shanghai.aliyuncs.com/images/verified-commit.png)

该标记表示这一次`commit`是经过签名验证的（signed with a verified signature）。配置过程参考[官方教程](https://docs.github.com/en/enterprise-server@3.7/authentication/managing-commit-signature-verification)。

### 克隆远程仓库
要克隆一个仓库，首先必须知道仓库的地址，然后使用`git clone`命令克隆。
> Git支持多种协议，包括`https`，但`ssh`协议速度最快。

如果本地仓库已经有了，那怎么办？新建一个空的远程仓库，再进行关联。

### 关联远程仓库
关联和添加远程仓库使用`git remote add`命令：
```bash
$ git remote add origin <git@newempty.git>  # 关联一个远程仓库
```

> `origin`是远程仓库的名字，这是Git默认的叫法，也可以改成别的。

把本地库的内容推送到远程，用`git push`命令：
```bash
$ git push -u origin master  # 推送当前分支到远程master分支
```
> 如果远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的分支内容推送到远程的`master`分支，还会把本地分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

从今往后，只要本地作了提交，就可以通过`git push origin master`命令提交修改。

### 解除远程关联
如果添加的时候地址写错了，或者就是想解除关联，可以用`git remote rm <repo name>`命令。

使用前，建议先用`git remote -v`查看远程库信息：
```bash
$ git remote -v  # 查看远程库信息
origin	git@github.com:leehyon/kohsruhe-site.git (fetch)
origin	git@github.com:leehyon/kohsruhe-site.git (push)
```
然后，根据名字解除，比如`git remote rm origin`。

## 核心逻辑二：分支管理（指针）
当我们创建新的分支`dev`时，Git新建了一个指针叫`dev`，指向`master`相同的一个`commit`，再把`HEAD`指针指向`dev`，就表示当前分支在`dev`上：
```bash
$ git switch -c dev  # 创建dev分支并切换
```

`git switch`命令加上`-c`参数表示创建并切换，相当于以下两条命令：
- `git branch dev`
- `git switch dev`

> Git创建一个分支很快，因为除了增加一个`dev`指针，更改`HEAD`的指向，没有更改工作区内容。

`git branch`命令会列出所有分支，带`*`号的为当前分支。

从现在开始，对工作区的修改和提交都是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针保持不变：

![New Branch](https://kohsruhe-image.oss-cn-shanghai.aliyuncs.com/images/branch_point.png)

如果我们在`dev`分支上的工作完成了，就可以把`dev`合并到`master`上。那Git怎么合并呢？最简单的方法，就是直接把`master`指向`dev`的当前提交。

所以Git合并分支也很快，就改改指针，也没有更改工作区内容。

合并分支后，甚至可以删除`dev`分支。删除`dev`分支就是把`dev`指针给删掉，删掉后，就剩`master`分支了。

到此为止，应该明白**针对分支的操作其实是针对指针的操作**。了解完这一点，Git就没啥了。

### 分支管理策略
在实际开发中，应该按照怎么样的策略进行分支管理呢？
- 首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活
- 干活都在`dev`分支上，等到新版本发布时，再把`dev`分支合并到`master`上，并在`master`分支发布新版本
- 修复bug时，建议通过创建新的bug分支进行修复，然后合并，最后删除
	- bug分支只用在本地修复bug，就没必要推到远程了
- 开发一个新feature，最好新建一个分支

### 多人协作
多人协作的工作模式通常是这样：
1.  首先，可以试图用`git push origin <branch-name>`推送自己的修改
2.  如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并
3.  如果合并有冲突，则解决冲突，并在本地提交
4.  没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送修改

## 标签管理
标签是版本库的一个快照。

Git的标签其实是指向某个`commit`的指针，同前面提到的分支类似，但区别是分支的指针可以移动，而标签不能移动（可以理解为一个**常量指针**）。

- `git tag <tagname>` 新建一个标签，默认为`HEAD`，也可以指定一个`commit id`
- `git tag -a <tagname> -m "<message>"` 指定标签信息
- `git tag` 查看所有标签
- `git push origin <tagname>` 推送一个本地标签
- `git push origin --tags` 推送全部未推送过的本地标签
- `git tag -d <tagname>` 删除一个本地标签
- `git push origin :refs/tags/<tagname>` 删除一个远程标签

## 参考资料
> 1. [Git教程 - 廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/896043488029600)
> 2. [Git - Book](https://git-scm.com/book/en/v2)
> 3. [Connecting to GitHub with SSH](https://docs.github.com/en/enterprise-server@3.7/authentication/connecting-to-github-with-ssh)
> 4. [Managing commit signature verification](https://docs.github.com/en/enterprise-server@3.7/authentication/managing-commit-signature-verification)
> 5. [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)
