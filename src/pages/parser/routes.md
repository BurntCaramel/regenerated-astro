---
title: Parsing Routes
description: Parsing Routes
layout: ../../layouts/MainLayout.astro
---

We can define a generator function for each route we want.

## Approach A: Route components return values

```js
import { parse, mustEnd } from 'yieldparser';

function* Home() {
  yield '/';
  yield mustEnd;
  return { type: 'home' };
}

function* Blog() {
  yield '/blog';
  yield mustEnd;
  return { type: 'blog' };
}

function* BlogPost() {
  yield '/blog/';
  const [postID] = yield /([-_\w]+)/;
  yield mustEnd;
  return { type: 'blogPost', postID };
}

function* Route() {
  return yield [Home, Blog, BlogPost];
}

function handleRequest(request) {
  const url = new URL(request.url);
  const route = parse(url.pathname, Route());
  if (!route.success) {
    return new Response(`Not found: ${url.pathname}`, { status: 404 });
  }

  if (route.value.type === 'home') {
    return renderHomePage();
  } else if (route.value.type === 'blog') {
    return renderBlogList();
  } else if (route.value.type === 'blogPost') {
    return renderBlogPost(route.value.postID);
  }
}
```

----

## Approach B: Route components return response handlers

```js
import { parse, mustEnd } from 'yieldparser';

import { findPost, renderBlogIndexHTML, renderBlogPostHTML } from './blog';

const htmlPreamble = `<!doctype html><html lang=en><meta charset=utf-8><body>`;

function* Home() {
  yield '/';
  yield mustEnd;

  return () => ({ title: 'Home', html: `<h1>Home page<h1>` });
}

function* Blog() {
  yield '/blog';
  yield mustEnd;

  return async () => {
    return { title: 'Blog', html: await renderBlogHTML() };
  };
}

function* BlogPost() {
  yield '/blog/';
  const [postID] = yield /([-_\w]+)/;
  yield mustEnd;

  return async () => {
    const post = await findPost(postID);
    return { title: post.title, html: renderBlogPostHTML(post) };
  };
}

function* Route() {
  return yield [Home, Blog, BlogPost];
}

async function handleRequest(request) {
  const url = new URL(request.url);
  const route = parse(url.pathname, Route());
  if (!route.success) {
    return new Response(`Not found: ${url.pathname}`, { status: 404 });
  }

  const { title, html } = await route.value;
  return new Response(`<!doctype html><html lang=en><meta charset=utf-8><title>${title}</title><body>${html}`, {
    status: 200,
    headers: new Headers({ 'content-type': 'text/html;charset=UTF-8' })
  });
}
```