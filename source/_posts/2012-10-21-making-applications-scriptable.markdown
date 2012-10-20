---
layout: post
title: "如何使用Qt把你的程序与脚本结合（Making Applications Scriptable翻译）"
date: 2012-10-21 03:22
comments: true
categories: Program
tags: [qt , C++ , QtScript]
---

初学Qt Script  [原文地址](http://doc.qt.digia.com/qt/scripting.html)

# 前言
Qt Script 是一种基于ECMAScript的脚本语言，建立的标准是ECMA-262

# 基本用法

首先我们创建一个`QScriptEngine`对象，调用evaluate()函数，把要运行的Javascript代码文本作为参数。

``` c++
QScriptEngine engine;
qDebug() << "the magic number is:" << engine.evaluate("1 + 2").toNumber();
```

脚本运行的结果是一个QScriptValue对象，可以转换成标准C++和Qt类型。

我们可以把一些自定义属性注册到脚本引擎，使得我们可以在脚本里使用。其中一种最简单的方法就是把属性注册到`全局对象（Global Object）`：

```c++
 engine.globalObject().setProperty("foo", 123);
 qDebug() << "foo times two is:" << engine.evaluate("foo * 2").
```
把属性引入到脚本引擎里，使得他们能够在脚本里使用。

# 在脚本引擎里使用QObject

任何基于QObject的对象都能在脚本里使用。

我们使用 QScriptEngine::newQObject() 函数来包装一个QObject对象，这样我们就可以在脚本里使用QObject的信号槽、属性、子对象。

下面这个例子里，我们把一个QObject对象作为脚本变量"myObject"的值

```c++ 
 QScriptEngine engine;
 QObject *someObject = new MyObject;
 QScriptValue objectValue = engine.newQObject(someObject);
 engine.globalObject().setProperty("myObject", objectValue);
```
 
上面的例子在脚本环境里创建了一个全局变量`myObject`，这样我们就可以在脚本环境通过变量来操作C++对象。注意一点，脚本变量的名称是可以任意的，跟QObject::objectName()没有任何关系。


