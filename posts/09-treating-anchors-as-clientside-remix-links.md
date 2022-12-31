---
slug: "treating-anchors-as-clientside-remix-links"
title: "Remix: Treating anchors as client-side links"
createdAt: "2023-01-01"
tags: ["remix", "javascript"]
status: "unlisted"
---

# Remix: Treating anchors as client-side links

Sometimes our app renders a `<a>` anchor element but we want it to work with a client-side navigation instead of a full page refresh. We can emulate what `<Link>` does in Remix by intercepting these clicks.

Let's dive right into the code:

```ts
import { useNavigate } from "@remix-run/react";
import { useEffect } from "react";

export function useClientNavigationLinks() {
  const navigate = useNavigate();

  useEffect(() => {
    const controller = new AbortController();
    document.addEventListener(
      "click",
      (event) => {
        let target = event.target;
        // Find the first parent anchor element
        while (target && target !== document) {
          if (target instanceof HTMLAnchorElement) {
            break;
          }
          target = (target as any)?.parentNode;
        }

        if (!(target instanceof HTMLAnchorElement)) return;

        const url = new URL(target.href, location.origin);
        if (
          url.origin === window.location.origin &&
          // Ignore clicks with modifiers
          !event.ctrlKey &&
          !event.metaKey &&
          !event.shiftKey &&
          // Ignore right clicks
          event.button === 0 &&
          // Ignore if `target="_blank"`
          [null, undefined, "", "self"].includes(target.target)
        ) {
          console.log(
            "Treating anchor as <Link> and navigating to:",
            url.pathname
          );
          event.preventDefault();
          navigate(url.pathname + url.search + url.hash);
        }
      },
      { signal: controller.signal }
    );

    return () => controller.abort();
  });
}
```

This hook will intercept all clicks on anchor elements and if they are internal links, it will call `navigate` instead of letting the browser do a full page refresh. It handles several edge cases:

- Supports markup like `<a><span>Click me</span></a>`. It will find the first parent anchor element on whatever the user clicks.
- It ignores clicks with modifiers (ctrl, meta, shift). This allows the user to open the link in a new tab or copy the link.
- Ignores right clicks for the same reason.
- Ignores links that are marked as `target="_blank"`. We want these to work as normal links and open in a new tab.

## Usage

Just call `useClientNavigationLinks()` in your app root component, or in any route component that renders anchor elements (if you don't want to apply it everywhere).

---

Props to [Jacob Ebey](https://twitter.com/ebey_jacob/status/1606831880208551936) for the idea! I extended the snippet in that Tweet to handle more edge cases (listed above).
