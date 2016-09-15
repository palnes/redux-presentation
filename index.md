[flowchart]: /flowchart.svg "flow"
[flowchartactions]: /flowchart-actions.svg "actions"
[flowchartconnect]: /flowchart-connect.svg "connect"
[flowchartmiddleware]: /flowchart-middleware.svg "middleware"
[flowchartreducers]: /flowchart-reducers.svg "reducers"
[flowchartstore]: /flowchart-store.svg "store"
[reducers]: /reducers.png "Reducers"

class: center, middle

# redux
### or: how i learned to stop worrying and love the atomic bomb

_by pål_

---
class: middle

## I'll walk you through

### ✔️ redux concepts
### ✔️ Connecting to React
### ✔️ Show and Tell 🎉

---

class: center, middle

## Compulsory flowchart

![][flowchart]

---
class: center, middle

# Store

![][flowchartstore]

---

## Store

- One global, immutable, **state**

- Creates **state** by executing **reducers** and combining their results

- State can only be changed by dispatching **actions** to **reducers**
```
    // Create store
    const store = redux.createStore(reducers, preloadedState, middlewares)
    // Get current state (entire)
    const myState = store.getState()
    // Dispatch action -> reducer
    const action = {
        type: 'HELLO_SAY',
        payload: 'Hello World!'
    };
    store.dispatch(action)
```

???

Global state, there is only one state tree. The store is a function that
calls the three provided functions to produce the state.

First is the rootReducer, composing the reducers into a state tree
Second is any pre-existing state, from URL, cookies, localStorage
Third up is the middlewares. Middlewares catch actions and massage their payloads.

State is created by calling reducers with an action of type `@@INIT`

Actions are called with the store's dispatch functions

---

class: center, middle

# Reducers

![][flowchartreducers]

---

# Reducers

Pure functions that take **state** and **action**, and return a new **state**
based on action **type** and **payload**.
```
store.dispatch(action = {
    type: 'FRUIT_ADD',
    payload: 'apple'
})
// ...
const reducer = (state = [], action) => {
    switch (action.type) {
        case 'FRUIT_ADD':
            return [...state, action.payload]
        case 'FRUIT_REMOVE':
            return state.filter(d => d !== action.payload);
        default:
            return state;
    }
}
// store.getState() => ['apple']
```
* Pure functions always return same output given same input
* Reducers _**always**_ return state.

---

# Composing reducers

```
const initialState = { fruits: [], edible: false }

const fruitReducer = (state = [], action) => {
    switch (action.type) {
        case 'FRUIT_ADD':
            return state.push(action.payload)
        case 'FRUIT_REMOVE':
            return state.filter(d => d !== action.payload);
        default:
            return state;
    }
}

const reducer = (state = initialState, action) => {
    switch(action.type) {
        case 'FRUIT_ADD':
            // fall through
        case 'FRUIT_REMOVE':
            return _.merge(state, { fruits: fruitReducer(state.fruits, action)})
        case 'EDIBLE_TOGGLE':
            return _.merge(state, { edible: !edible })
        default:
            return state;
    }
}

```

---

# combineReducers
`redux.combineReducers` allows composing reducers into a state tree.

```
const fruits = (state, action) => { ... }
const vegetables = (state, action) => { ... }

const rootReducer = combineReducers({
    fruits,
    vegetables
});
//  store.getState() => { fruits: ..., vegetables: ... }
```

---
class: center, middle

# Actions

![][flowchartactions]

---

# Actions

Actions are plain old Javascript objects with the following shape:
```
{
    type: 'USER_FIND', // Required
    payload: 'waldo', // Optional
    error: false, // Optional
    meta: {} // Optional, mostly used by middlewares
}
```

An action **MUST NOT** have properties other than `type`, `payload`, `error`, and `meta`.

---

# Action Creators
Actions are no fun if they're static.

```
const USER_ADD ='USER_ADD'

const addUser = username => ({
    type: USER_ADD,
    payload: username
});

const reducer = (state = [], action) => ({
    switch (action.type) {
        case USER_ADD:
            return state.push(action.payload)
        default:
            return state;
    }
});

store.dispatch(addUser('waldo'));
store.dispatch(addUser('oglaf'));
store.getState(); // ['waldo', 'oglaf']
```

???

Notice the constant used to share type between action and reducer.

---

# Action Errors
Actions representing errors should look like this:
```
{
    type: 'USER_FIND',
    payload: new Error('No such user'),
    error: true
}
```
An action whose error is `true` is analogous to a rejected `Promise`.

By convention, the payload is an `Error` object or derivative.

---
class: center, middle

# Middlewares

![][flowchartmiddleware]

---

# Middlewares
Middlewares allows an action **payload** to be altered before being applied to a **reducer**.
This allows, for example, **async** actions.
```
const USER_FIND ='USER_FIND'

const findUser = username => ({
    type: USER_FIND,
    payload: {
        promise: fetch(`/api/user/${username}`)
            .then(response => response.json())
            .then(json => json)
    }
});

store.dispatch(findUser('waldo'));
// redux-promise-middleware hooks the above action,
// resolves the promise and dispatches actions
// accordingly:
store.dispatch({ type: USER_FIND_PENDING });
store.dispatch({
    type: USER_FIND_FULFILLED,
    payload: ['waldo@gmail.com', 'waldo_sexy4112@hotmail.com']
});
```

---


## redux Concepts in summary

* **Store**

  Bundles **reducers** into **state** and dispatches **actions** to **reducers**.

* **Reducers**

  Pure functions that take **state** and **action**, and returns a new **state**.

* **Actions**

  * Plain object with **type** and **payload**.
  * Often wrapped in an **action creator** function.

* **Middleware**

  * Hooks action **payload** before applying **reducer**.
  * This allows for **async actions** with promises.

---

class: center, middle

# Connecting to React

![][flowchartconnect]

---

# Connecting the store to a React Application

```
import { createStore } from 'redux';
import { Provider } from 'react-redux';

const store = createStore(
    rootReducer,
    preloadedState,
    middlewares
);

ReactDOM.render((
    <Provider store={store}>
        <App />
    </Provider>),
    document.getElementById('app')
);
```
`<Provider/>` sets `store.getState()` in React's **context**.
This allows subcomponents to subscribe to store changes.

---

# Connecting state to a component

```
import { connect } from 'react-redux';

const Users = ({ users, findUser }) => (
    <div>
        <input type="text" onChange={event => findUser(event)}></input>
        <ul>{users.map(d => <li>{d}</li>)}</ul>
    </div>
);

const mapStateToProps = state => ({
    users: state => state.users // Assign `state.users` to `this.props.users`
});
const mapDispatchToProps = {
    findUser // Shorthand for `(id) => store.dispatch(findUser(id))`
}

const ConnectedComponent = connect(
    mapStateToProps,
    mapDispatchToProps
)(Users);
```

---

class: center, middle

# Show and Tell! 🎉


