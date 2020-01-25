---
title: Master useEffect
date: '2020-01-25'
spoiler: Master the useEffect hook.
cta: 'react'
---


### What is useEffect

useEffect hook helps you to catch changes in props or state inside a function component and respond to these changes.
It can be used to replace life cycle methods such as componentDidMount, componentDidUpdate and componentWillUnmount but it’s stronger, in this article we’ll dive in some examples starting of the basics and progressing into more complex stuff.

### The basics

```jsx
const Counter = props => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
};
```

Here we have a function component that can count using a state.
Let’s say we want to alert the count on each count change.
we can achieve this by adding an if statement:

```jsx
const Counter = props => {
  const [count, setCount] = useState(0);
  
  alert(`You clicked ${count} times`);
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
};
```

Well, that works, but what happens if we want to add another feature to our component? Let’s add an input that will define our counting steps size:

```jsx
const Counter = props => {
  const [count, setCount] = useState(0);
  const [countStep, setCountStep] = useState(1);

  alert(`You clicked ${count} times`);

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + Number(countStep))}>Click me</button>
      <input type="number" value={countStep} onChange={e => setCountStep(e.target.value)} />
    </div>
  );
};
```

Cool! But there’s a problem with this code. Notice we alert each render, that means every time we change the count or the count step a re-render happens and we alert the message again.
How can we solve it? We need an option to alert only if count has changed we need to catch a change in state (count) and respond to it (alert the user).
Enters useEffect hook:

```jsx
const Counter = props => {
  const [count, setCount] = useState(0);
  const [countStep, setCountStep] = useState(1);

  useEffect(() => {
    alert(`You clicked ${count} times`);
  }, [count]);

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + Number(countStep))}>Click me</button>
      <input type="number" value={countStep} onChange={e => setCountStep(e.target.value)} />
    </div>
  );
};
```

useEffect accepts 2 parameters: callback (function) and dependencies (array | undefined), On each render the useEffect hook checks if a change occurred in one of its dependencies and calls the callback.
The callback we passed is a function that alerts the count, the dependencies array contains the count variable meaning each change in the counter will fire callback.

### Cleanup function

After learning the basic usage of useEffect, catching props and state changes and respond, we can go deeper into useEffect cleanup function.
useEffect accepts a callback as a first argument, this callback function can return a function (or undefined), that means __the callback can’t return a number, string, array, promise and anything else that is not a function (or undefined)__.

```jsx
useEffect(() => {
  return ‘Hello World!’
})
// That code will return a warning in console: An effect function must return nothing besides a function, which is used for clean-up. You returned: Hello World!
```

That means you can’t write the callback as async since async function returns promise

```jsx
useEffect(async () => {
  const data = await fetchData()
  console.log(data)
})
// That code will throw an error: destroy is not a function

// One correct way to use async in useEffect is:
useEffect(() => {
  const asyncFunc = async () => {
    const data = await fetchData()
    console.log(data)
  }
  asyncFunc()
})
// That way we don’t return a promise
```

So what happens if we actually return a function?

```jsx
useEffect(() => {
  console.log(‘inside the callback function’)
  return () => console.log(‘inside the cleanup function!’)
})
```

useEffect will save the cleanup function in memory will call it:
 - Before each time it should call the callback function.
 - Once before the component unmounts.
 
Here’s an image showing the console logs:

![Screen Shot 2019-12-03 at 11.05.08](//images.ctfassets.net/afq2xws8d2dy/7vs9eTK8bs9NBbU6x4FnVu/b54c49f5a45f09a0c6e2c83c1c3d1070/Screen_Shot_2019-12-03_at_11.05.08.png)

How can we use the cleanup function?
Mainly it can be used as componentWillUnmount:

```jsx
useEffect(() => {
  return () => console.log(‘component will unmount’)
}, [])
```

Notice passing the ‘[]’ into the second parameter makes the cleanup function run __only__ before the component unmounts.

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setSeconds(seconds + 1);
    }, 1000);
    return () => clearInterval(id);
  }, [seconds]);

  return <div>{seconds}</div>;
}
```

Notice this example will cleanInterval and re-create interval on each seconds change but this is necessary for setting the correct seconds number, a better way to write it will be:

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <div>{seconds}</div>;
}
```

By removing the seconds variable from the dependencies useEffect will call the callback fnuction once when the component mounts and will call the cleanup function once before the component unmount.
Notice that by writing ‘setSeconds(s => s + 1);’ we can now get the current value of seconds and add 1 to it.
for more information about this example you can visit Dan Abramov’s Awsome [blog post about useEffect](https://overreacted.io/a-complete-guide-to-useeffect/#making-effects-self-sufficient “Dan Abramov blog post”).


### Recap

We’ve learned that by using useEffect you can catch changes in props or state inside a function component and respond to these changes.
Cleanup function is great for doing actions when the component unmounts.
Remember you can’t return anything but function (or undefined) inside useEffect callback.