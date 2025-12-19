---
layout: post
title: "React Redux - Example"
date: 2025-03-21 00:00:00 -0700
categories: tech
tags: react redux
---

I've decided to do a couple of Simplest Working Example posts. This will cover
Redux, a state management library for React. What that means in practical terms
is that it lets different components in an application access and modify the
same bit of state, like a counter.

## Project Setup

First setup a project using pnpm and vite:

{% highlight console %}
pnpm create vite counter-app --template react
cd counter-app
pnpm install
pnpm add @reduxjs/toolkit react-redux redux
{% endhighlight %}

## Actions and Reducers

First, we're creating a `features/counter` directory, and then creating
`actions.js` and `reducer.js` inside of that.

{% highlight javascript %}
// src/features/counter/actions.js
import { createAction } from '@reduxjs/toolkit';

export const increment = createAction('counter/increment');
export const decrement = createAction('counter/decrement');

// src/features/counter/reducer.js
import { createReducer } from '@reduxjs/toolkit';
import { increment, decrement } from './actions';

const initialState = {
  value: 0,
};

const counterReducer = createReducer(initialState, (builder) => {
  builder
    .addCase(increment, (state) => {
      state.value += 1;
    })
    .addCase(decrement, (state) => {
      state.value -= 1;
    });
});

export default counterReducer;
{% endhighlight %}

If this looks too much like magic, with `createAction`, then you can rewrite
it as,

{% highlight javascript %}
// src/features/counter/actions.js
export const INCREMENT = 'counter/increment';
export const DECREMENT = 'counter/decrement';

export const increment = () => ({
  type: INCREMENT,
});

export const decrement = () => ({
  type: DECREMENT,
});

// src/features/counter/reducer.js
import { INCREMENT, DECREMENT } from './actions';

const initialState = {
  value: 0,
};

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case INCREMENT:
      return { ...state, value: state.value + 1 };
    case DECREMENT:
      return { ...state, value: state.value - 1 };
    default:
      return state;
  }
}

export default counterReducer;
{% endhighlight %}

I kind of like the older, more verbose version.

## The Store

The action and the reducer work together to update the state. The client code,
different components of the application, dispatch actions, and then the reducer
catches those dispatched actions and updates the state. The the state is
updated in the client code using the `useSelector` hook. The state "lives" in
the react redux store.

{% highlight javascript %}

// src/store.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './features/counter/reducer';

const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

export default store;
{% endhighlight %}

## The Application

This is the top level app that holds the UI elements that we're interested in,
pulls reactive data from the store, and dispatches actions to update the store.

{% highlight javascript %}
// src/App.jsx
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement } from './features/counter/actions';

function App() {
  const count = useSelector((state) => state.counter.count);
  const dispatch = useDispatch();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(increment())}>Increment</button>
      <button onClick={() => dispatch(decrement())}>Decrement</button>
    </div>
  );
}

export default App;
{% endhighlight %}

This is what is sourced in `index.html`. It references our top level App
component, and it makes the store available to that App component via a higher
level Provider component.

{% highlight javascript %}
// src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';
import { Provider } from 'react-redux';
import store from './store';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
{% endhighlight %}
