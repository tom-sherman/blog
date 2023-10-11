---
slug: "designing-a-new-rsc-framework"
title: "Designing a new Server Component framework"
createdAt: "2023-10-11"
status: "unlisted"
---

# Designing a new React Server Component framework

- https://twitter.com/acdlite/status/1709973074387283980
- Would need to encourage suspense boundaries more than next.js does right now because every client component needs a boundary potentially, otherwise you block SSR for the whole page
- Could support multiple languages with the same framework.
- Could avoid implementing it on every language/runtime using shared Rust code with an adapter for each language
- Each runtime would need to hook into the JS bundler somehow to get client references
