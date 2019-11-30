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

**dispatch() is a lot like useRef() in the respect that it references the same reference function every time.** The dispatch function has the same memory reference on every render (it is the same one). So what we want to do is tell our function, "if all of your props are the same as they were on the last render, don't rerender.."

### React has a few different ways to handle this:

- **.memo()** - takes a function component and returns us a new one that is watching the props to see if the same one is returned. If it is, the re-render is skipped. We wrap our function in memo()

- **useCallback** - returns a new function that we can call. It gives us a function rather than the result.

- **useMemo** - useMemo will call the function once and if the dependencies haven't changed, it won't call the function again.

### React.memo()

```javascript
React.memo(function NewGrudge({onSubmit}) {
  ...
})
```

A caution with .memo() is not to wrap everything with it. We are still asking memo() to check the props, so we are still putting burden on our application.

When we try to use *memo()*, we can run into a pitfall, here we are wrapping *memo()* around our Grudge component:

```javascript
const Grudge = memo(({ grudge, onForgive }) => {
  console.log(`Rendering grudge: ${grudge.id}`);

  const forgive = () => onForgive(grudge.id);

  return (
    <article className="Grudge">
      <h3>{grudge.person}</h3>
      <p>{grudge.reason}</p>
      <div className="Grudge-controls">
        <label className="Grudge-forgiven">
          <input type="checkbox" checked={grudge.forgiven} onChange={forgive} />{' '}
          Forgiven
        </label>
      </div>
    </article>
  );
});
```

There is a problem though. In the parent component, we are defining a new function for addGrudge and toggleForgiveness every single render:

```javascript
const Application = () => {
  var [grudges, dispatch] = useReducer(reducer, initialState);
  console.log(grudges);

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

  const toggleForgiveness = id => {
    dispatch({
      type: GRUDGE_FORGIVE,
      payload: {
        id
      }
    });
  };

  return (
    <div className="Application">
      <NewGrudge onSubmit={addGrudge} />
      <Grudges grudges={grudges} onForgive={toggleForgiveness} />
    </div>
  );
};
```

So it is a **different addGrudge being passed to <NewGrudge/> on every render of the application and a different toggleForgiveness being passed to each individual <Grudge/>.** It is a brand new function that is doing exactly the same thing.

### Enter useCallback()

If useCallback() gets the same dependencies, it will return the same function reference.

```javascript
var addGrudge = useCallback(({ person, reason }) => {
    dispatch({
      type: GRUDGE_ADD,
      payload: {
        person,
        reason,
        forgiven: false,
        id: id()
      }
    });
  }, [dispatch])

var toggleForgiveness = useCallback(
    id => {
      dispatch({
        type: GRUDGE_FORGIVE,
        payload: {
          id
        }
      });
    },
    [dispatch]
  );
```

useCallback is using [dispatch] as a dependency. So useCallback is looking to see if *dispatch* changed. If hasn't changed for a particular component, useCallback isn't going to call it.

### So why couldn't we do this before we were using useReducer()?

The previous code for addGrudge and toggleForgiveness relied on useState() and was using the **grudges** array:

```javascript
var addGrudge = grudge => {
  grudge.id = id()
  grudge.forgiven = false;
  setGrudges([grudge, ...grudges]);
};

var toggleForgiveness = id => {
  setGrudges(
    grudges.map(grudge => {
      if (grudge.id !== id) return grudge;
      return {...grudge, forgiven: !grudge.forgiven};
    })
  )
}
```

On every render, **grudges** was a new array that was changing. This would have invalidated the dependencies of useCallback(). It would have depended on **grudges** and **setGrudges**, and any change to the grudges array would have triggered a re-render.

**Our dependencies (grudges and setGrudges) would have changed on every render, setting off useCallback()**