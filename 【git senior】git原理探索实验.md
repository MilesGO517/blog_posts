## 背景知识
### git的三个区域
- working directory
    - 也就是你当前所能操作的那些目录和文件
- history
    - 你所提交的所有记录，文件历史内容等等。git是个分布式版本管理系统，在你本地有项目的所有历史提交记录；文件历史记录；提交日志等等
- stage(index)
    - 暂存区域
    - 本质上是个文件，也就是.git/index

### git的三类对象及其关系
- commit对象
    - 用于表示一个提交
    - commit对象之间会组织成一棵树的结构
    - 每一次提交都会产生一个commit对象
    - 除了第一次提交产生的commit对象，其它的commit对象都会有父亲commit对象
    - 如果只是提交操作，则父亲节点只有1个
    - 如果是merge操作，则父亲节点会有2个 
    - commit对象包含的信息有
        - parent，父亲提交对象
        - tree, 根目录树对象
        - author，作者
        - committer，提交者
- tree对象
    - 用于表示一个目录
- blob对象
    - 用于表示一个文件
    - tree和blob对象可以看成是git内部采用的文件系统对象
    - 其组织结构和文件系统是很像的，很好理解
- 这些对象都保存在.git/objects/目录下，每一个对象都会生成1个40位的哈希值，前2位作为文件夹，后38位作为文件名
- 不同名，相同内容的文件，其哈希值相同，其blob对象也是共用的

## 实验1
```
$ git init test
Initialized empty Git repository in C:/Users/MilesGO/Desktop/git/test/.git/
$ cd test
$ find .git/objects -type f
```
此时由于没有任何的提交记录, `find .git/objects -type f`指令的返回结果为空，这条指令会列出.git/objects目录下的所有文件  
接下来在根目录下创建一个README文件，文件内容hello git
```
$ echo 'hello git' >> README
find .git/objects -type f
```
返回结果仍然为空，说明在工作区域做出的改变，如果没有提交到暂存区或历史版本库，是不会产生任何的git对象的
```
git add README
$ find .git/objects -type f
.git/objects/8d/0e41234f24b6da002d962a26c2495ea16a425f
$ git cat-file -t 8d0e4123
blob
$ git cat-file -p 8d0e4123
hello git
```
`git cat-file -t`指令可以查看一个git对象的类型，`git cat-file -p`指令可以查看一个git对象的内容  
我们将README文件添加到暂存区之后，生成了一个blob文件，对应了暂存区中的README，其内容就是我们刚刚添加的`hello git`  
接下来我们将暂存区的README提交
```
$ git commit -m 'add README'
[master (root-commit) 29c0715] add README
 1 file changed, 1 insertion(+)
 create mode 100644 README

$ find .git/objects -type f
.git/objects/28/324fe5bad6e397de7a37aa7d8c94b6bf176b83
.git/objects/29/c07152991c7b3ee41f9cb5ac1eff1f26610665
.git/objects/8d/0e41234f24b6da002d962a26c2495ea16a425f

$ git cat-file -t 28324fe5
tree

$ git cat-file -t 29c07152
commit
```
可以看到新生成了1个commit对象和一个tree对象，分别查看这两个对象的内容
```
$ git cat-file -p 29c07152
tree 28324fe5bad6e397de7a37aa7d8c94b6bf176b83
author MilesGO <164173218@qq.com> 1560070361 +0800
committer MilesGO <164173218@qq.com> 1560070361 +0800

add README
```
commit对象表明了其对应的根目录tree为28324f，也就是刚才新生成的tree对象
```
$ git cat-file -p 28324f
100644 blob 8d0e41234f24b6da002d962a26c2495ea16a425f    README
```
这个tree对象中只包含了一个blob对象8d0e41，也就是一开始我们添加的README文件

## 实验2
在实验1的基础上继续进行
在README文件中添加hello git again，commit后查看git对象情况
```
$ echo 'hello git again' >> README
$ git add README
$ git commit -m 'modify README'
[master 4a1d754] modify README
 1 file changed, 1 insertion(+)

$ find .git/objects -type f
.git/objects/28/324fe5bad6e397de7a37aa7d8c94b6bf176b83
.git/objects/29/c07152991c7b3ee41f9cb5ac1eff1f26610665
.git/objects/4a/1d75425a4cd5c3e206a4c69415c59802175039
.git/objects/8d/0e41234f24b6da002d962a26c2495ea16a425f
.git/objects/b9/641c1bd3e89e43cfaebf63305fc0a655635b50
.git/objects/f8/3422c24917dcb2c2f781c08d83ac2fbc363dd2

$ git cat-file -t 4a1d75
commit
$ git cat-file -t b9641c
tree
$ git cat-file -t f83422
blob
```
分别新增了1个commit，1个tree和1个blob对象
```
$ git cat-file -p 4a1d75
tree b9641c1bd3e89e43cfaebf63305fc0a655635b50
parent 29c07152991c7b3ee41f9cb5ac1eff1f26610665
author MilesGO <164173218@qq.com> 1560070808 +0800
committer MilesGO <164173218@qq.com> 1560070808 +0800

modify README
```
新的提交必然需要生成1个新的提交对象，其tree根目录对象为b9641c，也是新生成的，其parent对象为29c071，也就是第一次提交生成的提交对象  
```
$ 100644 blob f83422c24917dcb2c2f781c08d83ac2fbc363dd2    README
$ git cat-file -p b9641c
```
新生成的tree对象中，包含1条blob对象的信息
```
$ git cat-file -p f83422
hello git
hello git again
```
该blob对象也就是我们刚才修改后提交的README
到此为止，一共有2个commit对象，2个tree对象，2个blob对象

