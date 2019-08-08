git将引用保存在文件中，原理很简单

## 引用原理
`引用`指的是对`提交记录`的引用  
`提交记录`用`哈希值`唯一标识  
每个`引用`用一个文件表示，文件中保存`其引用的提交记录的哈希值`  
## 引用分类
- 分支
    - 可变, 在不同的时刻可以指向`不同的提交记录`
    - 本地分支
        - 对应`.git/refs/heads/目录`中的文件
        - 每个`本地仓库`有多个`本地分支`
    - 远程分支
        - 对应`.git/refs/remotes/<远端仓库名>/目录`中的文件
        - 每个`本地仓库`可以对应多个`远端仓库`, 同时每个`远端仓库`可以有多个`远端分支`
- tag
    - 对应`.git/refs/tags/目录`中的文件
    - 不可变, 除非删除后重新创建, 否则总是指向`特定的提交记录`
    - 每个git仓库可以有多个`tag`
- HEAD
    - 对应`.git/HEAD`文件
    - 可变
        - 通常指向某个`本地分支`，即引用的引用
        - 也可以直接指向`某个提交记录`，称为`HEAD detached`, 即分离头指针状态
        - 也可以指向`tag`，git将这种情况也处理成`HEAD detached`
        - 也可以指向`远端分支`, git将这种情况也处理成`HEAD detached`
    - 每个git仓库只有一个`HEAD`
    - 表示当前`工作区`检出的文件(或者说文件在修改之前)是属于哪个`提交记录`的
    - `git checkout 指令`，就是在改变HEAD的指向
        - `git checkout 本地分支名`
        - `git checkout 提交记录哈希值`, detached
        - `git checkout 远端分支名`, detached
        - `git checkout tag名`, detached

## 实验
```
$ git checkout master
Switched to branch 'master'

$ cat .git/HEAD
ref: refs/heads/master

$ cat .git/refs/heads/master
89d496d44f93d107a7eb404890cd15a14ba8845d
```  
checkout master后, HEAD指向master, master指向89d496

```
$ git checkout milestone
Note: checking out 'milestone'.
You are in 'detached HEAD' state. 
HEAD is now at eecc5fe milestone

$ cat .git/refs/tags/milestone
eecc5fe060e5b86957f931fd931beae4f206d4eb

$ cat .git/HEAD
eecc5fe060e5b86957f931fd931beae4f206d4eb
```  
checkout tag milestone后，HEAD指向eecc5f, detached HEAD







