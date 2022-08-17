---
title: YieldMachine
description: State Machines with Generator Functions
layout: ../../layouts/MainLayout.astro
---

- [GitHub repo](https://github.com/JavaScriptRegenerated/yieldmachine)
- [npm package](https://www.npmjs.com/package/yieldmachine)

## Installation

```bash
npm add yieldmachine
```

## Defining States

Here’s an example machine named `Switch`. It has two states: `Off` and `On`.

```js
function Switch() {
  function* Off() {
    yield on("flick", On);
  }
  function* On() {
    yield on("flick", Off);
  }

  return Off;
}
```

We return `Off` to make it the initial state.

Let’s see our machine in action:

```js
const machine = start(Switch);
machine.value.state; // "Off"

machine.next("flick");
machine.value.state; // "On"

machine.next("flick");
machine.value.state; // "Off"
```

So what’s going on here? Why are we nesting functions, and how does Yield Machine work?

When you call `start()` and pass your machine definition function, that function will be called.

It might be clearer if we pass our machine definition directly as an arrow function:

```js
const machine = start(() => {
  function* Off() {
    yield on("flick", On);
  }
  function* On() {
    yield on("flick", Off);
  }

  return Off;
});
machine.value.state; // "Off"

machine.next("flick");
machine.value.state; // "On"

machine.next("flick");
machine.value.state; // "Off"
```

So your function is called, and then the return value is used — which here is a reference to the inner function `Off`. This is how the initial state — available at `machine.value.state` — is populated.

Yield Machine sees that `Off` is a function and then calls it. Since it is a [generator function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*), it returns a [generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) which is able to be iterated through.

The `Off` state definition has one thing for iteration: the result of `on("flick", On)`. This defines a state transition. It declares that when the "flick" event is received, then transition to the `On` state.

```js
  function* Off() {
    yield on("flick", On);
  }
```

This only applies when we are currently in the `Off` state, which is why it is within the `Off` generator function. So when the machine is currently in the `Off` state and the `flick` event is sent, then it will transition to the `On` state.

We sent that event by running:

```js
machine.next("flick");
```

And then we’ll see that it has changed states to `On`:

```js
machine.value.state; // "On"
```

When Yield Machine makes the transition, it calls the new state definition’s function. So it will now call `On` and again see that it is a generator function returning a generator to be iterated through.

```js
  function* On() {
    yield on("flick", Off);
  }
```

Here it declares that when we are in the `On` state and the `flick` event is received, then transition to the `Off` state:

```js
machine.value.state; // "On"
machine.next("flick");
machine.value.state; // "Off"
```

Again it will now have called the `Off` state definition function and have registered its `flick` event transition. And since it decided to transition to `On`, then our machine will go back and forth between the two states:

```js
machine.value.state; // "Off"
machine.next("flick");
machine.value.state; // "On"
machine.next("flick");
machine.value.state; // "Off"
machine.next("flick");
machine.value.state; // "On"
```

----

## Defining Primitives

```js
function ClickedState(button) {
  function* Initial() {
    yield on("click", Clicked);
    yield listenTo(button, "click");
  }
  function* Clicked() {}

  return Initial;
}
```

## Nesting Machines
