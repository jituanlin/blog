---
layout: post
title: monocle-ts 补充文档
---

# monocle-ts 补充文档
本文为[monocle-ts](https://github.com/gcanti/monocle-ts)的指引文档的补充.

请先阅读[scala实现版本的文档](https://www.optics.dev/Monocle/).

知识体系预览:
![思维导图](https://raw.githubusercontent.com/jituanlin/public-docs/master/mindmaps/optics.png)

## 相关类型知识
在进入正题之前, 我们先介绍下一些涉及到的类型方面的术语.
这里所说的类型, 定义为符合某种约束的值得集合.

###  product type(交集类型)
例:
```typescript
interface People{
    name:string;
    age:number;
}
```
这里的`People`即为交集类型, 代表着`string`, `name` 是**同时**存在的.
`People`作为值的集合, 其组合的可能情况为 `符合string类型的值的个数` 
乘以 `符合number类型的值的个数` 种, 此即`product type`中的`product`(乘法)的由来.

### sum type(并集类型)
例:   
```typescript
enum Sex{
    Male,
    Female
}
```
这里的`Sex`即为交集类型, 代表着`Male`, `Female` 是**互斥**存在的.
`Sex`作为值得集合, 其组合的可能情况为 `符合Male类型的值(只有一个0)的个数` 乘以
`符合Female的类型的值(只有一个1)的个数` 种, 此即`sum type`的`sum`(加法)的由来.

## 什么是optics
optics为一种工具, 用于专注于复杂不可变数据结构中的*特定部分*进行检索和修改.

从其实现层面上, 是一组getter和setter的组合, 其类型表示分别为:
- getter = whole => part, 即, getter函数接受整个数据, 返回整体中的一部分
- setter = whole => value => whole, 即, setter作为高阶函数接受整个数据, 返回另外一个函数, 该函数接受一个值作为替换之用, 返回修改后的整个数据

其优势在于: 灵活, 可组合, 声明式.

## optics的种类
针对不同类型的数据, 有以下5中`optics`:
- `lens`, 用于`product type`形数据.
- `prisim`, 用于`sum type`形数据.
- `optional`, 用于可缺省值(`Option`)形数据.
- `traversal`, 用于可遍历形数据.
- `iso`, 用于对isomorphic(同构)的两个类型之间的转换.

### 使用`lens`




---
参考:
1. [思维导图源文件](https://github.com/jituanlin/public-docs/blob/master/mindmaps/optics.png)


