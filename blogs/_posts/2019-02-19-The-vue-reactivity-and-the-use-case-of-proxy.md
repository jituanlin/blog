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
    let dependencies = [];

    const observers = [];

    const collectObservable = executor => {
      dependencies = [];
      executor();
      observers.push({
        dependencies,
        executor
      });
    };

    const toReactive = target => {
      const copy = Object.assign({}, target);
      return Object.keys(target).reduce(
        (result, prop) =>
          Object.defineProperty(result, prop, {
            get() {
              dependencies.push(prop);
              return copy[prop];
            },
            set(val) {
              copy[prop] = val;
              observers
                .filter(({ dependencies }) => dependencies.includes(prop))
                .forEach(({ executor }) => executor());
            }
          }),
        {}
      );
    };

    const user = toReactive({ name: "Jituan" });
    const render = () => {
      document.body.textContent = `the user name is: ${user.name}`;
      console.log("render called");
    };

    collectObservable(render);

    // this will trigger the render
    setTimeout(() => (user.name = "JituanLin"), 2000);

    
    
    // this will not trigger the render, 
    // because the render not dependent the `age` property
    setTimeout(() => (user.age = 25), 2000);
``` 

The above code is a simple version of Mobx's `autorun` function or the internal way 
which Vue implement the reactivity.  

However, there is a limitation of this implementation:

```vue
    const user = toReactive({ name: "JituanLin" });

    user.age = 24;

    const render = () => {
      document.body.textContent = `the user age is: ${user.age}`;
      console.log("render called");
    };

    collectObservable(render);

    // this will not trigger the render, because the `age`
    // is not reactive
    setTimeout(() => (user.age = 25), 2000);
```

That is, register a new property directly will not be reactive.
But the `Proxy` way can handle this case:  

```js
    let dependencies = [];

    const observers = [];

    const collectObservable = executor => {
      dependencies = [];
      executor();
      observers.push({
        dependencies,
        executor
      });
    };

    const toReactive = target => {
      const copy = Object.assign({}, target);
      return new Proxy(target, {
        get(target, prop) {
          dependencies.push(prop);
          return target[prop];
        },
        set(target, prop, value) {
          target[prop] = value;
          observers
            .filter(({ dependencies }) => dependencies.includes(prop))
            .forEach(({ executor }) => executor());
        }
      });
    };

    const user = toReactive({ name: "JituanLin" });

    user.age = 24;

    const render = () => {
      document.body.textContent = `the user age is: ${user.age}`;
      console.log("render called");
    };

    collectObservable(render);

    // this will trigger the render
    setTimeout(() => (user.age = 25), 2000);
```

Because the `Proxy` provide the ability to intercept the property
`set`, `get` behavior and modify it **dynamically**, we could use it to avoid the limitation
of `defineProperty` way.

Further more, the `Proxy` has more behavior intercept ability than `defineProperty`.
Such as [function call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/apply),
[has in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/has), etc... 

It provide a strong way to extend current exist object/function without modify the
original code.

