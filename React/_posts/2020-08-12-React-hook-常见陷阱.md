---
layout: post
title: React hook 常见陷阱
---

# React hook 常见陷阱

## 为什么不将`deps`传入`useState`和`useEffect`中会导致其回调函数使用陈旧的状态

该部分内容为[此博客](https://dmitripavlutin.com/react-hooks-stale-closures/) 的归纳总结.

```jsx harmony
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
2. 每次我们点击`inc count`按钮, 都会触发整个组件重新渲染, 并新的`count`被*重新声明*(而非修改旧值), 其值为旧值加1.
3. 这个时候组件里的`count`, 并不是`intervalLogCount`里捕获的`count`, 这种**过时**的闭包称为`stale closure`.


---

参考:

- [关于 stale closure 的博客](https://dmitripavlutin.com/react-hooks-stale-closures/)
