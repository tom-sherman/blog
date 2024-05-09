---
title: "React Server Components for Terminal UIs"
slug: "tui-react-server-components"
createdAt: "2024-04-17"
status: "unlisted"
---

# React Server Components for Terminal UIs

I've been thinking about how we could use React Server Components to build terminal UIs. This is a bit of a thought experiment, but I think it's an interesting one.

## React for two worlds

Dan Abramov has this great analogy for RSCs: they're React components for two worlds. One world is on the server, the other is on the client.

This analogy is pretty obvious when it comes to the web. The server is the web server, the client is the browser. It's a little less obvious when it comes to terminal UIs. What is the server in this case? What is the client?

## Local server components

One way to think about it could be that the two worlds are two processes on the same machine. One process (the server) is async, it handles fetching data and co-ordinating the client. The other process (the client) is sync, it renders the UI and handles user input.

## Remote server components

Another way to think about it could be that the server components run on a central server, and the client components run on a local machine. This would match the way the web works.

Unfortunately our execution models probably don't allow for such a system in a secure way, at least not in Node.js. Matching the web exactly would mean downloading and executing the client components, which is a security risk. It may be possible to do this in a more secure way with a different runtime, eg. Deno.

## What about async client components?

The React team has suggested that async client components may be a thing in the future. This would allow us to fetch data on the client and render it in the same way we do on the server - this would possibly obviate the need for server components on the TUI
