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
这里所说的类型, 定义为符合某种约束的值的集合.

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
- `prism`, 亦用于有容错处理的类型转换.
- `optional`, 用于可缺省值(`Option`)形数据.
- `traversal`, 用于可遍历形数据.
- `iso`, 用于对isomorphic(同构)的两个类型之间的转换.
- `fold`, 用于对数据中的某一部分进行收集后, 作为数组进行操作.

## 如何使用`monocle-ts`

### `monocle-ts`模块结构
`monocle-ts`(v2.3.3开始), `import * as optics from 'monocle-ts'`将会导出一系列构造函数(`optics.Lens, optics.Prism ...`)和工厂函数(`optics.fromTraversable, optics.fromLens ...`)
和模块(`optics.traversal, optics.lens ...`).

形如`optics.Prism`等为`class`, 其中包括`get, modify, set, comoposeXXX`等常用方法.

形如`optics.fromTraversable`等为工厂函数, 返回类型皆为`optics.Prism`等`class`的对象.

形如`optics.traversal`等皆为模块, 模块中多为类型和**函数**定义, 其中类型如`optics.traversal.Traversal`皆为[ADT](https://jituanlin.github.io/%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B/2020-07-27-%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B%E5%AF%BC%E8%A8%80/#%E4%BB%A3%E6%95%B0%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B),
而非`class`.

### 代码示例
请参考[项目](https://github.com/jituanlin/cookbook/tree/master/js-stack/monocle-ts).

### 补充说明

#### `prism`
[monocle的官方文档中](https://www.optics.dev/Monocle/) 将`prism`定义为使用于`sum type`类型的`optics`, 
笔者认为更广泛地来讲, `prism`应该定义为: 用于两个[injective](https://www.wikiwand.com/en/Injective_function) 的类型(即,`B => Option<A>`且`A => B`)的转换.
如, `string => number`之间的转换是有可能失败的, 但`number => string`的类型转换总是可以成功的.

其`ADT`定义为:
```typescript
export interface Prism<S, A> {
  readonly getOption: (s: S) => Option<A>
  readonly reverseGet: (a: A) => S
}
```

详细代码示例请参考[项目](https://github.com/jituanlin/cookbook/tree/master/js-stack/monocle-ts) (下同, 不再赘述).

#### `iso`
`iso`即为同构(isomorphic)的缩写, 同构在此定义为: 用于两个[bijective](https://www.wikiwand.com/en/Bijection) 的类型(即,`B => A`且`A => B`)的转换.
即两个类型之间的转换是不会丢失信息的, 如object可通过`Object.entries`转换成`[key,value]`数组, 而`[key,value]`数组亦可通过`Object.fromEntries`
转换成object, 故转换之间之间没有任何信息丢失, 可称`Object.fromEntries`与`Object.entries`是同构的.

其`ADT`定义:
```typescript
export interface Iso<S, A> {
  readonly get: (s: S) => A
  readonly reverseGet: (a: A) => S
}
```

#### `traversal`
`traversal`用于对数据中的某一部分可遍历的数据(`Record`, `Array` ...)进行操作, 类似`map`之于`Array`.

`traversal`作为`ADT`, 其定义为:
```typescript
export interface Traversal<S, A> {
  readonly modifyF: ModifyF<S, A>
}

export interface ModifyF<S, A> {
  <F>(F: Applicative<F>): (f: (a: A) => HKT<F, A>) => (s: S) => HKT<F, S>
}
```

`modifyF`是`Traversal`作为一个`optics`的核心, `modify`即意味着修改, 而末尾的`F`即表示这个修改是带有副作用的.

字母`F`是`Functor`的缩写, 而函数式编程中使用`Functor`表示副作用, 故`fa`一般表示`Functor[A]`(scala语法), 
`fab`一般表示`Functor[A=>B]`. 

通过`modifyF`, 我们可以实现`set`, `modify`, 甚至`get`(理论上可行, monocle未如此实现)操作. 以下为其简化版实现:
```typescript
import * as fp from 'fp-ts';
import {Kind, URIS} from 'fp-ts/lib/HKT';

export class Traversal<S, A> {
  constructor(
    private readonly modifyF: <F extends URIS>(
      F: fp.applicative.Applicative1<F>
    ) => <B>(f: (a: A) => Kind<F, B>) => (s: S) => Kind<F, S>
  ) {}

  modify: <B>(f: (a: A) => B) => (s: S) => S = f =>
    this.modifyF(fp.identity.identity)(f);

  set: (a: A) => (s: S) => S = a => this.modify(() => a);
}
```

初始化`traversal`示例:
```typescript
export const traversalArray = <T>(): Traversal<T[], T> =>
  new Traversal<T[], T>(F => f => s => {
    const fbs = s.map(f);
    return fp.array.array.reduce(fbs, F.of([]), (fbs, fb) =>
      F.ap(
        F.map(fb, b => bs => bs.concat(b)),
        fbs
      )
    );
  });
```



除了通过直接传入`modifyF`初始化`traversal`, `optics.fromTraversable`还提供了一个便利的方法获取`traversal`, 其参数`Traversable<T>`为`ADT`, 该`ADT`核心函数为
`Traverse`, 其类型声明如下(Scala语法):

```
trait Traverse[F[_]]{
 def traverse[G[_]: Applicative, A, B](fa: F[A])(f: A => G[B]): G[F[B]]   
}
```

即, `traverse`函数接收`F[A]`, 将`F[A]`中的所有的`A`利用函数`f: A => G[B]`转化为一系列`G[B]`, 
后利用隐式参数Applicative的实例`G`中的方法`ap`和`of`, 将所有的`G[B]`串联并入`F`, 得到`G[F[B]]`.

关于`Traversable`的细节请参考[该文档](https://typelevel.org/cats/typeclasses/traverse.html).

#### `fold`
与`traversal`专注于可遍历数据中的每一个元素不同, `fold`将可遍历数据看做一个整体, 提供了类似`Array`的方法.

其主要方法为:
- `all`判断所有的元素是否符合传入的谓词函数.
- `getAll`返回以数组的形式返回所有的函数.
- `exist`判断是否有元素符合传入的谓词函数.

`fold`的构造方法为:
```typescript
  constructor(readonly foldMap: <M>(M: Monoid<M>) => (f: (a: A) => M) => (s: S) => M) {
    // ...
  }
```

其中`foldMap`的含义为(参见`Lens.asFold`): 从最终返回的函数`(s: S) => M`中的参数`s`取出一系列`A`, 传入函数`f`,
得到一系列`M`, 再将这一系列`M`使用`Monoid<M>`的实例的`concat`合并成一个最终`(s: S) => M`返回的`M`.

`Traversable.asFold`方法体现了*foldMap为Traverse的特殊化实现*这一条定理, 详情请阅读[此文档](https://typelevel.org/cats/datatypes/const.html).

---
参考:
- [思维导图源文件](https://github.com/jituanlin/public-docs/blob/master/mindmaps/optics.png).
- [scalac上一篇质量上乘的文章](https://scalac.io/scala-optics-lenses-with-monocle/)
- [如何利用modifyF实现get操作](https://typelevel.org/cats/datatypes/const.html)

--- 
注:
- monocle-ts的作者曾发表过一篇[文章](https://medium.com/@gcanti/introduction-to-optics-lenses-and-prisms-3230e73bfcfe), 其内容已过期, 不适合再作为参考.
 

