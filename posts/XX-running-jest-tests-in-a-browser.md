# Running Jest Tests in a Real Browser

If you didn't know, Jest is in my opinion the best testing framework for modern JavaScript and TypeScript. The developer experience is so smooth and lightning fast (lightning smooth?) which makes chasing chasing that holy grail of 100% unit test coverage a breeze. Unfortunately, like many things, unit tests do not offer you 100% protection. To explain why, I'd love for you to indulge me in telling a very short story;

## The unit test footgun

So picture this; I'm writing a new feature, following the rules, obeying TDD, watching the terminal light up green as I add code for my test cases. I finish writing my feature and all of my tests pass, great! It's [not Friday](https://twitter.com/kvlly/status/1098941288727093248?lang=en) so I deploy my changes to production, all of the logs look great so I go to bed after another successful day of being a [10x developer](https://medium.com/@stevenpcurtis.sc/the-dangerous-myth-of-the-10x-developer-534d0e9159c9#:~:text=Terminology,job%20specification%20and%20limited%20benefit).

The next day I log on to see a critical bug has been raised relating the perfect feature I finished yesterday. How could this happen? Weren't all of my tests green? Some of you may know where this is going. After diagnosing the bug for a couple of hours I find that the code I've written uses some relatively new feature of some Web API that isn't supported in several of my user's browsers. I slap myself on the back of the wrist for not considering this before deploying and get to work on a fix. But I ask myself, why wasn't this caught by unit tests?

The answer is down to Jest not being a _real_ browser. You have access to all of the various DOM APIs when writing Jest tests, but these are provided by [JSDOM](https://github.com/jsdom/jsdom) and your tests are actually running in Node. Jest has no native way to run your tests in a browser environment out of the box.

## Some alternative solutions before we get to the good stuff

There are some good ways to solve this. You could have a separate suite of using a different framework that can run in a real browser. In my experience though, this is difficult to sell to other developers. Now they have to write a load of test twice, one for Jest to get tight feedback loops and once for the browser. What makes this worse is that usually your browser testing framework will have a completely different API and sometimes need to be written in a completely different language.

Another solution could be to use jest-puppeteer. This is a great tool for spawning a headless version of Chrome and using the puppeteer API to write your tests. This technique works well as you're still in the Jest ecosystem, but now you have to choose if a test needs to be written using puppeteer or if it's ok to run in a Node and JSDOM environment. So in summary, we're looking for a solution that:

- Runs in real browsers that reflect the actual users of the application
- Doesn't waste our time by making us write tests twice or choose between environments
- Allow us to take advantage of the Jest APIs

## The good stuff

Here's the TL;DR of how we have our cake and eat it too:

Write all of your tests once using Jest as your framework. These will run in Node and JSDOM for local development to keep feedback loops tight. Then, usually part of CI, run these exact same tests in a browser environments.

---

Discuss this post in the GitHub issue or on Twitter.
