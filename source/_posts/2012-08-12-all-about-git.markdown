---
layout: post
title: "Git使用大全"
date: 2012-08-12 00:38
comments: true
categories: Server 
tags: [git]
---

虽然关于Git的使用有很多文章，但是没有一篇是全面而且解释清楚的，以致我还只是通晓几个Git的命令而已。有人觉得Git博大精深，其实不然，其实Git是简单的。就好比你吃一个苹果，你可以横着吃，可以竖着吃，你吃的方式可以有无限种，但没有人会去研究怎么吃苹果吧。只要你了解了本质，就会发现条条大路通罗马。

在这一篇文章里，你会学习到Git的用法。但我不会对Git的实现做详细的讲解，某些底层跟使用比较紧密的我会简单说明一下。



为了实验，我先在我的Github上新建一个名为`test_on_github`的仓库，不添加README和.gitignore，纯绿色的哦。

下面开始试验。

<!--more-->

# Local本地

这章的题目叫`Local本地`，是因为这一章的操作你都不需要连接互联网就能完成。这也是分布式版本管理的一个特点，每个人都是一个`完整`的版本（相对自己而言），你可以在你的本机上检出，提交，删除...

## 初始化
### 设置全局变量

名字和Email会作为Github显示的依据。你不设置，没问题，只是Github的提交里显示不出你的用户名而已。

```
git config --global user.name "DY.Feng"
git config --global user.email "yyfeng88625"
git config --global color.ui "always"
```

### 初始化新版本库

初始化，只会在根目录下创建一个`.git`的文件夹。

```
git init
```

初始化后的目录结构
	
```
$ tree .git/
.git/
├── branches
├── config
├── description
├── HEAD
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── prepare-commit-msg.sample
│   ├── pre-rebase.sample
│   └── update.sample
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
```
### 设置忽略文件

- 新建`.gitignore`文件并提交。
- 修改`.git/info/exclude`文件，支持正则表达式，例如`*.[oa]`等价于 `*.o` 和 `*.a`。这个方法只适用于你个人。


## 日常操作

### 添加文件到缓存区

- 添加个别文件。`git add somefile1 somefile2`
- 添加本目录所有txt文件。`git add *.txt`
- `递归`添加所有文件，不包括空目录。`git add .`

### 从缓存区删除文件

用法跟rm一样，`git rm --cached`。

- 删除单个文件。`git rm --cached somefile`
- 递归删除。`git rm --cached -r .`

### 查看当前缓存区的情况

当前缓存区的情况是`注释`掉的，这也正常嘛，还没正式提交。`git status`是查看当前缓存区（index）和HEAD之间的差异。
	
```
$ git status
```

例如下面的例子就是说明:当前缓存区比HEAD要多一个文件`c.txt`
	
```
    $ git status 
    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #       new file:   c.txt
    #
```

### 从HEAD签出文件

`签出`这个词在用惯svn的童鞋眼中肯定很熟悉，可你得注意，这里的签出是指从`本地仓库`签出而已，不是签出远程的服务器。

从HEAD签出文件意味着把HEAD时的状态的文件复制出来，再次强调`HEAD`指的是最后一次提交（git commit）的状态。如果当前文件已经加入到了`缓存区`，这个动作也会把他从缓存区删除。但*不建议*把这个作为`撤销加入缓存区`的方法，因为签出会把文件也变回最后一次提交的状态，如果你已经修改过文件，那你的修改就全无了。如果你要撤销文件加入缓存区，请用`git rm --cached`。

- 签出所有文件。`git checkout HEAD .`
- 签出所有txt文件。`git checkout HEAD *.txt`
- 签出个别文件。`git checkout HEAD somefile1 somefile2`

在这里面，我们不一定要用`HEAD`，就像我一开始说的，其实HEAD只是代表了一个32位的sha标记而已，指向最新的那次修改。`git log`就能看到本地所有的提交，每次提交都有一个32位的标记，我们的HEAD就是指向最新的那个标记。既然我们知道了原理，那我们就可以在任意一次`commit`里签出我们想要的文件。
	
```
$ git log #找到我们要签出的那次`commit`标记是多少
commit 341c1e6921b61468ffd38ac423d7cf1120d7b086
Author: DY.Feng <yyfeng88625@gmail.com>
Date:   Mon Aug 13 05:22:11 2012 +0800

    mod b to 1

commit 0b5f275b20e40b51cca7bd64b2d8e50693c6660b
Author: DY.Feng <yyfeng88625@gmail.com>
Date:   Mon Aug 13 05:21:24 2012 +0800

    add b,mod a to 1

commit f0b121cf65b11d6b58b30531eb2415ddf0cbe451
Author: DY.Feng <yyfeng88625@gmail.com>
Date:   Mon Aug 13 04:56:14 2012 +0800

    rm -

commit 5f1d4283f52a825ca706b8dd039afc0dee3aa1b2
Author: DY.Feng <yyfeng88625@gmail.com>
Date:   Mon Aug 13 04:23:06 2012 +0800

    first

$ git checkout 0b5f275b20e40b51cca7bd64b2d8e50693c6660b b.txt #签出倒数第二次提交时，我们的b.txt
```

