---
layout: post
title: Git Submodule
date: 2018-08-28 11:12:13
comments: true
tags:
- Git
- Submodule
categories:
- Develop
- Tools
---

# Git Submodule

> **运行环境**
> 
> Git 2.18.0.windows.1 **Running on** Win7x64
> 
> **参考文章**
> 
> [咖啡兔 : Git Submodule使用完整教程](http://www.kafeitu.me/git/2012/03/27/git-submodule.html)
> 
> **参考资料**
> 
> [stackoverflow : How to read the mode field of git-ls-tree's output](https://stackoverflow.com/questions/737673/how-to-read-the-mode-field-of-git-ls-trees-output)
> 
> [Curious git : How do git submodules work?](https://matthew-brett.github.io/curious-git/git_submodules.html)
> 
> [Curious git : Types of git objects](https://matthew-brett.github.io/curious-git/git_object_types.html)
> 
> [Curious git : Curious git](https://matthew-brett.github.io/curious-git/curious_git.html)
> 

**Git Submodule功能刚刚开始学习可能觉得有点怪异，本文将把命令行每一步操作的命令和结果都展现出来，以便更好的理解。**

<!-- more -->

## 1.对于共用资源的处理方式（为什么要使用Git Submodule？）

每个公司的系统都会有一套统一的系统风格，或者针对某一个大客户的多个系统风格保持统一，而且如果风格改动后要同步到多个系统中；这样的需求几乎每个开发人员都遇到，下面看看各个层次的程序员怎么处理：

假如对于系统的风格需要几个目录：css、images、js。

- 普通程序员，把最新版本的代码逐个复制到每个项目中，如果有N个项目，那就是要复制N x 3次；如果漏掉了某个文件夹没有复制…@（&#@#。

- 文艺程序员，使用Git Submodule功能，执行：git submodule update，然后冲一杯咖啡悠哉的享受着。

---

引用一段《Git权威指南》的话： 项目的版本库在某些情况虾需要引用其他版本库中的文件，例如公司积累了一套常用的函数库，被多个项目调用，显然这个函数库的代码不能直接放到某个项目的代码中，而是要独立为一个代码库，那么其他项目要调用公共函数库该如何处理呢？分别把公共函数库的文件拷贝到各自的项目中会造成冗余，丢弃了公共函数库的维护历史，这显然不是好的方法。

---

## 2.如何使用Git Submodule

> 本例通过模拟**一个项目**及**两个公共类库**来学习。
> 

### 2.1 准备工作

#### 2.1.1 建立三个版本库

> 可以在本地也可以在远程的“远程仓库”（lib1.git，lib2.git，pr.git）
> 本文就建立在本地了...
> 

1. 建立目录结构
```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ mkdir remote

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ cd remote/

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/remote
$ mkdir lib1.git

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/remote
$ mkdir lib2.git

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/remote
$ mkdir pr.git

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/remote
$ ls -a
./  ../  lib1.git/  lib2.git/  pr.git/
```

2. 初始化版本库
```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/remote
$ cd ./lib1.git/

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/remote/lib1.git
$ git --bare init
Initialized empty Git repository in D:/Work/Git/TestSubmodule/remote/lib1.git/

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/remote/lib1.git (BARE:master)
$ cd ../lib2.git/

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/remote/lib2.git
$ git --bare init
Initialized empty Git repository in D:/Work/Git/TestSubmodule/remote/lib2.git/

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/remote/lib2.git (BARE:master)
$ cd ../pr.git/

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/remote/pr.git
$ git --bare init
Initialized empty Git repository in D:/Work/Git/TestSubmodule/remote/pr.git
```

#### 2.1.2 建立lib1，lib2，pr本地仓库

> 此处通过克隆版本库快速建立本地仓库与版本库的联系
> 

1. 建立目录结构
```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/remote/pr.git (BARE:master)
$ cd ../../

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ mkdir -p libs/lib1

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ mkdir -p libs/lib2

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ mkdir pr

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ ls -a
./  ../  libs/  pr/  remote/

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ ls -a ./libs
./  ../  lib1/  lib2/
```

2. （通过克隆）建立本地仓库
```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ git clone /d/Work/Git/TestSubmodule/remote/lib1.git ./libs/lib1/
Cloning into './libs/lib1'...
warning: You appear to have cloned an empty repository.
done.

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ git clone /d/Work/Git/TestSubmodule/remote/lib2.git ./libs/lib2/
Cloning into './libs/lib2'...
warning: You appear to have cloned an empty repository.
done.

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ git clone /d/Work/Git/TestSubmodule/remote/pr.git ./pr
Cloning into './pr'...
warning: You appear to have cloned an empty repository.
done.
```

3. 为三个仓库都添加一点东西并**提交并推送**
```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ cd ./libs/lib1

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib1 (master)
$ git config user.name "lib1"

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib1 (master)
$ git config user.email  "lib1@."

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib1 (master)
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	l1.md

nothing added to commit but untracked files present (use "git add" to track)

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib1 (master)
$ git add .

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib1 (master)
$ git commit -am "init lib1 v0.1"
[master (root-commit) e32323b] init lib1 v0.1
 1 file changed, 2 insertions(+)
 create mode 100644 l1.md
 
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib1 (master)
$ git push
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 203 bytes | 203.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To D:/Work/Git/TestSubmodule/remote/lib1.git
 * [new branch]      master -> master
```

> lib2 添加文件 l2.md
> pr 添加文件 p.md
> 步骤同上，略
> 

### 2.2 在项目中添加Submodules

#### 2.2.1 添加Submodules
1. 添加lib1到指定位置
```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git submodule add /d/Work/Git/TestSubmodule/remote/lib1.git ./libs/lib1
# 从该行可以看到lib1被clone到了指定的libs/lib1目录下
Cloning into 'D:/Work/Git/TestSubmodule/pr/libs/lib1'...
done.
warning: LF will be replaced by CRLF in .gitmodules.
The file will have its original line endings in your working directory.
```

2. 添加lib2但不指定位置
```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git submodule add /d/Work/Git/TestSubmodule/remote/lib2.git/
# 从该行可以看到lib2别clone到了默认的当前目录下的lib2中
Cloning into 'D:/Work/Git/TestSubmodule/pr/lib2'...
done.
warning: LF will be replaced by CRLF in .gitmodules.
The file will have its original line endings in your working directory.
```

#### 2.2.2 观察变化
```bash
# 可以看到目录下增加了
# 记录模块信息的.gitmodules文件
# 存放lib2文件的lib2目录
# 指定存放lib1文件的目录的上级目录libs
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ ls -a
./  ../  .git/  .gitmodules  lib2/  libs/  p.md

# 查看.gitmodules内容
# 可以看到该文件记录了模块的位置和来源信息
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ cat .gitmodules
[submodule "libs/lib1"]
	path = libs/lib1
	url = D:/Work/Git/TestSubmodule/remote/lib1.git
[submodule "lib2"]
	path = lib2
	url = D:/Work/Git/TestSubmodule/remote/lib2.git/

# 查看项目/.git/config文件
# 可以看到模块信息已经注册到了config中
# （与网络上某些教程所描述的
# git submodule add 不会注册到config中，需要再 git submodule init
# 不符，可能是git版本差异）
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ cat ./.git/config
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
[remote "origin"]
	url = D:/Work/Git/TestSubmodule/remote/pr.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
[user]
	name = pr
	email = pr@.
[submodule "libs/lib1"]
	url = D:/Work/Git/TestSubmodule/remote/lib1.git
	active = true
[submodule "lib2"]
	url = D:/Work/Git/TestSubmodule/remote/lib2.git/
	active = true

# 进入并查看lib1目录
# 可以看到lib1的l1.md文件
# 以及一个.git文件
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ cd libs/lib1/

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr/libs/lib1 (master)
$ ls -a
./  ../  .git  l1.md

# 查看.git文件可以发现这是一个指向其他目录的“链接”
# 在Linux上可以通过它访问它指向的目录
# 但在Windows上就体现为了一个文件
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr/libs/lib1 (master)
$ cat .git
gitdir: ../../.git/modules/libs/lib1

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr/libs/lib1 (master)
$ cd .git
bash: cd: .git: Not a directory

# 进入这个“链接”指向的目录
# 可以发现这个目录是存放在当前项目的版本库中的lib1的版本库
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr/libs/lib1 (master)
$ cd ../../.git/modules/libs/lib1

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr/.git/modules/libs/lib1 (master)
$ ls -a
./   config       HEAD    index  logs/     packed-refs
../  description  hooks/  info/  objects/  refs/

# 查看其HEAD指针
# 指向了master
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr/.git/modules/libs/lib1 (master)
$ cat HEAD
ref: refs/heads/master

# 查看master
# 指向的是第一次提交（也就是当前最后一次提交）
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr/.git/modules/libs/lib1 (master)
$ cat refs/heads/master
e32323bc96b2e9753ba740e0efbed4a4fe2c0fc8

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr/.git/modules/libs/lib1 (master)
$ git log
commit e32323bc96b2e9753ba740e0efbed4a4fe2c0fc8 (HEAD -> master, origin/master, origin/HEAD)
Author: lib1 <lib1@>
Date:   Fri Aug 17 21:59:54 2018 +0800

    init lib1 v0.1

# 返回项目目录
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr/.git/modules/libs/lib1 (master)
$ cd /d/Work/Git/TestSubmodule/pr

# #####
# 查看项目状态及改动
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   .gitmodules
	new file:   lib2
	new file:   libs/lib1


MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git diff

# 请留意此处的变化
# 第一个变化是.gitmodules文件，文件模式为100644，表明一个普通文件，后面跟随着文本改动。
# 第二个与第三个是目录（并非普通文件），文件模式为160000，后面也记录着一个改动，内容是是模块当前版本的commit id（一个hash值）。
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git diff --cached
diff --git a/.gitmodules b/.gitmodules
new file mode 100644
index 0000000..7896297
--- /dev/null
+++ b/.gitmodules
@@ -0,0 +1,6 @@
+[submodule "libs/lib1"]
+       path = libs/lib1
+       url = D:/Work/Git/TestSubmodule/remote/lib1.git
+[submodule "lib2"]
+       path = lib2
+       url = D:/Work/Git/TestSubmodule/remote/lib2.git/
diff --git a/lib2 b/lib2
new file mode 160000
index 0000000..a2b0f7a
--- /dev/null
+++ b/lib2
@@ -0,0 +1 @@
+Subproject commit a2b0f7ac5f0f2cfe8f57dd24e7b6ab27922d6d7d
diff --git a/libs/lib1 b/libs/lib1
new file mode 160000
index 0000000..e32323b
--- /dev/null
+++ b/libs/lib1
@@ -0,0 +1 @@
+Subproject commit e32323bc96b2e9753ba740e0efbed4a4fe2c0fc8

```

#### 2.2.3 commit & push
```bash
# 注意 lib2 与 libs/lib1 记录的 160000 模式。
# 这是 Git 中的一种特殊模式
# 它本质上将一个提交记作一项目录记录的，而非一个子目录或者一个文件。
# 提交成功即确定下项目这一版本所添加的Submodules的版本。
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git commit -am "import: lib1 to ./libs/lib1, lib2 to ./lib2 (default path)"
[master 42a346a] import: lib1 to ./libs/lib1, lib2 to ./lib2 (default path)
 3 files changed, 8 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 lib2
 create mode 160000 libs/lib1

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 471 bytes | 471.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To D:/Work/Git/TestSubmodule/remote/pr.git
   48f1fd8..42a346a  master -> master

```

### 2.3 Clone带有Submodules的项目（仓库）

#### 2.3.1 git clone

- 在另一个位置克隆项目pr
```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ cd ..

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ git clone /d/Work/Git/TestSubmodule/remote/pr.git ./pr-clone
Cloning into './pr-clone'...
done.

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ cd pr-clone/

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ ls -a
./  ../  .git/  .gitmodules  lib2/  libs/  p.md

```

#### 2.3.2 查看Submodules状态

1. 查看.gitmodules
```bash
# 由于该文件受版本控制，因此里面的模块信息被一并克隆了下来
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ cat .gitmodules
[submodule "libs/lib1"]
	path = libs/lib1
	url = D:/Work/Git/TestSubmodule/remote/lib1.git
[submodule "lib2"]
	path = lib2
	url = D:/Work/Git/TestSubmodule/remote/lib2.git/

```

2. 查看.git目录
```bash
# 可以看到克隆得到的仓库的.git/config中没有模块的注册信息
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ cat .git/config
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
[remote "origin"]
	url = D:/Work/Git/TestSubmodule/remote/pr.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master

# .git中没有存放模块版本控制的modules目录
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ ls -a ./.git
./   config       HEAD    index  logs/     packed-refs
../  description  hooks/  info/  objects/  refs/

```

3. 查看模块目录下有什么
```bash
# 一片空白...
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ ls -a ./libs/lib1/
./  ../

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ ls -a ./lib2
./  ../

```

4. 使用git submodule查看
```bash
# 可以看到模块的hash码和路径
# 最前面的‘-’表示该模块还未检出
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ git submodule
-a2b0f7ac5f0f2cfe8f57dd24e7b6ab27922d6d7d lib2
-e32323bc96b2e9753ba740e0efbed4a4fe2c0fc8 libs/lib1
```

#### 2.3.2 注册并检出模块

1. 在当前克隆得到的仓库中注册模块

> git submodule init 用来初始化本地配置文件
> 
> 实际上是修改了.git/config文件，在该文件内注册了submodule库
> 

```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ git submodule init
Submodule 'lib2' (D:/Work/Git/TestSubmodule/remote/lib2.git/) registered for path 'lib2'
Submodule 'libs/lib1' (D:/Work/Git/TestSubmodule/remote/lib1.git) registered for path 'libs/lib1'

# 注册后.git仍无modules目录
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ ls -a ./.git
./   config       HEAD    index  logs/     packed-refs
../  description  hooks/  info/  objects/  refs/

# config中有了模块的信息
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ cat ./.git/config
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
[remote "origin"]
	url = D:/Work/Git/TestSubmodule/remote/pr.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
[submodule "lib2"]
	active = true
	url = D:/Work/Git/TestSubmodule/remote/lib2.git/
[submodule "libs/lib1"]
	active = true
	url = D:/Work/Git/TestSubmodule/remote/lib1.git

```

2. 检出模块

> git submodule update
> 
> 获取该模块版本库，检查当前模块版本是否与父项目当前版本的索引中记录的该模块的版本一致，一致则不做操作；不一致则检出目标commit id，HEAD将直接指向该commit id。
> 

```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ git submodule update
Cloning into 'D:/Work/Git/TestSubmodule/pr-clone/lib2'...
done.
Cloning into 'D:/Work/Git/TestSubmodule/pr-clone/libs/lib1'...
done.
Submodule path 'lib2': checked out 'a2b0f7ac5f0f2cfe8f57dd24e7b6ab27922d6d7d'
Submodule path 'libs/lib1': checked out 'e32323bc96b2e9753ba740e0efbed4a4fe2c0fc8'

# update后.git出现了模块的版本库文件夹，模块目录下也有了对应文件
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ ls -a ./.git
./   config       HEAD    index  logs/     objects/     refs/
../  description  hooks/  info/  modules/  packed-refs

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ ls -a libs/lib1/
./  ../  .git  l1.md

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ ls -a lib2
./  ../  .git  l2.md

```

- 上面的两个命令(git submodule init & update)其实可以简化
- 如果给 git clone 命令传递 --recursive 选项，它就会自动初始化并更新仓库中的每一个子模块。

### 2.4 修改Submodule

一个项目可以在自己的仓库中修改提交了；作为模块添加到其他项目中后也可以直接在其他项目中修改提交。

#### 2.4.1 在引用了该模块的项目中修改（lib1为例）

> 在上一节中克隆得到的项目中修改
> 
> 最后会将项目对模块的引用一并更新
> 

1. 首先进入模块目录，就可以查看模块版本状态（相当于进入了另一个仓库）
```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ cd ./libs/lib1/

# 可以看出该lib1处于头指针分离状态，处在原项目提交时该模块的版本上
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone/libs/lib1 ((e32323b...))
$ git status
HEAD detached at e32323b
nothing to commit, working tree clean

# 查看HEAD，直接指向原项目提交时该模块的commit id
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone/libs/lib1 ((e32323b...))
$ cat .git
gitdir: ../../.git/modules/libs/lib1

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone/libs/lib1 ((e32323b...))
$ cat ../../.git/modules/libs/lib1/HEAD
e32323bc96b2e9753ba740e0efbed4a4fe2c0fc8

```

2. 现在要修改lib1就需要先切换到master分支, 修改并提交（先不push）

```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone/libs/lib1 ((e32323b...))
$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone/libs/lib1 (master)
$ git diff
diff --git a/l1.md b/l1.md
index ebf6fdc..8bad4d8 100644
--- a/l1.md
+++ b/l1.md
@@ -1,2 +1,3 @@
 # lib1
 v 0.1
+v 0.2

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone/libs/lib1 (master)
$ git commit -am "IN pr-clone: update lib1 to 0.2"
[master ec3c940] IN pr-clone: update lib1 to 0.2
 1 file changed, 1 insertion(+)

```

3. 在push前先看一下pr-clone的状态

```bash
# 当前项目中的模块 libs/lib1 发生变化（即该模块有提交产生了新版本）
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone/libs/lib1 (master)
$ cd ../..

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   libs/lib1 (new commits)

no changes added to commit (use "git add" and/or "git commit -a")

# 差异中可以看出因为模块的版本发生变化，项目对该模块的引用的版本的记录也要进行更新
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ git diff
diff --git a/libs/lib1 b/libs/lib1
index e32323b..ec3c940 160000
--- a/libs/lib1
+++ b/libs/lib1
@@ -1 +1 @@
-Subproject commit e32323bc96b2e9753ba740e0efbed4a4fe2c0fc8
+Subproject commit ec3c940c4ab8406b08d0513f02955ca07bb995ef

```

**注意：如果现在执行git submodule update操作会让 libs/lib1 还原到之前的commit id上，但是刚刚的修改已经提交到了master分支上，因此不会丢失，只要再将 libs/lib1 master分支检出即可回到修改完成的版本上**
```bash
# 执行git submodule update使 lib1 回到了记录的 commit id 对应的版本上
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ git submodule update
Submodule path 'libs/lib1': checked out 'e32323bc96b2e9753ba740e0efbed4a4fe2c0fc8'

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean

# 检出 libs/lib1 master 分支来到修改后的版本上并push
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ cd libs/lib1/

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone/libs/lib1 ((e32323b...))
$ git checkout master
Previous HEAD position was e32323b init lib1 v0.1
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone/libs/lib1 (master)
$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Writing objects: 100% (3/3), 260 bytes | 260.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To D:/Work/Git/TestSubmodule/remote/lib1.git
   e32323b..ec3c940  master -> master

```

4. 将使用了新版lib1的pr-clone提交并push

这时查看pr-clone的状态
```bash
# 当前项目中的模块 libs/lib1 发生变化（提交产生了新版本）
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   libs/lib1 (new commits)

no changes added to commit (use "git add" and/or "git commit -a")

# 项目对该模块的引用的版本的记录需要更新
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ git diff
diff --git a/libs/lib1 b/libs/lib1
index e32323b..ec3c940 160000
--- a/libs/lib1
+++ b/libs/lib1
@@ -1 +1 @@
-Subproject commit e32323bc96b2e9753ba740e0efbed4a4fe2c0fc8
+Subproject commit ec3c940c4ab8406b08d0513f02955ca07bb995ef

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ git commit -am "follow lib1 v0.1 to v0.2 update"
[master 331116a] follow lib1 v0.1 to v0.2 update
 1 file changed, 1 insertion(+), 1 deletion(-)

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 362 bytes | 362.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To D:/Work/Git/TestSubmodule/remote/pr.git
   42a346a..331116a  master -> master
```

#### 2.4.2 在模块（项目）自己的仓库中修改（lib2为例）

> 相对于上一种修改方式，这种更为简单清晰。
> 

修改提交push完成！
```bash
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr-clone (master)
$ cd ../libs/lib2

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib2 (master)
$ git log
commit a2b0f7ac5f0f2cfe8f57dd24e7b6ab27922d6d7d (HEAD -> master, origin/master)
Author: lib2 <lib2@>
Date:   Fri Aug 17 22:05:13 2018 +0800

    init lib2 v0.1

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib2 (master)
$ git diff

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib2 (master)
$ git diff
diff --git a/l2.md b/l2.md
index 5c34c87..bceab83 100644
--- a/l2.md
+++ b/l2.md
@@ -1,2 +1,3 @@
 # lib2
 v 0.1
+v 0.2

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib2 (master)
$ git commit -am "IN selfRepo update lib2 to 0.2"
[master 772a158] IN selfRepo update lib2 to 0.2
 1 file changed, 1 insertion(+)

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib2 (master)
$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Writing objects: 100% (3/3), 253 bytes | 253.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To D:/Work/Git/TestSubmodule/remote/lib2.git
   a2b0f7a..772a158  master -> master
```

### 2.5 应用Submodule的修改

上一节列举了两种修改Submodule的方式，修改完成并推送后，要在以前引用了它们项目中应用这些更新。

- 模块版本状况的判断依据来源于索引中指定模块版本的 commit id。

此处有两种情况：
1. 项目当前版本索引中指定的模块版本已被更新：上一节中模块 lib1 更新后的版本直接被项目 pr 的克隆项目 pr-clone 引用并提交并push，那么项目 pr 拉取最新的版本后，索引中指定的模块版本已为新版，但项目中模块的文件为旧版，这时只需要在 .git/config 中已注册模块的情况下（若未注册需要执行git submodule init将模块信息注册到 .git/config 中）执行git submodule update即可将模块文件更新到指定版本了。

2. 项目当前版本索引中指定的模块版本未被更新：上一节中模块 lib2 在自己的仓库中更新后的版本后没有被其他项目引用，项目 pr 与 pr-clone 中当前版本索引中指定的模块 lib2 的版本仍为旧版，以 pr 为例，要更新 lib2 到最新版，需要进入 pr 项目下模块 lib2 的目录里，将分支切换到所需模块版本所在的分支并拉取更新（此处为master并拉取），检出需要的模块版本（此处为master最新版），返回项目目录中提交对索引中记录的模块版本的更新。

#### 2.5.1 索引中commit id已更新

```bash
# 拉取项目的更新
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git pull
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (3/3), done.
From D:/Work/Git/TestSubmodule/remote/pr
   42a346a..331116a  master     -> origin/master
Fetching submodule libs/lib1
From D:/Work/Git/TestSubmodule/remote/lib1
   e32323b..ec3c940  master     -> origin/master
Updating 42a346a..331116a
Fast-forward
 libs/lib1 | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

# 查看项目状态
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   libs/lib1 (new commits)

no changes added to commit (use "git add" and/or "git commit -a")

# 查看差异
# libs/lib1 第一版为 e3232 第二版为 ec3c9
# 此时索引中指定的为新版 ec3c9 而仓库模块文件还是旧版 e3232
# 可以看出 git 认为的新版本其实是没有来得及更新的旧版
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git diff
diff --git a/libs/lib1 b/libs/lib1
index ec3c940..e32323b 160000
--- a/libs/lib1
+++ b/libs/lib1
@@ -1 +1 @@
-Subproject commit ec3c940c4ab8406b08d0513f02955ca07bb995ef
+Subproject commit e32323bc96b2e9753ba740e0efbed4a4fe2c0fc8

# update 将索引中指定的新版的 libs/lib1 模块文件检出
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git submodule update
Submodule path 'libs/lib1': checked out 'ec3c940c4ab8406b08d0513f02955ca07bb995ef'

# 再看状态已经是干净的了
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

#### 2.5.2 索引中commit id未更新

```bash
# 由于索引中指定的 commit id 与旧版模块已经对应，因此状态是干净的
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean

# 进入模块 lib2 目录下并切换到需要的分支
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ cd lib2

# 因为是初始导入所以分支在master上，但更多的情况下模块处于头指针分离的状态
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr/lib2 (master)
$ git checkout master
Already on 'master'
Your branch is up to date with 'origin/master'.

# 拉取更新
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr/lib2 (master)
$ git pull
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (3/3), done.
From D:/Work/Git/TestSubmodule/remote/lib2
   a2b0f7a..772a158  master     -> origin/master
Updating a2b0f7a..772a158
Fast-forward
 l2.md | 1 +
 1 file changed, 1 insertion(+)

# 检出需要的版本
# 用最新版的情况下不需要这一步，拉取后即为最新...

# 查看状态：git 发现模块 lib2 版本发生了变化
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   lib2 (new commits)

no changes added to commit (use "git add" and/or "git commit -a")

# 查看差异：索引中的旧版 a2b0f 要更新到新版 772a1
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git diff
diff --git a/lib2 b/lib2
index a2b0f7a..772a158 160000
--- a/lib2
+++ b/lib2
@@ -1 +1 @@
-Subproject commit a2b0f7ac5f0f2cfe8f57dd24e7b6ab27922d6d7d
+Subproject commit 772a158c031f55febc383196a5c02eb58c0b13ed

# 因为索引中指定的模块版本发生了变化，需要提交
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git commit -am "follow lib2 v0.1 to v0.2 update"
[master b5de982] follow lib2 v0.1 to v0.2 update
 1 file changed, 1 insertion(+), 1 deletion(-)

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git push
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 315 bytes | 315.00 KiB/s, done.
Total 2 (delta 0), reused 0 (delta 0)
To D:/Work/Git/TestSubmodule/remote/pr.git
   331116a..b5de982  master -> master
```

## 3.涉及到的知识点

### Git 中的文件模式

From the Git [index-format.txt](https://github.com/git/git/blob/master/Documentation/technical/index-format.txt) file, regarding the mode:

```
32-bit mode, split into (high to low bits)

    4-bit object type
      valid values in binary are 1000 (regular file), 1010 (symbolic link)
      and 1110 (gitlink)

    3-bit unused

    9-bit unix permission. Only 0755 and 0644 are valid for regular files.
    Symbolic links and gitlinks have value 0 in this field.
```

Also, a directory object type (binary 0100) and group-writeable (0664 permissions) regular file are allowed as indicated by the fsck.c fsck_tree method. The regular non-executable group-writeable file is a non-standard mode that was supported in earlier versions of Git.
> 
> 此外，允许目录对象类型（二进制0100）和组可写（0664权限）常规文件，如fsck.c fsck_tree方法所示。 常规的非可执行组可写文件是早期版本的Git支持的非标准模式。
>

This makes valid modes (as binary and octal):
> 
> 这使得有效模式（二进制和八进制）：
>

0100000000000000 (040000): Directory

1000000110100100 (100644): Regular non-executable file

1000000110110100 (100664): Regular non-executable group-writeable file

1000000111101101 (100755): Regular executable file

1010000000000000 (120000): Symbolic link

1110000000000000 (160000): Gitlink
> 
> 0100000000000000（040000）：目录
> 
> 1000000110100100（100644）：常规非可执行文件
> 
> 1000000110110100（100664）：常规非可执行组可写文件
> 
> 1000000111101101（100755）：常规可执行文件
> 
> 1010000000000000（120000）：符号链接
> 
> 1110000000000000（160000）：Gitlink
>

- 这里的文件模式参考了常见的 UNIX 文件模式，但远没那么灵活。

### 查看gitlink

> 以 2.x 中的几个仓库为例，因为在 pr 引入 Submodule 后的提交中开始存在 gitlink ，而 pr 在最后一个版本相对前一版本更新了 lib2 的版本，所以此处选择 pr 最后两个版本进行对比查看。
> 

#### lib1 lib2 pr 的 git log

```bash
# lib1 的 git log
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule
$ cd libs/lib1

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib1 (master)
$ git log
commit ec3c940c4ab8406b08d0513f02955ca07bb995ef (HEAD -> master, origin/master)
Author: pr-clone/libs/lib1 <pr-clone/libs/lib1@>
Date:   Sat Aug 18 16:17:49 2018 +0800

    IN pr-clone: update lib1 to 0.2

commit e32323bc96b2e9753ba740e0efbed4a4fe2c0fc8
Author: lib1 <lib1@>
Date:   Fri Aug 17 21:59:54 2018 +0800

    init lib1 v0.1

# lib2 的 git log
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib1 (master)
$ cd ../lib2

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib2 (master)
$ git log
commit 772a158c031f55febc383196a5c02eb58c0b13ed (HEAD -> master, origin/master)
Author: lib2 <lib2@>
Date:   Sat Aug 18 18:43:26 2018 +0800

    IN selfRepo update lib2 to 0.2

commit a2b0f7ac5f0f2cfe8f57dd24e7b6ab27922d6d7d
Author: lib2 <lib2@>
Date:   Fri Aug 17 22:05:13 2018 +0800

    init lib2 v0.1

# pr 的 git log
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/libs/lib2 (master)
$ cd ../../pr

MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git log
commit b5de982213428040683bd765cda5dae5303a6a24 (HEAD -> master, origin/master)
Author: pr <pr@>
Date:   Sat Aug 18 19:30:54 2018 +0800

    follow lib2 v0.1 to v0.2 update

commit 331116ad97463cf9c7e69a2112cf19e1bbda1976
Author: pr-clone <pr-clone@>
Date:   Sat Aug 18 18:32:56 2018 +0800

    follow lib1 v0.1 to v0.2 update

commit 42a346aad4e27743984d40e9cc9f1dcf6606ee40
Author: pr <pr@>
Date:   Sat Aug 18 14:59:01 2018 +0800

    import: lib1 to ./libs/lib1, lib2 to ./lib2 (default path)

commit 48f1fd848327868b056e5088e975cafad13e46b9
Author: pr <pr@>
Date:   Sat Aug 18 13:46:03 2018 +0800

    init pr v0.1
```

#### pr 的倒数第二次提交

> 先看更新 lib2 之前的提交 331116ad97463cf9c7e69a2112cf19e1bbda1976
> 

```bash
# 显然这是一个commit类型的对象
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git cat-file -t 331116ad97463cf9c7e69a2112cf19e1bbda1976
commit

# 查看内容，得到此提交指向的目录树 tree 19a1795ef10408f533c194b9d799941eb59f9139
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git cat-file -p 331116ad97463cf9c7e69a2112cf19e1bbda1976
tree 19a1795ef10408f533c194b9d799941eb59f9139
parent 42a346aad4e27743984d40e9cc9f1dcf6606ee40
author pr-clone <pr-clone@> 1534588376 +0800
committer pr-clone <pr-clone@> 1534588376 +0800

follow lib1 v0.1 to v0.2 update

# 查看这棵树，可以看到 lib2 的文件模式为 160000 类型为 commit
# 对比上方 lib2 的 git log ，这串 hash码 就是 lib2 第一次提交的commit id
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git cat-file -p 19a1795ef10408f533c194b9d799941eb59f9139
100644 blob 7896297258c3b7833205c80ff7aa7843bf1a07ae    .gitmodules
160000 commit a2b0f7ac5f0f2cfe8f57dd24e7b6ab27922d6d7d  lib2
040000 tree 6c0d205680346cdd41957147913cba2ff5627f8b    libs
100644 blob eeeb88dcb36d4578b9a1d2c4fdba3b7e36428bcd    p.md

# 查看文件模式为 040000 的 libs 可以看到 lib1 像 lib2 一样
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git cat-file -p 6c0d205680346cdd41957147913cba2ff5627f8b
160000 commit ec3c940c4ab8406b08d0513f02955ca07bb995ef  lib1
```

#### pr 的最后一次提交

> 再来看 pr 更新 lib2 后的提交 b5de982213428040683bd765cda5dae5303a6a24
> 

```bash
# 直接查看它的目录树
# 对比上方 lib2 的 git log
# 可以看到文件模式为 160000 类型为 commit 的 lib2 这一行
# 中的 hash码 变为了 lib2 更新后提交的 commit id
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git ls-tree b5de982213428040683bd765cda5dae5303a6a24
100644 blob 7896297258c3b7833205c80ff7aa7843bf1a07ae    .gitmodules
160000 commit 772a158c031f55febc383196a5c02eb58c0b13ed  lib2
040000 tree 6c0d205680346cdd41957147913cba2ff5627f8b    libs
100644 blob eeeb88dcb36d4578b9a1d2c4fdba3b7e36428bcd    p.md

# 同样的命令查看 libs
# 可以看到 lib1 对应的 hash码 没有改变，因为 pr 的这次提交相对于
# 上一次提交没有改变 lib1 的版本
MiK@MiK-PC MINGW64 /d/Work/Git/TestSubmodule/pr (master)
$ git ls-tree 6c0d205680346cdd41957147913cba2ff5627f8b
160000 commit ec3c940c4ab8406b08d0513f02955ca07bb995ef  lib1
```

### 总结流程

本文中涉及到过的文件模式 160000 的解释为 gitlink ，意为一个git的链接，可以认为是指向一个commit id，在项目A中引入另一个项目B作为Submodule时，A不对引入的B的文件进行版本控制，而是在索引中用 gitlink 记录本项目下某个目录对应B的某一个版本的commit id

当一个项目使用 git submodule add 引入一个 Submodule 时，会在项目中生成记录模块信息的 .gitmodules 文件并注册到 .git/config 中，根据指定的目录clone模块版本库到 .git/modules 并检出到指定目录，该目录被索引记录为 gitlink 指向模块当前版本commit id。

当一个引入了Submodule的项目被检出时会创建每个模块在项目中的目录，并有一个被版本控制的 .gitmodules 文件记录着每个模块在当前项目的path及来源url。

执行 git submodule init 命令时，模块会被注册到 .git/config 中。

执行 git submodule update命令时，git会根据已注册模块的url来clone该模块版本库参考path存到 .git/modules 中并检出 gitlink 记录的版本到对应目录下。

## 感谢两大搜索引擎提供了强大的中英文资料支持 & Google 翻译