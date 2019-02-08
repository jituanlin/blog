---
layout: post
title: Passing events elegantly in React
image: /assets/img/passing-events-elegantly-in-react.jpg
---

# Passing events elegantly in React


## What is the problem
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
        payload={this.state.payload}/>
    )
  }
}
```
Just like we saw, above code using a trick that using `isNewEventFlag`  to indicate a new event was emitted.

Then, in `ChildComponent`, When each component detects a props update,
we make a strictly equal judgment to determine whether a new event has been accepted
.If true, then `doSomething`.

You may hold the opposite opinion that there is 'a simpler way' to achieve it.

Which is, using React `ref`,  we can bind a `ref` back to `ParentComponent`,
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

Disadvantages of this design is: 
>
Using the `ref`, the component `A` expose to anywhere, 
and the way it is used is not subject to any restrictions.
>
Component "A" is getting out of control, we hard to know when/who changes its state or execute its method.
>
Let's tell the truth, although the original developer split the code into several components, 
but the components did not play the role of isolation logic.

## How I resolve it

First, let us straighten out the problem:
1. A event emit/subscribe mechanism.
2. This mechanism must support emit/subscribe different component level.
3. There must be an easy way to clarify which components have emitted events and which components have subscribed to events.

For point `1`, I use **Rx.js** [Subject](https://github.com/ReactiveX/rxjs/blob/master/doc/subject.md) to emit/subscribe events.

If you think that introducing a library is too cumbersome, the minimized implementation is:
```js
export class Subject {
  constructor() {
    this.subscriptions = [];
  }
  next(value) {
    this.subscriptions.forEach(action => action(value));
  }
  subscribe(action) {
    this.subscriptions.push(action);
    return {
      unsubscribe: () => remove(this.subscriptions, action),
    };
  }
}
```
The `next` method is just corresponds **emit a event with value**.

The `subscribe` is just corresponds **bind a event handler, and return a instance with `unsubscribe` function which could be used to cancel the subscribe**
 
For point `2`, I use React [Context API](https://reactjs.org/docs/context.html) to handle it.

Using the `Context API`, we can share a `Subject instance` in a finite component tree, within which we can easily `emit / subscribe` events.

For point `3`, because we use `Context API`, we can easily find out which components `emit/subscribe` the `Subject instance` by search reference of `ContextType`.

Sample code is as follows:
```jsx
const SubjectContextType = React.createContext({
  subjectForEventA:new Subject()
})


class ChildComponent extends Component{
  static contextType=SubjectContextType
  
  componentDidMount(){
    this.subscription= this.context.subjects.subjectForEventA.subscribe(()=>{
      //do anything you want
    })
  }
  
  componentWillUnmount(){
    this.subscription.unsubscribe()
  }
  
}


class ParentComponent extends Component{
  
  constructor(props){
    super(props)
    this.subjects={
      subjectForEventA:new Subject()
    }
  }
  
  componentDidMount(){
    this.subjects.subjectForEventA.next('some value')
  }
  
  render(){
    return (
      <SubjectContextType.Provider value={this.subjects}>
        <ChildComponent/>
      </SubjectContextType.Provider>
    )
  }
}
```

## Conclusion
Using this way, the `ParentComponent` and `ChildComponent` achieve decoupling.
They can work independently whether `ParentComponent` emit events or if `ChildComponent` subscribe the events. 