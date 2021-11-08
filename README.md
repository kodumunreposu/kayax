<h1>
<div><b>kayaX</b></div>
</h1>

Tiny, easy and powerful **React state management** library


## What advantages does it have? ✨

<ul>
    <li>📦  ・<b>Tiny</b>: Less than 1kb package to manage your state in React and Preact.</li>
    <li>🌱  ・<b>Easy</b>: You don't need actions, reducers, selectors, connect, providers, etc. Everything can be done in the simplest and most comfortable way.</li>
    <li>🚀  ・<b>Powerful</b>: When a store property is updated, only its components are re-rendered. It's not re-rendering components that use other store properties.</li>
</ul>

<hr />

## Guide 🗺

- [What advantages does it have? ✨](#what-advantages-does-it-have-)
- [Guide 🗺](#guide-)
- [Installation 🧑🏻‍💻](#installation-)
- [Init your store 👩🏽‍🎨](#init-your-store-)
  - [createStore](#createstore)
  - [How to export](#how-to-export)
- [Manage the store 🕹](#manage-the-store-)
  - [useStore hook](#usestore-hook)
  - [getStore helper](#getstore-helper)
  - [withStore HoC](#withstore-hoc)
- [Register events after an update 🚦](#register-events-after-an-update-)
- [How to... 🧑‍🎓](#how-to-)
  - [Add a new store property](#add-a-new-store-property)
  - [Reset a store property](#reset-a-store-property)
  - [Reset all the store](#reset-all-the-store)
  - [Use more than one store](#use-more-than-one-store)
  - [Update several portions avoiding rerenders in the rest](#update-several-portions-avoiding-rerenders-in-the-rest)
  - [Define calculated properties](#define-calculated-properties)
- [Roadmap 🛣](#roadmap-)

## Installation 🧑🏻‍💻

```sh
yarn add kayax
# or
npm install kayax --save
```

## Init your store 👩🏽‍🎨

Each store has to be created with the `createStore` function. This function returns all the methods that you can use to consume and update the store properties.

### createStore

```js
import createStore from "kayax";

const { useStore } = createStore();
```

Or also with an initial store:

```js
const initialStore = {
  cart: { price: 0, items: [] },
};
const { useStore, getStore } = createStore(initialStore);
```

Or also with an event that is executed after every update:

```js
const initialStore = {
  cart: { price: 0, items: [] },
};

function onAfterUpdate({ store, prevStore }) {
  console.log("This callback is executed after an update");
}

const { useStore } = createStore(initialStore, onAfterUpdate);
```

_Input:_

| name            | type          | required | description                                                                                              |
| --------------- | ------------- | -------- | -------------------------------------------------------------------------------------------------------- |
| `initialStore`  | `object<any>` | `false`  | Object with your initial store.                                                                          |
| `onAfterUpdate` | `function`    | `false`  | Function that is executed after each property change. More [details](#register-events-after-an-update-). |

_Output:_

| name        | type    | description                                                                                                                                                                                             | example                                           |
| ----------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| `useStore`  | `Proxy` | Proxy hook to consume and update store properties inside your components. Each time the value changes, the component is rendered again with the new value. More [info](#usestore-hook).                 | `const [price, setPrice] = useStore.cart.price()` |
| `getStore`  | `Proxy` | Similar to `useStore` but without subscription. You can use it as a helper outside (or inside) components. Note that if the value changes, it does not cause a rerender. More [info](#getstore-helper). | `const [price, setPrice] = getStore.cart.price()` |
| `withStore` | `Proxy` | HoC with `useStore` inside. Useful for components that are not functional. More [info](#withstore-hoc).                                                                                                 | `withStore.cart.price(MyComponent)`               |

### How to export

We recommend using this type of export:

```js
// ✅
export const { useStore, getStore, withStore } = createStore({
  cart: { price: 0, items: [] },
});
```

This way you can import it with:

```js
// ✅
import { useStore } from '../store'
```

Avoid using a default export with all:

```js
// ❌
export default createStore({ cart: { price: 0, items: [] } });
```

Because then you won't be able to do this:

```js
// ❌  It's not working well with proxies
import { useStore } from '../store'
```

## Manage the store 🕹

### useStore hook

It's recommended to use the `useStore` hook as a proxy to indicate exactly what **portion of the store** you want. This way you only subscribe to this part of the store avoiding unnecessary re-renders.

```js
import createStore from "kayax";

const { useStore } = createStore({
  username: "Hamza",
  count: 0,
  age: 31,
  cart: {
    price: 0,
    items: [],
  },
});

function Example() {
  const [username, setUsername] = useStore.username();
  const [cartPrice, setCartPrice] = useStore.cart.price();

  return (
    <>
      <button onClick={() => setUsername("AnotherUserName")}>
        Update {username}
      </button>
      <button onClick={() => setCartPrice((v) => v + 1)}>
        Increment price: {cartPrice}€
      </button>
    </>
  );
}
```

However, it's also possible to use the `useStore` hook to use **all the store**.

```js
function Example() {
  const [store, setStore] = useStore();

  return (
    <>
      <button
        onClick={() =>
          setStore((s) => ({
            ...s,
            username: "AnotherUserName",
          }))
        }
      >
        Update {store.username}
      </button>
      <button
        onClick={() =>
          setStore((s) => ({
            ...s,
            cart: { ...s.cart, price: s.cart.price + 1 },
          }))
        }
      >
        Increment price: {store.cart.price}€
      </button>
    </>
  );
}
```

_Input:_

| name                  | type       | description                                                                                                                                                                                                                                    | example                                                                                                                                                                                                    |
| --------------------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Initial value         | `any`      | This parameter is **not mandatory**. It only makes sense for new store properties that have not been defined before within the `createStore`. If the value has already been initialized inside the `createStore` this parameter has no effect. | `const [price, setPrice] = useStore.cart.price(0)`                                                                                                                                                         |
| event after an update | `function` | This parameter is **not mandatory**. Adds an event that is executed every time there is a change inside the indicated store portion.                                                                                                           | `const [price, setPrice] = useStore.cart.price(0, onAfterUpdate)`<div><small>And the function:</small></div><div>`function onAfterUpdate({ store, prevStore }){ console.log({ store, prevStore }) }`</div> |

_Output:_

Is an `Array` with **3** items:

| name         | type       | description                                                                             | example                                                                                                                                                                                                                                                                                                                                                                 |
| ------------ | ---------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| value        | `any`      | The value of the store portion indicated with the proxy.                                | A store portion <div>`const [price] = useStore.cart.price()`</div>All store: <div> `const [store] = useStore()`</div>                                                                                                                                                                                                                                                   |
| update value | `function` | Function to update the store property indicated with the proxy.                         | Updating a store portion:<div>`const [count, setCount] = useStore.count(0)`</div>Way 1:<div>`setCount(count + 1)`</div>Way 1:<div>`setCount(c => c + 1)`</div><div>-------</div>Updating all store:<div>`const [store, updateStore] = useStore()`</div>Way 1:<div>`updateStore({ ...store, count: 2 }))`</div>Way 1:<div>`updateStore(s => ({ ...s, count: 2 }))`</div> |
| reset value  | `function` | Function that reset the store property indicated with the proxy to their initial value. | Reset store portion:<div>`const [,,resetCount] = useStore.count()`</div><div>`resetCount()`</div><small>_Put counter to 0 again (initial value defined inside the `createStore`)._</small><div>-------</div>Reset all store:<div>`const [,,resetStore] = useStore()`</div><div>`resetStore()`</div><small>_All store portions to their initial values._</small>         |

### getStore helper

It works exactly like `useStore` but with **some differences**:

- It **does not make a subscription**. So it is no longer a hook and you can use it as a helper wherever you want.
- It's **not possible to register events** that are executed after a change.

  ```js
  getStore.cart.price(0, onAfterPriceChange); // ❌

  function onAfterPriceChange({ store, prevStore }) {
    // ...
  }
  ```

  - If the intention is to register events that last forever, it has to be done within the `createStore`:

  ```js
  const { getStore } = createStore(initialStore, onAfterUpdate); // ✅

  function onAfterUpdate({ store, prevStore }) {
    // ..
  }
  ```

Very useful to use it:

- **Outside components**: helpers, services, etc.
- **Inside components**: Avoiding rerenders if you want to consume it inside events, when you only use the updaters `const [, setCount, resetCount] = getStore.count()`, etc.

Example:

```js
import { useState } from "react";

const { getStore } = createStore({ count: 0 });

function Example1() {
  const resetStore = getStore()[2];
  return <button onClick={resetStore}>Reset store</button>;
}

function Example2() {
  const [newCount, setNewCount] = useState();

  function saveIncreasedCount(e) {
    e.preventDefault();
    const [count, setCount] = getStore.count();
    if (newCount > count) setCount(newCount);
    else alert("You should increase the value");
  }

  return (
    <form onSubmit={saveIncreasedCount}>
      <input
        value={newCount}
        onChange={(e) => setNewCount(e.target.valueAsNumber)}
        type="number"
      />
      <button>Save the increased count value</button>
    </form>
  );
}
```

### withStore HoC

It's a wrapper of the `useStore` for non-functional components. Where you receive the same thing that the `useStore` hook returns inside `this.props.store`.

Example with a store portion:

```js
const { withStore } = createStore();

class Counter extends Component {
  render() {
    const [count, setCount, resetCount] = this.props.store;
    return (
      <div>
        <h1>{count}</h1>
        <button onClick={() => setCount((v) => v + 1)}>+</button>
        <button onClick={() => setCount((v) => v - 1)}>-</button>
        <button onClick={resetCount}>reset</button>
      </div>
    );
  }
}

// Similar to useStore.counter.count(0)
const CounterWithStore = withStore.counter.count(Counter, 0);
```

Example with all store:

```js
const { withStore } = createStore({ count: 0 });

class Counter extends Component {
  render() {
    const [store, setStore, resetStore] = this.props.store;
    return (
      <div>
        <h1>{store.count}</h1>
        <button onClick={() => setStore({ count: store.count + 1 })}>+</button>
        <button onClick={() => setStore({ count: store.count - 1 })}>-</button>
        <button onClick={resetStore}>reset</button>
      </div>
    );
  }
}

// Similar to useStore()
const CounterWithStore = withStore(Counter);
```

The **only difference** with the `useStore` is that instead of having 2 parameters (initialValue, onAfterUpdate), it has 3 where the **first one is mandatory** and the other 2 are not (**Component**, **initialValue**, **onAfterUpdate**).

## Register events after an update 🚦

It is possible to register an event after each update. This can be useful for validating properties, storing error messages, optimistic updates...

There are 2 ways to register:

- **Permanent** events: Inside `createStore`. This event will always be executed for each change made within the store.

  ```js
  export const { useStore, getStore } = createStore(
    initialStore,
    onAfterUpdate
  );

  function onAfterUpdate({ store, prevStore }) {
    // Add an error msg
    if (store.count > 99 && !store.errorMsg) {
      const [, setErrorMsg] = getStore.errorMsg();
      setErrorMsg("The count value should be lower than 100");
      return;
    }
    // Remove error msg
    if (store.count <= 99 && store.errorMsg) {
      const [, setErrorMsg] = getStore.errorMsg();
      setErrorMsg();
    }
  }
  ```

- **Temporal** events: Inside `useStore` / `withStore`. These events will be executed for each change in the store (or indicated portion) **only during the life of the component**, when the component is unmounted the event is removed.

  ```js
  function Count() {
    const [count, setCount] = useStore.count(0, onAfterUpdate);
    const [errorMsg, setErrorMsg] = useStore.errorMsg();

    // The event lasts as long as this component lives
    function onAfterUpdate({ store, prevStore }) {
      // Add an error msg
      if (store.count > 99 && !store.errorMsg) {
        setErrorMsg("The count value should be lower than 100");
        return;
      }
      // Remove error msg
      if (store.count >= 99 && store.errorMsg) {
        setErrorMsg();
      }
    }

    return (
      <>
        {errorMsg && <div className="erorMsg">{errorMsg}</div>}
        <div className="count">{count}</div>
        <button onClick={() => setCount((v) => v + 1)}>Increment</button>
      </>
    );
  }
  ```

## How to... 🧑‍🎓

### Add a new store property

You can use `useStore` / `getStore` / `withStore` even if the property does not exist inside the store, and create it on the fly.

```js
const { useStore } = createStore({ username: "Hamza" });

function CreateProperty() {
  const [price, setPrice] = useStore.cart.price(0); // 0 as initial value

  return <div>Price: {price}</div>;
}

function OtherComponent() {
  // store now is { username: 'Hamza', cart: { price: 0 } }
  const [store] = useStore();
  console.log(store.cart.price); // 0
  // ...
}
```

It's **not mandatory to indicate the initial value**, you can create the property in a following step with the updater.

```js
const { useStore } = createStore({ username: "Hamza" });

function CreateProperty() {
  const [cart, setCart] = useStore.cart();

  useEffect(() => {
    initCart();
  }, []);
  async function initCart() {
    const newCart = await fetch("/api/cart");
    setCart(newCart);
  }

  if (!cart) return null;

  return <div>Price: {cart.price}</div>;
}
```

### Reset a store property

You can use the 3th array item from `useStore` / `getStore` / `withStore`. It's a function to return the value to its initial value.

```js
const [item, setItem, resetItem] = useStore.item();
// ...
resetItem();
```

If you only want the reset function and not the value, we recommend using the `getStore` to avoid creating a subscription and avoid unnecessary rerenders.

```js
const [, , resetItem] = getStore.item();
// or...
const resetItem = getStore.item()[2];
```

### Reset all the store

The [same thing](#reset-a-store-property) works to reset the entire store to its initial value.

```js
const [store, setStore, resetStore] = useStore();
// ...
resetStore();
```

### Use more than one store

You can have as many stores as you want. The only thing you have to do is to use as many `createStore` as stores you want.

store.js

```js
import createStore from "kayax";

export const { useStore: useCart } = createStore({ price: 0, items: [] });
export const { useStore: useCounter } = createStore({ count: 0 });
```

Cart.js

```js
import { useCart } from "./store";

export default function Cart() {
  const [price, setPrice] = useCart.price();
  // ... rest
}
```

Counter.js

```js
import { useCounter } from "./store";

export default function Counter() {
  const [count, setCount] = useCounter.count();
  // ... rest
}
```

### Update several portions avoiding rerenders in the rest

If you do this it causes a rerender to all the properties of the store:

```js
// 😡
const [store, setStore] = useStore();
setStore({ ...store, count: 10, username: "" });
```

And if you do the next, you convert the whole store into only 2 properties (`{ count: 10, username: '' }`), and you will remove the rest:

```js
// 🥵
const [store, setStore] = useStore();
setStore({ count: 10, username: "" });
```

If you have to update several properties and you don't want to disturb the rest of the components that are using other store properties you can create a helper with `getStore`.

```js
export const { useStore, getStore } = createStore(initialStore);

export function setStore(fields) {
  Object.keys(fields).forEach((key) => {
    const setStoreField = getStore[key]()[1];
    setStoreField(fields[key]);
  });
}
```

And use it wherever you want:

```js
// 🤩
import { setStore } from "./store";

// ...
setStore({ count: 10, username: "" });
```

### Define calculated properties

It's possible to use the `getStore` together with the function that is executed after each update to have store properties calculated from others.

In this example the cart price value will always be a value calculated according to the array of items:

```js
export const { useStore, getStore } = createStore(
  {
    cart: {
      price: 0,
      items: [],
    },
  },
  onAfterUpdate
);

function onAfterUpdate({ store }) {
  const { items, price } = store.cart;
  const calculatedPrice = items.length * 3;

  // Price always will be items.length * 3
  if (price !== calculatedPrice) {
    const [, setPrice] = getStore.cart.price();
    setPrice(calculatedPrice);
  }
}
```

## Roadmap 🛣

- [x] React support
- [x] TypeScript support
- [ ] Vanilla JavaScript support
- [ ] Svelte support
- [ ] Vue support