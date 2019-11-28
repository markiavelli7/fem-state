# React Patterns and Anti-Patterns

When we're working with *props*, we have *PropTypes*. That is not the case with state.

## Should I Keep Something In React Component State?

### Can I Calculate It From Props?

Don't duplicate data from props in state. Calculate what we can in the *render()* method.

**Don't Use Component State**

### Am I Using It In the Render Method?

Don't keep something in the state if we don't use it for rendering. For example, API subscriptions are better off as custom private fields or variables in external modules.

**Don't Use Component State**

## Don't Use State For Derivation of Props

```javascript
class User extends Component {
  constructor(props) {
    super(props);
    this.state = {
      fullName: props.firstName + ' ' + props.lastName
    }
  }
}
```

The problem here is that since this is in the constructor, it won't get recalculated. If we make some modifications to the name in the class body, they aren't going to show up, because we are overriding them in the constructor.

## Derive Computed Properties Directly From the Props Themselves

```javascript
class User extends Component {
  render() {
    const {firstName, lastName} = this.props;
    const fullName = firstName + ' ' + lastName;
    return (
      <h1>{fullName}</h1>
    )
  }
}
```

### Alternatively, we can make our own method inside of the class

```javascript
class User extends Component {
  get fullName() {
    var {firstName, lastName} = this.props;
    return firstName + ' ' + lastName;
  }

  render() {
    return (
      <h1>{this.fullName}</h1>
    )
  }
}
```

The **get** keyword lets us call the method like a property without the parentheses. **this.fullName** instead of *this.fullName()*

## Don't Use State For Things That We Aren't Going to Render

