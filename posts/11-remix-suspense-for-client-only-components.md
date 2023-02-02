---
slug: "remix-suspense-for-client-only-components"
title: "Use Suspense for client-only components"
createdAt: "2023-02-02"
tags: ["remix", "javascript", "react"]
---

React 18 has this [cool pattern](https://beta.reactjs.org/reference/react/Suspense#providing-a-fallback-for-server-errors-and-server-only-content) for rendering content only on the client:

```tsx
export default function SomeRoute() {
  return (
    <Suspense fallback={<Loading />}>
      <Chat />
    </Suspense>
  );
}

function Chat() {
  if (typeof window === "undefined") {
    throw Error("Chat should only render on the client.");
  }
  // ...
}
```

Here we use a `<Suspense>` boundary to render a loading component on the server, and then render the `<Chat>` component on the client. This works because React will retry rendering trees inside a suspense boundary on the client after they throw an error on the server.

This is great for Remix apps, especially if you're using client-only components. We can move our chat component into `chat.client.tsx` to ensure that it's never included in our server bundle.

```tsx
import Chat from "~/components/chat.client";

export default function SomeRoute() {
  return (
    <Suspense fallback={<Loading />}>
      <Chat />
    </Suspense>
  );
}
```

What's great about this pattern is that we can seamlessly transition to a `lazy()` component, which will ensure the code is codesplit from our main client bundle, leading to faster initial page loads when the chat component is large.

```tsx
import { lazy } from "react";

const Chat = lazy(() => import("~/components/chat.client"));

export default function SomeRoute() {
  return (
    <Suspense fallback={<Loading />}>
      <Chat />
    </Suspense>
  );
}
```

And finally, there are some additional benefits here with regards to hydration. I won't go into all of these in this blog post, but you can read about them in the [React 18 architecture overview](https://github.com/reactwg/react-18/discussions/37).
