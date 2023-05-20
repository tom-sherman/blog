---
title: "Dynamic Open Graph Images with Cloudflare Workers"
slug: "dynamic-og-image-cloudflare-workers"
createdAt: "2022-12-26"
tags: ["typescript", "workers", "cloudflare"]
---

# Dynamic Open Graph Images with Cloudflare Workers

This blog dynamically renders Open Graph images using HTML and React, running on Cloudflare Workers. The rough steps are:

1. Use React (JSX) to define a component that renders some HTML
2. Use [Yoga](https://github.com/facebook/yoga) and [Satori](https://github.com/vercel/satori) to convert that HTML to SVG
3. Render the SVG to PNG using [resvg](https://github.com/RazrFalcon/resvg)

All three of these steps are done in either JS (1) or WASM (2 & 3), so they can be run in a Cloudflare Worker. The final result is a PNG image that can be used as an Open Graph image.

All of these steps are wrapped up in a single class `ImageResponse` that takes a React component and instantiates a `Response` object with the rendered PNG in the body. This class comes from the great [workers-og](https://github.com/kvnang/workers-og) project, which wraps all of steps above.

```tsx
new ImageResponse(<OgImage title={title} description={description} />, {
  width: 800,
  height: 600,
  fonts: [
    {
      family: "Roboto",
      data: await fetch(
        "https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap"
      ).then((res) => res.arrayBuffer()),
      weight: 400,
    },
  ],
});
```

## Props

The component needs to be dynamic, so it needs to accept props which can be passed to the workers via a query string eg.

```
/og-image?title=Hello%20World&description=This%20is%20a%20description
```

URL encoding makes me a little nervouse here (what if the title contains some special characters?) as does the the length (Cloudflare may have a limit on URL length, but I haven't found any documentation on this).

## Caching

While testing this I was coming up against slow response times. I tracked this down to the fact that I'm loading an external font (Roboto) from Google Fonts. This is a slow request, and it's happening on every request. To fix this I changed the code to load the font from a Cloudflare CDN, which should reduce latency, and I also cached this using the Cloudflare Cache API.

```tsx
async function fetchFont(url: string, ctx: Context) {
  const cache = await caches.open("fonts");
  const cached = await cache.match(url);
  if (cached) {
    console.log("font cache hit");
    return cached.arrayBuffer();
  }
  console.log("font cache miss");
  const response = await fetch(url);
  ctx.waitUntil(cache.put(url, response.clone()));

  return response.arrayBuffer();
}

new ImageResponse(<OgImage title={title} description={description} />, {
  width: 800,
  height: 600,
  fonts: [
    {
      family: "Roboto",
      url: await fetchFont(
        "https://cdnjs.cloudflare.com/ajax/libs/materialize/0.98.1/fonts/roboto/Roboto-Regular.ttf",
        context
      ),
      weight: 400,
    },
  ],
});
```

We use `waitUntil` to not block the response and to tell the worker to cache the font in the background.

Another problem I noticed was that sometimes I would hit the Cloudflare CPU time limit (10ms on the free plan). This I can only solve partly, by caching the whole `ImageResponse` using the same Cache API. This will reduce the CPU time on subsequent requests, but it will still be slow on the first request and could hit the timeout.

## Code

The full code is available on GitHub: https://github.com/tom-sherman/og-image-worker
