## Git 分支管理：简单操作分支进行开发

### 什么是分支？

上一篇，我们 git branch 显示的信息就是本地数据库中的各个分支名。

分支是为了在多人协作的时候，防止互相干扰，协同开发而诞生的。比如

- 主分支： main / master
- 分支A：登录功能，开发负责人： 小A
- 分支B：支付功能，开发负责人：小B
- 分支C：bug修复，开发负责人：小C

各个负责人从master拉取最新的特性，然后各自开发自己的功能，最终，各个负责人再将他们的分支并入到主分支中。

![](https://pic1.imgdb.cn/item/633a85c616f2c2beb1af99b0.png)



### 准备分支

#### git branch

我们在原先的 `git_test` 上继续测试，首先，我们先使用 git branch 查看一下有什么分支

```
$ git branch
* main
  master
```

---------------------

> 关于 git branch 更多常用的命令

`git branch -r`: 查看远程分支

```
$ git branch -r
  origin/main
  origin/master
```

`git branch -a`: 查看本地分支和远程分支

```
$ git branch -a
* main
  master
  remotes/origin/main
  remotes/origin/master
```

`git branch -v`: 查看各个分支最后一次提交

```
$ git branch -v
* main   b6d621a add note
  master 90c277e second revision
```

`git branch branchname`： 新建一条分支，新建后，并不会切换到新分支上，而是需要执行 `git checkout branchname` 才会切换到新的分支

`git branch -d branchname`： 删除指定分支

`git branch -D branchname`： 强制删除指定分支

`git branch -d -r branchname`： 强制删除远程的分支，之后 Git 还会告知还要再执行一次 push 操作，这样才能真正删除远程的分支

`git branch -merged`：查看哪些分支合并入当前分支

`git branch -no-merged`: 查看哪些分支未合并入当前分支

-------------------

现在我们的任务是建立一条新的分支 `feature/branch_test`

#### git checkout

我们上面学了 `git branch` 命令，可以知道用 `git branch feature/branch_test`来新建一个分支

然后我们可以用 `git checkout feature/branch_test` 来切换到这个分支上

但是还有更简单的，一步到位：

```
$ git checkout -b feature/branch_test
Switched to a new branch 'feature/branch_test'
```

再看看当前的分支情况

```
$ git branch
* feature/branch_test
  main
  master
```

我们修改一下 `example.txt`，然后提交分支

```
Hello, Git!

Learning git branch knowledge.
```

由于我们远程并没有 `feature/bracnh_test` 分支，在 push 时，可能会出现以下情况

```
$ git push
fatal: The current branch feature/branch_test has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin feature/branch_test
```

我们只要按照提示来，设置一下远程和本地的分支联系即可：`git push --set-upstream origin feature/branch_test`。现在，我们的远程分支如下：

![](https://pic1.imgdb.cn/item/6339dcd816f2c2beb1de1b7d.png)

-------

> 关于 git checkout 更多常用命令

1. 通过git checkout命令签出(切换)远程分支，这个语法是用于拉取一条远程分支的到本地分支的。

```
git checkout <remotebranch> origin/<remotebranch>
```

2. 放弃修改。

- `git checkout .`： 放弃所用更改，将工作区中（即还没有add过但已经修改的文件）所有改动都移除。
- `git checkout -- filename`： 放弃对指定文件的修改
- `git checkout -f`: 放弃工作区和暂存区的所有修改

-------------------

### 合并分支

现在我们已经准备好了分支，并且推送了内容。接下来我们将分支切换到 main 分支。

```
$ git checkout main
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
```

在正式讲合并分支的操作前，先简单介绍两种合并方式：`merge` 和 `rebase`。这两种方式都是将一个分支的修改合并到另一个分支，只不过方式有些不同而已。

------------------------

#### merge

我们可以使用 merge 命令来合并多个历史记录的流程到另一条分支中。

如下图所示：

![](https://pic1.imgdb.cn/item/633af73316f2c2beb18444f2.png)

我们从 master 分支上新建了一条分支 `feature/branch_test`，此时他们的特性都是相同的，我们假设他们的 commit 都是 `B`，接下来，我们在  `feature/branch_test` 上进行开发，假设进行了两次提交，commit 分别是 `X` 和 `Y`。

若 master 分支没有改动，那么这个合并会非常轻松，因为  `feature/branch_test` 分支包含了 master 分支所有的历史记录，Git 只需要把 commit  `X` 和 `Y` 移动到 master 分支 commit `B` 后面即可，如下图所示：

![](https://pic1.imgdb.cn/item/633afa0716f2c2beb18a8b99.png)

这种合并被称作 `fast-forward` （快进）合并。这是 git merge 默认的合并方式。仔细观察这种合并情况，它们实际上是一条线上的前后关系而已。这种合并没有冲突，合并时，只是把 master 指向了最新的一次 commit（即最新 commit 是 `Y`）。

显然，实际情况并不是那么理想，我们签出分支 `feature/branch_test`，master 分支可能会接着产生新特性。这种情况下就会如下图所示：

![](https://pic1.imgdb.cn/item/633afbc816f2c2beb18de8f5.png)

这时就不是简单的  `fast-forward` 合并了。我们知道  `fast-forward`  合并只是把 master 最新特性指向了 commit `Y`，但是上图的情况，直接指向  commit `Y` ，会导致 commit `C` 和 `D`丢失。因此，合并两个修改会生成一个提交记录，即生成一个新的 commit `E`，它包含了合并了两个分支的新特性。这种合并被称作 `non fast-forward` 合并。当然，生成这个新commit 时，合并的代码可能会有冲突，需要开发者去解决。

![](https://pic1.imgdb.cn/item/633afcd916f2c2beb1900fa1.png)

#### rebase

依然是 master 分支有新特性，`feature/branch_test` 分支也有新特性的情况

![](https://pic1.imgdb.cn/item/633afbc816f2c2beb18de8f5.png)

rebase 的做法与 merge 并不相同，它是这样进行合并的：

![](https://pic1.imgdb.cn/item/633afdf916f2c2beb1921de0.png)

rebase之后，会将 `feature/branch_test` 分支添加到 master 分支的后面，使历史记录变为一条直线。当然，这种做法，在移动 commit `X` 和 `Y` 时，可能会产生冲突，需要自行解决。

这里仅仅是简单提及 git 合并分支的两个命令，本篇会着重说明 merge 的用法，在下一篇讲解决冲突时，rebase 也会隆重登场。

-----------------------

回到正题，我们现在把 feature/branch_test 分支合并到 main 分支里吧。我们切换回到 main 分支后，原来在 feature/branch_test 分支的改动就没有了。现在，让我们合并分支：

```
$ git merge feature/branch_test
Updating b6d621a..92e32b0
Fast-forward
 example.txt | 4 +++-
 1 file changed, 3 insertions(+), 1 deletion(-)
```

这里看到了的信息表明，合并方式是 `Fast-forward`，并且看到了具体的文件修改变化。现在打开 example.txt 看看，文件中是否多了一句 `Learning git branch knowledge.`

### 删除分支

我们现在只是在本地上把 feature/branch_test 分支合并到 main 分支里，还需要执行一次 push 操作把本地的 main 分支推送到远程中。我们不必担心文件的改动需要我们再执行一次 add - commit - push 流程。我们只需要直接执行 push 即可，因为 add - commit 的操作，我们在 feature/branch_test 分支上已经完成了。如果不确定，可以用 git status 看看是否没有新的文件需要我们提交了。

```
$ git status
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
```

这里没有任何新增的文件，但是 status 告诉我们我们的分支超前了远程 main 分支一个提交，这个超前的提交就是我们合并后的新特性，所以我们把它 push 到远端即可。

现在，本地上的 feature/branch_test 分支已经完成了它的使命了。

```
$ git branch -d feature/branch_test
Deleted branch feature/branch_test (was 92e32b0).

$ git branch
* main
  master
```

执行 `git branch -d 分支名` 来删除 feature/branch_test 分支，现在本地上的 feature/branch_test 分支已经被删除了。但远端的 feature/branch_test 分支依然存在。我们现在不需要它了，我们可以执行下面的命令来删掉远程分支。当然，你也可以在 github 页面上，进入自己的仓库设置中删掉分支。

```
$ git push origin --delete feature/branch_test
To https://github.com/FelixWuu/git_test
 - [deleted]         feature/branch_test
```

至此，远端的 feature/branch_test 分支就被我们删除了。