## 背景知识
detached HEAD state，分离头指针，即HEAD指针直接指向提交记录的情况  
正常情况下，HEAD应指向某一分支
如果执行了`git checkout tag名`或`git checkout 远端分支名`或`git checkout 提交记录哈希值`，则HEAD会指向指向某一提交记录，这就是detached HEAD state

## 实验
先用git log和图形化软件git log查看一下开始实验前test仓库的提交记录树
```
$ git log
commit 314b5494eb06a9963f4e6f97075f2354c8a15f13 (HEAD -> master)
Author: MilesGO <164173218@qq.com>
Date:   Sun Jun 9 17:20:40 2019 +0800

    add main2.cpp

commit 12c2c80788dcd74a1cd739157c0a1011514d36b2
Author: MilesGO <164173218@qq.com>
Date:   Sun Jun 9 17:11:48 2019 +0800

    add src/main.cpp

commit 4a1d75425a4cd5c3e206a4c69415c59802175039
Author: MilesGO <164173218@qq.com>
Date:   Sun Jun 9 17:00:08 2019 +0800

    modify README

commit 29c07152991c7b3ee41f9cb5ac1eff1f26610665
Author: MilesGO <164173218@qq.com>
Date:   Sun Jun 9 16:52:41 2019 +0800

    add README
```
用git-fork查看之后，可以看到整棵提交记录树是线性的
我们直接用哈希值检出到倒数第二次提交
```
$ git checkout 12c2c8
Note: checking out '12c2c8'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 12c2c80 add src/main.cpp
```
翻译一下上面的内容，您正处于分离头指针状态，你可以做一些实验性的改动并且提交它们，但是这些提交都会被丢弃，除非你用一个分支指向它们
当前这个分支下还没有main2.cpp文件，我们添加这个文件
```
$ echo 'hello world again' >> main2.cpp
$ git add main2.cpp
$ git commit -m 'add main2.cpp again'
[detached HEAD 11ed7f1] add main2.cpp again
 1 file changed, 1 insertion(+)
 create mode 100644 main2.cpp

$ git checkout master
Warning: you are leaving 1 commit behind, not connected to
any of your branches:

  11ed7f1 add main2.cpp again

If you want to keep it by creating a new branch, this may be a good time
to do so with:

 git branch <new-branch-name> 11ed7f1

Switched to branch 'master'
```
翻译一下上面的内容，你把1个未用分支指向的提交记录给落掉了，其哈希值为11ed7f1，如果你想要保留这个提交，则请用`git branch <new-branch-name> 11ed7f1`指令创建1个新的分支并指向这个提交记录
```
$ git branch -av
* master 314b549 add main2.cpp

git branch pick 11ed7f1

$ git branch -av
* master 314b549 add main2.cpp
  pick   11ed7f1 add main2.cpp again
  
```
