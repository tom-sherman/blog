# My linting dogma

There are few things in this small subculture of humanity we call "Software Development" more tribal than linting.

I have thought framework for thinking about lint rules. In my book, lint rules must pass a collection of tests before they can be added to a project that has more than a few contributors.

- warnings are pointless
- formatting must be autofixable

changes to linting rules should be considedered as a breaking change to the DX. as the person deciding on whether to add the rule it should be treated with the same care and attention as a breaking change to your end users. this means:

- interviewing
- surverys
- testing

## Flowchart: "Should I add that new rule?"

TODO

---

Discuss this post in the [GitHub issue]() or [on Twitter]().
