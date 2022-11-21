---
title: "Git 子模块使用"
date: 2022-11-21T10:50:19+08:00
draft: false
tags: ["git","guide","feature"]
categories: ["articles"]
authors:
- "Leehyon"
---

## 前言
克隆他人项目时，有时候会看到仓库目录下有一个`.gitmodules`的文本文件，打开后有如下类似内容：
```bash
[submodule "foo/bar"]
	path = foo/bar
	url = https://example.com/foo.git
	branch = dev
```
上述内容表面该仓库使用了`git submodule`方式链接了其他仓库。如果要使用被链接的仓库，需要用以下命令进行初始化：
```bash
$ git submodule update --init --recursive
```

## 为什么使用Git子模块
其实背后的思想是模块化设计：
- 根据代码功能或维护周期把项目拆分成不同的子模块，并建立不同的Git仓库
- 一个模块能被多处使用，通过`git submodule`进行关联
- 主模块不必负责子模块的维护，只在必要时候同步更新

另外，如果项目依赖一个开源的第三方库时，也建议将第三方库设置为子模块。

## 主要内容
- 添加子模块到仓库并不会把子模块的源码添加进来，只是添加子模块的信息，比如`url`和`branch`
- 添加后的子模块不会随子模块仓库的更新而自动更新，以保证主仓库的稳定性

### 添加子模块
```bash
$ git submodule add <submodule_url> <path>
```
执行添加命令成功后，可以在当前路径中看到一个`.gitsubmodule`文件，里面的内容就是子模块的信息。

如果在添加子模块的时候想要指定分支，可以利用`-b`参数：
```bash
$ git submodule add -b <branch> <url> <path>
```
添加之后，仓库`<path>`目录下不会有任何变化，需要执行下面命令才会添加源码：
```bash
$ git submodule update --init --recursive
```

### 更新子模块
如果子模块仓库更新了，如有必要，需要手动进行更新，即进入子模块目录，执行`git pull`命令。

### 删除子模块
网上搜到的方法有点繁琐，其实可以使用`git submodule deinit`命令进行“优雅”删除：
```bash
$ git submodule deinit <submodule_name>
$ git rm <submodule_name>
```

这个命令如果添加上参数`--force`，则子模块工作区内即使有本地的修改，也会被移除。




