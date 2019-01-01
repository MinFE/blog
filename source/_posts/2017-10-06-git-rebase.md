---
title: 使用 git rebase 美化 git commit 流程
subtitle: 上篇文章讲到 commit message 规范，git 版本控制在日常团队协作中有着极其重要的作用，除了对 git 的 commit 信息进行规范化管理之外，还可以对 git 的 commit 做一些补丁修饰，即 git rebase。
date: 2017-10-06
cover: http://blog.static.minfive.com/post/17-10-06/git-rebase-cover.png
categories: 经验总结
tags:
    - git

author:
  nick: minfive
  link: 'https://github.com/Mrminfive'
---

### 前言

上篇文章讲到 [commit message 规范][commit-message-post]，git 版本控制在日常团队协作中有着极其重要的作用，除了对 git 的 commit 信息进行规范化管理之外，还可以对 git 的 commit 做一些补丁修饰，即 git rebase。


### 什么是 git rebase

在 git 中对于不同分支间的修改有两种方式：`merge` 和 `rebase`，merge 这个比较常用，用于合并分支，同样的，rebase 也可用于合并分支，虽然较少使用，但功能却比简单的 merge 强大的多。

rebase 即变基，顾名思义，就是改变基准点，以 commit 为基准点可以随意修改 commit 历史，以分支为基准点可以合并分支，同时整理 commit。

### 如何使用

上文讲到 rebase 可以以 commit 为基准点也可以以分支为基准点，那么以笔者为例，rebase 的主要用法如下：

* 修改 commit 历史，结合上一篇文章讲的 [commit message 规范][commit-message-post]，可以做到每一个 commit 仅切只做一件事情，单一原则。
* 合并分支，使分支保持单一链式，避免频繁的切换分支指向，导致分支 commit 混乱。(强迫症必备良药)

#### commit 的修改

对于 commit 的修改可以使用 `git commit -amend` 修改下 commit 信息，如果要修改多个 commit，那么就要使用 `git rebase` 了。

简单流程如下：

使用 `git log --oneline` 获取相应 commit 的哈希值（该 commit 为基准点，不参与本次修改）;

![git-log][git-log]

使用 `git rebase -i 5663aa4(指定的基准点)` 进入 vi 模式手动编辑选定区间内的 commit，可合并可编辑可删除，具体操作请参照 vi 界面的注释内容。

![git-rebase-commit][git-rebase-commit]

结合 [commit message 规范][commit-message-post] 就可以实现完美的单一任务原则 commit，在正式完成某个任务之后，可以通过 rebase 将无意义的 commit 合并在一起。

#### 分支的合并

分支的合并有两种方式：`merge` 和 `rebase`，两种方式的具体区别大家可以参考 [Pro Git Book](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA) 的解释。

简单来说：

对于相同情况下的 git 分叉：

![git-rebase-1][git-rebase-1]

merge 的合并如下图：

![git-rebase-2][git-rebase-2]

rebase 的合并如下图：

![git-rebase-3][git-rebase-3]

两者的区别在于 merge 只是简单的把所有改动整合到一个 commit 中，并保留 commit 纪录，还会产生恶心人的非必要合并 commit，而 rebase 则会根据最近的共同祖先作为基准点将改动依次进行排序，当前分支的改动永远置于最后，且不会产生合并 commit。

### 双刃剑

在日常团队协作开发一般都推荐 git flow 的工作流程，禁止使用 rebase 来合并分支，主要的原因在于 rebase 会改变整个分支的 commit 流向，极其容易产生版本冲突，所以，对于 rebase 的使用，笔者有几个小小的建议：

* 使用 rebase 修改 commit 只在本地分支发生
* 本地分支 push 到服务器仓库上前都应该先 `git rebase origin (branch)` 保证拉取最新代码。 
* 禁止使用 `git push --force` 覆盖服务器上的提交历史。

总而言之，rebase 是一个很强大的工具，只要操作得当，绝对是一大杀器。

### 总结

综上，笔者总结来一套用来结合 [commit message 规范][commit-message-post] 和 `git rebase` 的工作流程，具体如下：

假设团队开发项目除 `master` 分支外再有 `develop` 分支，那么在本地开发中应基于 `develop` 分支创建一个自定义分支：

``` shell
git checkout develop
git branch -b minfive
```

然后在自定义分支上执行任意开发，一个功能点开发完成后，使用 rebase 整理合并所有新增 commit 为单个符合 [commit message 规范][commit-message-post] 的 commit，每完成一个完整功能点整理合并一次 commit，开发完成后再通过 rebase 将整理过 commit 的自定义分支合并到 `develop` 分支。

到这一步，本地开发已完成，需要提交代码到服务器，那么请使用 `git pull --rebase` 拉取服务器最新 commit 并 rebase 合并到 `develop` 分支。有冲突解决冲突，没冲突则可以直接 push 代码上服务器。

> 注：应尽可能保持自定义分支与开发分支的同步，一般情况下，每更新一次开发分支，笔者都会基于开发分支重建一次自定义分支用来保证 rebase 基准点的准确性。

然后剩下的其它流程基本与 git flow 工作流程一致。

这样一套流程下来，commit 的流向如下图：

![git-rebase][git-rebase]

而不会出现多余的合并 commit 出现：

![git-merge][git-merge]

简直是强迫症解救神器啊！！！

> 另：附上 [demo][demo] 一枚。

### 参考文献

* [Pro Git book][proGit]
* [Git Community book][git-community-book]


[proGit]: https://git-scm.com/book/zh/v2
[git-community-book]: http://gitbook.liuhui998.com/4_2.html
[commit-message-post]: /2017/09/08/2017-09-08-git-commit-message/
[git-log]: http://blog.static.minfive.com/post/17-10-06/git-log-oneline.png
[git-rebase-commit]: http://blog.static.minfive.com/post/17-10-06/git-rebase-commit.png
[git-rebase-1]: http://blog.static.minfive.com/post/17-10-06/basic-rebase-1.png
[git-rebase-2]: http://blog.static.minfive.com/post/17-10-06/basic-rebase-2.png
[git-rebase-3]: http://blog.static.minfive.com/post/17-10-06/basic-rebase-4.png
[git-rebase]: http://blog.static.minfive.com/post/17-10-06/git-rebase.png
[git-merge]: http://blog.static.minfive.com/post/17-10-06/git-merge.png
[demo]: https://github.com/MinFE/git-rebase-demo