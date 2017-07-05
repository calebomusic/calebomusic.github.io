---
layout: post
title: An Intro to Higher-Order Reducers
---

Higher-order reducers are really cool.

They can be useful for keeping track your apps history, including redo or undo functionality, or logging. Moreover, understanding them is useful for functional programming in general and understanding Redux and reducers.

I'll assume you're slightly familiar with Redux, but not too familiar. On that note,  what is a **reducer**? Open up your Redux Bible to this [page](http://redux.js.org/docs/basics/Reducers.html) and we find that:

> The reducer is a pure function that takes the previous state and an action, and returns the next state.

**Pure functions** are functions that have no side-effects and always return the same output for the same input. For example consider the simple mathematical function, `addTwo`:

```javascript
function addTwo(n) {
    return n + 2;
}
```
This is a pure function, it is not making a difference to the outside environment in any way (no side-effects) and will return the same output for the same input. `2` will always return `4`.

Ok, so reducers take in an action and the previous state and return the next state -- that's it. Remembering our Redux, **actions** represent data sent to your apps store. For instance suppose we are making a simple calculator app. We dispatch a simple payload to the store in an addition action as follows:

```javascript
const ADD = "ADD"
const addTwo = (value) => (
  {
    type: ADD_TWO,
    value
  }  
)
```

This action simply carries the action type `ADD` and the value (using ES6 destructuring), which we could use to add two to the previous state.

The reducer could handle this action as follows:

```javascript
function valueReducer(state = initialstate, action) {
  switch(action.type) {
    case ADD:
      return Object.assign({}, {
        value: state.value + action.value
      });
    default:
      return state;
  }
}
```
`valueReducer` checks the action type, returning a state which includes the sum of the previous state and the value of the payload when the action is `ADD`. We're careful to return the initial state by default and return a new object with `Object.assign`.

That was a quick refresher on reducers and actions. What are higher-order reducers? Think of **higher-order reducers*** as reducers that wrap around other reducers or as reducers that return reducers. Suppose for our basic calculator app, we want to store a history, that is, we want to record every previous value. There are a number of ways this could be done and using higher-order reducers is one of those ways.

[picture]

Currently the state of our calculator app is very simple:
```javascript
{
  value: NUMBER
}
```
To keep track of history, let's change it to:
```javascript
{
  value:
    {
      current: NUMBER,
      history: [ ... ]
    }
}
```
Our previous root reducer probably looked like this:
```javascript
const RootReducer = combineReducers(
  {
    value: valueReducer,
  }
);
```

(`combineReducers` conveniently takes in reducers and combines them into one reducer which can be passed to the store.)

With our new `historyReducer`, our new root reducer will look something  like this:
```javascript
const RootReducer = combineReducers(
  {
    value: historyReducer(valueReducer),
  }
);
```
As you can see, `historyReducer` wraps around `valueReducer`. It stores the values of past application states in a history array. How so? An action like `ADD` will be dispatched and that action will be handled by the history reducer which will store the current value in the history array and then pass the `ADD` action with its payload to the `valueReducer`.
 In code:
```javascript
function historyReducer(state = initialstate, action) {
  switch (action.type) {
    case ADD:
      return {
        value: {
          history:  state.history.push(action.value),
          value: valueReducer(state, action)
        }
      };
    default:
      return state;
  }
}
```
To account for the above we will now have to change the `valueReducer`:
```javascript
function valueReducer(state = 0, action) {
  switch (action.type) {
    case ADD:
      return state.value + action.value;
    default:
      return state;
  }
}
```
Suppose we dispatch ADD twice with the payload of 1 and then with the payload of 3.

Application state initializes as follows:
```javascript
{
  value:
    {
      current: 0,
      history: []
    }
}
```

After dispatching `ADD` with a payload of 1 we have:
```javascript
{
  value:
    {
      current: 1,
      history: [0]
    }
}
```
... And with a payload of 3:
```javascript
{
  value:
    {
      current: 4,
      history: [0, 3]
    }
}
```
Super cool! That's a very simple implementation of a higher-order reducer. More useful uses of the idea concern ideas like [undo](http://redux.js.org/docs/recipes/ImplementingUndoHistory.html) or [optimistic updating](https://stackoverflow.com/questions/33009657/what-is-optimistic-updates-in-front-end-development). To do this, think of other actions that can be used like `BEGIN`, `END`, or `UNDO`...
