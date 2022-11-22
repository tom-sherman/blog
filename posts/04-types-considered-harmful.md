---
slug: "types-consdered-harmful"
title: "`@types` Considered Harmful"
createdAt: "2022-09-03"
tags: ["typescript", "javascript", "open-source"]
---

# `@types` Considered Harmful

> and I think we as a TypeScript community should actively discourage their usage.

Let me start by telling you a very short story:

Today I fixed a bug. The bug was caused by passing the [wrong prop](https://github.com/facebook/react-native/commit/18132c159773c6cedb2f74c7cf92a10987ea03d7) to a React Native component. I was passing `autoCompleteType`, the component wanted `autoComplete`.

> But don't you use TypeScript? Wouldn't this be a type error?

Absololutely, but on this occasion the types were distributed via a seperate channel - `@types/react-native` - and I, dear reader, forgot to upgrade the types at the same time as I upgrades the `react-native` package.

In an ideal world this example would convince you that any JS library worth taking remotely seriously should distribute their types in the same package as the code they relate to. I've worked a bit with Deno and I can tell you this really is an amazing world to live in. But to live in that world, we must kill all `@types/` packages.

If you're still not convinced, here are some other reasons:

- **Another repository to raise issues in.** "Do I raise this in the package or the DefinitelyTyped repo?", this kind of friction can prevent helpful issues from being raised.
- **JavaScript users get autocompletion for free** in more scenarios without their editor having to expensive and slow npm lookups in the background
- **Much easier to spelunk the source.** I can Cmd+Click to go to the types definition and the source code is nearby. With a seperate `@types/` package I have to go hunting in my node_modules folder. This is precious seconds I can spend making coffee or doomscrolling Twitter.

## In defence of `@types`

DefinitelyTyped, the folks that run it, and the folks that have contributed to it are to be appluaded. It's hard to see TypeScript spreading so far and wide without it, and for that - in the absense of dollars - I have to offer my thanks.

I just think we've outgrown them.

----

Discuss this post in the [GitHub discussion](https://github.com/tom-sherman/blog/discussions/10) or [on Twitter](https://twitter.com/tomus_sherman/status/1545436204119822336).
