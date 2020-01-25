---
title: React hooks - Explained
date: '2020-01-25'
spoiler: Clarifying React hooks purpose.
cta: 'react'
---

You might be new to hooks or you just want to sharp your hooks skills, either way in this article, I assume you’ve used React class components and know about React’s state and life cycles.



### Hooks?

Hooks are functions, the rules for using hooks are:
1. Don’t use them inside loops, conditions, or nested functions
2. Use hooks only inside function components

### useState

Let’s start off with the most common issues hooks are here to solve - state management.
Until hooks arrived, we had no way to save state inside a *function component* so we had to wrap it with a class component -> create a state in the class component -> inject the state via props.

Today we can do it with useState:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Add 1
      </button>
      {count}
    </div>
  );
}
```

useState is a function that receives initialValue as an argument and returns an array with two items: the first is the actual state value and the second is the state’s setter function.


### useEffect

Another major problem using function components is that we don’t have life cycle methods (componentDidMount, componentDidUpdate, componentWillUnmount, etc...).
Lets say we have a component that receives an userId as a prop, fetches the user’s name from a server and renders its name.

```jsx
class Username extends Component {
  constructor(props) {
    super(props);
    this.state = {
      username: ""
    };
    this.fetchAndSetUsername = this.fetchAndSetUsername.bind(this)
  }

  componentDidMount() {
    this.fetchAndSetUsername()
  }

  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchAndSetUsername()
    }
  }

  fetchAndSetUsername() {
    fetchUserName(this.props.userId).then(username => {
      this.setState({ username: username });
    });
  }

  render() {
    return <div>{this.state.username}</div>;
  }
}
```

In the example above, we get the name first when the component mounts using componentDidMount and whenever the userId is changing using componentDidUpdate.
Another way to understand it is “keep the name updated with the userId prop”.
We also have to use state as we need to save the name.

```jsx
function Username(props) {
  const { userId } = props;
  const [username, setUsername] = useState("");

  useEffect(() => {
    fetchUserName(userId).then(username => {
      setUsername(username);
    });
  }, [userId]);

  return <div>{username}</div>;
}
```

What happens here?
We create a name state using useState hook, then setting an useEffect hook to fetch the user’s name and set it to the state each time the userId changes.

useEffect is a function that accepts a callback function as the first argument and an array of dependencies as a second argument, It returns nothing.
useEffect is calling the callback function each time one of its dependencies has changed.
*redirect to another article to explain about cleanup function and advanced useEffect stuff*


### useMemo

As we discovered this far, some hooks are solving function component problems (or missing gaps from class components) but there’s another issue - performance.

```jsx
function FullnamesList(props) {
  const { users = [] } = props;
  const fullNames = users.map(user => {
    return user.firstName + " " + user.lastName;
  });

  return (
    <ul>
      {fullNames.map(fullName => {
        return <li>{fullName}</li>;
      })}
    </ul>
  );
}

```

In the example above we have a component that takes a list of users (each user is an object that contains a firstName and a lastName, both strings) as a prop and shows a list of their full names.

Do you see the problem?
The code runs over the whole list any time the component renders! that might become a performance hit on our app (assume a huge list or a more complex map calculations).

How can we solve it?
One way is to tell the component when to render (using React.memo). The second way is to calculate fullNames array only when “users” prop has changed (using useMemo hook).
Since this article is about React hooks, we will explore only useMemo, check out React.memo in [this article](https://reactjs.org/docs/react-api.html#reactmemo "React.memo article").

The useMemo solution:

```jsx
function FullnamesList(props) {
  const { users = [] } = props;
  const fullNames = useMemo(() => {
    return users.map(user => {
      return user.firstName + " " + user.lastName;
    });
  }, [users]);

  return (
    <ul>
      {fullNames.map(fullName => {
        return <li>{fullName}</li>;
      })}
    </ul>
  );
}
```

useMemo is a function that accepts a callback as the first argument and a dependency array as the second argument. useMemo returns a value, which value? The value returned from the callback function.
In simple words, useMemo runs the callback function and returns the same value the callback function returns, it runs the callback function again when one of its dependencies has changed.

*Read about memoization usages in a React application in this article*


### useCallback

Another performance enhancer is useCallback, in my opinion we can do well without it but it’s a great tool to have in your pocket, therefore if you’re feeling confused about hooks I recommend you to stop here and master the basic hooks (useState, useEffect and useMemo) before you continue.

To explain the problem useCallback comes to solve, I’ll use the previous useMemo’s example “FullnamesList” component.

Lets say we want to implement another prop to this component called "composeFullname” which is a function that takes a first name and last name as arguments and returns a fullname.

```jsx
const defaultComposeFullname = (firstName, lastName) => {
  return firstName + " " + lastName;
};

