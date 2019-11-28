# Hooks

```javascript
var [count, setCount] = React.useState(0);

var increment = () => setCount(count + 1);
var decrement = () => setCount(count - 1);
var reset = () => setCount(0);
```

When we call *useState()*, we pass in a default value as an argument, in this case 0.

When we call useState, we are returned an array with 2 item. The first item holds the value that we want to keep track of. The second holds a function that we are going to use for updating the value.

## Similarities and Differences Between setState()

### Handling multiple update calls in the same function:

```javascript
function incrementCount() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
}
```

Very similar to the asynchronous nature of the setState() call, if our count value was 0,React sees this as three calls to 0 + 1, so it will only execute the call once.

### Using functions inside of the state setter function:

```javascript
function incrementCount() {
  setCount(count => count + 1)
}
```

**In contrast to Class's setState, when we use Function's useState method, the setter function takes only one argument: it's associated value.**

### Returning From the Setter Function

```javascript
function incrementCount() {
  setCount(count => {
    if (count >= 10) return
    return count + 1
  })
}
```
This code causes a problem. As soon as we hit the max threshold of 10 and just try to return, our counter is going to show nothing on the screen. The reason for this is that we are returning nothing. Here is how we would fix that:

```javascript
function incrementCount() {
  setCount(count => {
    if (count >= 10) return count
    return count + 1
  })
}
```

Our fix is to return the current count to the user.

**this.setState was keeping track of state stored in an object, and here we are only keeping track of a primitive value. When we return nothing, we are setting that value to undefined.**

## No Callback Available In the Setter Function

Even though there is no callback available, React fills in that void by providing us another hook called useEffect().

```javascript
useEffect(() => {
  document.title = `Counter: ${count}`
}, [])
```

**useEffect()** takes a callback as the first argument, and a dependencies array as the second callback.

#### If we don't give useEffect() any dependencies, it will run on every single render. And if sets any amount of state, it will trigger a render, putting us into an endless loop.

Giving useEffect() an empty array is telling it, "Only use this effect when something that I care about changes."

```javascript
useEffect(() => {
  document.title = `Counter: ${count}`
}, [count])
```

In the above code, we are saying, "If count changes, run this effect."

## Writing a Custom Hook

Hooks are Just Functions

```javascript
function useLocalStorage(initialState, key) {
  function getItem() {
    var storage = localStorage.getItem(key);
    if (storage) return JSON.parse(storage).value;
    return initialState;
  }

  var [value, setValue] = useState(getItem());

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify({ value }))
  }, [value])

  return [value, setValue];
}
```

## What About ComponentWillUnmount(), How Does that Work with useEffect()?

Anytime that we set up something that persists (a websocket, setInterval), we want to be able to clean it up when we don't need it anymore.

**useEffect() gives us an API to handle this.**

Inside of useEffect, after our function that we pass in, if we return an object, useEffect will run that as well.

```javascript
useEffect(() => {
  setInterval(() => {
    console.log(`Count: ${count}`)
  }, 3000)
}, [count])
```

In this scenario, when we increment the count, all of the existing console.log() statements are still active, we are just adding new ones to the queue. What we want to do is clean up the statement after we are done using it.

Here is how we could clean up the previous set interval call:

```javascript
useEffect(() => {
  var id = setInterval(() => {
    console.log(`Count: ${count}`)
  }, 3000)
  return () => clearInterval(id);
}, [count]);
```

## Important Differences Between useState and Object State

In a class based component, at the end of the day the class returns an object with methods, etc. In JavaScript, we pass objects by reference and primitives by value.

When our state is stored on an object, we get the same object state back every time. **This is not the case with functions.**

With functions, when we use useState, we are getting a copy back, but we are running a new instance.

## So what do we do if we want to persist data between renders?

**Enter useRef()**

Ref gives us an object that looks like this:

```javascript
{
  current: null
}
```
Because it is an object and we are assigning to the current property on the object, the same object is returned (giving us persistence and the ability to read the value of current in between renders)


```javascript
function  Counter() {
  var [count, setCount] = useState(0);
  var countRef = useRef();

  var message = ""
  if (countRef.current < count) message = "Higher"
  if (countRef.current > count) message = "Lower"
  countRef.current = count;
}
```

**The important thing to note is that we are running our code that checks the value of the countRef.current before we call update its value. If we called our value checker code after the new countRef assignment, they would refer to the same thing.**

By using useRef we are persisting our data between the different function copies.

The current property of useRef persists across renders until **we** change it.

