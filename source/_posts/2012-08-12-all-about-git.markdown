---
layout: post
title: "Git使用大全"
date: 2012-08-12 00:38
comments: true
categories: Server 
tags: [git]
---

虽然关于Git的使用有很多文章，但是没有一篇是全面而且解释清楚的，以致我还只是通晓几个Git的命令而已。有人觉得Git博大精深，其实不然，其实Git是简单的。就好比你吃一个苹果，你可以横着吃，可以竖着吃，你吃的方式可以有无限种，但没有人会去研究怎么吃苹果吧。只要你了解了本质，就会发现条条大路通罗马。



因为囊括的东西比较多，所以我这篇文章比较。对于一个作者来说，罗列知识点很容易，但你要把知识点组织起来，让人思路清晰，脉络通畅，那就是难点了。

建议阅读方法：

0. 你可以先大概浏览一下`Local本地使用`和`Remote远程相关使用`这两章，感受一下Git的大概使用，这两章是讲述Git的在各种使用场景下的命令，不必精读。如果只是想查找命令的童鞋，看这章就可以了。
1. 再看看第一章`Git基础讲解`，最基本的概念和Git的运作方式会在这里讲述。
2. 当你遇到有不明白的术语或者词汇，不妨到`相关术语以及本文翻译`那章找找。
3. 再回来`Local本地使用`和`Remote远程相关使用`这两章。相信这个时候你应该能通过这命令，而知道他们底层究竟是做了什么操作。
4. 有空的时候可以看看`小技巧`
5. 如果对Git的使用还有什么疑问，可以留言。本人也是初学Git，可以共同探讨学习。

下面开始试验。

<!--more-->

# Git基础讲解

## Git是什么？
~~Git是一个版本控制软件...~~（地球人都知道的就不废话了，省略几百字）Git就是

- 一个带有`二次元平衡世界`、`时光倒流`功能的`文件管理系统`。
- 一个`键值对`的数据库，`键`是`值`的SHA1值。`tree .git/objects`你就能看到他们了。
- 一切皆对象

为什么说他是文件管理系统呢？那跟我电脑本地的文件管理系统有什么区别？这个就得从Git的`文件模型`--基础对象模型说起了。

## Git的基础对象模型（Git Objects）
  在Git系统里面有四种基础对象类型，分别是`blob`，`tree（树）`，`commit（快照）`，`tag`。几乎所有的Git都建立在管理操纵这些简单的结构之上，即它是建立在机器文件系统之上的一种自己的小型文件系统。

### blob对象

blob对象只包含了这个文件的`内容`，而没有包含修改日期、拥有者、*文件名*、目录路径之类的信息。下面我们来做个实验：
  
```
  $ echo 'Hello,Git!' > test.txt
  $ git hash-object test.txt
  63008ae88b4446dfc43b47f18aee5b427203b255
```

你可看到他生成了一个40位的`SHA1`值，这SHA1值就是通过`文件内容`计算出来的，所以说同一个文件只有一个SHA1值。如果你在你的电脑上做同样内容的一个文件，即使文件名不一样，计算出来的结果都是一样的。
  
现在我们把`test.txt`纳入我们的Git文件管理系统：

```
$ git add test.txt
$ git commit -m "add test.txt" #这句不是必须的，git add之后就已经纳入到Git里面去了
```

提交之后，`test.txt`的内容已经放进了我们Git的blob树了，我们可以根据SHA1值来看看文件的内容：

```
$ git cat-file blob 63008ae88b4446dfc43b47f18aee5b427203b255
Hello,Git!
$ git cat-file blob 63008a  #其实用前六位或者七位就可以了
Hello,Git!
$ git cat-file t 63008a  #查看对象类型
blob
```

blob只记录文件的内容的好处：
- Git可以快速的仅仅通过对象名称决定两个对象是否相同 
- 由于对象名称在每一个库（repository）中都由相同的方式计算得到，相同的内容存储到不同的库中将总被存储到相同的名称下，减少需要的硬盘空间
    
### commit对象

如果你有小用过Git，那对commit一定很熟悉，不就是提交嘛，其实不然。在这里的commit表示`快照`，是名词。跟blob一样，也是Git的`基础对象`之一。

查看一个快照（commit）对象，可以看到快照里记录了根目录树，作者，还有提交的信息。
```
$ git cat-file commit d5cb71
tree e84b991fd55acf5eb290a11c0ae4149ce4ffc8dd
author DY.Feng <yyfeng88625@gmail.com> 1345233032 +0800
committer DY.Feng <yyfeng88625@gmail.com> 1345233032 +0800

add a.txt
```
在下一小节`tree对象`里，我们将会学到如何创建一个快照对象，因为快照必须需要一个树对象。

### tree对象

树（tree）对象跟我们本地的目录树很相似，下面我们来试验：

准备试验用文件
```
$ mkdir bb && touch bb/a.txt  #新建一个文件夹并放置一个空文件在里面
$ git add bb/a.txt && git commit -m "add bb/a.txt" #提交
```

