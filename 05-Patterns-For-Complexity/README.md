## The Problem With useState()

We know that when we use the useState hook, we are getting a brand new value and a brand new function every single time. While this works in most cases, sometimes we want to do something a little bit more complicated.

**Redux** gave us the idea of having a reducer that everything goes into and new stuff comes out. We now have the **useReducer() hook,** which allows us to use a simplified version of what we might use in Redux.

**useReducer() does not provide all of the same functionality that we get with Redux.** Redux still has its use case.

## Arrays and Objects in JavaScript

Because Arrays and Objects are not primitive values, JS will treat a new array as an entirely different object, even if the contents of the object/array are exactly the same. This is why we usually have to return the contents of the old array inside of new arrays that we return:

```javascript
setGrudges([grudge, ...grudges])
```

## Reducers

**A reducer is a function that takes two arguments:**
- *The current state of the world (object or array)*
- *The action that just happened*

With those two pieces of data, a reducer figures out what the new state of the world should be.

This is the simplest possible reducer:

**Application.js**
```javascript
function reducer(state, action) {
  return state;
}

const Application = () => {
  const [grudges, dispatch] = useReducer(reducer, initialState);
```

useReducer takes two things as arguments:
- a reducer
- initial state

AND it returns:
- the current state of the world
- dispatch as the second element in the returned array

```javascript
const [grudges, dispatch]
```

dispatch allows us to send actions to the reducer:

```javascript
useEffect(() => {
    dispatch({ type: 'SOME_ACTION' });
  }, []);
```

**Our goal with useReducer() is to divorce the management of our state from the actual components that are rendering the state.**

If we are utilizing useState(), we end up directly managing state from within the components that are also responsible for rendering the state.

With useReducer, we are saying, **"we are going to give you the state from the reducer that we are managing for you and we are going to give you a dispatch function to tell us that stuff happened that we might need to adjust our state for."**

All of the state management is going to get pulled out of our application.

#### useReducer is very easy to unit test. We pass in a mock object that mirrors the state and pass in the action that we are expecting and then we see what will happen.

#### Pro-Tip: Store the action types in variables and reference them instead of re-writing a new string every time for an action type. WHY? No error will be thrown if we mistype a string. But if we mistype a variable, we will get an error..

```javascript
const GRUDGE_ADD = "GRUDGE_ADD"
const GRUDGE_FORGIVE = "GRUDGE_FORGIVE"
```

## Using Dispatch

The only thing that dispatch requires is that we pass in an object that has a type property:

```javascript
const addGrudge = grudge => {
  grudge.id = id();
  grudge.forgiven = false;
  dispatch({
    type: GRUDGE_ADD
  })
};
```

**The properties that the dispatch object recognizes:**
- *type*: necessary field
- *payload*: object that contains all the action's other data
- *meta*: rarely used
- *errors*: true/false

**type** and **payload** are used all of the time. **meta** and **errors**, not so much.

A dispatch example:

```javascript
function addGrudge({ person, reason }) {
  dispatch({
    type: GRUDGE_ADD,
    payload: {
      person,
      reason,
      forgiven: false,
      id: id()
    }
  });
}
```

The id() call is us using the uuid package to generate a unique id for use.

```javascript
import id from 'uuid/v4';
```

## Handling the Action

```javascript
function reducer(state, action) {
  if (action.type === GRUDGE_ADD) {
    return [action.payload, ...state];
  }
  return state;
}
```

