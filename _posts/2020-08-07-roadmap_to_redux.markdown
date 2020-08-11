---
layout: post
title:      "Roadmap to Redux"
date:       2020-08-07 11:34:29 -0400
permalink:  roadmap_to_redux
---



The goal of Redux is to allow components access to values from a global state, reducing prop drilling as well as confusing and complex passing down of values from parents, to children, to grandchildren components. Jumping into the middle of Redux can be confusing, and although it has a several recipes that can simply be followed, I find I can understand these smaller pieces of Redux once I have a big-picture view of how they all work together. So with that, let's go on a little road trip.

![](https://i.imgur.com/wzLtSHf.gif)

## 1. The Goal of Using Redux is to Update and Access Global State

We utilize Redux to trigger an action, that goes to a reducer, resulting in updated state. This is the essential 3-part process of Redux. It has a few more moving parts, but this is the gist of its existence. So let's break these 3 pieces down for a moment.

### Trigger an Action

A Javascript object called an action to change the state. This will always be composed of 2 parts:

1. The action type
2. The payload (data with which you’re applying the change)


```
action = {
  type: 'ADD_INTEREST',
  newInterest: {
    name: 'hockey',
    type: 'sport'
  }
}
```

### Use a Reducer to Reduce Current State with the Action's Payload
We then call a javascript function (a "reducer," a pure function) to process the action to a state

```
function changeState(state, action){      
  switch (action.type) {
    case 'INCREASE_COUNT':
      return {count: state.count + 1}
    case 'DECREASE_COUNT':
      return {count: state.count - 1}
    default:
      return state;
  }
}
```


### Use a "Dispatch" Function to Actually Change the State

We need a way to persist this info to the state (right now the reducer just simply returns a value), so we use a dispatch function:

```
function dispatch(action){
  state = changeState(state, action)
  return state
}

```

The dispatch incorporates both the action and the reducer.
To restate, Redux's 3-part process triggers an action, which goes to a reducer along with current state, resulting in dispatchng this new redution as the new state.


1. Write the reducer
2. Use the reducer’s action language to write the action
3. Write the dispatch wrapping it around the reducer and taking in the argument as an action

## Make the State Global
These examples have assumed that all of this code is in a single js file, and therefore the "state" being updated is actually local. Redux is meant to make a global state accessible, so now we need to look at setting up a global state.

Redux has a function called `createStore()`, that:
1. maintains a global state
2. reduces that state with an action
3. dispatches that reduction to a new state
4. stores the new state
5. allows access to global state with a getter method getState()

4 of the 5 of these are contained within the `createStore()` function. The only thing mentioned above that is external to the function is a the reducer it uses, which it takes as an argument. So the function can be illustrated like this:

 `createStore(reducer)` 
 
The  `createStore()` method is generic. It always returns a store (given a reducer) that will have a dispatch method and a getState method. The reducers will be unique to a program and therefore written separately.

## Access Global State

To access this global state created by createStore(), we add one more function from react-redux to the component (ideally a container) that needs access to global state:

`import { connect } from 'react-redux';`

In short, this connects `createStore()` to the component. Now that we're "connected" to the store, we need to actually access the store of data. We then utilize 2 functions within that component.

### Map State to Props

The first is `mapStateToProps `. This allows us to pass what data we want from the store into the component as props.

```
const mapStateToProps = state => {
  return {
    items: state.items
  };
};

```

Simple enough. Now if we called `{this.props.items.}` within our component, we'd have access to the values in the store's "items". If you had 12 different key/value pairs in your store, and only needed 2 in a particular component, you only list out what you need in the `mapStateToProps()` function.


### Map Dispatch to Props 

This is the part in our little roadtrip where the things might come undone.

![](https://i.imgur.com/URjbOYR.gif)

But no worries, we'll channel the indefatigable optimism of Clark Griswold and get through.

The component accessing global state through Redux will utilize one more function provided by redux: `mapDispatchToProps()`.

A boilerplate for this function looks like: 

```
const mapDispatchToProps = dispatch => {
  return {
    increaseCount: () => dispatch({ type: 'INCREASE_COUNT' })
  };
};
```

This is where I started having issues and taking a step back helps tremendously. Let's review:

The Redux 3-part process takes an action, reduces with the old state, and dispatches as the new state.

We have a Redux `createStore()` function that has an action and a dispatch function inside of it, but calls a reducer as an argument from without.

Good so far? Here's where I got derailed. Look back at our template *reducer* code:

```
function changeState(state, action){      
  switch (action.type) {
    case 'INCREASE_COUNT':
      return {count: state.count + 1}
    case 'DECREASE_COUNT':
      return {count: state.count - 1}
    default:
      return state;
  }
}
```

The reducer needs an action to return a new, reduced state. But that action is inside of the `createStore()` function, specifically, the argument of the `dispatch()` function...and `createStore()` calls the reducer as an argument, in order to execute its dispatch function:

```
function dispatch(action){
  state = changeState(state, action)
  return state
}

```
So we have bit of a chicken-and-egg dilemma here. The reducer needs a state and action to process. The action would be passed down from the dispatch function to the reducer as an argument. But since the reducer is now separate from Redux' `createStore()`, we need to pass the dispatch to a component (here, the App component) as props. Thus, the redux function `mapDispatchToProps()` comes into play in the app component:


```
const mapDispatchToProps = dispatch => {
  return {
    actionType: () => dispatch({ type: '<write your action type here>' })
  };
};

```

There is one other VERY important reason we need to pass the `dispatch()` function as props to the component. The vanilla `ispatch()` function does not come with any actions defined. It simply passes them in as an argument. So by funneling the dispatch to a component as props, it gives us the chance to assign the dispatch an action, before executing it (and consequently executing the reducer). One more time, the example of the mapDispatchToProps method *IN* the App component looks like this, being passed an action type of your choosing:

```
const mapDispatchToProps = dispatch => {
  return {
    increaseCount: () => dispatch({ type: 'INCREASE_COUNT' })
  };
};

```


## Putting It All Together

This function-as-props is then is called within the App component (here, using an event as the trigger):

`OnClick = () => { this.props.increaseCount() }`

The OnClick event then calls the props increaseCount, which in mapDispatchToProps assigned the dispatch funtion with the action.type  INCREASE_COUNT.

Dispatch then executes with this action, calling the reducer and passing the action as an argument to the reducer. The reducer executes, which returns a new state to dispatch function, which returns the state to the store in  `createStore()`:

```

function dispatch(action){
  state = changeState(state, action)
  return state
}
```

`dispatch()` knows to call the `changeState()` reducer because we passed the reducer into the  `createStore()` in the index.js file. This works now with only 1 reducer in your code, but i will have to be modified as other reducers are built.

If this makes sense, then you've made it to then end of our road trip and whatever the Redux version of WallyWorld is.

![](https://i.imgur.com/xXqSNQj.gif)
