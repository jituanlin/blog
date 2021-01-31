---
layout: post title: remote-data tags:       [函数式编程]
---

# `remote-data`

`remote-data`是一个ts库, 提供对*远程获取类型的数据*的抽象.

## 远程数据的串行计算`map`

在`React`中, 我们经常使用`hook`进行数据获取, 下面以`react-query`库为例展示一个 常见的获取数据例子:

```tsx
import {useQuery} from 'react-query'

const Demo = () => {
    const {data: users, error, isLoading, isError, isSuccess} = useQuery('fetchUsers', fetchUsers)
    const vipUsers = isSuccess ? users.data.filter(user => user.idVip) : []
    return <>
        <User users={users}/>
        <VipUsers vipUsers={vipUsers}/>
    </>
}
```

上面这段代码使用了`useQuery`获取数据, 其返回结果包含在这样一个*上下文*中:

1. 如果`fetchUser`请求成功, `isSuccess`将被设定为`true`, 且`data`为其对应的请求结果
2. 如果`fetchUser`请求失败, `isError`将被设定为`true`, 且`error`为其抛出的异常
3. 如果`fetchUser`正在执行, `isLoading`被设置为`true`

*`isError`, `isSuccess`, `isLoading` 为互斥关系, 即同一个时间里, 只有其中一个字段会为`true`*

随后, `const vipUsers = isSuccess ? users.data.filter(user => user.idVip) : []`基于`fetchUsers`的
状态计算出一个衍生于其结果的数据: `vipUsers`.

但这段代码存在问题:
> `vipUsers`只考虑了`fetchUsers`的`isSuccess`状态, 如果`fetchUser`为成功状态, 则返回其过滤后的数组,
> 否则一律返回空数组.

这使得`<VipUsers/>`组件接受到空数组后无从得知究竟是其依赖数据`users`在`loading`中呢, 还是`users`
获取失败导致的, 从而无法进一步根据其特定的状态进行展示.

这里的`vipUsers`作为`userd`的衍生数据, 其状态亦应与其一一对应, 即:

1. 若`users`处于`success`状态, `vipUsers`亦为`success`状态
2. 若`users`处于`error`状态, `vipUsers`亦为`error`状态
3. 若`users`处于`loadomg`状态, `vipUsers`亦为`loadog`状态

为了方便进一步表述, 在此我们将存在于`error`, `loading`, `success`*上下文*中的值定义为`RemoteData`, 其 类型声明如下:

```ts
export type RemoteData<E, A> = RemoteInitial | RemotePending | RemoteFailure<E> | RemoteSuccess<A>;

//constructors
export const failure = <E>(error: E): RemoteData<E, never> => ({
    _tag: 'RemoteFailure',
    error,
});
export const success = <A>(value: A): RemoteData<never, A> => ({
    _tag: 'RemoteSuccess',
    value,
});
export const pending: RemoteData<never, never> = {
    _tag: 'RemotePending',
    progress: none,
};
export const progress = (progress: RemoteProgress): RemoteData<never, never> => ({
    _tag: 'RemotePending',
    progress: some(progress),
});
export const initial: RemoteData<never, never> = {
    _tag: 'RemoteInitial',
};
```

*除了`error`, `loading`, `success`, `remote-data`还定义了一种状态`initial`, 表示该请求尚未发起*.

由此, 我们将`vipUser`的计算过程定义为: 给定映射(函数)`f: (a:A) => B`与`RemoteData<L,A>`, 若
`RemoteData<L,A>`处于`RemoteSuccess`状态, 则应用映射`f`, 将其转换为`RemoteData<L,B>`, 其实现如下:

```ts
map: <L, A, B>(fa: RemoteData<L, A>, f: FunctionN<[A], B>): RemoteData<L, B> =>
    isSuccess(fa) ? success(f(fa.value)) : fa
```

基于`map`操作, 我们可以将修复上述代码示例:

```tsx
import {useQuery} from 'react-query'
import {remoteData} from 'remote-data'
import {fromQuery} from './utils' // 一个帮助将query转换为RemoteData的函数, 文末给出其实现

const Demo = () => {
    const usersQuery = useQuery('fetchUsers', fetchUsers)
    const usersRd = fromQuery(usersQuery)
    const vipUsersRd = remoteData.map(users => users.filter(user => user.idVip), usersRd)

    return <>
        <User users={users}/>
        <VipUsers vipUsersRd={vipUsersRd}/>
    </>
}
```

修复后的版本中, `<VipUsers/>`接受一个`RemoteData`类型的参数, 可通过其状态判断其处于`error`, `loading`, `success`
的哪个状态, 从而展示对应的UI.

小结: `map`函数将*对一个包含于远程请求上下文中的数据进行串行计算*的过程进行抽象, 接受一个映射`f`和`RemoteData`, 返回另外一个`RemoteData`, 这种模式不仅仅应用于`RemoteData`类型,
也应用于我们常见的`Array`(`map`函数),
`Promise`(`then`函数)中.

