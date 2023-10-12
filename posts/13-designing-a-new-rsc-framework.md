---
title: "Designing a new Server Component framework"
slug: "designing-a-new-rsc-framework"
createdAt: "2023-10-11"
---

# Designing a new React Server Component framework

Notes on designing a server component framework that is runtime and language agnostic.

## What does that even mean?

{% tweet link="https://twitter.com/acdlite/status/1709973074387283980" %}

Imagine you're a Rails developer, or a Laravel developer, or you'd prefer to deploy only Rust on your server for some reason. You might, for various reasons, have an aversion for server-side JavaScript.

Our theoretical framework would allow you to build the entire UI-layer of your app with the React programming model while still taking advantage of your runtime and language of choice.

In this framework you'd be able to write your server components in any language you like, and then sprinkle in client components written in JS.

## Server Side Rendering

We would of course only be able to render our server components to HTML. SSR-ing the JS components isn't available to us because our constraint is that we don't want to run JS code on the server.

This means we need to be very diligent with pushing our client components down to the leaves of our application, otherwise half of the page wouldn't be able to be server side rendered and we'd fallback to the experience of a single page application on the first load.

## Suspense boundaries

We would also need to use Suspense boundaries within the server components to wrap client components in most cases. A client component is where HTML rendering stops, we need a suspense boundary to contain that blast radius.

It might make sense for this framework to enforce wrapping every client component in its own suspense boundary, although this might take away some design control.

## Two process architecture

_There has to be a better name for this..._

1. A client side portion that includes the bundler, router, and network wiring. This would be the more complex part but wouldd be agnostic to the server side.
2. A server side portion that is specific to the language and runtime

## Multi language

Our framework must extend to multiple languages. If we only support one then it misses the point of a shared understanding.

This might require several different implementations on the server side of the framework. It might be possible though to have some shared core in eg. portable Rust.

## Bundler

We would have hooks into and out of the bundler for the server side portion to interact with.

We need to use the bundler output to load client references (stubs for client components). We also need to output information to the bundler to support things like server actions.
