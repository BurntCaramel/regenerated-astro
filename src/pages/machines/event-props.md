---
title: Listening to external events with YieldMachine
description: Listening to external events
layout: ../../layouts/MainLayout.astro
---

```js
function KeyShortcutListener(el: HTMLElement) {
  function isEscapeKey(readContext: ReadContextCallback) {
    const event = readContext("event");
    return event instanceof KeyboardEvent && event.key === "Escape";
  }
  function isEnterKey(readContext: ReadContextCallback) {
    const event = readContext("event");
    return event instanceof KeyboardEvent && event.key === "Enter";
  }

  const openChoiceKeydown = new Map([
    [isEscapeKey, Closed],
    [null, Open as any],
  ]);
  const closedChoiceKeydown = new Map([
    [isEnterKey, Open],
    [null, Closed as any],
  ]);

  function* Open() {
    yield on("keydown", openChoiceKeydown);
    yield listenTo(el, ["keydown"]);
  }
  function* Closed() {
    yield on("keydown", closedChoiceKeydown);
    yield listenTo(el, ["keydown"]);
  }

  return Closed;
}

const aborter = new AbortController();
const input = document.createElement("input");
const machine = start(KeyShortcutListener.bind(null, input), {
  signal: aborter.signal,
});
machine.value.state; // "Closed"

input.dispatchEvent(new KeyboardEvent("keydown", { key: "Enter" }));
expect(machine.value).toMatchObject({
  state: "Open",
  change: 1,
});

input.dispatchEvent(new KeyboardEvent("keydown", { key: "Enter" }));
expect(machine.value).toMatchObject({
  state: "Open",
  change: 1,
});

input.dispatchEvent(new KeyboardEvent("keydown", { key: "a" }));
expect(machine.value).toMatchObject({
  state: "Open",
  change: 1,
});

input.dispatchEvent(new KeyboardEvent("keydown", { key: "Escape" }));
expect(machine.value).toMatchObject({
  state: "Closed",
  change: 2,
});

input.dispatchEvent(new KeyboardEvent("keydown", { key: "a" }));
expect(machine.value).toMatchObject({
  state: "Closed",
  change: 2,
});
```