## 实验3
创建src目录，在src目录下添加main.cpp，添加一些内容
```
$ mkdir src
$ echo 'hello world' >> src/main.cpp
$ git add --a
$ git commit -m 'add src/main.cpp'
[master 12c2c80] add src/main.cpp
 1 file changed, 1 insertion(+)
 create mode 100644 src/main.cpp

$ find .git/objects -type f
.git/objects/12/c2c80788dcd74a1cd739157c0a1011514d36b2
.git/objects/28/324fe5bad6e397de7a37aa7d8c94b6bf176b83
.git/objects/29/c07152991c7b3ee41f9cb5ac1eff1f26610665
.git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad
.git/objects/3f/68ce3f72d2ee5bec285180c6fb0aa3a7eee4ff
.git/objects/4a/1d75425a4cd5c3e206a4c69415c59802175039
.git/objects/8d/0e41234f24b6da002d962a26c2495ea16a425f
.git/objects/b9/641c1bd3e89e43cfaebf63305fc0a655635b50
.git/objects/c5/fb7c2c039f5f48bf89f922267016e8702bead8
.git/objects/f8/3422c24917dcb2c2f781c08d83ac2fbc363dd2

```
新添加的对象有
```
12c2c8
3b18e5
3f68ce
c5fb7c
```
查看这些新添加的对象
```
$ git cat-file -t 12c2c8
commit
$ git cat-file -p 12c2c8
tree 3f68ce3f72d2ee5bec285180c6fb0aa3a7eee4ff
parent 4a1d75425a4cd5c3e206a4c69415c59802175039
author MilesGO <164173218@qq.com> 1560071508 +0800
committer MilesGO <164173218@qq.com> 1560071508 +0800

add src/main.cpp

$ git cat-file -t 3b18e5
blob
$ git cat-file -p 3b18e5
hello world

$ git cat-file -t 3f68ce
tree
$ git cat-file -p 3f68ce
100644 blob f83422c24917dcb2c2f781c08d83ac2fbc363dd2    README
040000 tree c5fb7c2c039f5f48bf89f922267016e8702bead8    src

$ git cat-file -t c5fb7c
tree
$ git cat-file -p c5fb7c
100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad    main.cpp

```

## 实验4
不同文件名，相同文件内容
```
$ echo 'hello world' >> main2.cpp
$ git add main2.cpp
$ git commit -m 'add main2.cpp'
[master 314b549] add main2.cpp
 1 file changed, 1 insertion(+)
 create mode 100644 main2.cpp
```
在这种情况下，会新增哪些git对象呢？
```
$ find .git/objects -type f
.git/objects/12/c2c80788dcd74a1cd739157c0a1011514d36b2
.git/objects/28/324fe5bad6e397de7a37aa7d8c94b6bf176b83
.git/objects/29/c07152991c7b3ee41f9cb5ac1eff1f26610665
.git/objects/31/4b5494eb06a9963f4e6f97075f2354c8a15f13
.git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad
.git/objects/3e/c767e93a78c0ef5c4685eace73192bf6bc6848
.git/objects/3f/68ce3f72d2ee5bec285180c6fb0aa3a7eee4ff
.git/objects/4a/1d75425a4cd5c3e206a4c69415c59802175039
.git/objects/8d/0e41234f24b6da002d962a26c2495ea16a425f
.git/objects/b9/641c1bd3e89e43cfaebf63305fc0a655635b50
.git/objects/c5/fb7c2c039f5f48bf89f922267016e8702bead8
.git/objects/f8/3422c24917dcb2c2f781c08d83ac2fbc363dd2
```
其中314b54和3ec767是新增的
```
$ git cat-file -t 314b54
commit
$ git cat-file -p 314b54
tree 3ec767e93a78c0ef5c4685eace73192bf6bc6848
parent 12c2c80788dcd74a1cd739157c0a1011514d36b2
author MilesGO <164173218@qq.com> 1560072040 +0800
committer MilesGO <164173218@qq.com> 1560072040 +0800

add main2.cpp

$ git cat-file -t 3ec767
tree

$ git cat-file -p 3ec767
100644 blob f83422c24917dcb2c2f781c08d83ac2fbc363dd2    README
100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad    main2.cpp
040000 tree c5fb7c2c039f5f48bf89f922267016e8702bead8    src

$ git cat-file -p c5fb7c
100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad    main.cpp
```
main2.cpp和src/main.cpp共用了同一个blob对象

`git ls-files --stage`可以看到暂存区所有文件及其对应的blob文件
```
$ git ls-files --stage
100644 f83422c24917dcb2c2f781c08d83ac2fbc363dd2 0       README
100644 3b18e512dba79e4c8300dd08aeb37f8e728b8dad 0       main2.cpp
100644 3b18e512dba79e4c8300dd08aeb37f8e728b8dad 0       src/main.cpp
```
从这里也可以看到main2.cpp和src/main.cpp共用了同一个blob对象