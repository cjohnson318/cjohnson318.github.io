---
layout: post
title: "React Redux in TypeScript - Example"
date: 2025-03-22 00:00:00 -0700
tags: react redux
---

This is another Simplest Working Example post. This will cover Redux using
TypeScript.

## Project Setup

First setup a project using pnpm and vite:

{% highlight console %}
pnpm create vite counter-app-ts --template react-ts
cd counter-app-ts
pnpm install
pnpm add @reduxjs/toolkit react-redux redux
{% endhighlight %}

## Actions and Reducers

First, we're creating a `features/counter` directory, and then creating
`actions.js` and `reducer.js` inside of that.

{% highlight javascript %}
// src/features/counter/actions.ts
import { createAction } from '@reduxjs/toolkit';

export const increment = createAction('counter/increment');
export const decrement = createAction('counter/decrement');
export const incrementByAmount = createAction<number>('counter/incrementByAmount');

// src/features/counter/reducer.ts
import { createReducer } from '@reduxjs/toolkit';
import { increment, decrement, incrementByAmount } from './actions';

interface CounterState {
  count: number;
}

const initialState: CounterState = {
  count: 0,
};

const counterReducer = createReducer(initialState, (builder) => {
  builder
    .addCase(increment, (state) => {
      state.count += 1;
    })
    .addCase(decrement, (state) => {
      state.count -= 1;
    })
    .addCase(incrementByAmount, (state, action) => {
      state.count += action.payload;
    });
});

export default counterReducer;
{% endhighlight %}

If this looks too much like magic, with `createAction`, then you can rewrite
it as,

{% highlight javascript %}
// src/features/counter/actions.ts
export const INCREMENT = 'counter/increment';
export const DECREMENT = 'counter/decrement';
export const INCREMENT_BY_AMOUNT = 'counter/incrementByAmount';

interface IncrementAction {
  type: typeof INCREMENT;
}

interface DecrementAction {
  type: typeof DECREMENT;
}

interface IncrementByAmountAction {
  type: typeof INCREMENT_BY_AMOUNT;
  payload: number;
}

export type CounterActionTypes =
  | IncrementAction
  | DecrementAction
  | IncrementByAmountAction;

export const increment = (): IncrementAction => ({
  type: INCREMENT,
});

export const decrement = (): DecrementAction => ({
  type: DECREMENT,
});

export const incrementByAmount = (amount: number): IncrementByAmountAction => ({
  type: INCREMENT_BY_AMOUNT,
  payload: amount,
});

// src/features/counter/reducer.ts
import { INCREMENT, DECREMENT, INCREMENT_BY_AMOUNT, CounterActionTypes } from './actions';

interface CounterState {
  count: number;
}

const initialState: CounterState = {
  count: 0,
};

function counterReducer(
  state: CounterState = initialState,
  action: CounterActionTypes
): CounterState {
  switch (action.type) {
    case INCREMENT:
      return { ...state, count: state.count + 1 };
    case DECREMENT:
      return { ...state, count: state.count - 1 };
    case INCREMENT_BY_AMOUNT:
      return { ...state, count: state.count + action.payload };
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
// src/store.ts
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './features/counter/reducer';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
{% endhighlight %}

## The Application

This is the top level app that holds the UI elements that we're interested in,
pulls reactive data from the store, and dispatches actions to update the store.

{% highlight javascript %}
// src/App.tsx
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement, incrementByAmount } from './features/counter/actions';
import type { RootState, AppDispatch } from './store';

function App() {
  const count = useSelector((state: RootState) => state.counter.count);
  const dispatch: AppDispatch = useDispatch();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(increment())}>Increment</button>
      <button onClick={() => dispatch(decrement())}>Decrement</button>
      <button onClick={() => dispatch(incrementByAmount(5))}>Increment by 5</button>
    </div>
  );
}

export default App;
{% endhighlight %}

This is what is sourced in `index.html`. It references our top level App
component, and it makes the store available to that App component via a higher
level Provider component.

{% highlight javascript %}
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';
import { Provider } from 'react-redux';
import { store } from './store';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
{% endhighlight %}
