---
layout: post
title: Passing events elegantly in React
image: /assets/img/passing-events-elegantly-in-react.jpg
---

# Passing events elegantly in React


## what is the problem
Consider the following situation, in the React, sometimes we need to notify the children component to perform certain operations.
The focus is:
1. Children components should always perform actions when they are notified.
2. As the parent component of the event generator, and as a child component of the event consumer.

It's just a event emit/subscribe problem.
However, in React, we can naturally pass the **value** instead of passing the event.

Fortunately, we can use the **value** delivery mechanism to simulate event delivery.
Consider the following code:
```jsx
class ChildComponent extends Component{
  static getDerivedStateFromProps({isNewEventFlag,payload}){
    /**
     * When isNewEventFlag is not strictly equal,
     * mean the new event accepted.
     */
    if (isNewEventFlag!==this.props.isNewEventFlag){
      this.doSomething(payload)
    }
  }
}


class ParentComponent extends Component{
  constructor(props){
    super(props)
    this.state={
      isNewEventFlag:{},
      payload:null
    }
  }
  emitEvent(payload){
    this.setState({
      payload,
      /**
       * Because we create a new empty object reference every time,
       * when emitEvent called,
       * the ChildComponent getDerivedStateFromProps will be called,
       * and strictly equal will not be established, and
       * the ChildComponent will know should be call doSomething function.
       */
      isNewEventFlag: {}
    })
  }
  render(){
    return (
      <ChildComponent
        isNewEventFlag={this.state.isNewEventFlag}
        payload={this.state.payload}>
    )
  }
}
```
Just like we saw, above code using a trick that using `isNewEventFlag`  to indicate a new event was emitted.

Then, in `ChildComponent`, When each component detects a props update,
we make a strictly equal judgment to determine whether a new event has been accepted
.If true, then `doSomething`.

You may hold the opposite opinion that there is 'a simpler way' to achieve it.

Which is, using React `ref`,  we could bind a `ref` back to `ParentComponent`,
and call `ref.doSomething` whenever you want.

However, this is **anti-pattern**, and this is a typical example of the abuse of `ref`.

According the react official document, we should only use `refs` in following situation:
>
- Managing focus, text selection, or media playback.
- Triggering imperative animations.
- Integrating with third-party DOM libraries.

In our example, using `ref` broke the best practice:
>
Parent component should only pass props to children component,and
children component should only react to parent component by props.

This is not a pedantic over-design, but a lesson from my personal experience.

In the above example, the complexity of the code is kept to a certain extent 
because the example is relatively simple.

But in real world, bad design will pollution other modules.
Imagine the following situation:

The common component **A** expose `ref`  to upper level component `B`.

The upper level component did not use it.

But continue to pass to the root component `C`.

And `C` pass it to `n` level component `D`.

Finally, `D` component use it such as `ref.doSomething`.