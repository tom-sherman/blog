---
slug: "type-safe-remix-cloudflare-loader-context"
title: "Type safe context in Remix on Cloudflare Pages"
createdAt: "2023-02-11"
tags: ["remix", "typescript"]
---

# Type safe context in Remix on Cloudflare Pages

This is a pretty simple tip, but there wasn't a lot of documentation on how to do this, so I thought I'd share.

Cloudflare Pages passes [bindings](https://developers.cloudflare.com/pages/platform/functions/bindings/) to each Pages Function so you can access secrets, variables, and databases. You can distribute this context to each of your Remix loaders and actions with the `getLoadContext` option in `server.js`, I think in the Cloudflare Pages template the `server.js` file looks like this:

```ts
// server.js
import { createPagesFunctionHandler } from "@remix-run/cloudflare-pages";
import * as build from "@remix-run/dev/server-build";

const handleRequest = createPagesFunctionHandler({
  build,
  mode: process.env.NODE_ENV,
  getLoadContext: (context) => context.env,
});

export function onRequest(context) {
  return handleRequest(context);
}
```

The issue here is that TypeScript doesn't know the type of `context`, and so in each of the loaders and actions it'll be `any`. To fix this, we can convert the file to TypeScript (rename to `server.ts`) and make some adjustments.

First, we can add a type for the Cloudflare Function event context. We can add key/value pairs for each of our function bindings:

```ts
interface Env {
  GITHUB_TOKEN: string;
  DB: D1Database;
}

type Context = EventContext<Env, string, unknown>;
```

Next up is using TypeScript [declaration merging](https://www.typescriptlang.org/docs/handbook/declaration-merging.html) to override what the internal type of the Remix context is:

```ts
declare module "@remix-run/server-runtime" {
  interface AppLoadContext extends Env {}
}
```

And finally we can make typescript happy by giving it a type for our context value:

```ts
const handleRequest = createPagesFunctionHandler({
  build,
  mode: process.env.NODE_ENV,
  getLoadContext: (context: Context) => ({
    DB: context.env.DB,
    GITHUB_TOKEN: context.env.GITHUB_TOKEN,
  }),
});

export function onRequest(context: Context) {
  return handleRequest(context);
}
```

The full code should look like this:

```ts
import { createPagesFunctionHandler } from "@remix-run/cloudflare-pages";
import * as build from "@remix-run/dev/server-build";

interface Env {
  GITHUB_TOKEN: string;
  DB: D1Database;
}

type Context = EventContext<Env, string, unknown>;

declare module "@remix-run/server-runtime" {
  interface AppLoadContext extends Env {}
}

const handleRequest = createPagesFunctionHandler({
  build,
  mode: process.env.NODE_ENV,
  getLoadContext: (context: Context) => ({
    DB: context.env.DB,
    GITHUB_TOKEN: context.env.GITHUB_TOKEN,
  }),
});

export function onRequest(context: Context) {
  return handleRequest(context);
}
```

Now we can use the context in our loaders and actions:

```ts
export async function loader({ context }: DataFunctionArgs) {
  const { DB, GITHUB_TOKEN } = context;
  // ...
}
```
