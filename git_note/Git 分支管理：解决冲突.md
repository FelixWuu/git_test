## Git 分支管理：解决冲突

### 引入冲突

当两个修改并入同一条分支中，造成了相互干扰，合并就会产生冲突。

接下来，我们创建两个分支，并进行并行操作，人为制造合并的冲突。首先，我们从 git_test 项目的 main 分支创建两个新分支 `test/conflict1` 和 `test/conflict2`

```
$ git branch test/conflict1

$ git branch test/conflict2
```

现在我们切换到 `test/conflict1`

```
$ git checkout test/conflict1
Switched to branch 'test/conflict1'

$ git branch
  main
  master
* test/conflict1
  test/conflict2
```

我们在 `example.txt` 中新增一句话 `I'm used to merging branches with git merge.`, 现在 `test/conflict1` 分支的  `example.txt` 是这样的：

```
Hello, Git!

Learning git branch knowledge.

I'm used to merging branches with git merge.
```

然后我们提交 `test/conflict1` 这条分支。

接下来切换到 `test/conflict2` 分支，同样更改 `example.txt`（当你刚刚切换到  `test/conflict2` 分支后，这里的 `example.txt`依然和 main 分支的  `example.txt` 相同）。我们给文件加上另一句话，现在  `test/conflict2` 分支的 `example.txt` 内容如下：

```
Hello, Git!

Learning git branch knowledge.

I love using git rebase, it works well.
```

同样，我们将  `test/conflict2` 这条分支推送到远端数据库。

现在，我们的分支情况是这样的

![](https://pic1.imgdb.cn/item/633d976a16f2c2beb1739ac7.png)

我们先把 `test/conflict1` 合并到main，因为现在main是无任何改动的，将 `test/conflict1`  merge 过去是 `fast-forward` 模式，我们可以很轻松的将它合并到 main 中

```
$ git checkout main
Switched to branch 'main'
Your branch is up to date with 'origin/main'.

$ git merge test/conflict1
Updating 9b83394..949f645
Fast-forward
 example.txt | 4 +++-
 1 file changed, 3 insertions(+), 1 deletion(-)

$ git push
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/FelixWuu/git_test
   9b83394..949f645  main -> main

$ git branch -d test/conflict1
Deleted branch test/conflict1 (was 949f645).
```

然后，我们再合并 `test/conflict2` 到 `main` 分支

```
$ git branch
* main
  master
  test/conflict2

$ git merge test/conflict2
Auto-merging example.txt
CONFLICT (content): Merge conflict in example.txt
Automatic merge failed; fix conflicts and then commit the result.
```

我们可以看到，这次合并是有冲突的，我们在同一个文件下进行了不同的变更。Git 尝试自动解决冲突，但是解决冲突失败了，它提示 `Automatic merge failed; fix conflicts and then commit the result.` Git 自动合并失败，需要我们手动修复冲突，然后再进行提交。

### 解决冲突

由于 main 之前合并了`test/conflict1` 分支，现在，最新的特性已经在 main 分支中，main 分支最新的 commit 是下图中红色的圈

![](https://pic1.imgdb.cn/item/633d9bdb16f2c2beb17d5627.png)

 `test/conflict2` 中的特性会对 main 分支的特性造成影响，它们的差异产生了冲突，我们现在打开 main 分支的 `example.txt` 文件看看

```
Hello, Git!

Learning git branch knowledge.

<<<<<<< HEAD
I'm used to merging branches with git merge.
=======
I love using git rebase, it works well.
>>>>>>> test/conflict2
```

第5行到第9行就是冲突所在，`=======`上方的当前分支有的句子，下方则是 test/conflict2 合并过来的句子，我们需要自己选择这里的变更。

我们选择两者都保留，自己删掉不需要的东西即可。

```
Hello, Git!

Learning git branch knowledge.

I'm used to merging branches with git merge.

I love using git rebase, it works well.
```

> 如果你用了代码编辑器，如 vscode 等，这里的冲突还会智能的提供按钮让你选择，是保留原有分支的特性还是新合并进来的特性，还是两者一起保留。

现在我们修改了文件，重新提交即可。

```
$ git status
On branch main
Your branch is up to date with 'origin/main'.

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   example.txt

$ git add .

$ git commit -m "fix: resolve conflicts"
[main c970aea] fix: resolve conflicts

$ git push
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 12 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 449 bytes | 449.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/FelixWuu/git_test
   949f645..c970aea  main -> main
   
$ git branch -d test/conflict2
Deleted branch test/conflict2 (was 6f30a3f).
```

现在，git 分支就如下图所示了

![](https://pic1.imgdb.cn/item/633d9e5016f2c2beb182df15.png)

> **你也可以在网页上删除远程分支**
>
> ![](https://pic1.imgdb.cn/item/633d9f9916f2c2beb185b867.png)
>
> ![](https://pic1.imgdb.cn/item/633d9fde16f2c2beb1865bc6.png)

### 用 rebase 合并

我们再做一次上面引入冲突的操作，从 main 上创建两个分支。分别在两个不同的分支上的 example.txt 文件上新增一行不一样的内容，先合并其中一条分支 test/conflict3

```
$ git branch test/conflict3

$ git branch test/conflict4

$ git checkout test/conflict3
Switched to branch 'test/conflict3'

-- 修改 example.txt, 然后提交，就略过不贴上来了 --

$ git checkout main
Switched to branch 'main'
Your branch is up to date with 'origin/main'.

$ git merge  test/conflict3
Updating c970aea..ad478ee
Fast-forward
 example.txt | 4 +++-
 1 file changed, 3 insertions(+), 1 deletion(-)
 
$ git push
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/FelixWuu/git_test
   c970aea..ad478ee  main -> main
```

现在我们修改 `test/conflict4`

```
$ git rebase master
```

之后和merge时的操作相同，然后我们再使用 merge 命令 `git merge test/conflict4`，可以看到已经变成了Fast-forward模式。

### 再谈 merge

`git merge`会将多个提交序列合并进一个统一的提交历史。它的完整使用语法如下，共三种：

```
git merge [-n] [--stat] [--no-commit] [--squash] [--[no-]edit]
    [-s <strategy>] [-X <strategy-option>] [-S[<keyid>]]
    [--[no-]allow-unrelated-histories]
    [--[no-]rerere-autoupdate] [-m <msg>] [<commit>…]
    
git merge --abort

git merge --continue
```

当然，我们并不需要完全记住它的使用方法。以下几种是最常用的

**git merge**

`merge` 默认是fast-forward方式来merge，仅保留单条分支记录。主分支会直接将最新 commit 指向合并进来的分支的顶端，因此提交记录为一条直线，看不出来是其他分支合并进来的。

这个模式下的合并操作有一个弊端，当我们merge合并后，将会删除无用的分支，比如我们删除了 `test/conflict1` 分支。在这种情况下，**删除分支后，会丢掉分支的所有信息**。

**git merge --no-ff**

`--no-ff`指的是强行关闭fast-forward方式。如果要强制禁用Fast Forward模式，Git就会在merge时生成一个新的commit。它可以保存之前的分支历史。能够更好的查看 merge历史，以及branch 状态。

**git merge --squash**

`git merge --squash` 是用来把一些不必要commit进行压缩，合并的时候我们不希望把一些历史commit带过来，比如说临时的提价，一些无用的提交等等，这些commit是无需在主分支中里体现的。 --squash 也需要进行一次额外的commit来汇总信息，然后完成最终的合并。