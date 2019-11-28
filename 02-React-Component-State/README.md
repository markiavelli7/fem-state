# React State Management

## The Classic Class-Based Counter

```javascript
class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
    };

    this.increment = this.increment.bind(this);
    this.decrement = this.decrement.bind(this);
    this.reset = this.reset.bind(this);
  }

  increment() {
    this.setState({ count: this.state.count + 1 });
  }

  decrement() {
    this.setState({ count: this.state.count - 1 });
  }

  reset() {
    this.setState({ count: 0 });
  }

  render() {
    return (
      <div className="Counter">
        <p className="count">{this.state.count}</p>
        <section className="controls">
          <button onClick={this.increment}>Increment</button>
          <button onClick={this.decrement}>Decrement</button>
          <button onClick={this.reset}>Reset</button>
        </section>
      </div>
    );
  }
}
```

**this.setState() is asynchronous**

React is trying to avoid unnecessay re-renders.

So what will get from the increment method if we do something like this?

```javascript
increment() {
  this.setState({ count: this.state.count + 1 });
  this.setState({ count: this.state.count + 1 });
  this.setState({ count: this.state.count + 1 });
}

render() {...}
```

What will the count be after the user clicks the "Increment" button?

**1**

Effectively, we are queuing up state changes. **We told React, "we want you to set the state to the current number plus one."** And we basically told React that 3 times. What React heard is, **"set count to 0 + 1"**. Three times.

React will batch the calls, figure out the result and then efficiently make that change.

```javascript
Object.assign({},
  ourFirstCallToSetState,
  ourSecondCallToSetState,
  ourThirdCallToSetState
);
```

It merges all of the keys and if there are duplicate keys, the last key that was added will win.

## Using Functions as an Argument to setState

```javascript
increment() {
  this.setState((state) => {return {count: state.count + 1}});
  this.setState((state) => {return {count: state.count + 1}});
  this.setState((state) => {return {count: state.count + 1}});
}

render() {...}
```

What will *count* be incremented to this time?

**3**

This time, React is not batching. Functions can't be merged, so we sidestep the issue of our objects being merged.

#### React will play through each of the functions one-by-one.

Because we are working with functions, we can implement logic inside of them:

```javascript
increment() {
  this.setState(state => {
    if (state.count >= 10) return;
    return { count: state.count + 1 };
  });
}
```

We can also pass in props:

**index.js**

```javascript
<Counter max={15} step={5} />
```

**counter.js**

```javascript
increment() {
  var { max, step } = this.props;
  this.setState(state => {
    if (state.count >= max) return;
    return { count: state.count + step };
  });
}
```

While what we've done above works, the problem comes in when we want to unit test, we've got state and props kind of scattered through our files.

#### To help us with this, on top of just getting state, the function inside of this.setState also gets props:

```javascript
increment() {
  this.setState((state, { max, step }) => {
    if (state.count >= max) return;
    return { count: state.count + step };
  });
}
```

A careful eye will note that the function that we are passing into this.setState is a function just like any other function. We can pull it out and pass it in to *this.setState*.

```javascript
function incrementCounter(state, {max, step}) {
  if (state.count >= max) return;
  return { count: state.count + step };
}

increment() {
  this.setState(this.incrementCounter);
}
```

Extracting out our function gives us better reuse options and also better testing options. We can test this function individually without mounting extra components, etc.

## setState()'s Second Argument

So we know that for the first argument, setState can take either an object or a function. Additionally, setState can take a function as a second argument. **This function is ran after the state is updated.**

```javascript
increment() {
  this.setState((state, props) => {
    var { max, counter } = props
    if (state.count <= 15) return;
    return {count: state.count + step}
  }, () => {
    console.log(this.state)
  })
}
```

#### This second function is a callback function that will be called after the state has been updated.

A good use case for this callback is for when we want to show something somewhere else on the page when the state changes.

### Storing State in Local Storage

**This is an example, not to be used in production**

```javascript
function getStateFromLocalStorage() {
  var storage = localStorage.getItem('counterState')
  if (storage) return JSON.parse(storage)
  return { count: 0}
}

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = getStateFromLocalStorage();

    increment() {
      this.setState(this.incrementCounter, () => {
        localStorage.setItem('counterState', JSON.stringify(this.state))
      });
    }
```

The way that local storage works is that at a given key, we can store a string. We want to store an object, so we are going to need to JSON.parse it when it comes out.

Inside of the setState() callback function, we are setting our local storage "counterState" key with the current count.

**An important thing to note is that the setState callback function is does not receive state and props passed in like the first function does.**

If we wanted to use the state and props, we could create a function outside of the Counter Component and bind that to the class:

```javascript
function storeStateInLocalStorage() {
  localStorage.setItem('counterState', JSON.stringify(this.state))
}
```

Binding our function to the class:

```javascript
  increment() {
    this.setState(this.incrementCounter, () => {
      storeStateInLocalStorage.bind(this);
    });
  }
```

The other option that we have is to make storeStateInLocalStorage a method on our Counter class.