查看HEAD树，你会看到只有两个文件，一个是`test.txt`，另外一个是`树bb`，那还有一个文件`a.txt`呢？
```
$ git ls-tree HEAD
040000 tree 65a457425a679cbe9adf0d2741785d3ceabb44a7    bb
100644 blob 63008ae88b4446dfc43b47f18aee5b427203b255    test.txt
```

文件`a.txt`在`bb树`里面。
```
$git ls-tree 65a457425a679cbe9adf0d2741785d3ceabb44a7
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    a.txt
```

树也是Git的`基本对象`，所以准确来说：blob对象*不是*储存在树对象里面，而是树对象里面写有blob对象的`SHA1值`，所以tree对象可以查找到blob对象。

不知道你看到这里有没有发现，我们上面的树（tree）对象，我们并没有手动添加，在`git add`的时候也没有新增，而是在`git commit`之后增加的。究竟树对象是怎么新建的呢？下面我们来试验手动添加树对象：

重置我们的工作目录。
```
$ rm -rf * .git/
$ echo 'Good bye' > a.txt
$ git init 
$ git add a.txt
```

现在缓存区（stage）有我们的a.txt了。
```
$ git ls-files --stage
100644 c0ee9ab00ab41be0d401f00f7a4aaf2e478f9f1e 0       a.txt
$ find .git/objects -type f | sort
.git/objects/c0/ee9ab00ab41be0d401f00f7a4aaf2e478f9f1e
```

我们来创造一棵树吧，在你的电脑上应该会得到同样的一串SHA1值，因为这棵树里面的文件（文件名和文件SHA1值）我们是一样的。
```
git write-tree
e84b991fd55acf5eb290a11c0ae4149ce4ffc8dd
```

有了一棵树，我们就能创造出一个快照（commit）了。记得之前说过，一个快照里面包含了一个树对象的SHA1值，所以你如果要创造一个快照，就必须有一棵树。
```
$ echo "add a.txt" |git commit-tree e84b99
d5cb71ea0e9c6a2c6b198bd5e44b80bd72aac806
$ find .git/objects -type f | sort #我们现在有了一个文件，一棵树，还有一个快照
.git/objects/c0/ee9ab00ab41be0d401f00f7a4aaf2e478f9f1e
.git/objects/d5/cb71ea0e9c6a2c6b198bd5e44b80bd72aac806
.git/objects/e8/4b991fd55acf5eb290a11c0ae4149ce4ffc8dd
```

如果这个快照是有parent的，也就是说在这个快照之前，已经有了一次提交，创建过一个快照。那你可以指定`-p 选项`来指定parent。

我们把master分支头指向到`目前的快照`。
```
$ echo d5cb71ea0e9c6a2c6b198bd5e44b80bd72aac806 > .git/refs/heads/master
```

或者用更加安全的方法。
```
$ git update-ref ref/heads/master d5cb71
```

我们再把`HEAD`指向到master分支头（这一步在我的电脑上，`git init`的时候他已经帮我完成了）。
```
$ git symbolic-ref HEAD refs/heads/master
```

到目前为止，我们已经成功完成了一次`git commit`。
```
$ git log 
commit d5cb71ea0e9c6a2c6b198bd5e44b80bd72aac806
Author: DY.Feng <yyfeng88625@gmail.com>
Date:   Sat Aug 18 03:50:32 2012 +0800

    add a.txt
```


### 总结

究竟我们上面的所有动作，Git都是怎么储存的？让我们看看他都储存了什么文件吧：

```
$ rm -rf .git #我们把现有的.git删除，重新建立一个干净的git
$ git init 
$ tree .
.
├── bb
│   └── a.txt
└── test.txt
$ git commit -m "add all" -a #添加并提交所有的文件到git
$ find .git/objects -type f | sort
.git/objects/45/893175c9b60bdd4727c2a51ced8bae8a9c213f
.git/objects/5b/e1b8923c561189f1fcfc5bcf2d2fe0b7684e76
.git/objects/63/008ae88b4446dfc43b47f18aee5b427203b255
.git/objects/65/a457425a679cbe9adf0d2741785d3ceabb44a7
.git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391
```

从上面我们可以看到，现在有两个文件，两个目录（根目录和bb目录），一个快照（commit）。

这个就是我们的快照（commit），可以看到快照里记录了根目录树。

```
$ git cat-file -t 45893175c9b60bdd4727c2a51ced8bae8a9c213f
commit
$ git cat-file commit 45893175c9b60bdd4727c2a51ced8bae8a9c213f
tree 5be1b8923c561189f1fcfc5bcf2d2fe0b7684e76 
author DY.Feng <yyfeng88625@gmail.com> 1345226467 +0800
committer DY.Feng <yyfeng88625@gmail.com> 1345226467 +0800

add all
```

