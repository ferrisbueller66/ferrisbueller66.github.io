---
layout: post
title:      "Roadmap to Redux"
date:       2020-08-07 11:34:29 -0400
permalink:  roadmap_to_redux
---



The goal of Redux is to allow components access to values from a global state, reducing prop drilling and confusing and complex passing down of values from parents, to children, to grandchildren components. Jumping into the middle of Redux can be confusing, and although it has a several recipes that can simply be followed, I find I can understand these smaller pieces of Redux once I have a big-picture view of how they all work together. So with that, let's go on a little road trip.

![](https://i.imgur.com/wzLtSHf.gif)

## 1. The Goal of Using Redux is to Update and Access Global State

Redux updates we trigger an action, that goes to a reducer, resulting in updated state. This is the essential 3-part process of Redux. It has a few more moving parts, but this is the gist of its existence. So let's break these 3 pieces down for a moment.

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
A javascript function (a reducer, a pure function) to process the action to a state

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

Then need a way to persist this info to the state (right now the reducer just simply returns a value), so use a dispatch function:

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

## Access Global State

Redux has a function called `createStore()`, that:
1. maintains a global state
2. reduces that state with an action
3. dispatches that reduction to a new state
4. stores the new state
5. allows access to global state with a getter method getState()

4 of the 5 of these arecontained within the `createStore()` function. The only thing mentioned above that is external to the function is a the reducer it uses, which it takes as an argument. So the function can be illustrated like this:

 `createStore(reducer)` 
 
The  `createStore()` method is generic. It always returns a store (given a reducer) that will have a dispatch method and a getState method. The reducers will be unique to a program and therefore written separately.



### Map State to Props

### Map Dispatch to Props 
