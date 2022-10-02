## Git 的数据库

Git 是一个分布式版本管理系统，可以在任何时间点将文件的状态作为更新记录保存起来。

Git 有以下两种数据库：

- 远程数据库：有专有的服务器，可多人共享
- 本地数据库：用户个人使用，在自己机器上的数据库

平时用自己的机器开发，并将保存在本地数据中，然后可以将自己的开发成果推送到远程数据库，或者从远程数据库获取到自己之前的开发成果或别人开发的内容，保存到本地数据库。

本地数据库向远程数据库推送数据称为 push，从远程数据库获取内容到本地数据库成为 pull

接下来我们来一步步操作，看看如何创建一个数据库，并从本地新建到推送到远程。

## Git 入门之本地操作

在使用 Git 前，需要先建立一个仓库，有以下两种情形：

**新建数据库** 和 **复制远程数据库**

我们先从新建数据库开始

首先创建一个文件夹 `git_test`, 然后打开 git bash

**1. 新建一个仓库**

```
git init
```

会提示

```
Initialized empty Git repository in E:/workspace/git_test/.git/
```

我们会看到文件夹下仍然空空如也，但是我们在查看勾选了`隐藏的项目`时，会看到文件夹中多了一个 `.git` 文件夹，这个文件夹就包含了 git 初始化后所需要的各种东西

- hook：存放了一些 shell 脚本
- info：存放了仓库的一些信息
- logs（新建的项目，还未操作过，所有么有）：保存了所有更新的引用记录
- objects：存放了 git 对象
- refs：保存当前最新的一次提交的哈希值
- config：git 仓库的配置文件
- description：仓库的描述信息
- HEAD：用于映射 ref 引用，能够找到下一次提交时的前一个哈希值

现在我们在 `git_test` 文件夹中新建一个名为 example.txt 的文件，并在文件中写入

```
Hello, Git!
```

**2. 查看变更信息**

```
git status
```

这个命令用于查看我们上次提交之后是否对文件有更改。现在，我们可以使用 git status 命令，然后可以看到如下的信息

```
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        example.txt

nothing added to commit but untracked files present (use "git add" to track)
```

由于我们还没有提交过，所以显示 `No commits yet`

并且 Git 还提醒我们有未跟踪的文件，example.txt 不是目前历史记录的对象，需要将 example.txt 加入到索引中，可以使用 add 命令

**3. 将文件添加到暂存区**

我们可以使用 add 命令将文件保存到暂存区

```
git add <file>..
```

> <file>： 需要保存到暂存区的文件名

我们可以给 add 指定参数，指定参数「.」，可以把所有的文件加入到暂存区。

```
git add .
```

现在我们使用 `git add .`将 git_test 下的文件都保存到暂存区，然后再次使用 status

```
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   example.txt
```

看信息描述，example.txt 已经被加入到暂存区了，我们也就可以提交文件了。

我们可以使用 git commit 命令来提交文件

```
$ git commit -m "first commit"
[master (root-commit) 9660240] first commit
 1 file changed, 1 insertion(+)
 create mode 100644 example.txt
```

- git commit： 提交暂存区的内容添加到本地仓库中
- -m：本次提交的一些备注信息（推荐每次提交的时候这里都要写）

我们再次使用 status 确定状态，可以看到我们没有新的变更记录了。

```
$ git status
On branch master
nothing to commit, working tree clean
```

最后我们可以使用 git log 查询版本的历史

```
commit 96602409abe0d4ab8f5fdf46e5dc44b69f4de1d6 (HEAD -> master)
Author: ---- 这里是git账户的个人信息 ----
Date:   Sun Oct 2 16:51:33 2022 +0800

    first commit
```

我们可以看到我们本次 commit 的一个序列号（哈希id）：`96602409abe0d4ab8f5fdf46e5dc44b69f4de1d6`，说明已经提交成功了

git log 非常强大，它在复杂的 git 中可以查看分支历史，展示提交信息。是我们会常用到的命令之一，之后本系列的文章会更详细的介绍 git log。

## Git 入门之远程操作

为了将本地数据库的修改记录共享到远程数据库，必须上传本地数据库中存储的修改记录。这一过程称之为**推送(Push)**。

有时候，我们在远程看到一个项目，需要把它完整的复制下来，就需要进行**克隆（Clone）**操作就来复制远程数据库。执行克隆后，远程数据库的全部内容都会被下载。比如说你想下载这个教程，可以使用 `git clone`下载，只要 `git clone` 加上 url 即可。

