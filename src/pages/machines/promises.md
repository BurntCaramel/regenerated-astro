---
title: Nesting machines with YieldMachine
description: Compose simple machines into a larger system
layout: ../../layouts/MainLayout.astro
---

## Promises are state machines

A promise is an eventual value with three states: pending, resolved, and rejected.

That makes them excellent for composing within other state machines.

```js
const isAuthorized = fetchIsAuthorized();
const isAdmin = fetchIsAdmin();

const Pending = Symbol("pending");

function* UserStatus() {
  yield nest("isAuthorized", promiseMachine(isAuthorized, Pending));
  yield nest("isAdmin", promiseMachine(isAdmin, Pending));
}

const machine = start(UserStatus);
machine.value.state.isAuthorized; // Pending
machine.value.state.isAdmin; // Pending
machine.value.change; // 0

setTimeout(() => {
  machine.value.state.isAuthorized; // true
  machine.value.state.isAdmin; // false
  machine.value.change; // 2
}, 5000);
```

## Promise wrapper

```js
function promiseMachine(promise, initialState) {
  return function* Machine() {
    yield on("resolve", (value) => value);
    yield on("reject", (error) => error);

    return initialState;
  }
}
```

## Rejections

## Fetch
