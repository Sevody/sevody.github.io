---
title: git 合并操作总结
date: 2017-02-16 20:25:23
tags: git
---

git 合并经常需要用到的命令有: `git pull`、`git fetch` 、`git merge`、`git rebase`、`git cherry-pick`

有时候我们不知道该用哪个命令进行合并比较好，因此大概来总结下它们的区别。

<!--more-->

### git pull

`git pull` 命令的作用是，取回远程主机某个分支的更新，再与本地的指定分支合并

```bash
$ git pull <远程主机名> <远程分支名>:<本地分支名>
```

> `git pull` 相当于 `git fetch && git merge`

> `git pull rebase` 相当于 `git fetch && git rebase origin/master`

### git fetch

`git fetch` 用来取回所有远程分支的更新，也可以指定更新特定分支

```bash
$ git fetch <远程主机名> <分支名>
```

获取远程分支后，可以通过 `git checkout` 命令来读取远程分支的新内容

```bash
$ git checkout -b newBranch origin/master
```

### git merge

`git merge` 用来合并分支

把 feature 分支合到 master 分支

```bash
// 如果不设置第二个分支，默认是当前分支
$ git merge feature [master]`
```

有时候`merge` 后面会加上 `--no-ff` 或者 `--ff-only` 参数

#### -ff
`ff` 意思是 fast-forward, 使用 merge 时，默认会使用 fast-forward 的方式合并代码

如果合并的分支（master）是被合并分支（feature）的上游分支，则合并成功，不会产生 merge log，

如果合并的分支（master）不是被合并分支（feature）的直接上游分支（比如 master 在 checkout 出 feature 分支后，又进行了几次提交），不能使用 fast-forward 的方式合并代码，git 会进行一次三方合并（magic）,如果合并成功，就会产生一个 merge log, 如果有冲突产生，则合并失败，需要解决冲突并 commit 后才能合并.

#### --no-ff
如果加上 `--no-ff` 参数，就是默认使用三方合并的方式合并，就算合并的分支（master）是被合并分支（feature）的上游分支，也会产生一个 merge log
这种做法的好处是，忠实地记录了实际发生过什么，关注点在真实的提交历史上面

#### --ff-only
与 `--no-ff` 相反，`--ff-only` 表示只接受 fast-forward 方式的合并，如果不能直接使用 fast-forward 合并，会合并失败并报错。

### git rebase

`git rebase` 的用法有很多，主要是通过 **replay(回放)** commit来合并分支或者合并 commit log

```bash
# newbase 的默认值是 upstream
# upstream 的默认值是 origin/branch
# branch 的默认值是当前分支
$ git rebase [-i | --interactive] [--onto <newbase>] [<upstream> [<branch>]]
```

**git 会 截取 upstream..branch 之间的 commit ，然后在 newbase 的 HEAD 上 replay**

- 如果 repaly 过程中，有的 commit 已经出现在 newbase 上，则省略该 commit
- 如果 replay 过程中，出现冲突，则中断 rebase，并返回命令行
    - 手动解决冲突后，执行 `rebase --continue` 就能继续 replay
    - 如果放弃导致冲突的 commit，执行 `rebase --skip` 就会跳过该 commit, rebase 完后，该 commit 会**丢失**
    - 如果放弃这个 rebase，执行 `rebase --abort`

如果加上 -i 参数，就会进入到交互式的 rebase 设置页面，一般用来合并 commit

```
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
```

### git cherry-pick

`git cherry-pick` 与 rebase 类似，用来 **replay(重放)** 特定的 commit

```bash
$ git cherry-pick commit [commit2] [commit3]
```

通常用于只提取某个分支上的特定提交应用到当前分支，比如在 featureA 分支上修复了一个 bug 并作了 commit bugA，在 master 分支上只想获 取 bugA 的更改，就可以使用 `git cherry-pick bugAId`
如果产生冲突，处理方法跟 rebase 类似






