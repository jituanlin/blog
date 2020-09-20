---
layout: post
title: React vs Vue(2.x)
tags:       [React]
---

# React vs Vue(2.x)

## HOC/Hook vs mixin
逻辑复用的单位和方式不同, Vue(特指版本2.x, 下不赘述)使用*mixin*作为逻辑复用的单位和手段, 而React使用*高阶组件*(HOC)和*hook*
作为逻辑复用的单位和手段.

从设计上, hook优于HOC, HOC优于mixin.

### mixin的弊端

#### 隐式依赖
mixin使得各个逻辑单位之间存在隐式依赖(没有TS进行约束的情况下), 
如, 一个mixin可能依赖组件的某个方法, 或反过来, 组件自身可能依赖
某个mixin的某个方法, 或者mixin之间互相依赖. 

这种依赖是隐式的, 破坏了组件的封装性, 组件的方法和属性暴露给mixin
使用, 使得组件的实现依赖于mixin. 

#### 命名冲突
没有`symbol`的情况下, mixin容易导致命名冲突.

#### mixin之间相互耦合
mixin之间的互相依赖使得的mixin的实现高度耦合, 后续难以修改扩展.

#### mixin的真正问题
上述mixin带来的弊端都有先决条件: mixin与其他mixin或组件本身的互相依赖.
如果在开发mixin的时候, 使用其他方案处理"mixin与其他mixin或组件本身的互相依赖"的场景(比如直接复制粘贴),
则可以避免这种问题.

### HOC解决的问题
HOC使用*叠加装饰*代替mixin的*动态依赖*, 将mixin的图型依赖关系转化为为线性依赖关系.

### hook解决的问题
hook将HOC的线性依赖转换为树形的聚合依赖.

一个组件需要什么功能, 就注入什么hook, 一个hook如果依赖其他功能, 便在这个hook内部注入其他的hook.

这种树形的聚合依赖, 完全匹配了功能组合的心智模型, 理解起来更加直观, 易于维护.


## 虚拟DOM vs 依赖更新追踪
Vue可以做到精确更新, 只有当组件的state或者props发生变化的时候, 组件才会被render和触发component update.
而React中, 哪怕组件本身没有任何state和props, 只要该组件在其他组件的render函数中被调用, 都会触发组件的
render和update, 参考[实例代码证明](https://github.com/jituanlin/cookbook/blob/master/react-stack/src/pages/whether-react-update-if-props-nochanged/index.tsx).  

这使得React的性能优化非常依赖pure component或React.memo.

而对React来说, 为了做到精确更新, 其内部用到了属性拦截, 这使得在一开始进行template渲染的时候需要进行
依赖收集, 降低了初始渲染的速度.


---
参考:
- [React官方博客: mixin的弊端](https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html#:~:text=To%20ease%20the%20initial%20adoption,the%20same%20problem%20with%20composition.)
