# Git入门

## Windows上安装git

在Windows上使用Git，可以从Git官网直接[下载安装程序](https://git-scm.com/downloads)，（网速慢的同学请移步[国内镜像](https://pan.baidu.com/s/1kU5OCOB#list/path=%2Fpub%2Fgit)），然后按默认选项安装即可。

安转完成后，右键"Git Bash Here"或者开始菜单里找到“Git Bash”，跳出Git命令行窗口

安装完成后创建自己的git账户

~~~~
$ git config --global user.name "Your Name"
$ git config --global user.email "Your email"
~~~~

因为Git是分布式版本控制系统，所以，每个机器都必须自报家门：你的名字和Email地址。

注意`git config`命令的`--global`参数，用了这个全局参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。

## 初始化版本库

git通过仓库来管理仓库里面的文件，进入一个文件夹内初始化仓库

~~~
$ mkdir github
$ cd github/
$ git init
Initialized empty Git repository in E:/github/.git/
~~~

通过`git init`初始化仓库，仓库内会出现.git目录，这个目录是进行git版本控制的，不要修改。.git目录默认隐藏，通过`ls -a `命令可以看见。

~~~
$ ls -a
./  ../  .git/
~~~

## 把文件添加到仓库

编写一个`readme.txt`文件，把这个文件放到仓库`github`目录下（子目录也行），放在别的地方Git是无法找到这个文件的。

把一个文件放到到Git仓库需要两步，第一步通过`git add`命令把文件添加到暂存区index

~~~
$ git add readme.txt
~~~

执行`git add`命令，没有任何提示说明文件添加成功。

添加成功之后通过`git commit`把文件提交到仓库

~~~
$ git commit -m 'wrote a readme file'
[master (root-commit) 52edf3b] wrote a readme file
 1 file changed, 2 insertions(+)
 create mode 100644 readme.txt
~~~

`git commit`用于提交暂存区的文件到仓库，`-m`后面的是本次提交的注释信息，`1 file changed, 2 insertions(+)`表示一个文件改动，插入两行内容。git可以一次add多个文件，然后一次性commit。

## 查看状态和diff

成功添加文件到仓库后对源文件进行修改

~~~
git is a distribution version control system
git is free software
~~~

通过`git status`命令查看当前仓库的状态

~~~
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
~~~

运行结果显示`master`分支有 `readme.txt`文件被修改但是未提交。

通过`git diff`命令查看`index`暂存区的改变。

~~~
$ git diff
warning: LF will be replaced by CRLF in readme.txt.
The file will have its original line endings in your working directory
diff --git a/readme.txt b/readme.txt
index 347140a..3226a48 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1 +1,2 @@
 git is a version control system
+git is free software
~~~

`git diff`结果显示文件中修改的内容。在确认文件修改内容之后重新将修改提交，这一步和之前的命令一致。

~~~
$ git add readme.txt

$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   readme.txt
        
$ git commit -m 'add distribution'
[master c6d8007] add distribution
 1 file changed, 2 insertions(+), 1 deletion(-)
 
$ git status
On branch master
nothing to commit, working tree clean
~~~

可以看到在`git add`到暂存区之后结果显示文档的修改可以被`git commit`，再通过`git commit`提交到仓库。

## 版本回退

再次对文件修改

~~~
git is a distribution version control system
git is free software distributed under the GPL
~~~

~~~
$ git add readme.txt
$ git commit -m 'append GPL'
[master f0074eb] append GPL
 1 file changed, 1 insertion(+), 1 deletion(-)
~~~

每次在`git commit`的时候，就是保存了一次版本快照。当由于某些原因想把仓库中的文件恢复到最近的某一次提交，那么就需要从最近的快照恢复。在`commit`次数很多的时候，想查看历史版本快照，可以使用`git log`命令。

~~~
$ git log
commit f0074ebb5577f3c62e2862d535806847c1930b10 (HEAD -> master)
Author: zhouwei <chouwe052@163.com>
Date:   Fri Nov 29 11:14:25 2019 +0800

    append GPL

commit c6d800745507a323ba806547323479eb17974f7b
Author: zhouwei <chouwe052@163.com>
Date:   Fri Nov 29 10:51:38 2019 +0800

    add distribution

commit 90333efd7a60335539ef80ad9a67668c0381f87d
Author: zhouwei <chouwe052@163.com>
Date:   Fri Nov 29 10:41:50 2019 +0800

    first file
~~~

`git log`结果显示最近三次`git commit`的快照，通过提交注释信息就可以明显的观察到。如果觉得输出信息太多，可以加上`--pretty=online`，瞬间清爽有木有~

~~~
$ git log --pretty=oneline
f0074ebb5577f3c62e2862d535806847c1930b10 (HEAD -> master) append GPL
c6d800745507a323ba806547323479eb17974f7b add distribution
90333efd7a60335539ef80ad9a67668c0381f87d first file
~~~

上面结果中每次快照由`commit id(版本号)` 和commit提交注释组成，如`f0074`就是最近一次`commit id`。和SVN不一样，Git的版本号不是1,2,3...递增的数字，而是一个SHA1计算出来的一个十六进制数字。因为Git是分布式VCS，所以为了避免版本号冲突，不能采用递增的版本号。

在Git中使用`HEAD`表示当前版本，在上面结果中显示就是`f0074`那个版本，上个版本就是`HEAD^`，上上个版本就是`HEAD^^`，往上100版本就用`HEAD~100`表示。

现在想恢复到`add distribution`版本，需要`git reset`命令。

~~~
$ git reset --hard HEAD^
HEAD is now at c6d8007 add distribution
~~~

从结果可以看到当前版本在过去的`c6d8007`那个版本。现在查看文件中的内容是不是之前的版本内容。

~~~~
$ cat readme.txt
git is a distribution version control system
git is free software
~~~~

结果可以看到文件中内容是`add distribution`版本的内容。

此时再用`git log `命令查看版本记录就会发现刚刚`append GPL`版本不在记录中，如果此时想回到`append GPL`版本，就需要找到该版本的`commit id  f0074e` 。

~~~~
$ git log
commit c6d800745507a323ba806547323479eb17974f7b (HEAD -> master)
Author: zhouwei <chouwe052@163.com>
Date:   Fri Nov 29 10:51:38 2019 +0800

    add distribution

commit 90333efd7a60335539ef80ad9a67668c0381f87d
Author: zhouwei <chouwe052@163.com>
Date:   Fri Nov 29 10:41:50 2019 +0800

    first file
~~~~

执行`git reset`命令又回到未来的`append GPL`版本，查看文件信息的确为GPL版本。一来一回就好像在git版本控制系统中反复穿越。

~~~
$ git reset --hard f0074e
HEAD is now at f0074eb append GPL

$ cat readme.txt
git is a distribution version control system
git is free software distributed under the GPL
~~~

Git的版本回退速度很快，因为Git在内部有个指向当前版本的`HEAD`指针，当你版本回退的时候，Git仅仅把`HEAD`指针从指向`append GPL`改为指向`add ditribution`，顺便更新工作区。所以`HEAD`指向哪个版本号，当前就是那个版本。

如果关闭了当前窗口找不到`commit id`时，可以通过`git reflog`命令记录每一次命令。

~~~
$ git reflog
f0074eb (HEAD -> master) HEAD@{0}: reset: moving to f0074e
c6d8007 HEAD@{1}: reset: moving to HEAD^
f0074eb (HEAD -> master) HEAD@{2}: commit: append GPL
c6d8007 HEAD@{3}: commit: add distribution
90333ef HEAD@{4}: commit (initial): first file
~~~

从结果中可以获取到想要的`commit id`。

## 工作区和暂存区

工作区就是本地创建的目录，工作区有一个`.git`隐藏文件，这个不属于工作区，而是Git的版本库。

Git的版本库中有很多git的东西，包括indes(或者叫stage)暂存区，还有git帮我们创建的第一个分支`master`以及指向`master`的指针`HEAD`。

之前往Git版本库添加文件是分为两步

- 第一步用`git add`命令，实际就是把文件从工作区提交到暂存区
- 第二步用`git commit`命令，实际就是把文件从暂存区提交到当前分支。

现在对`readme.txt`文件随便修改，同时也在工作区添加一个`LICENSE.txt`文件，调用`git status`方法

~~~
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   readme.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        LICIENSE.txt

no changes added to commit (use "git add" and/or "git commit -a")
~~~

运行结果中可以发现`readme.txt`文件被修改，`LICENSE.txt`文件被标记为`untracked files`，从未被添加，未被版本库追踪到。现在`git add`这两个文件，再调用`git status`

~~~
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   LICIENSE.txt
        modified:   readme.txt
~~~

结果可以看到暂存区添加了新文件`LICENSE.txt`，所以`git add`命令是将所以要提交的修改添加到暂存区，`git commit`命令将暂存区的修改全部提到当前分支。

~~~~
$ git commit -m 'understand how git works'
[master 460eec8] understand how git works
 2 files changed, 1 insertion(+)
 create mode 100644 LICIENSE.txt
~~~~

`commit`之后查看工作区是干净的，不存在未提交未添加的文件。

~~~
$ git status
On branch master
nothing to commit, working tree clean
~~~

## 管理修改

 Git比其他的CVS优秀的原因就是Git跟踪并管理的是修改，而非文件。  你会问，什么是修改？比如你新增了一行，这就是一个修改，删除了一行，也是一个修改，更改了某些字符，也是一个修改，删了一些又加了一些，也是一个修改，甚至创建一个新文件，也算一个修改。 

下面实际操作一下，先给`readme.txt`文件添加一行，然后添加到暂存区。

~~~
$ cat readme.txt
git is a distribution version control system
git is free software distributed under the GPL
git is wocao
git consists of index

$ git add readme.txt
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   readme.txt
~~~

然后再修改

~~~
$ cat readme.txt
git is a distribution version control system
git is free software distributed under the GPL
git is wocao
git consists of index
git created by linus
~~~

再提交

~~~
$ git commit -m 'git is better than svn'
[master 0bee6e0] git is better than svn
 1 file changed, 1 insertion(+)
~~~

再查看

~~~
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
~~~

结果可以看到第二次修改没有被提交。

而之前的操作是第一次修改->`git add`->第二次修改->`git commit`。第二次修改的内容没有被提交到暂存区，所以`git commit`只讲第一次修改提交到当前分支。

提交后可以通过`git diff HEAD -- readme.txt`查看工作区和版本库的区别。

## 撤销修改

从前面我们知道Git跟踪的是修改，那么修改文件之后怎么撤销修改呢。

~~~
this is a license file
~~~

修改完之后调用`git status`命令发现可以通过`git restore <file>`命令丢弃工作区的修改。[也可以用`git checkout -- <file>`]

~~~
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   LICIENSE.txt

no changes added to commit (use "git add" and/or "git commit -a")
~~~

命令`git checkout -- readme.txt`意思就是，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：

丢弃工作区的修改：`readme.txt`修改后还没有`git add`到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

丢弃暂存区的修改：`readme.txt`已经`git add`到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。

~~~
$ git checkout -- LICIENSE.txt
~~~

还有一种撤销修改的场景

在对文件修改并`git add`到暂存区，在`commit`之前可以调用`git reset HEAD <file>`撤销(unstage)暂存区的修改，重新放回工作区。

 `git reset`命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用`HEAD`时，表示最新的版本。 

~~~
$ git reset HEAD LICIENSE.txt
Unstaged changes after reset:
M       LICIENSE.txt
~~~

再调用`git checkout LICENSE.TXT`丢弃工作区的修改，此时工作区也清理完毕。

~~~
$ git checkout LICIENSE.txt
Updated 1 path from the index

$ git status
On branch master
nothing to commit, working tree clean
~~~

## 删除文件

如果一个文件添加到版本库中，而此时直接通过资源管理器或者`rm`命令删除了该文件，那么如何实现版本库中的文件也删除？

~~~~
$ git add test.txt

$ git commit -m "add test.txt"
[master b84166e] add test.txt
 1 file changed, 1 insertion(+)
 create mode 100644 test.txt
~~~~

调用`git rm <file>`删除版本库中的文件， 并且`git commit`： 

```
$ git rm test.txt
rm 'test.txt'

$ git commit -m "remove test.txt"
[master d46f35e] remove test.txt
 1 file changed, 1 deletion(-)
 delete mode 100644 test.txt
```

调用`git checkout -- <file>`把删除的文件恢复到最新版本。

## 添加远程库

在本地版本库中获取github远程库地址，调用命令关联远程库。 添加后，远程库的名字就是`origin`，这是Git默认的叫法，也可以改成别的，但是`origin`这个名字一看就知道是远程库。 

~~~
$ git remote add origin git@github.com:WeeiiiiiCoder/booknotes.git
~~~

把本地库的文件推送到远程库

~~~
$ git push -u origin master
~~~

 由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。 

以后通过`git push origin master `推送本地master到远程库。

## 克隆远程库

和上面一样获取github仓库地址，并调用`git clone`命令克隆远程库到本地

~~~
$ git clone git@github.com:WeeiiiiiCoder/booknotes.git
~~~

## 创建与合并分支

Git的每次`commit`可以串成一个时间线，默认只有一条时间线，就是master分支。`HEAD`严格来说不是指向提交，而是指向`master`，而`master`才是指向提交的，所以`HEAD`就是指的当前分支。

每次提交，`master`都会指向新的提交点，而`HEAD`指向`master`。当在`master`指向的某个提交点创建了新的分支`dev`之后，这个`dev`分支也会指向该提交点，如果`HEAD`再切换到`dev`，就表示分支为`dev`。

此时，对工作区的提交和修改就是在`dev`分支上进行的了，`dev`指针会指向新的`commit`提交点，而`master`指针还是指向原来的提交点。

如果`dev`分支工作完成，想把`dev`合并到`master`上。Git怎么合并呢？最简单的方法，就是直接把`master`指向`dev`的当前提交，就完成了合并。 所以Git合并分支也很快！就改改指针，工作区内容也不变！   合并完分支后，甚至可以删除`dev`分支。删除`dev`分支就是把`dev`指针给删掉，删掉后，我们就剩下了一条`master`分支：

先创建`dev`分支，并切换到该分支。

```
$ git checkout -b dev
Switched to a new branch 'dev'
```

`-b`表示先创建分支，再切换到新分支，相当于

~~~
$ git branch dev
$ git checkout dev
~~~

或者创建并切换到新的`dev`分支，可以使用：

```
$ git switch -c dev
```

切换完分支之后可以查看当前分支，`*`表示当前分支。

~~~
$ git branch
* dev
  master
~~~

然后对工作区的`readme.txt`文件添加一行，并进行提交。

~~~
Creating a new branch is quick.

$ git add readme.txt
$ git commit -m 'create branch'
[dev c6ddfbe] create branch
 1 file changed, 1 insertion(+)
~~~

在切换到`master`分支，查看该文件

~~~
$ git switch master
Switched to branch 'master'

$ git branch
  dev
* master

$ cat readme.txt
git is a distribution version control system
git is free software distributed under the GPL
git is wocao
git consists of index
git tracks changes
git created by linus
~~~

发现`dev`分支上的提交并没有在`master`分支上，`master`分支还是之前的提交点的状态。

所以此时需要把`dev`分支的内容合并到`master`分支上。调用`git merge <branch>`合并指定分支到当前分支。

~~~
$ git merge dev
Updating 0bee6e0..c6ddfbe
Fast-forward
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
~~~

在合并分支的结果中， 注意到上面的`Fast-forward`信息，Git告诉我们，这次合并是“快进模式”，也就是直接把`master`指向`dev`的当前提交，所以合并速度非常快，但也有特殊情况。

此时在查看文件内容

~~~
$ cat readme.txt
git is a distribution version control system
git is free software distributed under the GPL
git is wocao
git consists of index
git tracks changes
git created by linus
Creating a new branch is quick.
~~~

文件内容为`dev`分支最新提交的。

此时有需要可以删除`dev`分支了

~~~
$ git branch -d dev
Deleted branch dev (was c6ddfbe).
~~~

## 解决冲突

刚刚在`dev`分支对`readme.txt`文件修改提交后直接切换到`master`分支，并没有在`master`

分支对`readme.txt`文件进行提交修改。但是如果刚好两个分支都对同一个文件进行提交修改，那么就可能产生冲突。

~~~
# 创建并切换到新的pro分支
$ git switch -c pro
Switched to a new branch 'pro'
# 修改提交readme.txt
$ git add readme.txt
$ git commit -m 'add simple'
[pro 83b2f7a] add simple
 1 file changed, 1 insertion(+), 1 deletion(-)
# 切换回master分支
$ git checkout master
Switched to branch 'master'
# 修改提交readme.txt文件
$ git add readme.txt
$ git commit -m 'add & simple'
[master e790cb2] add & simple
 1 file changed, 1 insertion(+), 1 deletion(-)
# 合并pro分支到master分支,readme.txt文件产生冲突
$ git merge pro
Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt
Automatic merge failed; fix conflicts and then commit the result.
~~~

此时需要进入`readme.txt`文件解决冲突。

~~~
$ vi readme.txt
git is a distribution version control system
git is free software distributed under the GPL
git is wocao
git consists of index
git tracks changes
git created by linus
<<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick and simple.
>>>>>>> pro
~~~

修改为`pro`分支的内容，再重新提交

~~~
$ git add readme.txt
$ git commit -m 'fix conflict'
[master 57972e7] fix conflict
~~~

此时可以查看分支合并情况

~~~
$ git log --graph --pretty=oneline --abbrev-commit
*   57972e7 (HEAD -> master) fix conflict
|\
| * 83b2f7a (pro) add simple
* | e790cb2 add & simple
|/
* c6ddfbe create branch
* 0bee6e0 git is better than svn
* 3455379 git tracks changes-2
* 716afe6 git track changes
* 460eec8 understand how git works
* f0074eb append GPL
* c6d8007 add distribution
* 90333ef first file
~~~

最后删除分支

~~~
$ git branch -d pro
Deleted branch pro (was 83b2f7a).
~~~

## 分支管理策略

通常Git在进行分支合并的时候就使用`Fast Forward`模式，但是这种模式在删除分支后，会丢失分支信息。如果强制禁用`Fast forward`模式，Git就会在merge的时候生成一个新的commit id，这样分支历史就会看到分支信息。下面演示一下

先创建被切换到`dev`分支

~~~
$ git switch -c dev
~~~

再修改`readme.txt`文件，并提交

~~~
$ git add readme.txt

$ git commit -m 'add merge'
~~~

切换回`master`分支

~~~
$ git switch master
~~~

禁用`fast Forward`模式合并分支

~~~
$ git merge --no-ff -m 'add merge' dev
~~~

调用日志会出现`dev`分支的commit id

~~~~
$ git log --graphy --pretty=oneline
*   7c02e8c9bd6d7fa7b456307b203d3763c4bb5953 (HEAD -> master) Merge branch 'dev'
|\
| * 137c44b9d869c657e9ba9a1fec1b226548550f7a (dev) add merge
|/
*   57972e7e8511250e7725ff11d74a64ac851bc037 fix conflict
~~~~

## bug分支

软件开发中，bug就像家常便饭一样。有了bug就需要修复，在Git中，由于分支是如此的强大，所以，每个bug都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除。

当你在`dev`分支上工作时，接到一个修复一个代号101的bug的任务，很自然地，你想创建一个分支`issue-101`来修复它，但是`dev`分支上的工作还没有结束

```
$ git status
On branch dev
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   hello.py

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   readme.txt
```

并不是你不想提交，而是工作只进行到一半，还没法提交，预计完成还需1天时间。但是，必须在两个小时内修复该bug，怎么办？

幸好，Git还提供了一个`stash`功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：

```
$ git stash
Saved working directory and index state WIP on dev: f52c633 add merge
```

现在，用`git status`查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。

首先确定要在哪个分支上修复bug，假定需要在`master`分支上修复，就从`master`创建临时分支：

```
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 6 commits.
  (use "git push" to publish your local commits)

$ git checkout -b issue-101
Switched to a new branch 'issue-101'
```

现在修复bug，需要把“Git is free software ...”改为“Git is a free software ...”，然后提交：

```
$ git add readme.txt 
$ git commit -m "fix bug 101"
[issue-101 4c805e2] fix bug 101
 1 file changed, 1 insertion(+), 1 deletion(-)
```

修复完成后，切换到`master`分支，并完成合并，最后删除`issue-101`分支：

```
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 6 commits.
  (use "git push" to publish your local commits)

$ git merge --no-ff -m "merged bug fix 101" issue-101
Merge made by the 'recursive' strategy.
 readme.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

太棒了，原计划两个小时的bug修复只花了5分钟！现在，是时候接着回到`dev`分支干活了！

```
$ git checkout dev
Switched to branch 'dev'

$ git status
On branch dev
nothing to commit, working tree clean
```

工作区是干净的，刚才的工作现场存到哪去了？用`git stash list`命令看看：

```
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge
```

工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：

一是用`git stash apply`恢复，但是恢复后，stash内容并不删除，你需要用`git stash drop`来删除；

另一种方式是用`git stash pop`，恢复的同时把stash内容也删了：

```
$ git stash pop
On branch dev
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   hello.py

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   readme.txt

Dropped refs/stash@{0} (5d677e2ee266f39ea296182fb2354265b91b3b2a)
```

再用`git stash list`查看，就看不到任何stash内容了：

```
$ git stash list
```

你可以多次stash，恢复的时候，先用`git stash list`查看，然后恢复指定的stash，用命令：

```
$ git stash apply stash@{0}
```

在master分支上修复了bug后，我们要想一想，dev分支是早期从master分支分出来的，所以，这个bug其实在当前dev分支上也存在。

那怎么在dev分支上修复同样的bug？重复操作一次，提交不就行了？

有木有更简单的方法？

有！

同样的bug，要在dev上修复，我们只需要把`4c805e2 fix bug 101`这个提交所做的修改“复制”到dev分支。注意：我们只想复制`4c805e2 fix bug 101`这个提交所做的修改，并不是把整个master分支merge过来。

为了方便操作，Git专门提供了一个`cherry-pick`命令，让我们能复制一个特定的提交到当前分支：

```
$ git branch
* dev
  master
$ git cherry-pick 4c805e2
[master 1d4b803] fix bug 101
 1 file changed, 1 insertion(+), 1 deletion(-)
```

Git自动给dev分支做了一次提交，注意这次提交的commit是`1d4b803`，它并不同于master的`4c805e2`，因为这两个commit只是改动相同，但确实是两个不同的commit。用`git cherry-pick`，我们就不需要在dev分支上手动再把修bug的过程重复一遍。

有些聪明的童鞋会想了，既然可以在master分支上修复bug后，在dev分支上可以“重放”这个修复过程，那么直接在dev分支上修复bug，然后在master分支上“重放”行不行？当然可以，不过你仍然需要`git stash`命令保存现场，才能从dev分支切换到master分支。

## Feature分支

当项目开发一个新特性的时候，建议新建一个feature分支，在这个分支上开发，合并最后删除该分支。

实战

新feature，并在feature分支未合并之前删除分支时会提示该分支还没有完全合并，可以用`git branch -D <branch>`强制删除分支。

~~~
$ git checkout -b ship
Switched to a new branch 'ship'

$ vi ship.py

$ git add ship.py

$ git commit -m 'add feature ship'
[ship 6e0844a] add feature ship
 1 file changed, 1 insertion(+)
 create mode 100644 ship.py

$ git checkout master
Switched to branch 'master'

$ git branch -d ship
error: The branch 'ship' is not fully merged.
If you are sure you want to delete it, run 'git branch -D ship'.

$ git branch -D ship
Deleted branch ship (was 6e0844a).
~~~

## 多人协作

当在本地克隆远程仓库的时候，实际Git自动把本地的`master`分支和远程仓库的`master`分支对应起来了，远程仓库默认名称是`origin`。

`git remote`命令可以查看远程库信息

~~~
$ git remote
origin
~~~

或者，用`git remote -v`显示更详细的信息：

```
$ git remote -v
origin  git@github.com:michaelliao/learngit.git (fetch)
origin  git@github.com:michaelliao/learngit.git (push)
```

上面显示了可以抓取和推送的`origin`的地址。如果没有推送权限，就看不到push的地址。

### 推送分支

推送本地分支到远程仓库，推送时要指定本地分支， 这样，Git就会把该分支推送到远程库对应的远程分支上 。 如果推送失败，先用`git pull`抓取远程的新提交； 

~~~
$ git push origin master
$ git push origin dev
~~~

### 抓取分支

从远程库克隆项目

~~~
git clone git@server-name:address/xxx.git
~~~

刚克隆的时候本地只能看到`master`分支，可以通过`git branch`检验

所以在本地创建`dev`分支对应远程库`origin`的`dev`分支，在该分支上进行操作。 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`； 

~~~
$ git checkout -b dev origin/dev
~~~

