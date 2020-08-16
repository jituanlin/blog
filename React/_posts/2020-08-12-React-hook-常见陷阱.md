---
layout: post
title: React hook 常见陷阱
---

# React hook 常见陷阱

## 为什么不将`deps`传入`useMemo`和`useEffect`中会导致其回调函数使用陈旧的`state`

该部分内容为[此博客](https://dmitripavlutin.com/react-hooks-stale-closures/) 的归纳总结.

```tsx
import React, { useEffect, useState } from "react";

export const StaleClosureExample = () => {
  const [count, setCount] = useState(0);
  const incCount = () => setCount(count + 1);

  const intervalLogCount = () => {
    const intervalId = setInterval(() => console.log(count), 1000);
    return () => clearInterval(intervalId);
  };
  useEffect(intervalLogCount, []);

  return (
    <div>
      <p>count:{count}</p>
      <button onClick={incCount}>inc count</button>
    </div>
  );
};
```

上述代码中, `useEffect`输出的`count`永远为`1`, 其执行流程如下:
1. `StaleClosureExample`第一次加载到DOM中时,  `intervalLogCount`被调用, 这个时候该函数捕获的闭包里,
`count`等于`0`.
2. 每次我们点击`inc count`按钮, 都会触发整个组件重新渲染, 并新的`count`被**重新声明**(而非修改旧值), 其值为旧值加1.
3. 这个时候组件里的`count`, 并不是`intervalLogCount`里捕获的`count`, 这种**过时**的闭包称为`stale closure`.

由于存在`stale closure`的问题, 所以React在设计的时候要求将`deps`传入`useEffect`或`useMemo`, React会监听`deps`, 当其发生改变的时候,
对于`useEffect`, 会调用回调函数返回的清理函数, 再重新调用传入的回调函数, 对于`useMemo`, React会调用回调函数生成
新的值, 而后触发重新渲染.

## 避免使用`useEffect`监听`stateA`的变化`setState`B
示例:
```tsx

```
在`useEffect`中`setState`的本质是:

`stateB`单向, 部分依赖`steateA`, 即, 当`stateA`发生变化(或特定变化)时, `stateB`会随着变化, 
反之, `stateB`的变化不会影响`stateA`.

这便导致问题:

- 这种使用方式蔓延开来后, 项目中的某一`state`, 开发者都很难确定其状态, 也难以确定修改其状态后会影响哪些状态. 
因为该状态可能依赖其他状态, 而其他状态又依赖其他状态, 依赖链条过长后难以追踪. 反之, 该状态又有可能是其他状态
的依赖, 修改后可能会触发一系列其他状态的修改, 牵一发而动全身.

- 单向部分依赖的状况在真实的业务开发中很少见, 很多时候出现需要通过`useEffect`修改`state`是开发者逻辑设计上的不严谨. 
  

---

参考:

- [关于 stale closure 的博客](https://dmitripavlutin.com/react-hooks-stale-closures/)
