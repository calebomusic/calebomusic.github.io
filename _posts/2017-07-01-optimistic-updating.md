---
layout: post
title: Optimistic Updating with Redux
---

This is a follow up post to [An Intro to Higher-order Reducers](https://calebomusic.github.io/2017/05/01/higher-order-reducers-redux.html). For another intro to higher-order reducers or reducer enhancers see the [Redux docs](http://redux.js.org/docs/recipes/reducers/ReusingReducerLogic.html).

One nifty technique that can be implemented with higher-order reducers is optimistic updating. What is optimistic updating?

Let's start with a problem. Suppose you are making a todo app and you want to update the title of the todo. The "backend" consists of a server side rails app, the "frontend" consists of a client side Redux app. Now suppose that the title of the todo resides in two places in the app: a todo list and a todo detail. Asana (link) is an instance of this pattern and [Shmasana](shmasana.herokuapp.com) is as well:

![Todo List / Todo Detail](https://github.com/calebomusic/Shmasana/blob/master/docs/screenshots/charles.png?raw=true)

One way to implement this: when the title of a todo changes in the todo detail, send a request to the backend updating the todo's title. Once the todo title is successfully updated, you fire off a response to the frontend, updating the todo's title in the todo detail and todo list. Unfortunately, this can cause problems.

If someone is typing at a reasonably high speed and the request-response cycle between the frontend and backend is not up to snuff or fast enough, the user may be frustrated by the following situation: They type in "t" (and so fire off the request-response cycle) and then type in "o" (an so fire off another request-response cycle) -- but as soon as they type "o" the first response is received by the frontend, updating the todo's title to "t". The user will experience typing in "to", which is transformed into "t". Disappearing letters! Now if they quickly press "o" again, the todo's title will end up being updated to "too"... This makes for slightly unpleasant UI.

One way to fix this is with optimistic updating: update the UI instantly without waiting for a successful save on the backend. Hence the optimism: update the app without validation. However, if the update was invalid one will need to return to a previous valid state.

How might one do this? With a higher-order reducer!

From a high-level, when a todo is updated we will dispatch an update todo action. This action will update the UI and also fire off an api call to the backend. The todo reducer will be wrapped by an optimistic update reducer which will keep track of when the todo list was changed, when it was successfully updated, and revert to a previous state if necessary.

In more detail: if a todo's title is changed, dispatch `UPDATE_TODO` action. As a payload, this action has a `status` (handled by `OptimisticReducer`) and the `todo` (handled by `TodoReducer`). This action (1) dispatches `receiveTodoBegin` and (2) fires off an api call to the backend.

`receiveTodoBegin` optimistically updates the UI with a todo that carries the status `BEGIN`. The api call to the backend may look like this (using callbacks):

```javascript
updateTodo(action.todo, successfulUpdate, revertOnError);
```
Or, using promises, something like this:
```javascript
updateTodo().then((todo) => successfulUpdate(todo)).catch((todo) => revertOnError(todo))
```
I won't get into the various ways of handling async calls with Redux whether one is utilizing middleware, thunk, or saga is immaterial here. Importantly: on a successful update we chain fulfillment function or pass in a success callback that changes the todo's status from `BEGIN` to `END`. On a failed update we chain a rejection function or pass in an error callback that dispatches the todo with the status `REVERT`.

A higher-order reducer will wrap around the todo reducer and handle the status of the todo:

```javascript
const OptimisticReducer = (reducer) => {
  return (oldState = { past: [], present: {}}, action) => {
    let newState = merge({}, oldState)
    switch (action.status) {
      case BEGIN:
        return {
          past: [ ...newState.past],
          present: TodoReducer(newState, action)
        };
      case END:
        newState.past.push(action);

        if (newState.past.length > 1) {
          newState.past.shift();
        }

        return newState;
      case REVERT:
        const prevAction = newState.past[newState.past.length - 1];

        if (prevAction) {
          return {
            past: [prevAction],
            present: TodoReducer(newState, prevAction)
          };
        } else {
          return newState; // Or throw an error
        }
      default:
        return oldState;
    }
  }
}
```

First, `newState` is created using lodash's merge function (remembering that state is immutable). If an action has the status of `BEGIN` we pass the action to the `TodoReducer` which updates the state with the new todo (nothing exciting going on there, it's an ordinary reducer). If the backend is successfully updated, `OptimisticReducer` will receive an action with the status of `END` and we will save the todo as `past` and delete the previous past valid todo (if there was one). Remember, this action will be executed by the success callback or fulfillment function of `updateTodo`. Finally, if the error callback or reject function is executed, an action with the status `REVERT` will be dispatched. This will replace the existing todo with the last valid todo.

That's a relatively simple way to implement optimistic updating with redux then. Here's a helpful write up on [Undo History with Redux](http://redux.js.org/docs/recipes/ImplementingUndoHistory.html) from really quite nice docs.
