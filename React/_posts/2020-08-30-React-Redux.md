---
layout: post
title: React Redux
---

# React Redux
本文为对React Redux的学习笔记, 其内容待补充完善.

## React Redux如何进行状态改变通知
React Redux 使用`Context API`进行状态集中管理.
 
对于进行`connect`的组件, Redux通过创建一个`memo`过的functional component, 在这个组件中通过`useContext`实现对全局
状态的绑定, 故若在reducer中直接对全局状态进行*mutation*, 不会触发这些connect组件重新渲染, 因为`useContext`是对
Context进行引用比较判断是否状态发生了变化.

对于*class component*, React Redux将其包裹在`Context.Provider`之下, 于此实现通知组件需要根据状态改变的改变进行重新渲染.

对于*functional component*, React Redux直接使用*useContext*实现状态改变通知.

