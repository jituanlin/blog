---
layout: post
title: React Redux
---

# React Redux
本文为对React Redux的学习笔记, 其内容待补充完善.

## React Redux如何进行状态改变通知
React Redux 使用`Context API`进行状态集中管理.\

对于*class component*, React Redux将其包裹在`Context.Provider`之下, 于此实现通知组件需要根据状态改变的改变进行重新渲染.

对于*functional component*, React Redux直接使用*useContext*实现状态改变通知.