function FullnamesList(props) {
  const { users = [], composeFullname = defaultComposeFullname } = props;
  const fullNames = useMemo(() => {
    return users.map(user => {
      return composeFullname(user.firstName, user.lastName);
    });
  }, [users, composeFullname]);

  return (
    <ul>
      {fullNames.map(fullName => {
        return <li>{fullName}</li>;
      })}
    </ul>
  );
}
```

Note that as we implement composeFullname, we also need to add composeFullname to the dependencies array in useMemo so it will recalculate the list of full names whenever composeFullname is changed.

What about useCallback??

Time to implement FullnamesList!

```jsx
const users = [{ firstName: "John", lastName: "Doe" }, { firstName: "Jane", lastName: "Doe" }];

function App(props) {
  const composeFullname = (firstName, lastName) => {
    return lastName + ' ' + firstName
  }

  return <FullnamesList users={users} composeFullname={composeFullname} />;
}
```

That's cool! we can control how to compose the full names outside the FullnamesList component. But there's a problem with this code.
composeFullname is redefined each App's render, meaning FullnamesList gets a different composeFullname prop (that does the same thing) and that's causing unnecessary computations inside FullnamesList (see the dependency array inside FullnamesList's useMemo).

So now we need a way to define a function that won't be redefined any render.
useCallback to the rescue:

```jsx
const users = [{ firstName: "John", lastName: "Doe" }, { firstName: "Jane", lastName: "Doe" }];

function App(props) {
  const composeFullname = useCallback((firstName, lastName) => {
    return lastName + ' ' + firstName
  }, [])

  return <FullnamesList users={users} composeFullname={composeFullname} />;
}
```

useCallback is a function that accepts a callback as the first argument and a dependency array as the second argument, same as useMemo, the difference is that useCallback returns a memoized version of the callback from the first argument, meaning it won't re-create a function each render.
It'll re-create the function when one of its' dependencies has changed.

### useRef

useRef is a great tool used for keeping mutable variables.
I can't explain it any better than the [React docs](https://reactjs.org/docs/hooks-reference.html#useref "useRef React docs") so I won't waste your time.

### useContext

[Context](https://reactjs.org/docs/context.html "React context") is a basic React concept that lets multiple components share the same data.
From my experience, using context is pretty rare and not suggested most of the times, as the React docs says: "...it makes component reuse more difficult".
If you ask me, useContext is not a necessety, so to avoid confusion, I will elaborate on it on another article.

### useReducer

I'm not using the useReducer hook since I can achive the same result using useState, see [this cool article](https://medium.com/free-code-camp/why-you-should-choose-usestate-instead-of-usereducer-ffc80057f815 "Medium article why-you-should-choose-usestate-instead-of-usereducer").


## Summary

Hooks are a great concept that might sometimes be confusing.
Every hook serves a purpose, so when struggling with which hook to use, try asking yourself what is the problem you're trying to solve? Is it a state you need? Do you need to perform an action every prop/state change? Is it a performance issue?...
I've made a list of hooks clustered by purpose, made to help you answer these questions:

State management:
 - useState
 - useReducer

Lifecycle and prop/state change effects:
 - useEffect

Memoization (performance):
 - useCallback
 - useMemo


Mutable variables:
 - useRef

Context (HOC, prop drilling):
 - useContext




I hope that article will help you get started and use hooks more often.