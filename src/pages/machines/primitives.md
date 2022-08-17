---
title: Using primitives with YieldMachine
description: Increment numbers and use symbols as states
layout: ../../layouts/MainLayout.astro
---

While it’s fantastic to have finite states, we often need to model more than that.

Yield Machine lets you use JavaScript primitives — booleans, numbers & symbols — to model your states.

This lets you create states that are similar to what you’d make using React’s `useReducer` hook.

## Boolean toggle

```ts
function* Toggle() {
  yield on(
    "toggle",
    map((current: boolean) => !current)
  );

  return false;
}

const machine = start(Toggle);
machine.value.state; // false
machine.next("toggle");
machine.value.state; // true
machine.next("toggle");
machine.value.state; // false
```

Here the initial state is `false`. We listen to the `toggle` event and perform a “not” operation on the current state going from `false` when `true`, and from `true` when `false`.

## Incrementing numbers

Here is a counter, now world famous as a demonstration of state libraries:

```ts
function* Counter() {
  yield on(
    "increment",
    map((n: number) => n + 1)
  );

  return 0;
}

const machine = start(Counter);
machine.value.state; // 0
machine.next("increment");
machine.value.state; // 1
machine.next("increment");
machine.value.state; // 2
```

We can go further though. Wouldn’t it be great if we can increment by chunks of ten at a time?

```ts
function* Counter() {
  yield on(
    "+1",
    map((n: number) => n + 1)
  );
  yield on(
    "+10",
    map((n: number) => n + 10)
  );

  return 0;
}

const machine = start(Counter);
machine.value.state; // 0
machine.next("+1");
machine.value.state; // 1
machine.next("+1");
machine.value.state; // 2
machine.next("+10");
machine.value.state; // 12
machine.next("+10");
machine.value.state; // 22
machine.next("+1");
machine.value.state; // 23
```

## Symbol states

```ts
const Closed = Symbol("closed");
const File = Symbol("file");
const Edit = Symbol("edit");
const View = Symbol("view");

function* Menu() {
  function closeIfCurrent(id: symbol) {
    return map((current: symbol) => (current === id ? Closed : id));
  }
  yield on("file", closeIfCurrent(File));
  yield on("edit", closeIfCurrent(Edit));
  yield on("view", closeIfCurrent(View));
  yield on(
    "close",
    map(() => Closed)
  );

  return Closed;
}

const machine = start(Menu);
machine.value.state; // Closed
machine.next("file");
machine.value.state; // File
machine.next("close");
machine.value.state; // Closed
machine.next("file");
machine.value.state; // File
machine.next("file");
machine.value.state; // Closed
machine.next("file");
machine.value.state; // File
machine.next("edit");
machine.value.state; // Edit
machine.next("view");
machine.value.state; // View
machine.next("edit");
machine.value.state; // Edit
machine.next("edit");
machine.value.state; // Closed
machine.next("close");
machine.value.state; // Closed
```

## Coming soon? — your own custom state

```js
function* Counter() {
  yield on("increment", (n: number) => n + 1);

  return 0;
}

const On = Symbol("On");
const Off = Symbol("Off");
function* Switch() {
  yield on("flick", (current: symbol) => (current === On ? Off : On));

  return 0;
}
```
