---
layout: post
title: The Vue Reactivity and the use case of Proxy
---

# The Vue Reactivity and the use case of Proxy

The [Vue official document](https://vuejs.org/v2/guide/reactivity.html) explains the principle of responsiveness in detail.
However, in this blog, we will focus on the implementation of Vue's way.
After that, we will explore the new way(`Proxy`) to achieve the same effect with less [`caveats`](https://vuejs.org/v2/guide/reactivity.html#Change-Detection-Caveats).
Finally, we will see the real world `Proxy  ` use case.

## The Vue Reactivity
The core of Vue Reactivity is: 
> Bind some hooks when the target's property change or get.
 
In the Vue world, there are two key concepts:
1. the `observer`, such as `template`
2. the `observable`, such as `object`

Consider following code:
```jsx 
<p> the user name is: {{user.name}}</p>
```

In the above code, the observer is the `template` with the content which refer the property `name` of `user` `observable`.
When the `user.name` change, the template will rerender so the changed content will be showed.

Let us express the `template` in the `regular JS way`:
```jsx
  const template = () => document.body.textContent = `the user name is: ${user.name}`;
```
And then add some `magic` let the `template` function will be call again when the `user.name` changed.
The complete code is as follows: 
```js
  const observers = [];

  const collectObservable = (observer) => {
    observer();
    observers.push(observer);
  };

  const toReactive = (target) => {
    const copy = Object.assign({}, target);
    return Object.keys(target).reduce(
      (result, prop) => Object.defineProperty(result, prop, {
        get() {
          return copy[prop];
        },
        set(val) {
          copy[prop] = val;
          observers.forEach(observer => observer());
        }
      }),
      {}
    );
  };

  const user = toReactive({name: 'Jituan'});
  const template = () => document.body.textContent = `the user name is: ${user.name}`;

  collectObservable(
    template
  );

  setTimeout(
    () => user.name = 'JituanLin',
    2000
  );
``` 
