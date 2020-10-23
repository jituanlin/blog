---
layout: post
title: 内存管理中的 stack 和 heep
tags:       [js]
---

#  内存管理中的 stack 和 heep
本文为记录一个理解上的疑问, 待后续理清后补充.

许多文章中如此写道:
> JS中的primitive value 储存在stack中, 而object存储在heep中.

根据我现在的理解, 这是错误的, 其自洽的解释是(以java的内存实现为例):
1. 所有的object都是储存在heap中.
2. primitive和object reference类型的变量储存在哪, 依据变量所属方区分.
方法相关的变量(local variable/parameter/...)储存在stack上,
作用于超出方法作用域的, 储存于heap上.
3. 其他机制在此忽略...

之所以说这是自洽的, 是因为这符合函数调用栈的使用机制:
> 函数调用完后自动释放函数所属的变量(local variable/parameter/...)的内存.



--- 
参考:
- [stackoverflow上的回答](https://stackoverflow.com/questions/13624462/where-does-class-object-reference-variable-get-stored-in-java-in-heap-or-in-s)
