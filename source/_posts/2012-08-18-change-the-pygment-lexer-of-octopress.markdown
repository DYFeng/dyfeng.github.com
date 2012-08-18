---
layout: post
title: "修改octopress的pygment语法"
date: 2012-08-18 18:43
comments: true
categories: Linux
tags: [octopress , pygement , python,regular expression,lexer]
---

最近我在写一篇关于Git使用的文章，里面贴了大量`bash`命令。我的语法高亮是用Octopress自带的`pygements.rb`，用起来总觉得不大爽，因为他高亮的地方实在是太少了，只有少量的命令有高亮。自己动手丰衣足食，下面记录我修改的过程：

<!-- more -->

# Octopress语法高亮原理
我要修改Octopress的语法高亮，当然得搞清楚他的来龙去脉。

首先在`plugins`目录下你会看到`pygments_code.rb`插件，这个插件的功能就是调用`pygements.rb`库，把代码转换成html，还在你项目的`.pygements-cache`目录下缓存代码块。代码样式他其实是通过css调节的，他生成的只是一个堆html标签。

`pygements.rb`库其实是ruby封装了python的`pygements`库，而且`pygements.rb`里面已经自带了一个`pygements`库（这真不是一个好主意，系统讲究的应该是低耦合）。

就这样...

1. Octopress调用pygments_code.rb插件。
2. pygments_code.rb插件调用pygements.rb库。
3. pygements.rb库调用python的pygements库。

完成了对代码的语法高亮。

# 修改Octopress语法高亮规则

你先要确保你的电脑上其他地方没有安装python的`pygements`。

```bash
$ sudo apt-get remove python-pygments
```

，我们先找出你`pygments.rb`的安装目录：
```bash
$ bundle show pygments.rb
/var/lib/gems/1.9.1/gems/pygments.rb-0.2.13
```

修改`pygements.rb`自带python`pygments`库的语法文件。
```bash
$ cd YOUR_PYGEMENTS.RB_PATH/vendor/pygments-main/
$ vim pygments/lexers/shell.py
```

找到`BashLexer`下`tokens`字典，`basic`下，大概修改成这个样子。
```python
    # (r'\b(alias|bg|bind|break|find|builtin|caller|cd|command|compgen|'
    #  r'complete|declare|dirs|disown|echo|cat|enable|eval|exec|exit|'
    #  r'export|false|fc|fg|getopts|hash|help|history|jobs|kill|let|'
    #  r'local|logout|popd|printf|pushd|pwd|read|readonly|set|shift|'
    #  r'shopt|source|suspend|test|time|times|trap|true|type|typeset|'
    #  r'ulimit|umask|unalias|unset|wait'
    #  r')\s*\b(?!\.)',
    #  Name.Builtin),
    (r'(?<=\$\s)\b\w+',Name.Builtin),
```

一开始我是想遍历`PATH`下所有目录，把里面的命令都动态加上，但python的正则抗议太多选项了，所以我就唯有手动改规则了。现在的规则是以`$ `开头的一个单词定义为命令。

最后一步千万不要漏了，清除`pygments_code.rb`插件的缓存，重新生成页面

```bash
$ rm -rf .pygments-cache/*
$ rake generate
$ rake preview
```

现在你应该看到bash的语法高亮`亮`多了吧。

# 参考文献

1. [如何编写pygements的语法规则](http://pygments.org/docs/lexerdevelopment/)，来自pygements官网
2. [python正则表达式参考](http://docs.python.org/library/re.html)，来自python官网
