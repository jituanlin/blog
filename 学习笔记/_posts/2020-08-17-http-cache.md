---
layout: post
title: HTTP cache
tags:       [网络]
---

# HTTP cache
本文为[该文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching) 的学习笔记.

思维导图如下:
![思维导图](https://github.com/jituanlin/public-docs/blob/master/public-mindmaps/HTTP%20cache.png?raw=true)

no-cache和no-store的区别:
- no-cache: 每次复用cache前向服务器发送请求校验资源的有效性.
- no-store: 禁止缓存资源.

图示:
![no cache和no store的区别](https://i.stack.imgur.com/OpJIC.png)

--- 
参考:
- [思维导图源文件](https://github.com/jituanlin/public-docs/blob/master/public-mindmaps/HTTP%20cache.xmind)
- [no-cache vs no-store](https://stackoverflow.com/questions/7573354/what-is-the-difference-between-no-cache-and-no-store-in-cache-control#:~:text=As%20far%20as%20I%20know,it%20first%20with%20the%20source.)