```
git clone https://github.com/FelixWuu/git_test.git
```

很多情况下一个项目都是由多人一起维护的，那么我们就需要拉去别人的修改到本地的数据库，进行**拉取(Pull)** 操作就可以把远程数据库的内容更新到本地数据库。进行拉取，就是从远程的数据库把别人的修改记录下载到本地数据库中。

### Push

我们试着将本地数据库推送到远程吧。我们先上 github 创建一个远程仓库，按默认的来就行。然后生成的仓库的 URL 如下：`https://github.com/FelixWuu/git_test`

在这之前，我们需要将本地数据库与远程数据库关联起来。我们可以使用 `git romote` 命令来关联远程仓库。remote 命令是这样的：

```
git remote add <name> <url>
```

- name 表示我们给远程仓库起的名字
- url 是远程仓库的 URL

我们尝试连接我们这个项目的地址（没有返回任何东西）

```
$ git remote add origin https://github.com/FelixWuu/git_test
```

- 我们将远程仓库命名为 origin
  - 我推荐远程仓库都叫 origin，因为我们执行 push 或 pull 的时候，如果省略了远程仓库的名字，那么会使用默认的名字 `origin`，因此一般情况下，远程仓库都叫做 `origin`

关联了之后，我们可以用 remote 来查看远程仓库的信息

```
$ git remote -v
origin  https://github.com/FelixWuu/git_test (fetch)
origin  https://github.com/FelixWuu/git_test (push)
```

现在，我们将远程仓库推送上去。我们使用 `git pull` 命令，将我们的内容推送到远程的 master 分支上。

```
$ git push -u origin master
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 224 bytes | 224.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
remote:
remote: Create a pull request for 'master' on GitHub by visiting:
remote:      https://github.com/FelixWuu/git_test/pull/new/master
remote:
To https://github.com/FelixWuu/git_test
 * [new branch]      master -> master
branch 'master' set up to track 'origin/master'.
```

尝试输入 git branch 吧，它可以看到我们本地的信息

```
$ git branch
* master
```

关于 git branch，我们之后会详细聊聊它，现在，你只需要知道它的功能是展示本地的分支。

--------

> 如果 Push 失败，很有可能是没有配置好 ssh-key，这里简述一下 ssh key 配置的完整步骤

1. 检查一下是否存在 ssh-key

切换到 .ssh 文件夹下

```
cd ~/.ssh
```

然后看看有什么文件

```
ls
```

看看是否存在 id_rsa 和 id_rsa.pub文件，如果存在，则说明已经有 ssh-key 了

2. 如果没有 ssh-key，则生成一下

```
ssh-keygen -t rsa -C "你的登录账号"
```

之后 git 会询问你一些信息，直接回车，默认同意

3. 查看生成的密钥

```
cat id_rsa.pub
```

复制密钥内容

4. 到 github 添加 ssh-key

点击 github 用户的头像，点击 setting，选择 SSH and GPG keys，新建一个 SSH key （New SSH key 按钮）

然后给你的密钥取个名字，把之前复制的密钥内容拷贝进去，添加即可。

5. 验证是否配置成功

```
ssh -T git@github.com
```

可以看到返回

```
$ ssh -T git@github.com
Hi FelixWuu! You've successfully authenticated, but GitHub does not provide shell access.
```

--------

### Clone

你会发现，经过push之后，我们在 https://github.com/FelixWuu/git_test 并没有我们推送上去的内容，这是因为我们推送上去的分支名是 master，而现在，分支名为 main 的才是主分支。

> 自2020年10月1日后，`Github`会将所有新建的仓库的默认分支从`master`修改为`main`

我们切换分支，即可以看到文件

