---
slug: "typescript-pattern-matched-types-proposal"
title: "Pattern Matched Types Proposal"
createdAt: "2023-01-03"
tags: ["typescript", "functional-programming"]
---

# Pattern Matched Types Proposal for TypeScript

[![""](https://user-images.githubusercontent.com/9257001/210391155-736cde4e-fb23-4285-a0bd-09d227734e5f.png)](https://twitter.com/sebastienlorber/status/1610293422727766016)


I propose a new feature for TypeScript that allows for pattern matching on types. This proposal is a [straw man](https://en.wikipedia.org/wiki/Straw_man_proposal) and I haven't quite thought through all the details, but I think it's a good starting point for discussion.

## The problem

TypeScript conditional types can become very unweildy when you nest them. They are also (subjectively) ugly and some developers find them hard to read and grok.

## The solution

Pattern matching on types. This would allow you to declaratively flatten nested conditional types and make them more readable.

This is a feature in many functional languages such as [Haskell](http://learnyouahaskell.com/syntax-in-functions#:~:text=Pattern%20matching%20consists%20of%20specifying,function%20bodies%20for%20different%20patterns) where conditional code is written in a declarative style.

```ts
type RouteParams<string>                                        = Record<string, string>;
type RouteParams<`${infer Start}:${infer Param}/${infer Rest}`> = {[k in Param | keyof RouteParams<Rest>]: string};
type RouteParams<`${infer Start}:${infer Param}`>               = {[k in Param]: string};
type RouteParams<any>                                           = {};
```

This is a type that extracts the route parameters from a string. It's equivalent to this current syntax:

```ts
type ExtractRouteParams<T extends string> = string extends T
  ? Record<string, string>
  : T extends `${infer Start}:${infer Param}/${infer Rest}`
  ? { [k in Param | keyof ExtractRouteParams<Rest>]: string }
  : T extends `${infer Start}:${infer Param}`
  ? { [k in Param]: string }
  : {};
```

Pattern matching solves the problem of unwieldy nested conditional types with a minimal syntax change. It should just require the addition of allowing literal types in generic argument positions. In my opinion this is preferable to adding new syntax for conditional types such as `if`/`else`, which is in affect a larger language feature.

This style has the benefits of looking similar to function overloads in TypeScript, which users are already familiar with. Like function overloads, the order of the patterns is important. The first pattern that matches is the one that is used and so the order of the patterns should be from most specific to least specific.

An alternative to the `any` base case could be to allow for mixing of literal and generic types in patterns. This would allow for the base case to be specified as a `_` to denote an empty pattern

```ts
type RouteParams<string>                                        = Record<string, string>;
type RouteParams<`${infer Start}:${infer Param}/${infer Rest}`> = {[k in Param | keyof RouteParams<Rest>]: string};
type RouteParams<`${infer Start}:${infer Param}`>               = {[k in Param]: string};
type RouteParams<_>                                       = {};
```

This would also encorporate the idea of a wildcard pattern, which is a common feature in pattern matching.

```ts
type Lucky<7> = "You're lucky";
type Lucky<N extends number> = IsEven<N> extends true ? "At least you chose even" : "You're unlucky";
```

As demonstrated also in the above snipped, this also supports capturing the matched value in a generic argument. Useful for mixing and matching pattern matching and classic conditional types.

## Maybe this is stupid

There's a lot of details to hash out here, and maybe the effort is not worth it to simply remove the need for nested ternaries. I'm not sure right now and would love to hear thoughts! Tag me on [Mastodon](https://fosstodon.org/@tomsherman) (`@tomsherman@fosstodon.org `) or [Twitter](https://twitter.com/tomus_sherman) and let me know what you think.
