---
title: "Remix: Satisfying TypeScript with the `satisfies` keyword"
slug: "remix-typescript-satisfies"
createdAt: "2022-11-23"
tags: ["typescript", "javascript", "remix"]
---

# Remix: Satisfying TypeScript with the `satisfies` keyword

## TL;DR - show me the code!

TypeScript 4.9 adds a new keyword: `satisfies`. You can read about it on [the blog post](https://devblogs.microsoft.com/typescript/announcing-typescript-4-9/#satisfies) but the abridged version is that it allows you to specify a value conforms to a specific type, but without changing (either widening or narrowing) the type of the value.

We can use this to our advantage when defining Remix loaders.

```ts
type Loader = (args: DataFunctionArgs) => Promise<Response> | Response;

export const loader = (async () => new Response("Hello, world!")) satisfies Loader;
```

## Explanation of the problem

Let's first look at an example that we would like to have a type error:

```tsx
export const loader = () => {
  const message = "hello world";
};

export default function Route() {
  const message = useLoaderData<typeof loader>();
  return <h1>{message}</h1>;
}
```

In this example, we would get the dreaded and often confusing error:

> Error: You defined a loader for route "routes/route" but didn't return anything from your `loader` function. Please return a value or `null`.

Another scenario where this can be annoying is if you have a loader like this:

```ts
export const loader = () => {
  json({ message: "hello world" });
};
```

You would get a type error when you try to access `useLoaderData<typeof loader>().message`, but it won't point you to the missing return. It will just say that `message` doesn't exist on `never`.

## A first attempt

This is an error that should be easy to catch at compile time. A first attempt at doing this could be to add a return type to the function - but what should that type be? For the `useLoaderData<typeof loader>` technique to work correctly we need to return a `TypedResponse`, which in my opinion is a little unsightly:

```ts
export const loader = (): TypedResponse<{ message: string }> => {
  // Now it's obvious we need to put a return here
  return json({ message: "hello world" });
};
```

It can also be really annoying to have to change the type annotation all the time. The DX of having `useLoaderData<typeof loader>` infer the type is really nice and it's a shame to lose it.

Another albeit minor problem with this is that we still have to type the function arguments, further increasing the visual noise.

```ts
export const loader = ({ request }): TypedResponse<{ message: string }> => {
  return json({ message: request.status });
};
```

## The `satisfies` keyword

With the `satisfies` keyword we can regain the DX lost by adding these type annotations:

```ts
type Loader = (args: DataFunctionArgs) => Promise<Response> | Response;

export const loader = (({ request }) => {
  return json({ message: "hello world" });
}) satisfies Loader;
```

Some notes here:

- We define our own `Loader` type. This can be as specific as you like for a set of loaders you want to use it on. Above I've defined the most general version, but you may want for this type to only return `Promise<Response>` for example. Or even `Promise<TypedResponse<Record<string, unknown>>>` to ensure it always returns an object JSON response.
- We must define our loader as an arrow function. `satisfies` only works on expressions, it doesn't work on function declarations.
- We had to introduce some parenthesis around our arrow function. Without them TypeScript wouldn't know which part of the expression to apply the `satisfies` keyword to. It looks strange at first, but it's not too bad.
