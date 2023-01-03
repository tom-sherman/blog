---
slug: "remix-blog-syntax-highlighting"
title: "Adding Syntax Highlighting to My Remix Blog"
createdAt: "2023-01-03"
tags: ["remix", "javascript"]
---

# Adding Syntax Highlighting to My Remix Blog

This is my journey of adding syntax highlighting to my [Remix](https://remix.run) blog.

## Attempt 1: Prism and Highlight.js

- [Prism](https://prismjs.com/)
- [Highlight.js](https://highlightjs.org/)

I feel like these are the de-facto standard for syntax highlighting. They are both do a decent job of syntax highlighting and are able to run on both the client and server. This is great because it means the code blocks would be highlighted even if JavaScript is disabled and there is no flash of unstyled content.

They have a massive downside though in my opinion: the highlighting is really low fidelity. I spend all day in Visual Studio Code and I'm used to the highlighting being really good. I don't want to go back to the 90s.

## Attempt 2: Shiki

[Shiki](https://shiki.matsu.io/) is absolutely beautiful. It uses the TextMate grammars and VS Code themes to do the highlighting which means it's maximally accurate. This is exactly what I want.

The catch is that it requires the theme, grammar, and a WASM binary to be fetched at runtime. THis means that the code blocks will not be highlighted until the JavaScript has loaded and the WASM binary has been fetched. And we can't easily run it on the server.

Also by default it loads these assets from unpkg.com which has questionable availability and inconsistent performance.

## Attempt 2.1: Shiki with self hosted assets

We can solve the problem of the assets being fetched from unpkg.com by self hosting them. This is simple to do
