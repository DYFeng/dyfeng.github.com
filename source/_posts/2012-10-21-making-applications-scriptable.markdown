---
layout: post
title: "如何使用Qt把你的程序与脚本结合（Making Applications Scriptable翻译）"
date: 2012-10-21 03:22
comments: true
categories: Program
tags:[qt , C++ , QtScript]
---

(原文地址)[http://doc.qt.digia.com/qt/scripting.html]

# 前言
Qt Script 是一种基于ECMAScript的脚本语言，建立的标准是ECMA-262

# 基本用法

首先我们创建一个`QScriptEngine`对象，调用evaluate()函数，把要运行的Javascript代码文本作为参数。

```
 QScriptEngine engine;
 qDebug() << "the magic number is:" << engine.evaluate("1 + 2").toNumber();
```
