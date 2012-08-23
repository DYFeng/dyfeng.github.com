---
layout: post
title: "解决Firefox菜单不弹出和右键失效的问题"
date: 2012-08-23 13:19
comments: true
categories: linux
tags: [firefox , ubuntu , fcitx ， kde ]
---

最近一次kubuntu升级后，我发现一个很奇怪的现象，我的Firefox某些时候菜单栏不能弹出，右键也失灵了。准确来说，是弹出了，又很快地焦点转移，导致他又关闭了。Google下发现遇到这个问题的童鞋还真不少，而且解决的方法各有不同，看来这个问题是Firefox一直存在的。

<!-- more -->
网上的解决方法：

- 升级你的Firefox到14+。
- 重装你的Firefox
- 插件冲突

后来我发现他有个规律，一开始Firefox是好的，后来我在里面搜索打中文的时候，灵异的事情发生了，右键和菜单又能用了。这时候我才发现是我的输入法`fcitx`的问题。

解决的方法是安装`fcitx-frontend-gtk2`和`fcitx-frontend-gtk3`，因为`fcitx-frontend-qt4`我是一直有装的，所以就不重复了。

```bash
$ sudo apt-get install fcitx-frontend-gtk2 fcitx-frontend-gtk3
```

想起我这个ubuntu是先装了`Unity`界面，后来我卸载了`Unity`，重新回到我`KDE`的怀抱。可能就是在清理软件的时候，多手删了，善哉善哉。
