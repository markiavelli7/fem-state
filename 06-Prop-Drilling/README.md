# Prop Drilling

Prop drilling occurs when we have deep component trees.

## The Context API

Context provides a way to pass data through the component tree without having to pass props down manually at every level.

### createContext()

When we call createContext(), we get two components: a **Provider** and a **Consumer**.

```javascript
import React, {createContext} from "react";

var SuperCoolContext = React.createContext();

SuperCoolContext.Provider;
SuperCoolContext.Consumer;
```

Consumer and Provider are components in their own right.

```javascript
<CountContext.Provider value={0}>
  <CountContext.Consumer>
    {value => <p>{value}</p>}
  </CountContext.Consumer>
</ CountContext.Consumer>
```

We pass in the value to the **Provider** as a prop, and then any nested child in the component tree can access the value using a callback function inside of the **Consumer**.

**{value => <p>{value}</p>}** is a render prop

### Passing State Into the Context

```javascript
var CountContext = createContext()

class CountProvider extends Component {
  state = {count: 0};

  function increment() { 
    this.setState(({count}) => ({count: count + 1}))
  }

  function decrement() {
    this.setState(({count}) => ({count: count - 1}))
  }

  render() {
    var {increment, decrement} = this
    var {count} = this.state;
    var value = {count, increment, decrement}

    return (
      <CountContext.Provider value={value}>
        {this.props.children}
      </CountContext.Provider>
    )
  }
}
```

## Refactoring Grudges to Use Context

**GrudgeContext.js**

```javascript
import React, { useReducer, createContext, useCallback } from 'react';

import id from 'uuid/v4';
import initialState from './initialState';

var GrudgeContext = createContext();

const GRUDGE_FORGIVE = 'GRUDGE_FORGIVE';
const GRUDGE_ADD = 'GRUDGE_ADD';

function reducer(state, action) {
  if (action.type === GRUDGE_ADD) {
    return [action.payload, ...state];
  } else if (action.type === GRUDGE_FORGIVE) {
    return state.map(grudge => {
      if (grudge.id !== action.payload.id) return grudge;
      return { ...grudge, forgiven: !grudge.forgiven };
    });
  }
  return state;
}
export const GrudgeProvider = ({ children }) => {
  var [grudges, dispatch] = useReducer(reducer, initialState);

  var addGrudge = useCallback(
    ({ person, reason }) => {
      dispatch({
        type: GRUDGE_ADD,
        payload: {
          person,
          reason,
          forgiven: false,
          id: id()
        }
      });
    },
    [dispatch]
  );

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

  var value = {
    grudges, addGrudge, toggleForgiveness
  }

  return (
    <GrudgeContext.Provider value={value}>
      {children}
    </GrudgeContext.Provider>
  )
};
```

With the GrudgeProvider component set up, we can use it as a wrapper and any child component that needs to access the grudges state or the addGrudge or toggleForgiveness methods will be able to do that using the GrudgeProvider.

## Wrapping the Application with the Provider

**index.js**

```javascript
import {GrudgeProvider} from './GrudgeContext'

ReactDOM.render(
  <GrudgeProvider>
    <Application />
  </GrudgeProvider>,
  rootElement
)
```

Now, any component inside of the application can hook into the Context to get access to the data and methods.

If there is only a part of the application that needs access to the Context, we can just wrap that part.

We did lose all of our performance optimizations. **This is because memo() is looking into context for the prop changes between renders and it really doesn't know what changed.**

With the clean code organization of using Context, we lose some of the ability to make performance optimizations. **As well as testability.**

#### The way to handle testing is to have a TestContext that we wrap the application in.