![](https://pic1.imgdb.cn/item/6339a5d416f2c2beb1857263.png)

借此机会，我们熟悉一下 clone。就用 clone 把项目拉下来吧。我们现在要做的事情如下：

1. 克隆原仓库
2. 创建和推送 main 分支

选择任意一个文件夹，然后打开 git bash，把 git_test clone下来

```
$ git clone https://github.com/FelixWuu/git_test.git
Cloning into 'git_test'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 6 (delta 0), reused 3 (delta 0), pack-reused 0
Receiving objects: 100% (6/6), done.
```

然后切换到 git_test 项目下

```
cd git_test/
```

这时候，我们再使用 git branch 看看分支信息

```
$ git branch
* main
```

可以看到我们现在所处的分支是 main 分支，里面空空如也（如果你在建立项目时选择了生成一个 README，那么 main 分支会带有一个 README.md 文件，该文件一般用来写项目的说明）

我们再把原来项目中的example.txt拷贝到现在clone下来的文件夹中，然后 git status 看看结果

```
$ git status
On branch main
Your branch is up to date with 'origin/main'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        example.txt

nothing added to commit but untracked files present (use "git add" to track)
```

我们依然走一遍 add - commit - push 的流程 （记得 push 到 main 分支）

```
$ git add .

$ git commit -m "push to main"
[main 2b34357] push to main
 1 file changed, 1 insertion(+)
 create mode 100644 example.txt

$ git push -u origin main
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 12 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 290 bytes | 290.00 KiB/s, done.
Total 3 (delta 0), reused 1 (delta 0), pack-reused 0
To https://github.com/FelixWuu/git_test.git
   6972647..2b34357  main -> main
branch 'main' set up to track 'origin/main'.
```

![](https://pic1.imgdb.cn/item/6339a92116f2c2beb18d1f6d.png)

现在，我们可以在 main 分支下看到我们的 example.txt 了

-----------

当然，我们也可以自己设置主分支

![](https://pic1.imgdb.cn/item/6339aa4116f2c2beb18f239a.png)

--------------------

### Pull

现在我们试试 pull 命令。它的格式是：`git pull <repository> <refspec>...`

我们试着拉去 master 分支。因为我们在新的文件夹中操作，它并不知道 master 分支是什么，现在先按照如下命令执行

```
$ git checkout -b master origin/master
Switched to a new branch 'master'
branch 'master' set up to track 'origin/master'.
```

它的意思是切换到本地分支 master 并关联远程的 master 分支

```
$ git branch
  main
* master
```

现在查看本地分支中可以看到两条分支了，并且master分支前面有一个 `*` 号，且master分支被高亮，这说明我们当前处于 master 分支

执行一次 git pull 看看

```
$ git pull origin master
From https://github.com/FelixWuu/git_test
 * branch            master     -> FETCH_HEAD
Already up to date.
```

现在，我们切换到老的 git_test 项目中，修改一下 example.txt 并推送上去，然后新增一个文件 example2.txt.

我们给 example.txt 新增一句话

*example.txt*

```
Hello, Git!

Hi, Git! We meet again!
```

并新建一个 example2.txt，随便填写点内容

*example2.txt*

```
I'm NutCat.
```

然后在老的文件夹中开启一个新的 git bash，将改动推送到远端（使用 `git status` 可以看到我们对文件的改动）

```
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   example.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        example2.txt
```

推送流程，这里我们 `git push`不指定分支，Git 知道我们当前分支关联的远程分支是 origin/master， 并默认将其推送到对应的远程分支上。

```
$ git add .

$ git commit -m "second revision"
[master 90c277e] second revision
 2 files changed, 4 insertions(+), 1 deletion(-)
 create mode 100644 example2.txt

$ git push
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 12 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (4/4), 324 bytes | 324.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/FelixWuu/git_test
   9660240..90c277e  master -> master
```

![](https://pic1.imgdb.cn/item/6339af5416f2c2beb1993f32.png)

现在回到我们新克隆的文件夹中，可以看到文件还是只有 example.txt的。现在打开 git bash，试着再 `git pull` 一次。

```
$ git pull
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 4 (delta 0), reused 4 (delta 0), pack-reused 0
Unpacking objects: 100% (4/4), 304 bytes | 1024 bytes/s, done.
From https://github.com/FelixWuu/git_test
   9660240..90c277e  master     -> origin/master
Updating 9660240..90c277e
Fast-forward
 example.txt  | 4 +++-
 example2.txt | 1 +
 2 files changed, 4 insertions(+), 1 deletion(-)
 create mode 100644 example2.txt
```

我们成功的拉去到了项目文件，并且改动和新增的文件都出现再我们文件夹中了。核对一下，内容一致。

- 这里 pull 也没有指定分支名，因为我们之前切换到了master分支了，这条分支也关联了远程的master分支，因此pull时 Git 知道选择哪条分支。如下，括号中的就是当前分支的分支名。

```
Administrator@NutCat MINGW64 /e/temp/git_test (master)
```

