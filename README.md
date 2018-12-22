### Relax - A Promise Based State Management Library

[![npm version](https://badge.fury.io/js/%40relax-js%2Frelax.svg)](https://badge.fury.io/js/%40relax-js%2Frelax)

A spin-off of Redux. Rewritten from scratch to simplify how you manage state. No more action types, switch statements, middleware, and especially no reducers! Plus it's tiny: `650 B` gzipped! That's ~70% smaller than Redux: `2.23kb` gzipped.

Basic Usage:

[![Relax Tutorial](https://i.ibb.co/pjnfM3j/relax-play.png)](https://youtu.be/GLzRHx0DAX0 "Relax Counter Tutorial")

- [Example Code](https://github.com/relax-js/relax/tree/master/examples)
- [Setup](#create-store)
- [Managing Branches of State](https://github.com/relax-js/relax/tree/master/examples/branches)
- [Asyncronous Actions](#asyncronous-actions)
- [Return Values](#dispatch-response)
- [Debugging](#debugging)

Complimentary Libraries
- [React With Relax](https://github.com/relax-js/react-relax)

---
#### Create Store
This is the most basic setup of creating a store. To read about the parameters it accepts, such as setting an initial state, see [here](https://github.com/relax-js/relax/tree/master/examples/createStore)
```js
const store = createStore();
```

Now say you want to update the value of a DOM element every time the state is changed.
```js
const rerender = ({ state }) => {
    // code to update a DOM value
    element.value = state.value;
}

store.subscribe(rerender);
```

Create an action to change the value in state
```js
const updateValue = (newValue) => {
    return {
        value: newValue,
    }
}

// Dispatch the action
store.dispatch(updateValue(47))
console.log(store.getState()) // { value: 47 }
```
That was a super basic example. If you're familiar with Redux you may be wondering where the reducer is --- there are no reducers! What you return from your action is **merged** with the current state.

If you're thinking, "I can see this getting messy really fast" don't worry, managing different branches of state can be pretty simple! See [here](https://github.com/relax-js/relax/tree/master/examples/branches) to continue reading on that.

#### Asyncronous Actions
Here's a basic example on asyncronous actions. To read more on this, see [here](https://github.com/relax-js/relax/tree/master/examples/async).
```js
// asynchronous action
const fetchData = (api) => {
    return fetch(api)
        .then(data => data)
        .then(result => ({
            apiData: result,
        });
}

store.dispatch(fetchData('/someApi'));
// When fetch promise resolves
console.log(store.getState()) // { value: 47, apiData: {...} }
```

#### Thenable Actions
Async actions are very common and sometimes callbacks to those actions are desireable.

Well, now you chain `.then` to dispatch (same with any Promise methods: `.catch`, `.finally`, etc.)
```js
store.dispatch(fetchData('/someApi')).then(() => {
    store.dispatch(runAfterFetch());
});

// Btw, you can chain `.then` to synchronous actions too :)
store.dispatch(updateValue(100)).then(() => {
    // do whatever after value is updated
});
```

But wait, there's more to `dispatch` and `.then`. It provides an object of parameters!
```js
store.dispatch(...).then(({ result, state, dispatch }) => {})
```

#### Dispatch Response
Property | Description
---|---
result | the return value from your action (what changed)
state | current state
dispatch | same as `store.dispatch`

#### Subscribe Response
Property | Description
---|---
... | same response as dispatch
unsubscribe | function to unsubscribe: `unsubscribe()`

#### Debugging
You can hook up to the Redux Dev Tools extension, if you have it installed, like this:
```js
const devTools = window.__REDUX_DEVTOOLS_EXTENSION__.connect({
    name: `Instance Name`, // optional
});
devTools.init(store.getState());
store.subscribe(({ result, state }) => {
    const action = (result && result._action) || 'state changed';
    devTools.send(action, state);
});
```
**Warning**
• I don't recommend using this in production so be sure to wrap it in `process.env.NODE_ENV !== 'production'` to avoid potential issues.
• This is a basic example to show state diffs. Other dev tools features would require more logic.

Typically with action creators in Redux you'd create a `type` which would also display in the dev tools.

This is how you pass a *type* with Relax. Take the action `updateValue` for example:
```js
const updateValue = (newValue) => ({
    _action: 'updateValue',
    value: newValue
});
```
`_action` is a string and is helpful with debugging in dev tools. You could subscribe and watch for an `_action` if you really wanted to.