查看根目录树，可以看到他里面包含了`test.txt文件`和`bb目录`的SHA1值，就像一个指针一样指向他们。
```
$ git cat-file -t 5be1b8923c561189f1fcfc5bcf2d2fe0b7684e76
tree
$ git ls-tree  5be1b8923c561189f1fcfc5bcf2d2fe0b7684e76 
040000 tree 65a457425a679cbe9adf0d2741785d3ceabb44a7    bb
100644 blob 63008ae88b4446dfc43b47f18aee5b427203b255    test.txt
```

查看`test.txt`
```
$ git cat-file -t 63008ae88b4446dfc43b47f18aee5b427203b255
blob
$ git cat-file blob 63008ae88b4446dfc43b47f18aee5b427203b255
Hello,Git!
```

这个是`bb目录`树，里面只有一个文件`a.txt`。
```
$ git cat-file -t 65a457425a679cbe9adf0d2741785d3ceabb44a7
tree
$ git ls-tree 65a457425a679cbe9adf0d2741785d3ceabb44a7
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    a.txt
```

这个是`bb目录`里面的`a.txt`。

```
$ git cat-file -t e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
blob
```

一共增加了5个文件。

这个时候我们改改`bb/a.txt`文件。
```
$ echo 'Good' >bb/a.txt
```

让Git跟踪`bb/a.txt`文件。你会看到里面会新增了一个文件，他就是新修改的`bb/a.txt`文件。
```
$ git add .
$ find .git/objects -type f | sort
.git/objects/45/893175c9b60bdd4727c2a51ced8bae8a9c213f                                                  
.git/objects/5b/e1b8923c561189f1fcfc5bcf2d2fe0b7684e76                                             
.git/objects/63/008ae88b4446dfc43b47f18aee5b427203b255                     
.git/objects/65/a457425a679cbe9adf0d2741785d3ceabb44a7                            
.git/objects/cd/9dd73b78cf79e797c97b8fd5d8b92e28d4437f                                
.git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391
```

即使我们现在用`git checkout`和`git reset`，新增加的那个文件也不会消失，虽然我们现在无法通过正常的途径来获取他。
```
$ git checkout HEAD                           
M       bb/a.txt $ find .git/objects -type f | sort     
.git/objects/45/893175c9b60bdd4727c2a51ced8bae8a9c213f                           
.git/objects/5b/e1b8923c561189f1fcfc5bcf2d2fe0b7684e76                           
.git/objects/63/008ae88b4446dfc43b47f18aee5b427203b255                           
.git/objects/65/a457425a679cbe9adf0d2741785d3ceabb44a7                           
.git/objects/cd/9dd73b78cf79e797c97b8fd5d8b92e28d4437f                           
.git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391                           
$ git reset --hard HEAD 
HEAD is now at 4589317 add all                      
$ find .git/objects -type f | sort            
.git/objects/45/893175c9b60bdd4727c2a51ced8bae8a9c213f                           
.git/objects/5b/e1b8923c561189f1fcfc5bcf2d2fe0b7684e76                           
.git/objects/63/008ae88b4446dfc43b47f18aee5b427203b255                           
.git/objects/65/a457425a679cbe9adf0d2741785d3ceabb44a7                           
.git/objects/cd/9dd73b78cf79e797c97b8fd5d8b92e28d4437f                           
.git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391
```

从这里我们就可以看出Git与其他的版本管理系统有什么区别。最大的区别就是Git记录的其实是每个文件的快照，但他不管这个快照是什么时候在什么目录被什么人建立的，因为这些他都可以通过`tree对象`和`commit对象`来推算出来。



# Local本地使用

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




# Remote远程相关使用

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



# 小技巧

查看`别名`代表的SHA1值
```
$git rev-parse (HEAD|master|...)
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
  
- 提交/快照（commit）。这个`commit`我个人认为，他有两重含义。做动词的时候表示`提交`，例如`git commit`。做名词的时候表示`快照`，是Git的基本对象之一`commit对象`。例如`git log`时你会看到你之前很多次的提交，其实每一次提交都是一次对你工作目录的快照。

- 签出（checkout）。与`reset`的区别：


- 分支（branch）。分支这个对于了解svn的人来说应该很好理解。其实Git的分支只是快照（commit）的一个别名，或者说是一堆快照的别名。例如他定义了这一堆快照叫`branch of development`。

- 标签（tag）。标签这个就是一个快照（commit）的别名。跟`branch`相似，只不过他只能定义一个快照，而且他有自己的概要描述。

- master。master就是主分支，只是一个约定俗成的名字而已，没有什么特殊。

    
# 参考文献

在学习Git的过程中，看过的文献实在是太多了，下面只列出我浏览时间超过5分钟的。

1. 不错的一个[Git常用命令思维导图](http://blog.csdn.net/liuysheng/article/details/7191846)，本文就是基于这个脉络来写的。
2. 关于Git的方方面面（英文）。[原文地址](http://newartisans.com/2008/04/git-from-the-bottom-up/) [PDF下載](http://ftp.newartisans.com/pub/git.from.bottom.up.pdf)
3. [Git基础对象模型介绍](http://guibin.iteye.com/blog/1013279)
