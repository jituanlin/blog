---
layout: post
title: contramap
tags:       [函数式编程]
---

# `contramap`
`contramap`是一个函数, 我们先来介绍下它的使用 -- 排序规则的组合.

## `contramap`与排序
假设我们将对类型`A`的排序`方法`记作`Ord<A>`.

又假设, 我们现在有一个类型`B`, 已知类型映射`B => A`以及`Ord<A>`, 如何求得针对类型`B`的排序方法
`Ord<B>`?

从实现上, 我们可以将类型`B`映射到`A`(通过`B => A`), 然后对得到的`A`进行排序, 得到的次序即为
类型`B`的排序次序.

暂且把这个通过映射`B => A`和排序方法`Ord<A>`得到的`Ord<B>`的方法称为`contramap`, 其类型声明如下:
```ts
type Contramap = <A, B>(f: (a: B) => A) => (ordA: Ord<A>) => Ord<B> 
```

将`Ord`替换为类型参数`F`, 我们得到了`contramap`的通用版本:

```ts
type Contramap = <F, A, B>(f: (a: B) => A) => (fa: F<A>) => F<B> 
```

## `contramap`与`map`
`map`函数我们都熟悉, `Array`类型的`map`函数类型签名如下:

```ts
type Map = <A, B>(f: (a:A) => B) => (fa: Array<A>) => Array<B>
```

同样的, 通过将`Array`替换成类型参数`F`, 我们得到其通用版本:

```ts
type Map = <F, A, B>(f: (a:A) => B) => (fa: F<A>) => F<B>
```

通过观察`contramap`与`map`的类型签名, 我们不难看出它们之间的联系:

```ts
type Map =       <F, A, B>(f: (a:A) => B) => (fa: F<A>) => F<B>
type Contramap = <F, A, B>(f: (a:B) => A) => (fa: F<A>) => F<B> 
```

即, 映射`f`的`A`和`B`在`contramap`中是颠倒过来的, 这也是`contra`(相反)命名的由来.

这是其表面联系, 让我们再继续深究下去.

## type constructor `F` 的含义

`F` type constructor 在函数式编程意味着`上下文`(context), 即:

```ts
type Map = <F, A, B>(f: (a:A) => B) => (fa: F<A>) => F<B>
```

意味着将**上下文**`F`中的`A`取出, 通过映射`A => B`映射为`B`, 再重新*装*回**上下文**`F`中, 
对`Array`来说, 这个上下文即`数组`(有次序的N个类型为`A`的值).

而:
```ts
type Contramap = <A, B>(f: (a:B) => A) => (fa: Ord<A>) => Ord<B>
```
则有着**映射串联**的意味, 有些抽象? 不着急, 我们想看看`Ord`在`fp-ts`的声明:

```ts
export interface Ord<A>{
  readonly compare: (second: A) => (first: A) => Ordering
}

export type Ordering = -1 | 0 | 1
```

即, `Ord` 是一个对象, 其中包含着`A => A => Ordering`的映射`compare`.

为了帮助进一步理解, 让我们将`Ord<A>`简化为`A => A => Ordering`

即, `Ord<A>`等价于`A => A => Ording`(类型别名), 即:
> `Ord<A>`是一个`A => A => Ording`映射

让我们将`Ord<A>`简化为`A => A => Ordering`, 替换进再来看看`contramap`:
```ts
type Contramap =
    <A, B>(f: (a:B) => A) => 
        (fa: ((a1:A) => (a2: A) => Ordering)) => 
            (b1:B) => (b2: B) => Ordering
```
即, `Ord`的`contramap`是**将两个映射串联起来**, 它接受映射`f: A => B` 和另外
一个映射`fa: A => A => Ording`, 返回另外一个映射`fb: B => B => Ording`.

当我们往得到的映射`fb: B => B => Ording`中填入类型为`B`的值的时候, 这个映射
将两个参数`B`同时转换为`A`, 再传入映射`fa`中, 得到的`Ording`即为`fb`的返回值.

参考`fp-ts`对`Ord`的`contramap`的实现:
```ts
export const contramap: Contravariant1<URI>['contramap'] = (f) => (fa) =>
  fromCompare((second) => (first) => fa.compare(f(second))(f(first)))
```

## 总结

`F`是一个`上下文`, 它不仅仅可以是容器(`Array`, `Set`, `Map`, `Record`), 还可以
是映射`F<A,B> = (A => B)`(一元函数又名Arrow), `F<A> = (A => F<B>)`(ReaderT又名Kleisli)等等.

*ps: 如果这个总结给你带来了更大的疑惑, 那么请关注后续的其他博客, 会有进一步的讲解*.