### 提交修改到本地仓库

- 提交之前`git add`过的。`git commit -m "提交信息"`
- 提交所有修改，省略`git add`步骤。`git commit -m "提交信息" -a`
- 提交单个文件，注意此文件必须是`git add`之后的。`git commit -m "提交信息" somefile`
- 增补提交，复用HEAD留言（修改小错误，而不增加提交记录，掩盖自己的小马虎）。`git commit -C HEAD -a --amend`

*注意：*修改过的文件在`git commit`之前也必须`git add`一次。

### 撤销本地仓库的修改

其实准确的来说不是撤销修改，而是把世界恢复到某个提交的时间点。




# Remote远程

简单的说，这一章你需要连接网络才能完成。

## 初始化

### 克隆版本库

克隆版本库这个是我们最常用`获取代码`的方法。
	
```
git clone git@github.com:DYFeng/test_on_github.git
```

克隆后你会发现Github已经帮你填写好几样东西：远程`origin`配置和分支`master`配置。origin其实是一个`别名`，远程仓库remote的别名。而分支配置`[branch "master"]`里则写明了：我的远程端是`origin`。其实这个`origin`也可以自己改一个名字，origin这个只是大众默认名字而已。
	
```
$ git@github.com:DYFeng/test_on_github.git
$ cd test_on_github
$ cat .git/config
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
[remote "origin"]
    fetch = +refs/heads/*:refs/remotes/origin/*
    url = git@github.com:DYFeng/test_on_github.git
[branch "master"]
    remote = origin
    merge = refs/heads/master
```

### 手动添加删除远程仓库

其实就是生成上面`[remote "origin"]`小节的东西。
	
```
git remote add origin git@github.com:DYFeng/test_on_github.git
```

### 删除别名和相关分支
	
```
git remote rm origin
```

### 创建一个空本地仓库

没有远程配置也没有分支配置，而且他把`.git`里面的文件都新建到外面了
	
```
git init --bare
$ cat config 
[core]
    repositoryformatversion = 0
    filemode = true
    bare = true
```

# 相关术语以及本文翻译

个人英语水平不高，很多名词我是按照自己的理解来翻译的，所以相关术语我都会标上英文原词，免得误人子弟。下面是本文出现过的术语以及对应的英文原词。

- 本地文件/工作目录（working tree）。他持有`实际文件`。

- 缓存区（index/stage）。临时保存你的改动，例如`git add`就是保存到缓存区。缓冲区文件是`.git/index`。

- 本地仓库（repository）。仓库位于工作目录下的`.git`目录，本地仓库储存的实际文件在`.git/objects`目录。仓库里面包含了：
  - 你所有的提交（commits）
  - 定义了`HEAD`
  - 包含了所有分支（branche）
  - 包含了所有的标签（tag）
  - 你这个仓库的配置，远程服务端地址等。
  
- HEAD。

  *HEAD不是一个实际的仓库！！*所谓的HEAD，其实就像一个`指针`，指向你最近一次快照（commit）。
  
  例如`cat .git/HEAD`后得到`ref: refs/heads/master`这么一串东西，其实他的意思就是现在的`HEAD`指向`.git/refs/heads/master`文件。再跟踪下去我们就会发现`.git/refs/heads/master`里面包含了一串32位的sha码。没错，这就是真正的提交结果，也就是`git commit`之后git给回你的一个标记。这个标记跟你`git log`第一条commit的标记是一样的。
  
  什么时候这个指针会移动？
  - 你`git chekcout`的时候
  - 你`git commit`的时候
  
- 提交/快照（commit）。这个`commit`我个人认为，他做动词的时候表示`提交`，例如`git commit`，做名词的时候表示`快照`，例如`git log`时你会看到你之前很多次的提交，其实每一次提交都是一次对你工作目录的快照。

- 签出（checkout）。与`reset`的区别：


- 分支（branch）。分支这个对于了解svn的人来说应该很好理解。其实Git的分支只是快照（commit）的一个别名，或者说是一堆快照的别名。例如他定义了这一堆快照叫`branch of development`。

- 标签（tag）。标签这个就是一个快照（commit）的别名。跟`branch`相似，只不过他只能定义一个快照，而且他有自己的概要描述。

- master。master就是主分支，只是一个约定俗成的名字而已，没有什么特殊。

- 对象/基础对象。
  在Git系统里面有四种基础对象类型，分别是`blob`，`tree`，`commit（快照）`，`tag`。几乎所有的Git都建立在管理操纵这些简单的结构之上，即它是建立在机器文件系统之上的一种自己的小型文件系统。

- blob。
  blob对象只包含了这个文件的文件名和内容。



# 参考文献

1. 不错的一个[Git常用命令思维导图](http://blog.csdn.net/liuysheng/article/details/7191846)，本文就是基于这个脉络来写的。
2. 关于Git的方方面面（英文）。[原文地址](http://newartisans.com/2008/04/git-from-the-bottom-up/) [PDF下載](http://ftp.newartisans.com/pub/git.from.bottom.up.pdf)

