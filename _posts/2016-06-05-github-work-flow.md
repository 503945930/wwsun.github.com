---
layout: post
title: 理解Github工作流
category: technique
---

使用Github很久了，但却很少的梳理Github的工作流，因此，本文将简单的介绍一种简单的基于Github的工作流。
Github工作流非常的轻量级，它是以分支为基础的，以此用于支持项目和团队开发。

<!--more-->

## Github工作流

### 创建分支

当你正在开发一个项目的时候，你或许有很多的想法或者功能特性需要花费时间去付诸实施，有些已经在做了，
有些可能还没有。分支就是为了帮你管理这种工作流而诞生的。

当你在项目中创建了一个新的分支的时候，也就为你创建了一个尝试新想法的环境。所有的改动，都不会影响到`master`分支，
因此你可以自由的实验和提交变更，当你确定分支上的开发没有问题时，此时可以合并到`master`上，但最好经过review。

**分支**

分支是Git的一个核心概念，并且整个Github工作流都是基于分支的。只有一条规则：**`master`分支上的内容一定是可部署的**。

因此，当你需要修复bug或者增加新功能的时候，请务必创建一个分支，并在这个新的分支上进行开发。分支名最好是描述性的：
例如`refactor-authentication`，`user-content-cache-key`，`make-retina-avatars`，
因此其他人可以很直观的直到这个分支的作用。

### 在分支上添加commits

一旦你的分支被创建好了，现在就是开始增加变更的时候了。当你增加、编辑、或删除文件时，你都需要提交一个commit，
并将它加入到你的分支上。这种提交变更到过程可以确保能过追踪你在分支上的工作进度。

Commit也形成了你工作历史的一条透明的轨迹，从而让其他人能理解你做了什么，以及为什么这么做。每次commit的时候，
都会有一条有关该commit的message，用于描述为什么需要做这些变动。此外，每次commit都可以看作为时一个独立的变更单元。
这让你能够在发现问题的时候会滚变更，或者让你能够回到其他提交节点上。

**Commit Message***

Commit message非常的重要，因为它们会伴随着变更被上传到服务器上，能够让开发者轻松的识别提交的变更历史和原因。
也能过让其他人更快的理解你的工作过程。

### 开启一个Pull Request

Pull Request会初始化一个有关你提交历史的讨论界面。由于它紧密的与底层的Git仓库集成在一起，
任何人都可以看到哪些变更将会被merge，并让开发者决定是否执行merge操作。

在开发过程中，你可以任何时候开启一个Pull Request：即使没有代码变更，只是分享一个截图或一些想法，
以及你需要帮助和提供建议的时候，或者是你已经准备好了让别人Review你的代码的时候。对于Github而言，
你还可以在Pull Request的message中 @someone 某些人，这样你可以像特定的人询问反馈。

Pull Request对于向开源项目贡献代码时非常的有用。如果你使用的时 Fork 或 Pull Model，
Pull Request提供了一种通知项目维护者有关你提交的变更的最佳途径。如果在使用一个共享的代码库（多人协同），
通过Pull Request你可以在变更被合并到`master`分支前进行code review或者时开启对某个建议特性的讨论。

### 讨论和Review你的代码

一旦一个Pull Request被启动了，review你代码的人或团队可能会有一系列的问题和评论。比如，
你的代码没有符合项目的指导规范，变更缺少单元测试，或者是你的代码写的很好。设计Pull Request的目的就是鼓励和记录这种讨论过程。

在有关你的代码的讨论和反馈过程中，你可以继续提交变更。比如说，某个人评论你忘记了做某个事情，或者说你的代码有某个bug，
你可以在该分支中继续修复问题，并提交变更，推送这些新的提交。Github能够将你的这些新的变更同步到远程服务器，
并在Pull Request的界面中显示出来，以便获得其他人的review和评论。

提示：Github的Pull Request的评论框支持Markdown格式，你可以插入图片或者emoji，代码块等等。

### 部署

一旦你的Pull Request被review通过，并且通过了测试，你可以部署你的变更，并测试在生产环境中的工作状况。
如果你的分支存在问题，你也可以回滚变更，重新将生产环境切换回`master`分支。

### 合并

现在，分支上的代码已经部署成功，并工作正常，你便可以将代码合并到`master`分支上了。

一旦合并成功后，Pull Request回保存一份代码历史变更的记录。因为它们是可被搜索的，它们允许任何人回顾合并历史，
查询当初的PR讨论过程。

通过在你的Pull Request中包含一些特殊的关键字，你可以将代码和相关的issue联系在一起。当你的Pull Request被提交的时候，
相关的issue也会被关闭。例如，你可以在PR结束的时候在评论框中输入`Closes #32`，它将会关闭仓库中的编号为32的issue。

## 参考资料

1. [Understanding the Github Flow](https://guides.github.com/introduction/flow/)
2. [Closing issues via commit messages](https://help.github.com/articles/closing-issues-via-commit-messages/)
3. [基于git的源代码管理模型——git flow](http://www.ituring.com.cn/article/56870)
4. [Git资料汇总](https://github.com/xirong/my-git)