## 远程数据的串行计算`chain`(别名`flatMap`,`bind`)

在上一个小节中, 将`map`等价于`Promise.prototype.then`, 然而`Promise.prototype.then`不仅支持
`(f:(a:A)=>B, pa:Promose<A>) => Promise<B>`的操作(对标`RemoteData`的`map`函数), 还支持
`(f:(a:A)=>Promise<B>, pa:Promose<A>) => Promise<B>`的操作.

其实`RemoteData`也支持此类操作, 不过由于*单一职责原则*将其拆分为`chain`函数:

```ts
chain: <L, A, B>(fa: RemoteData<L, A>, f: FunctionN<[A], RemoteData<L, B>>): RemoteData<L, B> =>
    isSuccess(fa) ? f(fa.value) : fa
```

即, `map`函数是将一个`远程数据的上下文`中的值取出进行计算, 而后将计算结果重新包裹进一个`远程数据的上下文`.

而, `chain`函数是将一个`远程数据的上下文`中的值取出进行计算, 其计算结果为另外一个`远程数据的上下文`, 而后将两层*上下文*进行一个*拍平操作*(flat/join), 最终返回另外一个包裹于`远程数据的上下文`中的值.

代码示例如下:

```tsx
import {useQuery} from 'react-query'
import {remoteData} from 'remote-data'
import {fromQuery} from './utils' // 一个帮助将query转换为RemoteData的函数, 文末给出其实现

const Demo = () => {
    const usersQuery = useQuery('fetchUsers', fetchUsers)
    const usersRd = fromQuery(usersQuery)

    const noPaidUsersQuery = usersQuery('fetchNoPaidUsers', fetchNoPaidUsers)
    const noPaidUsersRd = fromQuery(noPaidUsersQuery)

    const paidVipUsersRd = remoteData.chian(users => {
        return remoteData.map(
            noPaidUsers => {
                const notPaidUserIds = noPaidUsers.map(({id}) => id)
                return users.filter(user => user.idVip && !notPaidUserIds.includes(user.id))
            },
            noPaidUsersRd
        )

    }, usersRd)

    return <>
        <User users={users}/>
        <VipUsers vipUsersRd={paidVipUsersRd}/>
    </>
}
```
上述例子中, 基于`usersRd`计算返回的是另外一个`RemoteData`, 为了避免`RemoteData`的嵌套, 需要使用`chain`而不是`map`.

小结, `chain`操作支持*对一个`RemoteData`进行串行计算, 其返回值同样为`RemoteData`, 将两个`RemoteData`进行`join`/`flat`
从而避免了两个`RemoteData`的嵌套*.

## `Promise.all`?
`Promise.all`支持将`[Promise<A>, Promise<B>, Promise<C>/*...*/]`(即*Array of Promise*)转换为
`Promise<[A, B, C /*...*/]>`.

这种操作逻辑不仅仅应用于`Promise`, 也出现在其他带`上下文`的计算中, 例如`RemoteData`, 然后我们对其进行通用化抽象(为了简化定义, 我们
使用可高阶类型(HKT)的语法, ts现不支持该特性):

```ts
type Sequence<M,F> = (ma:M<F<A>>) => F<M<A>>
// Promise.all为sequence的具象化
type PromiseALl = Sequence<Array, Promise>
// RemoteData.sequnce同样为其具象化
type RemoteDataSequnce<M> = Sequence<M, RemoteData>
```

`remote-data`对其的实现如下:
```ts
//Traversable
	traverse: <F>(F: Applicative<F>) => <E, A, B>(
		ta: RemoteData<E, A>,
		f: (a: A) => HKT<F, B>,
	): HKT<F, RemoteData<E, B>> => {
		if (isSuccess(ta)) {
			return F.map(f(ta.value), a => remoteData.of(a));
		} else {
			return F.of(ta);
		}
	}
	sequence: <F>(F: Applicative<F>) => <L, A>(ta: RemoteData<L, HKT<F, A>>): HKT<F, RemoteData<L, A>> =>
		remoteData.traverse(F)(ta, identity)
```

这段代码需要读者了解`Applicative`, `Traversable`和`fp-ts`的`HKT`实现, 超出本文的范围, 故先给出一个引子, 后续博客会继续阐述.

小结, `sequence`是对类型是`Promise.all`操作的抽象化与通用化, 其本质是对两个嵌套的不同*上下文*进行*里外翻转*.

## 总结
`remote-data`从`远程数据获取上下文`出发, 提供了一系列对于其操作的便利的函数, 这些函数使用了常见
的(`fp-ts`)的代数类型进行定义, 从而实现了抽象的统一, 可结合`react-query`进行使用.

## 参考
- [remote-ts github仓库](https://github.com/devexperts/remote-data-ts)
- [fromQuery的实现](https://github.com/jituanlin/cookbook/blob/master/react-stack/src/utils/remoteData.ts)
- [Traversable教程](https://typelevel.org/cats/typeclasses/traverse.html)
- [Applicative教程](https://typelevel.org/cats/typeclasses/applicative.html)
