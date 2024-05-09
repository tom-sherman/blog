---
slug: row-level-security-server-components
createdAt: 2024-05-09
title: "Row Level Security for auth in Server Components"
status: "draft"
---

# Row Level Security for auth in Server Components

I've been going back and forth on whether Row Level Security (RLS) is a good fit for authorization in React Server Components (RSC).

## Server Components Auth Best Practices

All of my thinking around this topic is based around a few best practices for auth in server components.

- The source of truth for data access should where you read your data. This is lower level than the layout or even the page/component level. Typically this is called the "Data Access Layer" or "DAL"
- Middleware is a good place for basic auth checks eg. "does this request have a valid token?". But it should be treated as a performance optimization, not a security boundary

Based on these articles:

- https://nextjs.org/docs/app/building-your-application/authentication#creating-a-data-access-layer-dal
- https://nextjs.org/blog/security-nextjs-server-components-actions

## The case for RLS

- SQL is your entire DAL. Can be a good boundary. We don't need to worry about auditing JS for auth checks, it's described in a declarative way in the database
- DTOs can be written in JS to allow the frontend to move quickly and not have to worry about the specifics of the database schema

## The case against RLS

- It's easier to iterate on authorization logic in your application code than in your database. Server components are "JS all the way up" and it's jarring when you have to switch to SQL to implement authorization logic
- Having your DTO and your DAL in the same language (potentially in the same module) is good for iteration speed, testing
- Authorization logic now requires migrations and potential downtime
- You still need to wrap your SQL queries in a JS interface for example to wrap them in `React.cache()`. We may as well put the auth logic in that layer too
