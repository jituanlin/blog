---
layout: post
title: tree的fold和unfold
tags:       [函数式编程]
---

# tree的fold和unfold
`fp-ts`提供了`tree`的常见操作, 其中`fold`和`unfoldTree`可分别应用于`tree`的折叠和展开.

`fp-ts`中`tree`的定义:
```ts
export type Forest<A> = ReadonlyArray<Tree<A>>

export interface Tree<A> {
  readonly value: A
  readonly forest: Forest<A>
}
```

## `fold`: `tree`的折叠

`fold`通过递归的方式, 自底向上, 自左向右, 将一棵树折叠为一个单一的值, 其实现如下:
```ts
export const fold = <A, B>(f: (a: A, bs: ReadonlyArray<B>) => B): ((tree: Tree<A>) => B) => {
  const go = (tree: Tree<A>): B => f(tree.value, tree.forest.map(go))
  return go
}
```
其中`fold`接受一个函数`f:(a: A, bs: ReadonlyArray<B>) => B`, `f`的参数`a`将接受遍历时节点的`value`, 而
`bs`则会接受已经*折叠过*的子节点的值(故此算法为自底向上遍历整棵树).

## `fold`在`React`中的使用

使用场景上, `fold`可用于React的渲染, 代码示例如下:
```tsx
const coRec = (self: S, children: readonly JSX.Element[]) => {
  return (
    <Section self={self} key={self.id}>
      {children}
    </Section>
  );
};

export const renderTreeSection = (treeSection: Tree<S>): JSX.Element => {
  return tree.fold(coRec)(treeSection);
};

const Section = (props: {self: S; children: readonly ReactNode[]}) => {
    return (
        <Styled isSelected={props.self.isSelected}>
            <div className="self">
                <div className="title">{props.self.title}</div>
                <pre className="content">{props.self.content}</pre>
            </div>
            <div className="children">{props.children}</div>
        </Styled>
    );
};
```

上述代码示例借助`fold`将一棵树自底向上, 传入`Section`组件, 最终得到一个`ReactNode`.

### 渲染性能问题
然而这段代码在对`tree`的修改的时候(对树进行immutable式修改可借助`monocle-ts`, 文末给出代码示例), 会触发`React`
对整棵树进行重新渲染(哪怕对`Section`进行`React.memo`也无济于事), 在树的节点众多的时候会导致性能低下.

这是由于`fold`的实现中是使用`tree.forest.map(go)`递归获取当前节点的子节点的`Section`, 哪怕`go`返回
的遍历(通过`forest.map`)后的`Section[]`中的每一个元素都是被命中`React.memo`的cache的, 其作为数组的引用
也是全新的, 简化版本的示例如下:
```ts
const ns = [1,2,3]
ns.map(n => n) !== ns
```

故, `React.memo`在对其进行`shadow compare`的时候, 永远返回`false`, 从而触发`Section`的重新渲染.

### 解决方案
综上所述, 我们自定义`React.memo`的`compare`逻辑, 使其在对比`Section[]`的时候, 若`Section[]`中
的每一个元素(`Section`)都与上一次渲染相同(`shadow compare`), 则返回`true`(O(n)), 不再简单地使用
整个数组进行`shadow compare`, 从而避免无效渲染.

代码实现:

```ts
type CoRecArgs = {
  self: S;
  children: readonly ReactNode[];
};
const eqBySelf = eq.contramap<S, CoRecArgs>(({self}) => self)(eq.eqStrict);

const eqByChildren = eq.contramap<readonly ReactNode[], CoRecArgs>(
  ({children}) => children
)(readonlyArray.getEq<ReactNode>(eq.eqStrict));

const eqSectionProps = eq.getMonoid<CoRecArgs>().concat(eqBySelf, eqByChildren);

export const Section = React.memo(_Section, eqSectionProps.equals);
```

## `unfoldTree`
`unfoldTree`用于将`一颗种子`展开, 最终形成一颗树, 其实现如下:
```ts
export const unfoldTree = <B, A>(b: B, f: (b: B) => readonly [A, ReadonlyArray<B>]): Tree<A> => {
  const [a, bs] = f(b)
  return { value: a, forest: unfoldForest(bs, f) }
}

export const unfoldForest = <B, A>(bs: ReadonlyArray<B>, f: (b: B) => readonly [A, ReadonlyArray<B>]): Forest<A> =>
    bs.map((b) => unfoldTree(b, f))
```

其运行步骤为:
1. 将参数`b: B`传出函数`f: (b: B) => readonly [A, ReadonlyArray<B>]`, 得到`[a, bs]`, 
其中`a`(`A`)作为当前节点的`value`, `bs`(`ReadonlyArray<B>`)数组的元素作为生成对应子节点的*种子*.
   
2. 将得到的子节点的种子间接地再次传入`unfoldTree`(借助`unfoldForest`), 递归生成子节点.

这种*递归函数返回当前值和进行下一次递归的种子*的模式称为`coRecusive`.

### 使用`unfoldTree`将数据库返回的一维数据展开成一棵树
假设数据库返回的目录树的数据是这样的:
```ts
export const titles: readonly Title[] = [
  {
    id: 1,
    title: 'level1',
    parentId: null,
  },
  {
    id: 2,
    title: 'level2-1',
    parentId: 1,
  },
  {
    id: 3,
    title: 'level2-2',
    parentId: 1,
  },
  {
    id: 4,
    title: 'level3-1',
    parentId: 2,
  },
  {
    id: 5,
    title: 'level3-2',
    parentId: 2,
  },
];
```
通过`unFoldTree`可将其展开成一棵树:

```ts
export const treeTitle: TreeTitle = F.tree.unfoldTree<Title, number>(1, id => {
  const self = titles.find(R.propEq('id', id), titles)!;
  const seeds = F.pipeable.pipe(
    titles,
    R.filter<Title, 'array'>(R.propEq('parentId', self.id) as any),
    R.pluck('id')
  );

  return [self, seeds];
});
```

## 总结
通过`fold`和`unfoldTree`我们可以将一棵树折叠成一个值, 或者将一个值展开成一棵树. 

额外的, 在处理`unfoldTree`的时候, *获取种子对应的节点的`value`和其子节点的种子* 这个过程可能是
`带上下文`(`contextful`)的(比如异步), 这个时候可以使用`fp-ts`的`unfoldTreeM`进行处理.


## 引用
- [使用`monocle-ts`对树进行immutable式修改](https://github.com/polymona/cookbook/blob/master/react-stack/src/pages/using-tree-in-front-end/optics/section.ts) 
- [可运行的所有示例的demo](https://github.com/polymona/cookbook/tree/master/react-stack/src/pages/using-tree-in-front-end)
