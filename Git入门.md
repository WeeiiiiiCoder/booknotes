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



