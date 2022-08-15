---
title: State Machines
description: State Machines with Generator Functions
layout: ../../layouts/MainLayout.astro
---

<template id=examples-template>
    <style>
        :host { display: block; padding: 1rem; }
        [data-result] { padding: 0.25em 0.5em; background: #fff3; border-radius: 4px; }
    </style>
    <output><slot name=result><code data-result>loading…</code></slot></output>
    <div style="margin-top: 1rem">
      <slot name=mainElement></slot>
    </div>
</template>

<style>
  button, summary {
    cursor: pointer;
  }
  button {
    background: black;
    color: white;
  }
</style>

Library used: [yieldmachine](https://github.com/JavaScriptRegenerated/yieldmachine)

## Click

<machines-example machine="ClickedState">
    <button slot=mainElement type=button>Click Listener</button>
</machines-example>

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

---

## Focus

<machines-example machine="FocusState">
  <textarea slot=mainElement></textarea>
</machines-example>

```js
function* FocusState(el) {
  const checkingFocused = new Map([
    [() => el.ownerDocument.activeElement === el, Active],
    [null, Inactive],
  ]);

  function* Active() {
    yield listenTo(el.ownerDocument, ["focusin"]);
    yield listenTo(el, ["blur"]);
    yield on("focusin", checkingFocused);
    yield on("blur", checkingFocused);
  }
  function* Inactive() {
    yield listenTo(el, ["focus"]);
    yield on("focus", Active);
  }

  return checkingFocused;
}
```

---

## Details

<machines-example machine="DetailsListener">
    <details slot=mainElement>
        <summary>Click to toggle</summary>
        <div><em>Some details that are shown only when expanded.</em></div>
    </details>
</machines-example>

```js
function* DetailsListener(el) {
  yield listenTo(el, ["toggle"]);
  yield on("toggle", compound(CheckingOpen));

  function* Closed() {}
  function* Open() {}
  function* CheckingOpen() {
    yield cond(el.open, Open);
    yield always(Closed);
  }

  return CheckingOpen;
}
```

---

## Document Visibility

<machines-example machine="DocumentVisibilityListener">
    <div slot=mainElement><em>This listens to the <code>visibilitychange</code> event. Switch between tabs to trigger it.</em></div>
</machines-example>

```js
function* DocumentVisibilityListener() {
  yield listenTo(document, ["visibilitychange"]);
  yield on("visibilitychange", compound(Checking));

  function* Visible() {}
  function* Hidden() {}
  function* Checking() {
    yield cond(document.visibilityState === "visible", Visible);
    yield always(Hidden);
  }

  return Checking;
}
```

---

## Fetch

```js
import { entry, on, start } from "yieldmachine";

const exampleURL = new URL("https://example.org/");
function fetchData() {
  return fetch(exampleURL);
}

// Define a machine just using functions
function Loader() {
  // Each state is a generator function
  function* idle() {
    yield on("FETCH", loading);
  }
  // This is the ‘loading’ state
  function* loading() {
    // This function will be called when this state is entered.
    // Its return value is available at `loader.results.fetchData`
    yield entry(fetchData);
    // If the promise succeeds, we will transition to the `success` state
    // If the promise fails, we will transition to the `failure` state
    yield on("SUCCESS", success);
    yield on("FAILURE", failure);
  }
  // States that don’t yield anything are final
  function* success() {}
  // Or they can define transitions to other states
  function* failure() {
    // When the RETRY event happens, we transition from ‘failure’ to ‘loading’
    yield on("RETRY", loading);
  }

  // Return the initial state from your machine definition
  return idle;
}

const loader = start(Loader);
loader.current; // "idle"

loader.next("FETCH");
loader.current; // "loading"

loader.results.then((results) => {
  console.log("Fetched", results.fetchData); // Use response of fetch()
  loader.current; // "success"
});
```

<script type="module" is:inline>
import {
  start,
  on,
  compound,
  listenTo,
  entry,
  cond,
  always,
  accumulate,
} from 'https://cdn.jsdelivr.net/npm/yieldmachine@0.5.1/dist/yieldmachine.module.js';

function ClickedState(button) {
  console.log("button", button);
  function* Initial() {
    yield on('click', Clicked);
    yield listenTo(button, ['click']);
  }
  function* Clicked() {}

  return Initial;
}

function* FocusState(el) {
  console.log("Focus state", el);
  const checkingFocused = new Map([
    [() => {
      console.log("checking focused", el.ownerDocument.activeElement, el);
      return el.ownerDocument.activeElement === el
    }, Active],
    [null, Inactive],
  ]);

  function* Active() {
    yield listenTo(el.ownerDocument, ["focusin"]);
    yield listenTo(el, ["blur"]);
    yield on("focusin", checkingFocused);
    yield on("blur", checkingFocused);
  }
  function* Inactive() {
    yield listenTo(el, ["focus"]);
    yield on("focus", Active);
  }

  return checkingFocused;
}

function* DetailsListener(el) {
  yield listenTo(el, ['toggle']);
  yield on('toggle', compound(CheckingOpen));

  function* Closed() {}
  function* Open() {}
  function* CheckingOpen() {
    yield cond(el.open, Open);
    yield always(Closed);
  }

  return CheckingOpen;
}

function* DocumentVisibilityListener() {
  yield listenTo(document, ['visibilitychange']);
  yield on('visibilitychange', compound(Checking));

  function* Visible() {}
  function* Hidden() {}
  function* Checking() {
    yield cond(document.visibilityState === 'visible', Visible);
    yield always(Hidden);
  }

  return Checking;
}

const machineRegistry = new Map();
machineRegistry.set('ClickedState', ClickedState);
machineRegistry.set('FocusState', FocusState);
machineRegistry.set('DetailsListener', DetailsListener);
machineRegistry.set('DocumentVisibilityListener', DocumentVisibilityListener);

class MachinesExample extends HTMLElement {
  constructor() {
    super();

    const templateEl = document.getElementById('examples-template');
    const template = templateEl.content;
    const clone = template.cloneNode(true);
    // const clone = this.ownerDocument.importNode(template, true);
    // const clone = this.ownerDocument.createRange().createContextualFragment(templateEl.innerHTML);
    const shadowRoot = this.attachShadow({ mode: 'open' });
    shadowRoot.appendChild(clone);
    const [mainElement] = shadowRoot.querySelector('slot[name=mainElement]').assignedElements();
    const [outputEl] = shadowRoot.querySelector('slot[name=result]').assignedElements({ flatten: true });
    
    const machineName = this.getAttribute('machine');
    const machineDefinition = machineRegistry.get(machineName);
    if (!machineDefinition) {
      console.error("No machine defined with name", machineName);
    }

    const abortController = new AbortController;
    const machine = start(machineDefinition.bind(null, mainElement));
    
    machine.eventTarget.addEventListener('StateChanged', this);
    
    Object.assign(this, { machine, outputEl });
    Object.preventExtensions(this);
  }
  
  connectedCallback() {
    this.update();
  }

  disconnectedCallback() {
    this.machine.abort();
  }

  handleEvent(event) {
    this.update();
  }

  update() {
    this.outputEl.textContent = `${this.machine.value.state} ${this.machine.value.change}`;
  }
}
customElements.define('machines-example', MachinesExample);
</script>
