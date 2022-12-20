---
slug: "mastodon-domain-alias-on-remix"
title: "Remix: Creating a Mastodon alias with your custom domain"
createdAt: "2022-12-20"
tags: ["typescript", "javascript", "remix", "mastdodon"]
---

# Remix: Creating a Mastodon alias with your custom domain

https://fosstodon.org/@tomsherman/109547084606024642

This technique allows me to use my custom domain as an alias to my Mastodon without having to host a fully fledged Mastodon instance. In practice it allows people to tag me in posts with `@me@tom-sherman.com` and also search `me@tom-sherman.com` to find me on Mastodon.

The solution is pretty easy in Remix, you just need to create a new resource route that returns some JSON. That route needs to be at `/.well-known/webfinger` and return a JSON object that looks like this:

```json
{
  "subject": "...",
  "aliases": ["..."],
  "links": [
    {
      "rel": "self",
      "type": "application/activity+json",
      "href": "..."
    }
  ]
}
```

Remix file system routing gets in the way a bit here, the route needs to be created at `app/routes/[.]well-known/webfinger.ts` (note the `.` in the filename is surrounded by square brackets).

Mastodon instances will send a GET request to `/.well-known/webfinger` with a query parameter `resource` that contains the username it's querying for. We just need to detect a resource query paramater of me@tom-sherman.com and return the JSON object above. Here's the full code, with some hopefully helpful comments:

```ts
import type { LoaderArgs } from "@remix-run/cloudflare";

export function loader({ request }: LoaderArgs) {
  const url = new URL(request.url);
  const resourceQuery = url.searchParams.get("resource");

  if (!resourceQuery) {
    // We don't care about requests that aren't from mastodon, so we can bail out at this point
    return new Response("Missing resource query parameter", {
      status: 400,
    });
  }

  if (resourceQuery !== "acct:me@tom-sherman.com") {
    // You may want to tweak this code path, but essentially I didn't want every username on tom-sherman.com to point to my Mastodon profile.
    // If you want to do that, you can remove this if statement and the code will work as expected. You can also include other usernames in the check if you want multiple aliases.
    return new Response("Not found", {
      status: 404,
    });
  }

  return new Response(
    JSON.stringify({
      // Replace this with your Mastodon handle
      subject: "acct:tomsherman@fosstodon.org",
      aliases: [
        // Put links to your mastodon profile here
        "https://fosstodon.org/@tomsherman",
        "https://fosstodon.org/users/tomsherman",
      ],
      links: [
        {
          rel: "http://webfinger.net/rel/profile-page",
          type: "text/html",
          // Change this to your Mastodon profile
          href: "https://fosstodon.org/@tomsherman",
        },
        {
          rel: "self",
          type: "application/activity+json",
          // Change this to your Mastodon profile
          href: "https://fosstodon.org/users/tomsherman",
        },
        {
          rel: "http://ostatus.org/schema/1.0/subscribe",
          // Change this to your Mastodon instance
          template: "https://fosstodon.org/authorize_interaction?uri={uri}",
        },
      ],
    }),
    {
      headers: {
        "content-type": "application/jrd+json",
      },
    }
  );
}
```

Full code on GitHub [here](https://github.com/tom-sherman/tom-sherman.com/blob/main/app/routes/%5B.%5Dwell-known/webfinger.ts).
