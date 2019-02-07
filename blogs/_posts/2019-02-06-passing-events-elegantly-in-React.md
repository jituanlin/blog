---
layout: post
title: Passing events elegantly in React
image: /assets/img/passing-events-elegantly-in-react.jpg
---

## what is the problem
Consider the following situation, in react, sometimes, we need notify child component to execute some action.
The point is:
1. Child component should always execute action whenever accept notification.
2. The parent component as the notification producer, and the child component as the notification consumer.

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