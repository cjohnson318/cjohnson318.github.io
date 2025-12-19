---
layout: post
title:  "React Hook Examples"
date:   2024-06-20 00:00:00 -0700
categories: tech
tags: react
---

React Hooks are a powerful addition to the React toolkit that allow you to use
state and other React features without writing classes, and passing props 
between components

## Using `useState` and `useEffect`

The most fundamental React hooks are `useState` and `useEffect`. The `useState`
hook handles reactive data so that changes to that data force the UI to
rerender. The first argument of the `useEffect` function is an anonymous
function that performs a side effect when there is a change in state. The
second argument of `useEffect` allows you to provide a list of variables that
you want `useEffect` to react to. Alternatively, you can pass an empty array
as the second argument if you want `useEffect` to react to any and all state
changes.

{% highlight react %}
import { useState, useEffect } from "react";

export default function App() {
  const [value, setValue] = useState(1);
  const [count, setCount] = useState(0);

  useEffect(() => {
    if (value != 1) {
      setCount((count) => count + 1);
    }
  }, [value]); // watch specifically for changes in value

  const double = () => {
    setValue((value) => value * 2);
  };

  return (
    <div className="App">
      <p>
        Here, value is being doubled, and count tracks the number of doublings
      </p>
      <p>Value: {value}</p>
      <p>Count: {count}</p>
      <button onClick={double}>Double value</button>
    </div>
  );
}
{% endhighlight %}

### What's Happening

  - `useState`: This hook is used to manage state within functional components. We create two state variables: `value` initialized to 1 and `count` initialized to 0.
  - `useEffect`: This hook is used to perform side effects in functional components. It runs after every render. In this case, we're using it to increment count whenever value changes (except for the initial render).
  - `double`: This function doubles the `value` state.

### How it Works

  1. The component initially renders with value as 1 and count as 0.
  2. Clicking the "Double value" button calls the double function, updating value to 2.
  3. The `useEffect` hook runs due to the change in `value`.
  4. Inside `useEffect`, we increment `count` by 1.
  5. The component re-renders with the updated `value` and `count`.

## Using `useContext`

The next most useful React hook is `createContext`. This hook allows users to
share data and trigger side effects across components without passing props.
This simple example has two components, `ControllerComponent` and
`JobComponent`. These two components use and update a global value, `status`.
The global value is manages by a `useState` hook, and it is shared between the
two components via the `useContext` hook. This arrangement allows us to 
control and display `status` from either the `ControllerComponent` or the
`JobComponent`, and have `status` update in both components.

{% highlight react %}
import { useState, useContext, createContext } from "react";

const statuses = {
  play: "Playing",
  pause: "Paused"
};

function ControllerComponent() {
  const [status, setStatus] = useContext(StatusContext);
  function controllerBtn() {
    if (status == statuses.play) {
      setStatus(statuses.pause);
    } else {
      setStatus(statuses.play);
    }
  }
  return (
    <div>
      <div>Controller Component</div>
      <button onClick={controllerBtn}>Toggle</button>
    </div>
  );
}

function JobComponent() {
  const [status, setStatus] = useContext(StatusContext);
  function jobBtn() {
    if (status == statuses.play) {
      setStatus(statuses.pause);
    } else {
      setStatus(statuses.play);
    }
  }
  return (
    <div>
      <div>Job Component</div>
      <div>Status: {status}</div>
      <button onClick={jobBtn}>Toggle</button>
    </div>
  );
}

const StatusContext = createContext();

export default function App() {
  const [status, setStatus] = useState(statuses.play);
  return (
    <div className="App">
      <StatusContext.Provider value={[status, setStatus]}>
        <ControllerComponent />
        <JobComponent />
      </StatusContext.Provider>
    </div>
  );
}
{% endhighlight %}

### What's Happening

  - `statuses`: This object defines a simple object containing possible values for the status: play and pause.
  - `useState` in `App`: The `App` component holds the initial state using `useState` and sets the initial status to "play."
  - `StatusContext`: We create a React Context using createContext to hold the current status and a function to update it (setStatus).
  - `useContext`: Both `ControllerComponent` and `JobComponent` use the `useContext` hook to access the shared state provided by `StatusContext`. Their functionalities are similar:
    - They read the current `status` from the context.
    - They define a function (`controllerBtn` or `jobBtn`) that toggles the status between "play" and "pause" based on the current value.
    - They update the context using the `setStatus` function provided by the context.

### How it Works

  1. The `App` component renders and creates a `StatusContext` provider, wrapping both `ControllerComponent` and `JobComponent`.
  2. The context provider passes the current `status` and the `setStatus` function as an array to its children.
  3. Each child component uses `useContext` to access these values.
  4. Clicking the "Toggle" button in either component calls its respective function (controllerBtn or jobBtn).
  5. These functions access the current `status` and update it to the opposite value (play/pause) using the `setStatus` function from the context.
  6. The context updates with the new status.
  7. Both components re-render, displaying the updated status and allowing them to react based on the current state.
